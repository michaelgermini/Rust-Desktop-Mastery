# Chapitre 5 : Organiser un Vrai Projet

## Introduction

Un projet Rust professionnel ne se résume pas à un seul fichier `main.rs`. Ce chapitre présente les meilleures pratiques pour structurer une application maintenable, testable et évolutive.

---

## 5.1 Cargo Workspace

### Pourquoi un workspace ?

Un workspace Cargo permet de :
- Diviser le code en crates réutilisables
- Compiler une seule fois les dépendances partagées
- Maintenir une cohérence de versions
- Faciliter les tests et le développement parallèle

### Structure d'un workspace

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

### Configuration du workspace

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

# Profil de release optimisé
[profile.release]
lto = true
strip = true
panic = "abort"
codegen-units = 1
```

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

### Commandes workspace

```bash
# Compiler tout le workspace
cargo build

# Compiler une crate spécifique
cargo build -p mon_app_core

# Lancer l'application
cargo run -p mon_app

# Tests de tout le workspace
cargo test

# Tests d'une crate
cargo test -p mon_app_core
```

---

## 5.2 Crates Modulaires

### Principe de séparation

```
┌─────────────────────────────────────────────────┐
│                     app                          │
│        (Point d'entrée, orchestration)          │
├─────────────────────────────────────────────────┤
│                      ui                          │
│    (Interface utilisateur, composants)          │
├─────────────────────────────────────────────────┤
│                     core                         │
│   (Logique métier pure, aucune dépendance I/O)  │
├─────────────────────────────────────────────────┤
│                   storage                        │
│      (Base de données, fichiers, cache)         │
└─────────────────────────────────────────────────┘

Règle : les dépendances vont de haut en bas
       ui et storage dépendent de core
       core ne dépend de rien d'applicatif
```

### Crate `core` : logique métier pure

```rust
// crates/core/src/lib.rs
pub mod domain;
pub mod services;
pub mod error;

pub use domain::*;
pub use error::CoreError;
```

```rust
// crates/core/src/domain/mod.rs
mod invoice;
mod customer;
mod product;

pub use invoice::*;
pub use customer::*;
pub use product::*;
```

```rust
// crates/core/src/domain/invoice.rs
use serde::{Deserialize, Serialize};
use rust_decimal::Decimal;

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct Invoice {
    pub id: InvoiceId,
    pub customer_id: CustomerId,
    pub items: Vec<InvoiceItem>,
    pub status: InvoiceStatus,
    pub created_at: DateTime<Utc>,
}

impl Invoice {
    pub fn subtotal(&self) -> Decimal {
        self.items.iter().map(|i| i.total()).sum()
    }
    
    pub fn tax(&self, rate: Decimal) -> Decimal {
        self.subtotal() * rate
    }
    
    pub fn total(&self, tax_rate: Decimal) -> Decimal {
        self.subtotal() + self.tax(tax_rate)
    }
}

// Pas de dépendance à la base de données ici !
// Pas de dépendance à l'UI ici !
// Logique pure et testable
```

### Crate `storage` : persistence

```rust
// crates/storage/src/lib.rs
pub mod database;
pub mod repository;
pub mod migrations;

pub use database::Database;
pub use repository::*;
```

```rust
// crates/storage/src/repository/invoice_repository.rs
use mon_app_core::{Invoice, InvoiceId, CustomerId};
use anyhow::Result;

/// Trait définissant les opérations sur les factures
pub trait InvoiceRepository {
    fn save(&self, invoice: &Invoice) -> Result<()>;
    fn find_by_id(&self, id: InvoiceId) -> Result<Option<Invoice>>;
    fn find_by_customer(&self, customer_id: CustomerId) -> Result<Vec<Invoice>>;
    fn delete(&self, id: InvoiceId) -> Result<bool>;
}

/// Implémentation SQLite
pub struct SqliteInvoiceRepository {
    db: rusqlite::Connection,
}

impl InvoiceRepository for SqliteInvoiceRepository {
    fn save(&self, invoice: &Invoice) -> Result<()> {
        // Implémentation SQLite
        self.db.execute(
            "INSERT OR REPLACE INTO invoices (id, data, customer_id, created_at)
             VALUES (?1, ?2, ?3, ?4)",
            params![
                invoice.id.0,
                serde_json::to_string(invoice)?,
                invoice.customer_id.0,
                invoice.created_at.timestamp(),
            ],
        )?;
        Ok(())
    }
    
    // ... autres méthodes
}
```

### Crate `ui` : interface

```rust
// crates/ui/src/lib.rs
pub mod app;
pub mod components;
pub mod screens;
pub mod theme;

pub use app::App;
pub use theme::Theme;
```

```rust
// crates/ui/src/screens/invoice_screen.rs
use mon_app_core::Invoice;
use egui::{Ui, Response};

pub struct InvoiceScreen {
    invoices: Vec<Invoice>,
    selected: Option<usize>,
}

impl InvoiceScreen {
    pub fn new(invoices: Vec<Invoice>) -> Self {
        Self { invoices, selected: None }
    }
    
    pub fn render(&mut self, ui: &mut Ui) {
        ui.heading("Factures");
        
        for (idx, invoice) in self.invoices.iter().enumerate() {
            let is_selected = self.selected == Some(idx);
            
            if ui.selectable_label(is_selected, &format!("#{}", invoice.id)).clicked() {
                self.selected = Some(idx);
            }
        }
    }
}
```

---

## 5.3 Architecture de Dossiers Claire

### Structure complète recommandée

```
mon_application/
├── Cargo.toml
├── Cargo.lock
├── README.md
├── LICENSE
├── .gitignore
│
├── crates/
│   ├── app/
│   │   ├── Cargo.toml
│   │   └── src/
│   │       ├── main.rs          # Point d'entrée
│   │       └── cli.rs           # Arguments ligne de commande
│   │
│   ├── core/
│   │   ├── Cargo.toml
│   │   └── src/
│   │       ├── lib.rs
│   │       ├── domain/          # Entités métier
│   │       │   ├── mod.rs
│   │       │   ├── invoice.rs
│   │       │   ├── customer.rs
│   │       │   └── product.rs
│   │       ├── services/        # Logique métier
│   │       │   ├── mod.rs
│   │       │   ├── billing.rs
│   │       │   └── reporting.rs
│   │       └── error.rs
│   │
│   ├── ui/
│   │   ├── Cargo.toml
│   │   └── src/
│   │       ├── lib.rs
│   │       ├── app.rs           # Application egui
│   │       ├── theme/           # Design system
│   │       │   ├── mod.rs
│   │       │   ├── colors.rs
│   │       │   ├── spacing.rs
│   │       │   └── typography.rs
│   │       ├── components/      # Composants réutilisables
│   │       │   ├── mod.rs
│   │       │   ├── button.rs
│   │       │   ├── input.rs
│   │       │   ├── table.rs
│   │       │   └── modal.rs
│   │       └── screens/         # Écrans de l'app
│   │           ├── mod.rs
│   │           ├── dashboard.rs
│   │           ├── invoices.rs
│   │           └── settings.rs
│   │
│   └── storage/
│       ├── Cargo.toml
│       └── src/
│           ├── lib.rs
│           ├── database.rs      # Connexion SQLite
│           ├── migrations/      # Migrations SQL
│           │   ├── mod.rs
│           │   ├── v001_initial.sql
│           │   └── v002_add_index.sql
│           └── repository/      # Repositories
│               ├── mod.rs
│               ├── invoice_repo.rs
│               └── customer_repo.rs
│
├── tests/                       # Tests d'intégration
│   ├── common/
│   │   └── mod.rs
│   ├── invoice_flow_test.rs
│   └── export_test.rs
│
├── benches/                     # Benchmarks
│   └── search_benchmark.rs
│
├── assets/                      # Ressources statiques
│   ├── icons/
│   ├── fonts/
│   └── images/
│
├── config/                      # Fichiers de config exemple
│   └── default.toml
│
└── scripts/                     # Scripts de build/deploy
    ├── build-release.sh
    └── package-windows.ps1
```

### Conventions de nommage

```rust
// Fichiers et modules : snake_case
// crates/core/src/domain/invoice_item.rs
mod invoice_item;

// Structs et Enums : PascalCase
pub struct InvoiceItem { ... }
pub enum InvoiceStatus { ... }

// Fonctions et méthodes : snake_case
fn calculate_total() { ... }
impl Invoice {
    fn add_item(&mut self) { ... }
}

// Constantes : SCREAMING_SNAKE_CASE
const MAX_ITEMS_PER_INVOICE: usize = 100;
const DEFAULT_TAX_RATE: Decimal = dec!(0.20);

// Traits : PascalCase, souvent avec suffixe -able ou -er
trait Billable { ... }
trait InvoiceRepository { ... }
```

---

## 5.4 Tests et Benchmarks

### Tests unitaires (dans chaque crate)

```rust
// crates/core/src/domain/invoice.rs

#[cfg(test)]
mod tests {
    use super::*;
    use rust_decimal_macros::dec;
    
    fn sample_invoice() -> Invoice {
        Invoice {
            id: InvoiceId(1),
            customer_id: CustomerId(1),
            items: vec![
                InvoiceItem {
                    description: "Service A".to_string(),
                    quantity: dec!(2),
                    unit_price: dec!(100),
                },
                InvoiceItem {
                    description: "Service B".to_string(),
                    quantity: dec!(1),
                    unit_price: dec!(50),
                },
            ],
            status: InvoiceStatus::Draft,
            created_at: Utc::now(),
        }
    }
    
    #[test]
    fn test_subtotal_calculation() {
        let invoice = sample_invoice();
        assert_eq!(invoice.subtotal(), dec!(250));  // 2*100 + 1*50
    }
    
    #[test]
    fn test_tax_calculation() {
        let invoice = sample_invoice();
        let tax_rate = dec!(0.20);  // 20%
        assert_eq!(invoice.tax(tax_rate), dec!(50));  // 250 * 0.20
    }
    
    #[test]
    fn test_total_with_tax() {
        let invoice = sample_invoice();
        let tax_rate = dec!(0.20);
        assert_eq!(invoice.total(tax_rate), dec!(300));  // 250 + 50
    }
    
    #[test]
    fn test_empty_invoice() {
        let invoice = Invoice {
            items: vec![],
            ..sample_invoice()
        };
        assert_eq!(invoice.subtotal(), dec!(0));
    }
}
```

### Tests d'intégration (dossier tests/)

```rust
// tests/invoice_flow_test.rs
use mon_app_core::{Invoice, InvoiceStatus};
use mon_app_storage::{Database, InvoiceRepository, SqliteInvoiceRepository};
use tempfile::tempdir;

fn setup_test_db() -> (Database, tempfile::TempDir) {
    let dir = tempdir().unwrap();
    let db_path = dir.path().join("test.db");
    let db = Database::open(&db_path).unwrap();
    db.run_migrations().unwrap();
    (db, dir)
}

#[test]
fn test_invoice_crud_flow() {
    let (db, _dir) = setup_test_db();
    let repo = SqliteInvoiceRepository::new(&db);
    
    // Create
    let invoice = Invoice::new(CustomerId(1));
    repo.save(&invoice).unwrap();
    
    // Read
    let loaded = repo.find_by_id(invoice.id).unwrap();
    assert!(loaded.is_some());
    assert_eq!(loaded.unwrap().customer_id, CustomerId(1));
    
    // Update
    let mut updated = invoice.clone();
    updated.status = InvoiceStatus::Sent;
    repo.save(&updated).unwrap();
    
    let loaded = repo.find_by_id(invoice.id).unwrap().unwrap();
    assert_eq!(loaded.status, InvoiceStatus::Sent);
    
    // Delete
    assert!(repo.delete(invoice.id).unwrap());
    assert!(repo.find_by_id(invoice.id).unwrap().is_none());
}

#[test]
fn test_find_by_customer() {
    let (db, _dir) = setup_test_db();
    let repo = SqliteInvoiceRepository::new(&db);
    
    // Créer plusieurs factures
    let customer_1 = CustomerId(1);
    let customer_2 = CustomerId(2);
    
    repo.save(&Invoice::new(customer_1)).unwrap();
    repo.save(&Invoice::new(customer_1)).unwrap();
    repo.save(&Invoice::new(customer_2)).unwrap();
    
    // Vérifier le filtrage
    let invoices_1 = repo.find_by_customer(customer_1).unwrap();
    assert_eq!(invoices_1.len(), 2);
    
    let invoices_2 = repo.find_by_customer(customer_2).unwrap();
    assert_eq!(invoices_2.len(), 1);
}
```

### Benchmarks avec Criterion

```toml
# Cargo.toml (racine ou crate spécifique)
[dev-dependencies]
criterion = { version = "0.5", features = ["html_reports"] }

[[bench]]
name = "search_benchmark"
harness = false
```

```rust
// benches/search_benchmark.rs
use criterion::{black_box, criterion_group, criterion_main, Criterion, BenchmarkId};
use mon_app_core::Invoice;
use mon_app_storage::{Database, InvoiceRepository, SqliteInvoiceRepository};

fn create_test_data(repo: &SqliteInvoiceRepository, count: usize) {
    for i in 0..count {
        let mut invoice = Invoice::new(CustomerId((i % 100) as i64));
        invoice.add_item(InvoiceItem {
            description: format!("Item {}", i),
            quantity: dec!(1),
            unit_price: dec!(100),
        });
        repo.save(&invoice).unwrap();
    }
}

fn benchmark_search(c: &mut Criterion) {
    let db = Database::open_in_memory().unwrap();
    let repo = SqliteInvoiceRepository::new(&db);
    
    // Préparer les données
    create_test_data(&repo, 10_000);
    
    let mut group = c.benchmark_group("Invoice Search");
    
    for size in [100, 1000, 10000].iter() {
        group.bench_with_input(
            BenchmarkId::from_parameter(size),
            size,
            |b, &_size| {
                b.iter(|| {
                    black_box(repo.find_by_customer(CustomerId(42)).unwrap())
                })
            },
        );
    }
    
    group.finish();
}

fn benchmark_serialization(c: &mut Criterion) {
    let invoice = Invoice::new(CustomerId(1));
    
    c.bench_function("serialize invoice", |b| {
        b.iter(|| {
            black_box(serde_json::to_string(&invoice).unwrap())
        })
    });
    
    let json = serde_json::to_string(&invoice).unwrap();
    c.bench_function("deserialize invoice", |b| {
        b.iter(|| {
            black_box(serde_json::from_str::<Invoice>(&json).unwrap())
        })
    });
}

criterion_group!(benches, benchmark_search, benchmark_serialization);
criterion_main!(benches);
```

```bash
# Lancer les benchmarks
cargo bench

# Résultat : rapport HTML dans target/criterion/
```

---

## Résumé du Chapitre

### Structure recommandée

| Crate | Responsabilité | Dépendances |
|-------|----------------|-------------|
| `app` | Point d'entrée, orchestration | Toutes les autres |
| `core` | Logique métier pure | Aucune crate interne |
| `ui` | Interface utilisateur | `core` |
| `storage` | Persistence | `core` |

### Commandes essentielles

```bash
# Développement
cargo build                    # Compiler tout
cargo run -p mon_app           # Lancer l'app
cargo test                     # Tous les tests
cargo test -p mon_app_core     # Tests d'une crate

# Release
cargo build --release          # Build optimisé
cargo bench                    # Benchmarks

# Maintenance
cargo clippy                   # Linter
cargo fmt                      # Formatage
cargo doc --open               # Documentation
```

---

**Dans le prochain chapitre, nous aborderons l'async et la concurrence pour des applications desktop réactives.**
