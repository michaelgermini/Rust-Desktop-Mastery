# 14.2 Séparation Logique

## Entités métier (Core)

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
}
```

## Ports (Interfaces)

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
```

## Cas d'usage (Application)

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

## Implémentations (Infrastructure)

```rust
// crates/infrastructure/src/repositories/sqlite_invoice_repository.rs

use core::{Invoice, InvoiceId, InvoiceRepository, CoreError};
use rusqlite::{Connection, params};
use std::sync::Mutex;

pub struct SqliteInvoiceRepository {
    conn: Mutex<Connection>,
}

impl InvoiceRepository for SqliteInvoiceRepository {
    fn save(&self, invoice: &Invoice) -> Result<InvoiceId, CoreError> {
        // Implémentation SQLite
        // ...
    }
    
    // Autres méthodes...
}
```

## Résumé

- **Entités** : Logique métier pure dans Core
- **Ports** : Traits définissant les contrats
- **Use cases** : Orchestration dans Application
- **Implémentations** : Code technique dans Infrastructure
