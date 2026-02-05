# 24.1 De Zéro à Application Finie

## Architecture complète

```
notevault/
├── Cargo.toml                    # Workspace
├── crates/
│   ├── core/                     # Entités, logique métier
│   │   ├── Cargo.toml
│   │   └── src/
│   │       ├── lib.rs
│   │       ├── entities/
│   │       │   ├── note.rs
│   │       │   └── tag.rs
│   │       ├── services/
│   │       │   └── search.rs
│   │       └── ports/
│   │           └── repository.rs
│   │
│   ├── application/              # Use cases
│   │   ├── Cargo.toml
│   │   └── src/
│   │       ├── lib.rs
│   │       ├── commands/
│   │       │   ├── create_note.rs
│   │       │   └── update_note.rs
│   │       └── queries/
│   │           └── get_notes.rs
│   │
│   ├── infrastructure/           # Implémentations
│   │   ├── Cargo.toml
│   │   └── src/
│   │       ├── lib.rs
│   │       ├── database/
│   │       │   └── sqlite.rs
│   │       └── repositories/
│   │           └── note_repository.rs
│   │
│   └── ui/                       # Interface
│       ├── Cargo.toml
│       └── src/
│           ├── lib.rs
│           ├── app.rs
│           ├── screens/
│           │   ├── editor.rs
│           │   └── list.rs
│           └── components/
│               └── search_box.rs
│
└── src/
    └── main.rs                   # Point d'entrée
```

## Structure du projet

### Cargo.toml workspace

```toml
[workspace]
members = [
    "crates/core",
    "crates/application",
    "crates/infrastructure",
    "crates/ui",
]

[workspace.dependencies]
egui = "0.27"
eframe = "0.27"
rusqlite = "0.31"
serde = { version = "1.0", features = ["derive"] }
tokio = { version = "1", features = ["full"] }
```

### Dépendances par crate

```toml
# crates/core/Cargo.toml
[dependencies]
serde = { workspace = true }
chrono = "0.4"

# crates/application/Cargo.toml
[dependencies]
core = { path = "../core" }
serde = { workspace = true }

# crates/infrastructure/Cargo.toml
[dependencies]
core = { path = "../core" }
application = { path = "../application" }
rusqlite = { workspace = true }
serde = { workspace = true }

# crates/ui/Cargo.toml
[dependencies]
application = { path = "../application" }
egui = { workspace = true }
eframe = { workspace = true }
```

## Dépendances principales

```toml
# src/Cargo.toml (racine)
[dependencies]
ui = { path = "crates/ui" }
infrastructure = { path = "crates/infrastructure" }
```

## Résumé

- **Workspace Cargo** : Organisation en crates séparées
- **Clean Architecture** : Core → Application → Infrastructure/UI
- **Dépendances** : Chaque crate dépend seulement de ce qu'il faut
- **Structure claire** : Facile à naviguer et maintenir
