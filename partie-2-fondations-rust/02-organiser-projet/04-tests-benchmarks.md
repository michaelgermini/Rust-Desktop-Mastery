# 5.4 Tests et Benchmarks

## Tests unitaires (dans chaque crate)

### Structure des tests

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

## Tests d'intégration (dossier tests/)

### Setup commun

```rust
// tests/common/mod.rs
use mon_app_storage::{Database, InvoiceRepository, SqliteInvoiceRepository};
use tempfile::tempdir;

pub fn setup_test_db() -> (Database, tempfile::TempDir) {
    let dir = tempdir().unwrap();
    let db_path = dir.path().join("test.db");
    let db = Database::open(&db_path).unwrap();
    db.run_migrations().unwrap();
    (db, dir)
}
```

### Test d'intégration complet

```rust
// tests/invoice_flow_test.rs
use mon_app_core::{Invoice, InvoiceStatus};
use mon_app_storage::{Database, InvoiceRepository, SqliteInvoiceRepository};
use tempfile::tempdir;

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
```

## Benchmarks avec Criterion

### Configuration

```toml
# Cargo.toml
[dev-dependencies]
criterion = { version = "0.5", features = ["html_reports"] }

[[bench]]
name = "search_benchmark"
harness = false
```

### Benchmark de recherche

```rust
// benches/search_benchmark.rs
use criterion::{black_box, criterion_group, criterion_main, Criterion, BenchmarkId};
use mon_app_core::Invoice;
use mon_app_storage::{Database, InvoiceRepository, SqliteInvoiceRepository};

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

criterion_group!(benches, benchmark_search);
criterion_main!(benches);
```

### Lancer les benchmarks

```bash
# Lancer les benchmarks
cargo bench

# Résultat : rapport HTML dans target/criterion/
```

## Résumé

- **Tests unitaires** : Dans chaque crate, à côté du code
- **Tests d'intégration** : Dans `tests/`, testent plusieurs crates ensemble
- **Benchmarks** : Dans `benches/`, mesurent les performances
- **Criterion** : Outil standard pour les benchmarks Rust
