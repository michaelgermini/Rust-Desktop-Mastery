# Chapitre 14 : Clean Architecture pour Apps Desktop

## Introduction

La Clean Architecture, popularisée par Robert C. Martin, propose une organisation du code en couches concentriques où les dépendances pointent vers l'intérieur. Ce chapitre adapte ces principes au développement desktop en Rust.

---

## 14.1 Les Couches de l'Architecture

### Vue d'ensemble

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

### Structure de projet

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

---

## 14.2 Couche Core

### Entités

```rust
// crates/core/src/entities/invoice.rs

use rust_decimal::Decimal;
use chrono::{DateTime, Utc};
use serde::{Deserialize, Serialize};

/// Identifiant unique de facture (Value Object)
#[derive(Debug, Clone, Copy, PartialEq, Eq, Hash, Serialize, Deserialize)]
pub struct InvoiceId(pub i64);

/// Statut d'une facture
#[derive(Debug, Clone, Copy, PartialEq, Eq, Serialize, Deserialize)]
pub enum InvoiceStatus {
    Draft,
    Sent,
    Paid,
    Cancelled,
}

/// Entité Facture - logique métier pure
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct Invoice {
    pub id: InvoiceId,
    pub number: String,
    pub customer_id: CustomerId,
    pub items: Vec<InvoiceItem>,
    pub status: InvoiceStatus,
    pub issue_date: DateTime<Utc>,
    pub due_date: DateTime<Utc>,
    pub notes: Option<String>,
}

impl Invoice {
    /// Créer une nouvelle facture (brouillon)
    pub fn new(customer_id: CustomerId) -> Self {
        Self {
            id: InvoiceId(0),  // Sera assigné par le repository
            number: String::new(),  // Sera généré
            customer_id,
            items: Vec::new(),
            status: InvoiceStatus::Draft,
            issue_date: Utc::now(),
            due_date: Utc::now() + chrono::Duration::days(30),
            notes: None,
        }
    }
    
    /// Sous-total HT
    pub fn subtotal(&self) -> Decimal {
        self.items.iter().map(|i| i.total()).sum()
    }
    
    /// TVA
    pub fn vat(&self, rate: Decimal) -> Decimal {
        self.subtotal() * rate
    }
    
    /// Total TTC
    pub fn total(&self, vat_rate: Decimal) -> Decimal {
        self.subtotal() + self.vat(vat_rate)
    }
    
    /// Marquer comme envoyée
    pub fn send(&mut self) -> Result<(), InvoiceError> {
        match self.status {
            InvoiceStatus::Draft => {
                if self.items.is_empty() {
                    return Err(InvoiceError::EmptyInvoice);
                }
                self.status = InvoiceStatus::Sent;
                Ok(())
            }
            _ => Err(InvoiceError::InvalidStatusTransition),
        }
    }
    
    /// Marquer comme payée
    pub fn mark_paid(&mut self) -> Result<(), InvoiceError> {
        match self.status {
            InvoiceStatus::Sent => {
                self.status = InvoiceStatus::Paid;
                Ok(())
            }
            _ => Err(InvoiceError::InvalidStatusTransition),
        }
    }
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct InvoiceItem {
    pub description: String,
    pub quantity: Decimal,
    pub unit_price: Decimal,
}

impl InvoiceItem {
    pub fn total(&self) -> Decimal {
        self.quantity * self.unit_price
    }
}
```

### Ports (Interfaces)

```rust
// crates/core/src/ports/mod.rs

use crate::entities::*;
use crate::error::CoreError;

/// Port pour la persistence des factures
/// Implémenté dans la couche Infrastructure
pub trait InvoiceRepository: Send + Sync {
    fn save(&self, invoice: &Invoice) -> Result<InvoiceId, CoreError>;
    fn find_by_id(&self, id: InvoiceId) -> Result<Option<Invoice>, CoreError>;
    fn find_by_customer(&self, customer_id: CustomerId) -> Result<Vec<Invoice>, CoreError>;
    fn find_all(&self) -> Result<Vec<Invoice>, CoreError>;
    fn delete(&self, id: InvoiceId) -> Result<bool, CoreError>;
    fn next_number(&self) -> Result<String, CoreError>;
}

/// Port pour la génération de PDF
pub trait PdfGenerator: Send + Sync {
    fn generate_invoice(&self, invoice: &Invoice, customer: &Customer) -> Result<Vec<u8>, CoreError>;
}

/// Port pour l'envoi d'emails
pub trait EmailSender: Send + Sync {
    fn send_invoice(&self, invoice: &Invoice, customer: &Customer, pdf: &[u8]) -> Result<(), CoreError>;
}
```

### Services métier

```rust
// crates/core/src/services/billing.rs

use crate::entities::*;
use crate::ports::*;
use crate::error::CoreError;
use rust_decimal::Decimal;

/// Service de facturation - logique métier pure
pub struct BillingService {
    vat_rate: Decimal,
}

impl BillingService {
    pub fn new(vat_rate: Decimal) -> Self {
        Self { vat_rate }
    }
    
    /// Calculer le total d'une facture
    pub fn calculate_totals(&self, invoice: &Invoice) -> InvoiceTotals {
        let subtotal = invoice.subtotal();
        let vat = invoice.vat(self.vat_rate);
        let total = subtotal + vat;
        
        InvoiceTotals { subtotal, vat, total, vat_rate: self.vat_rate }
    }
    
    /// Valider une facture avant envoi
    pub fn validate_for_sending(&self, invoice: &Invoice) -> Result<(), Vec<ValidationError>> {
        let mut errors = Vec::new();
        
        if invoice.items.is_empty() {
            errors.push(ValidationError::EmptyInvoice);
        }
        
        if invoice.number.is_empty() {
            errors.push(ValidationError::MissingNumber);
        }
        
        for (i, item) in invoice.items.iter().enumerate() {
            if item.quantity <= Decimal::ZERO {
                errors.push(ValidationError::InvalidQuantity(i));
            }
            if item.unit_price < Decimal::ZERO {
                errors.push(ValidationError::NegativePrice(i));
            }
        }
        
        if errors.is_empty() {
            Ok(())
        } else {
            Err(errors)
        }
    }
}

pub struct InvoiceTotals {
    pub subtotal: Decimal,
    pub vat: Decimal,
    pub total: Decimal,
    pub vat_rate: Decimal,
}
```

---

## 14.3 Couche Application

### Commands (Use Cases d'écriture)

```rust
// crates/application/src/commands/create_invoice.rs

use core::{Invoice, InvoiceItem, CustomerId, InvoiceRepository};
use crate::error::AppError;

/// Commande pour créer une facture
pub struct CreateInvoiceCommand {
    pub customer_id: CustomerId,
    pub items: Vec<InvoiceItemInput>,
    pub notes: Option<String>,
}

pub struct InvoiceItemInput {
    pub description: String,
    pub quantity: f64,
    pub unit_price: f64,
}

/// Handler de la commande
pub struct CreateInvoiceHandler<R: InvoiceRepository> {
    repository: R,
}

impl<R: InvoiceRepository> CreateInvoiceHandler<R> {
    pub fn new(repository: R) -> Self {
        Self { repository }
    }
    
    pub fn handle(&self, cmd: CreateInvoiceCommand) -> Result<Invoice, AppError> {
        // 1. Créer l'entité
        let mut invoice = Invoice::new(cmd.customer_id);
        
        // 2. Ajouter les items
        for item in cmd.items {
            invoice.items.push(InvoiceItem {
                description: item.description,
                quantity: Decimal::from_f64(item.quantity).unwrap_or_default(),
                unit_price: Decimal::from_f64(item.unit_price).unwrap_or_default(),
            });
        }
        
        invoice.notes = cmd.notes;
        
        // 3. Générer le numéro
        invoice.number = self.repository.next_number()?;
        
        // 4. Persister
        let id = self.repository.save(&invoice)?;
        invoice.id = id;
        
        Ok(invoice)
    }
}
```

### Queries (Use Cases de lecture)

```rust
// crates/application/src/queries/get_invoices.rs

use core::{Invoice, InvoiceRepository, InvoiceStatus, CustomerId};
use crate::error::AppError;

/// Critères de filtrage
pub struct InvoiceFilter {
    pub status: Option<InvoiceStatus>,
    pub customer_id: Option<CustomerId>,
    pub search_query: Option<String>,
}

/// Query handler
pub struct GetInvoicesQuery<R: InvoiceRepository> {
    repository: R,
}

impl<R: InvoiceRepository> GetInvoicesQuery<R> {
    pub fn new(repository: R) -> Self {
        Self { repository }
    }
    
    pub fn execute(&self, filter: InvoiceFilter) -> Result<Vec<Invoice>, AppError> {
        let mut invoices = self.repository.find_all()?;
        
        // Appliquer les filtres
        if let Some(status) = filter.status {
            invoices.retain(|i| i.status == status);
        }
        
        if let Some(customer_id) = filter.customer_id {
            invoices.retain(|i| i.customer_id == customer_id);
        }
        
        if let Some(query) = filter.search_query {
            let query = query.to_lowercase();
            invoices.retain(|i| {
                i.number.to_lowercase().contains(&query)
            });
        }
        
        Ok(invoices)
    }
}
```

---

## 14.4 Couche Infrastructure

### Repository SQLite

```rust
// crates/infrastructure/src/repositories/sqlite_invoice_repository.rs

use core::{Invoice, InvoiceId, CustomerId, InvoiceRepository, CoreError};
use rusqlite::{Connection, params};
use std::sync::Mutex;

pub struct SqliteInvoiceRepository {
    conn: Mutex<Connection>,
}

impl SqliteInvoiceRepository {
    pub fn new(db_path: &str) -> Result<Self, CoreError> {
        let conn = Connection::open(db_path)?;
        let repo = Self { conn: Mutex::new(conn) };
        repo.init_schema()?;
        Ok(repo)
    }
    
    fn init_schema(&self) -> Result<(), CoreError> {
        let conn = self.conn.lock().unwrap();
        conn.execute(
            "CREATE TABLE IF NOT EXISTS invoices (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                number TEXT NOT NULL UNIQUE,
                customer_id INTEGER NOT NULL,
                status TEXT NOT NULL,
                issue_date TEXT NOT NULL,
                due_date TEXT NOT NULL,
                notes TEXT,
                items_json TEXT NOT NULL
            )",
            [],
        )?;
        Ok(())
    }
}

impl InvoiceRepository for SqliteInvoiceRepository {
    fn save(&self, invoice: &Invoice) -> Result<InvoiceId, CoreError> {
        let conn = self.conn.lock().unwrap();
        let items_json = serde_json::to_string(&invoice.items)?;
        
        if invoice.id.0 == 0 {
            // Insert
            conn.execute(
                "INSERT INTO invoices (number, customer_id, status, issue_date, due_date, notes, items_json)
                 VALUES (?1, ?2, ?3, ?4, ?5, ?6, ?7)",
                params![
                    invoice.number,
                    invoice.customer_id.0,
                    format!("{:?}", invoice.status),
                    invoice.issue_date.to_rfc3339(),
                    invoice.due_date.to_rfc3339(),
                    invoice.notes,
                    items_json,
                ],
            )?;
            Ok(InvoiceId(conn.last_insert_rowid()))
        } else {
            // Update
            conn.execute(
                "UPDATE invoices SET 
                    number = ?1, customer_id = ?2, status = ?3,
                    issue_date = ?4, due_date = ?5, notes = ?6, items_json = ?7
                 WHERE id = ?8",
                params![
                    invoice.number,
                    invoice.customer_id.0,
                    format!("{:?}", invoice.status),
                    invoice.issue_date.to_rfc3339(),
                    invoice.due_date.to_rfc3339(),
                    invoice.notes,
                    items_json,
                    invoice.id.0,
                ],
            )?;
            Ok(invoice.id)
        }
    }
    
    fn find_by_id(&self, id: InvoiceId) -> Result<Option<Invoice>, CoreError> {
        let conn = self.conn.lock().unwrap();
        let mut stmt = conn.prepare(
            "SELECT id, number, customer_id, status, issue_date, due_date, notes, items_json
             FROM invoices WHERE id = ?1"
        )?;
        
        let invoice = stmt.query_row([id.0], |row| {
            Ok(self.row_to_invoice(row)?)
        }).optional()?;
        
        Ok(invoice)
    }
    
    fn find_all(&self) -> Result<Vec<Invoice>, CoreError> {
        let conn = self.conn.lock().unwrap();
        let mut stmt = conn.prepare(
            "SELECT id, number, customer_id, status, issue_date, due_date, notes, items_json
             FROM invoices ORDER BY issue_date DESC"
        )?;
        
        let invoices = stmt.query_map([], |row| {
            Ok(self.row_to_invoice(row)?)
        })?.collect::<Result<Vec<_>, _>>()?;
        
        Ok(invoices)
    }
    
    fn next_number(&self) -> Result<String, CoreError> {
        let conn = self.conn.lock().unwrap();
        let year = chrono::Utc::now().format("%Y");
        
        let count: i64 = conn.query_row(
            "SELECT COUNT(*) FROM invoices WHERE number LIKE ?1",
            [format!("{}-%", year)],
            |row| row.get(0),
        )?;
        
        Ok(format!("{}-{:04}", year, count + 1))
    }
    
    // ... autres méthodes
}
```

---

## 14.5 Assemblage Final

### Point d'entrée

```rust
// src/main.rs

use std::sync::Arc;
use infrastructure::SqliteInvoiceRepository;
use application::{CreateInvoiceHandler, GetInvoicesQuery};
use ui::App;

fn main() -> eframe::Result<()> {
    // 1. Initialiser l'infrastructure
    let db_path = dirs::data_dir()
        .unwrap()
        .join("mon_app")
        .join("data.db");
    
    std::fs::create_dir_all(db_path.parent().unwrap()).ok();
    
    let invoice_repo = Arc::new(
        SqliteInvoiceRepository::new(db_path.to_str().unwrap())
            .expect("Failed to open database")
    );
    
    // 2. Créer les handlers
    let create_invoice = CreateInvoiceHandler::new(Arc::clone(&invoice_repo));
    let get_invoices = GetInvoicesQuery::new(Arc::clone(&invoice_repo));
    
    // 3. Créer l'application UI
    let app = App::new(create_invoice, get_invoices);
    
    // 4. Lancer
    eframe::run_native(
        "Mon Application",
        eframe::NativeOptions::default(),
        Box::new(|_cc| Ok(Box::new(app))),
    )
}
```

---

## Résumé du Chapitre

### Principes clés

| Couche | Responsabilité | Dépendances |
|--------|----------------|-------------|
| Core | Logique métier pure | Aucune |
| Application | Orchestration, Use cases | Core |
| Infrastructure | Implémentations techniques | Core, Application |
| UI | Interface utilisateur | Application |

### Avantages

- **Testabilité** : Core testable sans base de données
- **Maintenabilité** : Changements localisés
- **Évolutivité** : Nouvelles features sans casser l'existant
- **Flexibilité** : Changer de base de données facilement

---

**Dans le prochain chapitre, nous verrons comment implémenter une communication efficace entre les couches avec un event bus.**
