# 18.2 Factures

## Génération de factures PDF

```rust
use printpdf::*;

pub struct PdfGenerator {
    fonts: PdfFonts,
}

impl PdfGenerator {
    pub fn generate_invoice(&self, invoice: &Invoice, customer: &Customer) -> Vec<u8> {
        let (doc, page1, layer1) = PdfDocument::new(
            "Invoice",
            Mm(210.0), Mm(297.0), // A4
            "Layer 1",
        );
        
        let current_layer = doc.get_page(page1).get_layer(layer1);
        
        // En-tête
        current_layer.use_text(
            &format!("FACTURE N° {}", invoice.number),
            24.0,
            Mm(20.0), Mm(270.0),
            &self.fonts.bold,
        );
        
        // Informations client
        current_layer.use_text(
            &customer.name,
            12.0,
            Mm(20.0), Mm(240.0),
            &self.fonts.regular,
        );
        
        // Tableau des items
        let mut y = 200.0;
        for item in &invoice.items {
            current_layer.use_text(
                &item.description,
                10.0,
                Mm(20.0), Mm(y),
                &self.fonts.regular,
            );
            current_layer.use_text(
                &format!("{:.2} €", item.total()),
                10.0,
                Mm(160.0), Mm(y),
                &self.fonts.regular,
            );
            y -= 8.0;
        }
        
        // Total
        current_layer.use_text(
            &format!("TOTAL: {:.2} €", invoice.total(Decimal::new(20, 2))),
            14.0,
            Mm(140.0), Mm(50.0),
            &self.fonts.bold,
        );
        
        doc.save_to_bytes().unwrap()
    }
}
```

## Mise en page professionnelle

```rust
impl PdfGenerator {
    fn render_invoice_header(&self, layer: &PdfLayerReference, invoice: &Invoice) {
        // Logo (si disponible)
        // Coordonnées entreprise
        // Numéro de facture
        // Date
    }
    
    fn render_customer_info(&self, layer: &PdfLayerReference, customer: &Customer) {
        // Nom et adresse client
        // Coordonnées de facturation
    }
    
    fn render_items_table(&self, layer: &PdfLayerReference, items: &[InvoiceItem]) {
        // Tableau avec colonnes : Description, Qté, Prix unitaire, Total
        // Lignes pour chaque item
    }
    
    fn render_totals(&self, layer: &PdfLayerReference, invoice: &Invoice) {
        // Sous-total HT
        // TVA
        // Total TTC
    }
}
```

## Calculs automatiques

```rust
impl Invoice {
    pub fn calculate_totals(&self, vat_rate: Decimal) -> InvoiceTotals {
        let subtotal = self.items.iter().map(|i| i.total()).sum();
        let vat = subtotal * vat_rate;
        let total = subtotal + vat;
        
        InvoiceTotals { subtotal, vat, total, vat_rate }
    }
}
```

## Résumé

- **PDF professionnel** : Mise en page soignée
- **Sections** : En-tête, client, items, totaux
- **Calculs** : Automatiques depuis les données
- **Format standard** : A4, marges appropriées
