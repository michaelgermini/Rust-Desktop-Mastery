# Structure du Projet Rust Books

## Organisation des fichiers

```
rust_books/
├── README.md                          # Introduction générale du livre
├── books.md                           # Table des matières complète
├── STRUCTURE.md                       # Ce fichier - documentation de la structure
│
├── partie-1-mindset-rust-desktop/    # Partie I
│   ├── README.md                      # Introduction de la partie
│   ├── 01-pourquoi-rust/             # Chapitre 1 (dossier avec sous-sections)
│   │   ├── README.md
│   │   ├── 01-fatigue-js-electron.md
│   │   ├── 02-performance-native.md
│   │   ├── 03-securite-memoire.md
│   │   ├── 04-binaire-unique.md
│   │   ├── 05-offline-first.md
│   │   └── 06-souverainete-donnees.md
│   ├── 02-desktop-is-back/           # Chapitre 2 (dossier avec sous-sections)
│   │   ├── README.md
│   │   ├── 01-retour-apps-locales.md
│   │   ├── 02-ux-superieure.md
│   │   ├── 03-ia-locale-sqlite.md
│   │   ├── 04-cas-concrets.md
│   │   └── 05-quand-pas-web.md
│   └── 03-penser-produit/            # Chapitre 3 (dossier avec sous-sections)
│       ├── README.md
│       ├── 01-ship-plutot-que-demo.md
│       ├── 02-logiciel-8h-jour.md
│       ├── 03-dette-ux-vs-technique.md
│       └── 04-design-avantage-concurrentiel.md
│
├── partie-2-fondations-rust/         # Partie II
│   ├── README.md
│   ├── 01-rust-pragmatique/          # Chapitre 4 (dossier avec sous-sections)
│   │   ├── README.md
│   │   ├── 01-ownership-simplement.md
│   │   ├── 02-borrow-checker.md
│   │   ├── 03-erreurs-idiomatiques.md
│   │   └── 04-patterns-utiles.md
│   ├── 02-organiser-projet.md        # Chapitre 5 (fichier unique)
│   ├── 03-async-concurrence.md       # Chapitre 6 (fichier unique)
│   └── 04-logs-debug-profiling.md    # Chapitre 7 (fichier unique)
│
├── partie-3-interfaces-modernes/     # Partie III
│   ├── README.md
│   ├── 01-panorama-frameworks.md     # Chapitre 8
│   ├── 02-construire-ui-egui.md      # Chapitre 9
│   ├── 03-design-system.md           # Chapitre 10
│   ├── 04-composants-reutilisables.md # Chapitre 11
│   ├── 05-patterns-ux.md             # Chapitre 12
│   └── 06-wireframe-vers-code.md     # Chapitre 13
│
├── partie-4-architecture-logiciel/    # Partie IV
│   ├── README.md
│   ├── 01-clean-architecture.md      # Chapitre 14
│   ├── 02-event-bus.md               # Chapitre 15
│   ├── 03-gestion-etat.md            # Chapitre 16
│   ├── 04-stockage-local.md          # Chapitre 17
│   ├── 05-fichiers-import-export.md  # Chapitre 18
│   └── 06-plugins-extensibilite.md   # Chapitre 19
│
├── partie-5-blocs-metiers/          # Partie V
│   ├── README.md
│   ├── 01-mini-erp.md                # Chapitre 20
│   ├── 02-moteur-recherche.md        # Chapitre 21
│   ├── 03-dashboard-visualisation.md # Chapitre 22
│   ├── 04-ia-locale.md               # Chapitre 23
│   └── 05-cas-etude-complet.md       # Chapitre 24
│
├── partie-6-finition-qualite/       # Partie VI
│   ├── README.md
│   ├── 01-performance-percue.md      # Chapitre 25
│   ├── 02-accessibilite-ergonomie.md # Chapitre 26
│   ├── 03-packaging-multiplateforme.md # Chapitre 27
│   └── 04-tests-ui-qualite.md        # Chapitre 28
│
├── partie-7-production-business/    # Partie VII
│   ├── README.md
│   ├── 01-licences-modeles-economiques.md # Chapitre 29
│   ├── 02-distribution-marketing.md  # Chapitre 30
│   ├── 03-support-maintenance.md     # Chapitre 31
│   ├── 04-monetisation-avancee.md     # Chapitre 32
│   └── 05-vision-long-terme.md       # Chapitre 33
│
└── annexes/                          # Annexes
    ├── README.md
    ├── A-cargo-toml-complet.md
    ├── B-cheatsheet-egui.md
    ├── C-patterns-rust.md
    ├── D-checklist-lancement.md
    ├── E-ressources-liens.md
    └── F-glossaire.md
```

## Convention de nommage

### Chapitres avec sous-sections détaillées
- **Format** : `XX-nom-chapitre/` (dossier)
- **Contenu** : 
  - `README.md` : Introduction et index des sous-sections
  - `01-sous-section.md`, `02-sous-section.md`, etc.

### Chapitres simples
- **Format** : `XX-nom-chapitre.md` (fichier unique)
- **Contenu** : Chapitre complet dans un seul fichier

## Règles de réorganisation

1. **Chapitres détaillés** : Si un chapitre a 4+ sous-sections substantielles, créer un dossier avec sous-sections
2. **Chapitres simples** : Si un chapitre est relativement court ou monolithique, garder un fichier unique
3. **Pas de doublons** : Un chapitre existe soit comme fichier, soit comme dossier, jamais les deux
4. **README cohérents** : Chaque partie et chaque dossier de chapitre a un README.md

## Statut actuel

✅ **Partie I** : Complètement réorganisée avec dossiers pour tous les chapitres
✅ **Partie II** : Chapitre 4 réorganisé en dossier, autres chapitres en fichiers
⏳ **Parties III-VII** : Structure cohérente avec fichiers uniques
✅ **Annexes** : Structure complète

## Prochaines étapes suggérées

1. Compléter les sous-sections manquantes pour les parties restantes
2. Uniformiser le format (tous en dossiers ou tous en fichiers selon la longueur)
3. Ajouter des liens croisés entre chapitres
4. Créer un index global avec liens vers tous les chapitres
