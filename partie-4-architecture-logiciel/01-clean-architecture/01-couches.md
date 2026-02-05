# 14.1 Core / App / UI / Infra

## Vue d'ensemble

```
┌─────────────────────────────────────────────────────────┐
│                    Infrastructure                        │
│  (SQLite, Fichiers, API externes, Système)              │
├─────────────────────────────────────────────────────────┤
│                         UI                               │
│  (egui, Composants, Écrans, Thème)                      │
├─────────────────────────────────────────────────────────┤
│                    Application                           │
│  (Use cases, Orchestration, Commands)                   │
├─────────────────────────────────────────────────────────┤
│                        Core                              │
│  (Entités, Règles métier, Interfaces)                   │
└─────────────────────────────────────────────────────────┘

Règle : Les dépendances vont UNIQUEMENT vers l'intérieur
        UI et Infra dépendent de Application
        Application dépend de Core
        Core ne dépend de RIEN
```

## Structure de projet

```
mon_application/
├── Cargo.toml                    # Workspace
├── crates/
│   ├── core/                     # Couche Core
│   │   ├── Cargo.toml
│   │   └── src/
│   │       ├── lib.rs
│   │       ├── entities/         # Entités métier
│   │       ├── services/         # Logique métier pure
│   │       ├── ports/            # Interfaces (traits)
│   │       └── error.rs
│   │
│   ├── application/              # Couche Application
│   │   ├── Cargo.toml
│   │   └── src/
│   │       ├── lib.rs
│   │       ├── commands/         # Use cases
│   │       ├── queries/          # Lectures
│   │       └── state.rs          # État applicatif
│   │
│   ├── infrastructure/           # Couche Infrastructure
│   │   ├── Cargo.toml
│   │   └── src/
│   │       ├── lib.rs
│   │       ├── database/         # SQLite
│   │       ├── filesystem/       # Fichiers
│   │       └── repositories/     # Implémentations
│   │
│   └── ui/                       # Couche UI
│       ├── Cargo.toml
│       └── src/
│           ├── lib.rs
│           ├── app.rs
│           ├── screens/
│           └── components/
│
└── src/
    └── main.rs                   # Point d'entrée
```

## Responsabilités de chaque couche

### Core (Couche interne)

- **Entités métier** : Structures de données avec logique métier
- **Services métier** : Calculs et règles métier pures
- **Ports (Traits)** : Interfaces définissant les contrats
- **Aucune dépendance externe** : Pas de SQLite, pas d'UI, rien

### Application

- **Commands** : Use cases d'écriture (créer, modifier, supprimer)
- **Queries** : Use cases de lecture (lister, rechercher)
- **Orchestration** : Coordonne les appels aux repositories et services
- **Dépend de Core uniquement**

### Infrastructure

- **Repositories** : Implémentations concrètes des ports (SQLite, fichiers)
- **Services externes** : API HTTP, emails, etc.
- **Dépend de Core et Application**

### UI

- **Écrans** : Vues de l'application
- **Composants** : Widgets réutilisables
- **Gestion d'état UI** : État local des composants
- **Dépend de Application uniquement**

## Avantages

- **Testabilité** : Core testable sans base de données
- **Maintenabilité** : Changements localisés
- **Évolutivité** : Nouvelles features sans casser l'existant
- **Flexibilité** : Changer de base de données facilement

## Résumé

- **4 couches** : Core, Application, Infrastructure, UI
- **Dépendances vers l'intérieur** : Core ne dépend de rien
- **Séparation claire** : Chaque couche a un rôle précis
- **Workspace Cargo** : Organisation en crates séparées
