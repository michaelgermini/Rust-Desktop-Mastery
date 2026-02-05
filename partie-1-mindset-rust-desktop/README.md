# Partie I — Le Mindset Rust Desktop

Cette première partie pose les fondations philosophiques et stratégiques du développement desktop en Rust. Avant d'écrire une seule ligne de code, il est crucial de comprendre *pourquoi* nous faisons ces choix et *comment* penser notre travail.

## Chapitres

1. **[Pourquoi Rust pour le logiciel moderne](./01-pourquoi-rust/)**
   - La fatigue JS / Electron / SaaS
   - Performance native
   - Sécurité mémoire
   - Binaire unique
   - Offline-first
   - Souveraineté des données

2. **[Desktop is back](./02-desktop-is-back/)**
   - Le retour des apps locales
   - UX supérieure au navigateur
   - IA locale, SQLite, fichiers
   - Cas concrets : ERP, notes, outils, automation
   - Quand ne pas utiliser le web

3. **[Penser produit, pas repo GitHub](./03-penser-produit/)**
   - Ship plutôt que démo
   - Logiciel utilisable 8 heures par jour
   - Dette UX vs dette technique
   - Le design comme avantage concurrentiel

## Objectifs d'apprentissage

À la fin de cette partie, vous serez capable de :

- Argumenter le choix de Rust pour un projet desktop
- Identifier les cas d'usage où le desktop surpasse le web
- Adopter une mentalité produit orientée utilisateur
- Évaluer les compromis entre performance, UX et time-to-market

## Le changement de paradigme

Pendant 15 ans, le web a été la réponse par défaut à tout besoin logiciel. Cette ère touche à sa fin pour de nombreux cas d'usage :

| Approche Web/SaaS | Approche Desktop Rust |
|-------------------|----------------------|
| Latence réseau | Réponse instantanée |
| Abonnement mensuel | Achat unique |
| Données sur serveur tiers | Données locales |
| Dépendance internet | Fonctionne offline |
| Complexité infrastructure | Un fichier binaire |

Ce n'est pas un retour en arrière, c'est une évolution vers des logiciels plus respectueux des utilisateurs et plus durables.
