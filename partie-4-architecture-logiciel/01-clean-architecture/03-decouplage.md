# 14.3 Découplage

## Dependency Inversion Principle

Le principe d'inversion des dépendances stipule que :
- Les modules de haut niveau ne doivent pas dépendre des modules de bas niveau
- Les deux doivent dépendre d'abstractions (traits)

## Traits et interfaces

```rust
// Core définit le contrat
pub trait InvoiceRepository: Send + Sync {
    fn save(&self, invoice: &Invoice) -> Result<InvoiceId, CoreError>;
    fn find_by_id(&self, id: InvoiceId) -> Result<Option<Invoice>, CoreError>;
}

// Application utilise le trait (pas l'implémentation)
pub struct CreateInvoiceHandler<R: InvoiceRepository> {
    repository: R,
}

// Infrastructure implémente le trait
impl InvoiceRepository for SqliteInvoiceRepository {
    // ...
}
```

## Injection de dépendances

```rust
// Dans main.rs ou un module de composition

fn main() -> eframe::Result<()> {
    // 1. Créer l'implémentation concrète
    let invoice_repo = Arc::new(
        SqliteInvoiceRepository::new("data.db")
            .expect("Failed to open database")
    );
    
    // 2. Injecter dans les handlers (qui acceptent le trait)
    let create_invoice = CreateInvoiceHandler::new(Arc::clone(&invoice_repo));
    let get_invoices = GetInvoicesQuery::new(Arc::clone(&invoice_repo));
    
    // 3. Utiliser dans l'application
    let app = App::new(create_invoice, get_invoices);
    
    // ...
}
```

## Avantages du découplage

### Testabilité

```rust
// Mock repository pour les tests
pub struct MockInvoiceRepository {
    invoices: Arc<Mutex<Vec<Invoice>>>,
}

impl InvoiceRepository for MockInvoiceRepository {
    fn save(&self, invoice: &Invoice) -> Result<InvoiceId, CoreError> {
        let mut invoices = self.invoices.lock().unwrap();
        invoices.push(invoice.clone());
        Ok(InvoiceId(invoices.len() as i64))
    }
    
    // ...
}

#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_create_invoice() {
        let repo = MockInvoiceRepository::new();
        let handler = CreateInvoiceHandler::new(repo);
        
        // Test sans base de données réelle
        let cmd = CreateInvoiceCommand { /* ... */ };
        let result = handler.handle(cmd);
        assert!(result.is_ok());
    }
}
```

### Flexibilité

```rust
// Changer d'implémentation facilement
let invoice_repo: Arc<dyn InvoiceRepository> = if use_sqlite {
    Arc::new(SqliteInvoiceRepository::new("data.db")?)
} else {
    Arc::new(FileInvoiceRepository::new("data/")?)
};
```

## Résumé

- **Dependency Inversion** : Dépendre d'abstractions, pas de concret
- **Traits** : Contrats définis dans Core
- **Injection** : Dépendances passées au constructeur
- **Testabilité** : Mocks faciles à créer
- **Flexibilité** : Changer d'implémentation sans modifier le code métier
