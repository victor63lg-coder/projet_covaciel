# README — Antoine (élève 3) — Historique des versions

Code Python (Raspberry Pi) pour le système de chronométrage automatique multi-voitures : capteurs VL53L0X, lecture QR code, communication XBee, archivage CSV + photos.

---

## v1 — `code antoine chrono v1.pdf`

Version initiale du système de chronométrage.

- Capteurs de distance **VL53L0X** via multiplexeur I2C **TCA9548A** (adresse 0x70)
  - 2 capteurs de ligne (canaux 0 et 1) pour détecter le sens de passage
  - 4 capteurs verticaux (canaux **3, 5, 6, 7**) pour détecter les voitures
- Seuil de passage unique : `SEUIL_PASSAGE = 200 mm`
- Communication XBee sur `/dev/ttyUSB0` à 9 600 bauds
  - Envoi avec `\r\n` comme fin de ligne
- Thread caméra USB indépendant (reconnexion automatique) pour lecture QR code via **pyzbar**
- Import de **numpy** (`import numpy as np`)
- Dossier d'archivage non défini (pas de `BASE_DIR`)
- Pas encore de diagnostic au démarrage

---

## v1 + XBee — `code antoine chrono + xbee v1.pdf`

Refonte partielle : amélioration du chronométrage et de la configuration.

**Modifications par rapport à chrono v1 :**
- Canaux verticaux mis à jour : `[3, 4, 5, 6, 7]` → **ajout du canal 4** (5 capteurs au lieu de 4)
- Séparation des seuils : `SEUIL_LIGNE = 350 mm` et `SEUIL_VERTICAL = 455 mm` (au lieu d'un seul `SEUIL_PASSAGE = 200 mm`)
- Ajout de `BASE_DIR = "/home/pi/capture_covaciel"` pour l'archivage des résultats
- Ajout de `DELAI_LOOP = 0.001` (constante de timing explicite)
- Suppression de l'import `numpy` (non utilisé)
- Fin de ligne XBee simplifiée : `\r\n` → `\n` uniquement
- XBee : ajout d'un log `[XBEE][INFO] Envoi '...' xN` avant chaque envoi
- Code moins commenté mais plus structuré

---

## v2 + XBee — `code antoine chrono +xbee v2.pdf`

Version finale : code propre, commentaires complets, architecture documentée.

**Modifications par rapport à chrono + xbee v1 :**
- **Ajout de docstrings** sur toutes les fonctions importantes (`init_xbee`, `xbee_send_line`, `camera_thread`, etc.)
- Commentaires inline expliquant chaque bloc (I2C, TCA9548A, thread caméra, multiplexeur)
- En-tête du fichier : ajout du titre `COVACIEL - SYSTÈME DE CHRONOMÉTRAGE MULTI-VOITURES` et du nom de l'auteur **Antoine NICOLAS**
- Les séparateurs de section passent de `###` (43 `#`) à `###` (63 `#`) → mise en forme plus lisible
- Ajout du commentaire `# Petit délai pour stabiliser` dans `init_xbee()`
- Ajout du commentaire `# Petit délai pour éviter collisions radio` dans `xbee_send_line()`
- Commentaire dans le thread caméra : `# On stocke la dernière image`
- Logique fonctionnelle identique à v1 + XBee (mêmes constantes, mêmes seuils, même architecture)
- Version considérée comme la version finale documentée du projet Antoine
