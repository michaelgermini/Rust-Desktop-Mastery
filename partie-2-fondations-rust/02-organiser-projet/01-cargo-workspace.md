# 5.1 Cargo Workspace

## Pourquoi un workspace ?

Un workspace Cargo permet de :
- **Diviser le code en crates réutilisables** : Séparation claire des responsabilités
- **Compiler une seule fois les dépendances partagées** : Gain de temps et d'espace
- **Maintenir une cohérence de versions** : Toutes les crates utilisent les mêmes versions de dépendances
- **Faciliter les tests et le développement parallèle** : Tests isolés par crate

## Structure d'un workspace

```
mon_application/
├── Cargo.toml              # Workspace root
├── Cargo.lock              # Versions verrouillées (partagé)
├── crates/
│   ├── app/                # Application principale
│   │   ├── Cargo.toml
│   │   └── src/
│   │       └── main.rs
│   ├── core/               # Logique métier
│   │   ├── Cargo.toml
│   │   └── src/
│   │       └── lib.rs
│   ├── ui/                 # Interface utilisateur
│   │   ├── Cargo.toml
│   │   └── src/
│   │       └── lib.rs
│   └── storage/            # Persistence
│       ├── Cargo.toml
│       └── src/
│           └── lib.rs
├── tests/                  # Tests d'intégration
└── benches/                # Benchmarks
```

## Configuration du workspace

### Cargo.toml racine

```toml
# Cargo.toml (racine)
[workspace]
resolver = "2"
members = [
    "crates/app",
    "crates/core",
    "crates/ui",
    "crates/storage",
]

# Dépendances partagées (workspace inheritance)
[workspace.dependencies]
serde = { version = "1.0", features = ["derive"] }
tokio = { version = "1.0", features = ["full"] }
anyhow = "1.0"
tracing = "0.1"

# Profil de release optimisé (appliqué à toutes les crates)
[profile.release]
lto = true
strip = true
panic = "abort"
codegen-units = 1
```

### Cargo.toml d'une crate membre

```toml
# crates/core/Cargo.toml
[package]
name = "mon_app_core"
version = "0.1.0"
edition = "2021"

[dependencies]
serde.workspace = true      # Utilise la version du workspace
anyhow.workspace = true
```

```toml
# crates/app/Cargo.toml
[package]
name = "mon_app"
version = "0.1.0"
edition = "2021"

[dependencies]
mon_app_core = { path = "../core" }
mon_app_ui = { path = "../ui" }
mon_app_storage = { path = "../storage" }
tokio.workspace = true
tracing.workspace = true
```

## Commandes workspace

### Compilation

```bash
# Compiler tout le workspace
cargo build

# Compiler une crate spécifique
cargo build -p mon_app_core

# Build release pour toutes les crates
cargo build --release

# Build release pour une crate
cargo build --release -p mon_app
```

### Exécution

```bash
# Lancer l'application principale
cargo run -p mon_app

# Lancer avec arguments
cargo run -p mon_app -- --config config.toml
```

### Tests

```bash
# Tous les tests du workspace
cargo test

# Tests d'une crate spécifique
cargo test -p mon_app_core

# Tests avec output détaillé
cargo test -- --nocapture
```

### Autres commandes utiles

```bash
# Clippy sur tout le workspace
cargo clippy --workspace

# Formatage de tout le workspace
cargo fmt --all

# Documentation pour toutes les crates
cargo doc --workspace --open

# Vérifier les dépendances obsolètes
cargo outdated
```

## Avantages pratiques

### Compilation partagée

```bash
# Premier build : compile toutes les dépendances
$ cargo build
   Compiling serde v1.0.0
   Compiling tokio v1.0.0
   Compiling mon_app_core v0.1.0
   Compiling mon_app_ui v0.1.0
   Compiling mon_app v0.1.0
Finished in 45s

# Modification dans core seulement
$ cargo build
   Compiling mon_app_core v0.1.0
   Compiling mon_app v0.1.0
Finished in 2s  # Beaucoup plus rapide !
```

### Gestion des versions

```toml
# Une seule source de vérité pour les versions
[workspace.dependencies]
serde = "1.0"  # Toutes les crates utilisent cette version

# Si vous devez mettre à jour :
# 1. Changez ici
# 2. Toutes les crates sont mises à jour automatiquement
```

## Résumé

- **Workspace** : Groupe de crates liées partageant un `Cargo.lock`
- **Workspace inheritance** : Dépendances définies une fois, réutilisées partout
- **Commandes** : `-p` pour cibler une crate spécifique
- **Avantage principal** : Compilation plus rapide et gestion simplifiée des versions
