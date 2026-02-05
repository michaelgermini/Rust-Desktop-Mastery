# Annexe A : Cargo.toml Complet Commenté

Ce fichier présente un `Cargo.toml` complet pour une application desktop professionnelle.

```toml
[package]
name = "mon-app-desktop"
version = "1.0.0"
edition = "2021"
authors = ["Votre Nom <email@example.com>"]
description = "Application de gestion pour PME"
license = "Proprietary"
repository = "https://github.com/org/mon-app"
readme = "README.md"
keywords = ["desktop", "business", "management"]
categories = ["gui", "business"]

# Informations pour le packaging
[package.metadata.bundle]
name = "Mon Application"
identifier = "com.example.monapp"
icon = ["assets/icon.icns", "assets/icon.ico", "assets/icon.png"]
copyright = "Copyright © 2024 Votre Société"
category = "Productivity"

# ═══════════════════════════════════════════════════════════════════════════════
# DÉPENDANCES PRINCIPALES
# ═══════════════════════════════════════════════════════════════════════════════

[dependencies]

# ─────────────────────────────────────────────────────────────────────────────────
# Interface Graphique
# ─────────────────────────────────────────────────────────────────────────────────

# Framework UI principal (mode immédiat)
eframe = "0.27"
egui = "0.27"

# Graphiques et visualisation
egui_plot = "0.27"

# ─────────────────────────────────────────────────────────────────────────────────
# Async et Concurrence
# ─────────────────────────────────────────────────────────────────────────────────

# Runtime async
tokio = { version = "1.37", features = ["full"] }

# Channels pour communication inter-threads
crossbeam-channel = "0.5"

# État partagé thread-safe (swap atomique)
arc-swap = "1.7"

# Token d'annulation
tokio-util = { version = "0.7", features = ["rt"] }

# ─────────────────────────────────────────────────────────────────────────────────
# Persistance et Base de Données
# ─────────────────────────────────────────────────────────────────────────────────

# SQLite embarqué
rusqlite = { version = "0.31", features = ["bundled"] }

# Migrations de base de données
refinery = { version = "0.8", features = ["rusqlite"] }

# ─────────────────────────────────────────────────────────────────────────────────
# Sérialisation
# ─────────────────────────────────────────────────────────────────────────────────

# JSON
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"

# TOML (configuration)
toml = "0.8"

# CSV (import/export)
csv = "1.3"

# ─────────────────────────────────────────────────────────────────────────────────
# Gestion des erreurs
# ─────────────────────────────────────────────────────────────────────────────────

# Erreurs simplifiées pour applications
anyhow = "1.0"

# Erreurs typées pour bibliothèques
thiserror = "1.0"

# ─────────────────────────────────────────────────────────────────────────────────
# Logging et Diagnostic
# ─────────────────────────────────────────────────────────────────────────────────

# Logging structuré
tracing = "0.1"
tracing-subscriber = { version = "0.3", features = ["env-filter", "json"] }

# Export Chrome trace (profiling)
tracing-chrome = "0.7"

# ─────────────────────────────────────────────────────────────────────────────────
# Dates et Temps
# ─────────────────────────────────────────────────────────────────────────────────

# Manipulation de dates
chrono = { version = "0.4", features = ["serde"] }

# ─────────────────────────────────────────────────────────────────────────────────
# Calculs Numériques
# ─────────────────────────────────────────────────────────────────────────────────

# Nombres décimaux (calculs financiers)
rust_decimal = { version = "1.35", features = ["serde"] }

# ─────────────────────────────────────────────────────────────────────────────────
# Identifiants
# ─────────────────────────────────────────────────────────────────────────────────

# UUID v4
uuid = { version = "1.8", features = ["v4", "serde"] }

# ─────────────────────────────────────────────────────────────────────────────────
# Recherche Full-Text
# ─────────────────────────────────────────────────────────────────────────────────

# Moteur de recherche
tantivy = "0.22"

# ─────────────────────────────────────────────────────────────────────────────────
# Cryptographie et Sécurité
# ─────────────────────────────────────────────────────────────────────────────────

# Chiffrement AES-GCM
aes-gcm = "0.10"

# Dérivation de clé (mot de passe)
argon2 = "0.5"

# Signature (licences)
ed25519-dalek = "2.1"

# Hachage
sha2 = "0.10"

# Encodage base64
base64 = "0.22"

# Encodage hex
hex = "0.4"

# ─────────────────────────────────────────────────────────────────────────────────
# Génération de Documents
# ─────────────────────────────────────────────────────────────────────────────────

# Génération PDF
printpdf = "0.7"

# ─────────────────────────────────────────────────────────────────────────────────
# Dialogues Système
# ─────────────────────────────────────────────────────────────────────────────────

# Dialogues natifs (open file, save file)
native-dialog = "0.7"

# Ouvrir fichiers/URLs avec l'application par défaut
open = "5.1"

# ─────────────────────────────────────────────────────────────────────────────────
# Mise à Jour Automatique
# ─────────────────────────────────────────────────────────────────────────────────

# Auto-update depuis GitHub
self_update = "0.39"

# ─────────────────────────────────────────────────────────────────────────────────
# IA Locale (optionnel)
# ─────────────────────────────────────────────────────────────────────────────────

# Framework ML
candle-core = { version = "0.4", optional = true }
candle-nn = { version = "0.4", optional = true }
candle-transformers = { version = "0.4", optional = true }

# Tokenizer
tokenizers = { version = "0.15", optional = true }

# ═══════════════════════════════════════════════════════════════════════════════
# DÉPENDANCES DE DÉVELOPPEMENT
# ═══════════════════════════════════════════════════════════════════════════════

[dev-dependencies]

# Fichiers temporaires pour tests
tempfile = "3.10"

# Benchmarking
criterion = "0.5"

# Assertions améliorées
pretty_assertions = "1.4"

# Mocking
mockall = "0.12"

# ═══════════════════════════════════════════════════════════════════════════════
# FEATURES
# ═══════════════════════════════════════════════════════════════════════════════

[features]
default = []

# Activer l'IA locale
local-ai = ["candle-core", "candle-nn", "candle-transformers", "tokenizers"]

# Mode développement avec logs détaillés
dev = []

# ═══════════════════════════════════════════════════════════════════════════════
# PROFILS DE BUILD
# ═══════════════════════════════════════════════════════════════════════════════

[profile.dev]
opt-level = 0
debug = true

[profile.release]
opt-level = 3
lto = true          # Link-Time Optimization
strip = true        # Supprimer les symboles de debug
panic = "abort"     # Plus petit binaire
codegen-units = 1   # Meilleure optimisation

[profile.release-with-debug]
inherits = "release"
debug = true
strip = false

# ═══════════════════════════════════════════════════════════════════════════════
# BENCHMARKS
# ═══════════════════════════════════════════════════════════════════════════════

[[bench]]
name = "search_benchmark"
harness = false

[[bench]]
name = "rendering_benchmark"
harness = false
```

## Notes d'utilisation

### Installation des dépendances

```bash
cargo build
```

### Compilation Release

```bash
cargo build --release
```

### Avec IA locale

```bash
cargo build --release --features local-ai
```

### Exécution des benchmarks

```bash
cargo bench
```
