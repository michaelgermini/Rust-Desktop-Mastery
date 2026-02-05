# 28.1 Tests Unitaires

## Tests de la logique métier

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

## Tests des composants UI isolés

```rust
#[cfg(test)]
mod ui_tests {
    use super::*;
    
    #[test]
    fn test_button_rendering() {
        let ctx = egui::Context::default();
        let tokens = DesignTokens::light();
        
        let mut button_state = ButtonState::default();
        
        ctx.run(Default::default(), |ctx| {
            egui::CentralPanel::default().show(ctx, |ui| {
                let response = Button::new("Test")
                    .variant(ButtonVariant::Primary)
                    .show(ui, &tokens);
                
                assert!(response.clicked() == false);  // Pas encore cliqué
            });
        });
    }
    
    #[test]
    fn test_kpi_card_calculation() {
        let card = KpiCard::new("Revenue", 1000.0)
            .with_change(5.5);
        
        assert_eq!(card.value, "1000");
        assert_eq!(card.change, Some(5.5));
    }
}
```

## Mocking et fixtures

```rust
pub struct MockInvoiceRepository {
    invoices: Arc<Mutex<Vec<Invoice>>>,
}

impl InvoiceRepository for MockInvoiceRepository {
    fn save(&self, invoice: &Invoice) -> Result<InvoiceId, CoreError> {
        let mut invoices = self.invoices.lock().unwrap();
        let id = InvoiceId(invoices.len() as i64 + 1);
        let mut invoice = invoice.clone();
        invoice.id = id;
        invoices.push(invoice);
        Ok(id)
    }
    
    fn find_by_id(&self, id: InvoiceId) -> Result<Option<Invoice>, CoreError> {
        let invoices = self.invoices.lock().unwrap();
        Ok(invoices.iter().find(|i| i.id == id).cloned())
    }
    
    // Autres méthodes...
}

#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_create_invoice_with_mock() {
        let repo = MockInvoiceRepository::new();
        let handler = CreateInvoiceHandler::new(repo);
        
        let cmd = CreateInvoiceCommand {
            customer_id: CustomerId(1),
            items: vec![],
            notes: None,
        };
        
        let result = handler.handle(cmd);
        assert!(result.is_ok());
    }
}
```

## Résumé

- **Tests unitaires** : Logique métier isolée
- **Tests UI** : Composants testés individuellement
- **Mocks** : Repositories mockés pour tests rapides
- **Fixtures** : Données de test réutilisables
