# 5.2 Crates Modulaires

## Principe de séparation

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

## Crate `core` : logique métier pure

### Structure de base

```rust
// crates/core/src/lib.rs
pub mod domain;
pub mod services;
pub mod error;

pub use domain::*;
pub use error::CoreError;
```

### Domain entities

```rust
// crates/core/src/domain/mod.rs
mod invoice;
mod customer;
mod product;

pub use invoice::*;
pub use customer::*;
pub use product::*;
```

### Exemple : Invoice

```rust
// crates/core/src/domain/invoice.rs
use serde::{Deserialize, Serialize};
use rust_decimal::Decimal;
use chrono::{DateTime, Utc};

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

## Crate `storage` : persistence

### Structure

```rust
// crates/storage/src/lib.rs
pub mod database;
pub mod repository;
pub mod migrations;

pub use database::Database;
pub use repository::*;
```

### Repository pattern

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

## Crate `ui` : interface

### Structure

```rust
// crates/ui/src/lib.rs
pub mod app;
pub mod components;
pub mod screens;
pub mod theme;

pub use app::App;
pub use theme::Theme;
```

### Exemple d'écran

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

## Résumé

- **core** : Logique métier pure, aucune dépendance applicative
- **storage** : Persistence, dépend de `core`
- **ui** : Interface, dépend de `core`
- **app** : Orchestration, dépend de tout
