# README — Victor (élève 1) — Historique des versions

Code ESP32 pour le contrôle de la voiture autonome (direction servo + moteur ESC via JSON série).

---

## v1.0 — `code victor v1.pdf`

Version initiale du contrôleur ESP32.

- Réception de commandes JSON via Serial (USB depuis le Raspberry Pi)
- Contrôle de la direction (servo GPIO 13) et du moteur ESC (GPIO **15**)
- Lissage progressif direction + vitesse (`SMOOTH_STEP = 10`, `SMOOTH_DELAY = 5 ms`)
- Cooldown entre commandes : 40 ms
- Filtre angulaire : ignore les variations < 3°
- Limites servo : MIN=1200, MAX=1650, NEUTRE=1450
- Plage vitesse limitée à [1200 ; 1800] µs
- Commandes supportées : `drive` (angle + speed) et `stop`
- Pas de watchdog de communication

---

## v1.1 — `code Victor v1.1.pdf`

Évolution mineure, essentiellement un ralentissement du lissage.

**Modifications par rapport à v1.0 :**
- `SMOOTH_DELAY` passé de **5 ms à 20 ms** → mouvements plus lents et plus contrôlés
- ESC déplacé de GPIO 15 → GPIO **14**
- Ajout d'un commentaire explicite sur la limite PWM vitesse [1200 ; 1800]
- Aucun watchdog ajouté

---

## v1.2.1 — `code Victor 1.2.1.pdf`

Introduction du watchdog de communication (failsafe).

**Modifications par rapport à v1.1 :**
- Ajout du **watchdog** (`COMM_TIMEOUT = 3000 ms`) : si aucune commande reçue depuis 3 s, retour automatique au neutre
- `SMOOTH_DELAY` passé de 20 ms à **120 ms** → lissage encore plus lent
- Plage vitesse élargie de [1200 ; 1800] à **[1200 ; 2000] µs**
- Correction du bug `alreadyWarned` : la variable `static` était dupliquée dans le bloc `else`, maintenant placée en dehors
- Le `stop` ajoute un `delay(1000)` pour une petite pause moteur

---

## v1.2.2 — `code Victor v1.2.2.pdf`

Ajustement des limites mécaniques de la direction + ajout de commentaires pédagogiques détaillés.

**Modifications par rapport à v1.2.1 :**
- `SERVO_MIN` passé de 1200 à **1150 µs**
- `SERVO_NEUTRE` passé de 1450 à **1400 µs** → recentrage de la direction
- Ajout de nombreux **commentaires en français** sur chaque ligne de logique (filtrage, mapping, contrainte, parsing JSON…)
- Correction du bug `alreadyWarned` : la variable `static` maintenant correctement placée à l'extérieur des blocs `if/else`
- Logique identique à v1.2.1 sinon

---

## v1.2.3 — `Code Victor v1.2.3.txt`

Refactorisation propre avec constantes ESC nommées + correction du mapping d'angle.

**Modifications par rapport à v1.2.2 :**
- Ajout de constantes dédiées à l'ESC : `ESC_MIN = 1200`, `ESC_MAX = 1700`, `ESC_NEUTRE = 1500`
- Ces constantes remplacent les valeurs hardcodées dans `constrain()` et `checkCommTimeout()`
- `SMOOTH_DELAY` réduit de 120 ms à **60 ms** → lissage intermédiaire plus réactif
- Mapping d'angle corrigé : la plage de contrainte passe de **[-180 ; 180]** à **[-90 ; 90]** et le `map()` utilise `SERVO_MIN → SERVO_MAX` (au lieu de `SERVO_MAX → SERVO_MIN`) → inversion de sens corrigée
- Code plus lisible et commenté
