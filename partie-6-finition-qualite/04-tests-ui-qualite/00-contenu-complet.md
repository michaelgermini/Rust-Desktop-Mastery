# Chapitre 28 : Tests UI et Assurance Qualité

## Introduction

Les tests garantissent que l'application fonctionne correctement et continue de fonctionner après chaque modification.

---

## 28.1 Tests Unitaires

```rust
#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_invoice_calculation() {
        let mut invoice = Invoice::new(CustomerId(1));
        
        invoice.items.push(InvoiceItem {
            description: "Service A".to_string(),
            quantity: Decimal::new(2, 0),
            unit_price: Decimal::new(10000, 2),  // 100.00
        });
        
        invoice.items.push(InvoiceItem {
            description: "Service B".to_string(),
            quantity: Decimal::new(1, 0),
            unit_price: Decimal::new(5000, 2),   // 50.00
        });
        
        assert_eq!(invoice.subtotal(), Decimal::new(25000, 2));  // 250.00
        
        let vat_rate = Decimal::new(20, 2);  // 20%
        assert_eq!(invoice.vat(vat_rate), Decimal::new(5000, 2));  // 50.00
        assert_eq!(invoice.total(vat_rate), Decimal::new(30000, 2));  // 300.00
    }
    
    #[test]
    fn test_invoice_status_transitions() {
        let mut invoice = Invoice::new(CustomerId(1));
        invoice.items.push(InvoiceItem {
            description: "Test".to_string(),
            quantity: Decimal::ONE,
            unit_price: Decimal::new(100, 0),
        });
        invoice.number = "2024-001".to_string();
        
        // Draft -> Sent : OK
        assert!(invoice.send().is_ok());
        assert_eq!(invoice.status, InvoiceStatus::Sent);
        
        // Sent -> Sent : Erreur
        assert!(invoice.send().is_err());
        
        // Sent -> Paid : OK
        assert!(invoice.mark_paid().is_ok());
        assert_eq!(invoice.status, InvoiceStatus::Paid);
    }
}
```

---

## 28.2 Tests d'Intégration

```rust
// tests/integration_tests.rs

use tempfile::tempdir;

#[test]
fn test_full_invoice_workflow() {
    // Setup
    let dir = tempdir().unwrap();
    let db = Database::open(&dir.path().join("test.db")).unwrap();
    db.run_migrations().unwrap();
    
    let customer_repo = SqliteCustomerRepository::new(&db);
    let invoice_repo = SqliteInvoiceRepository::new(&db);
    
    // Créer un client
    let customer = Customer {
        id: CustomerId(0),
        name: "Test Client".to_string(),
        email: "test@example.com".to_string(),
        address: Address::default(),
        vat_number: None,
    };
    let customer_id = customer_repo.save(&customer).unwrap();
    
    // Créer une facture
    let mut invoice = Invoice::new(customer_id);
    invoice.items.push(InvoiceItem {
        description: "Service".to_string(),
        quantity: Decimal::new(1, 0),
        unit_price: Decimal::new(10000, 2),
    });
    
    let invoice_id = invoice_repo.save(&invoice).unwrap();
    
    // Vérifier
    let loaded = invoice_repo.find_by_id(invoice_id).unwrap().unwrap();
    assert_eq!(loaded.items.len(), 1);
    assert_eq!(loaded.customer_id, customer_id);
    
    // Nettoyer
    assert!(invoice_repo.delete(invoice_id).unwrap());
    assert!(invoice_repo.find_by_id(invoice_id).unwrap().is_none());
}
```

---

## 28.3 Tests de Snapshot UI

```rust
// Capturer l'état visuel pour comparaison
pub fn capture_ui_snapshot(ctx: &egui::Context, name: &str) -> UiSnapshot {
    let output = ctx.run(Default::default(), |ctx| {
        // Render UI
    });
    
    UiSnapshot {
        name: name.to_string(),
        shapes: output.shapes,
        text_output: output.text_output,
    }
}

#[test]
fn test_invoice_list_snapshot() {
    let ctx = egui::Context::default();
    let invoices = vec![
        Invoice::sample(),
        Invoice::sample(),
    ];
    
    let snapshot = capture_ui_snapshot(&ctx, "invoice_list");
    
    // Comparer avec le snapshot de référence
    let reference = load_reference_snapshot("invoice_list");
    assert_snapshots_match(&snapshot, &reference);
}
```

---

## 28.4 Checklist Qualité

```rust
/// Checklist de qualité avant release
pub struct QualityChecklist {
    pub tests_pass: bool,
    pub no_warnings: bool,
    pub no_clippy_lints: bool,
    pub documentation_complete: bool,
    pub changelog_updated: bool,
    pub version_bumped: bool,
}

impl QualityChecklist {
    pub fn run() -> Self {
        Self {
            tests_pass: run_tests(),
            no_warnings: check_warnings(),
            no_clippy_lints: run_clippy(),
            documentation_complete: check_docs(),
            changelog_updated: check_changelog(),
            version_bumped: check_version(),
        }
    }
    
    pub fn is_ready(&self) -> bool {
        self.tests_pass
            && self.no_warnings
            && self.no_clippy_lints
            && self.changelog_updated
            && self.version_bumped
    }
}
```

```bash
#!/bin/bash
# scripts/quality-check.sh

echo "🧪 Running tests..."
cargo test --all

echo "📝 Checking formatting..."
cargo fmt --check

echo "🔍 Running clippy..."
cargo clippy -- -D warnings

echo "📚 Building docs..."
cargo doc --no-deps

echo "✅ All checks passed!"
```

---

## Résumé

- **Tests unitaires** : Logique métier isolée
- **Tests d'intégration** : Workflow complet
- **Snapshots UI** : Régression visuelle
- **Checklist CI** : Qualité automatisée
