# README — Noémie (élève 2) — Historique des versions

Code C++ (Raspberry Pi) pour la navigation du robot : LIDAR, capteur ultrason Grove, et communication XBee.

---

## Série 1 — Code LIDAR seul

### lidar v1 — `code noémie lidar v1.pdf`

Version initiale, LIDAR + ultrason, sans XBee.

- Utilise la bibliothèque SLAMTEC (`sl_lidar`) sur `/dev/ttyUSB0` à 256 000 bauds
- Ajout de `pigpio` pour lire un capteur ultrason single-pin (GPIO 22)
- Fonction `readUltrasonic()` : impulsion 10 µs + mesure de l'écho
- Scan LIDAR en boucle, données brutes analysées par tranche angulaire
- Pas de XBee, pas de communication avec l'ESP32
- `configSerialPort()` utilise l'API `termios` classique

### lidar v2 — `code noémie lidar v2.pdf`

Simplification — suppression du capteur ultrason, focus LIDAR seul.

**Modifications par rapport à lidar v1 :**
- **Suppression complète** du capteur ultrason et de `pigpio`
- Ajout d'une logique de **pilotage moteur** : calcul de `speed` et `sendAngle` selon la distance frontale (`avg0`)
  - `avg0 > 500` → avance rapide (1680 µs)
  - `avg0 >= 300` → avance modérée (1650 µs)
  - `avg0 < 100` → marche arrière (1350 µs, angle inversé +180°)
- Envoi de commandes JSON `{"action":"drive","angle":X,"speed":Y}` vers l'ESP32 via `/dev/ttyUSB1`
- Code plus court et orienté contrôle

---

## Série 2 — LIDAR + Ultrason

### lidar + ultrason v1 — `code noémie lidar + ultrason v1.pdf`

Fusion du LIDAR et de l'ultrason Grove, sans XBee.

**Modifications par rapport à lidar v2 :**
- Réintroduction de `pigpio` et du capteur **ultrason Grove** (`readGroveUltrasonic()`)
  - Impulsion trigger 10 µs, lecture de l'écho, formule `duration / 58.0f` (en cm)
- Le capteur ultrason complète le LIDAR pour la détection d'obstacles proches
- `configSerialPort()` toujours en `termios` classique (baud fixe B115200)
- Pas encore de XBee

---

## Série 3 — LIDAR + Ultrason + XBee

### lidar + ultrason + xbee v1 — `code noémie lidar + ultrason + xbee v1.pdf`

Première version avec communication XBee (attente du signal START).

**Modifications par rapport à lidar + ultrason v1 :**
- Ajout de l'**initialisation XBee** sur `/dev/ttyUSB2` à 9 600 bauds
- Le programme attend le message `"START"` via XBee avant de démarrer la boucle principale
- Lecture XBee ligne par ligne avec `serialReadLine()` (lecture caractère par caractère)
- `configSerialPort()` toujours en `termios` classique

### lidar + ultrason + xbee v2 — `code noémie lidar + ultrason + xbee v2.pdf`

Refonte de la configuration série pour supporter des bauds non-standards.

**Modifications par rapport à xbee v1 :**
- `configSerialPort()` réécrite en utilisant **`termios2` + `ioctl`** (`TCGETS2`/`TCSETS2`) au lieu de `termios` classique → permet de configurer n'importe quel baudrate (non limité aux constantes B9600, B115200…)
- `serialReadLine()` remplacée par `serialReadSome()` qui lit un buffer entier d'un coup (jusqu'à 256 octets) au lieu de caractère par caractère → plus efficace
- Ajout d'un buffer d'accumulation `xbeeBuffer` pour reconstituer les lignes complètes
- Suppression des includes `<termios.h>` et `<map>`, ajout de `<sys/ioctl.h>` et `<asm/termbits.h>`

### lidar + ultrason + xbee v3 — `code noémie lidar + ultrason + xbee v3.pdf`

Refactorisation majeure : architecture globale propre, gestion des signaux robuste.

**Modifications par rapport à xbee v2 :**
- Introduction de **variables globales nommées** : `lidarChannel`, `motor_fd`, `xbee_fd`, `stopRequested`
- `ctrlc_handler()` modifié : au lieu d'appeler `exit()` directement, il positionne le flag `volatile sig_atomic_t stopRequested = 1` → arrêt propre de la boucle principale
- Ajout d'une **constante de période** : `LOOP_PERIOD_MS = 50` pour cadencer la boucle
- Nouveau parser XBee **`readXbeeLine()`** avec buffer statique interne :
  - Détecte les octets non-imprimables (alerte mode API XBee)
  - Filtre les `\r`, assemble les lignes sur `\n`
  - Protection contre débordement (reset au-delà de 256 caractères)
- Ajout de `resetXbeeParser()` (stub de remise à zéro du parser)
- Includes enrichis : `<cerrno>`, `<cstring>` pour la gestion d'erreurs
- Suppression du `<map>` (non utilisé)

### lidar + ultrason + xbee v4 — `code noémie lidar + ultrason + xbee v4.pdf`

Version stable — identique à v3 sur la structure, avec ajustements de la logique de navigation.

**Modifications par rapport à xbee v3 :**
- Structure du code et fonctions utilitaires identiques à v3
- Ajustements dans la logique de décision LIDAR (seuils de distance, vitesses cibles)
- Corrections mineures dans la boucle principale de navigation
- Version considérée comme la plus aboutie de la série Noémie
