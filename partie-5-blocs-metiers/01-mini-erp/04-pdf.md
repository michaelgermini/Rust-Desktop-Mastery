# 20.4 PDF

## Génération de factures PDF

```rust
use printpdf::*;

pub struct InvoicePdfGenerator {
    company_info: CompanyInfo,
    fonts: PdfFonts,
}

pub struct CompanyInfo {
    pub name: String,
    pub address: Address,
    pub siret: Option<String>,
    pub vat_number: Option<String>,
}

impl InvoicePdfGenerator {
    pub fn generate(&self, invoice: &Invoice, customer: &Customer) -> Vec<u8> {
        let (doc, page, layer) = PdfDocument::new("Invoice", Mm(210.0), Mm(297.0), "Main");
        let current_layer = doc.get_page(page).get_layer(layer);
        
        // Logo et en-tête entreprise
        self.draw_header(&current_layer);
        
        // Informations facture
        self.draw_invoice_info(&current_layer, invoice);
        
        // Informations client
        self.draw_customer_info(&current_layer, customer);
        
        // Tableau des items
        self.draw_items_table(&current_layer, invoice);
        
        // Totaux
        let breakdown = VatCalculator::new(invoice.vat_rate).calculate(invoice);
        self.draw_totals(&current_layer, &breakdown);
        
        // Pied de page
        self.draw_footer(&current_layer);
        
        doc.save_to_bytes().unwrap()
    }
    
    fn draw_items_table(&self, layer: &PdfLayerReference, invoice: &Invoice) {
        let mut y = 180.0;
        
        // En-têtes
        layer.use_text("Description", 10.0, Mm(20.0), Mm(y), &self.fonts.bold);
        layer.use_text("Qté", 10.0, Mm(100.0), Mm(y), &self.fonts.bold);
        layer.use_text("Prix unit.", 10.0, Mm(120.0), Mm(y), &self.fonts.bold);
        layer.use_text("Total HT", 10.0, Mm(160.0), Mm(y), &self.fonts.bold);
        
        y -= 8.0;
        
        // Lignes
        for item in &invoice.items {
            layer.use_text(&item.description, 9.0, Mm(20.0), Mm(y), &self.fonts.regular);
            layer.use_text(&format!("{}", item.quantity), 9.0, Mm(100.0), Mm(y), &self.fonts.regular);
            layer.use_text(&format!("{:.2} €", item.unit_price), 9.0, Mm(120.0), Mm(y), &self.fonts.regular);
            layer.use_text(&format!("{:.2} €", item.subtotal()), 9.0, Mm(160.0), Mm(y), &self.fonts.regular);
            y -= 6.0;
        }
    }
    
    fn draw_totals(&self, layer: &PdfLayerReference, breakdown: &VatBreakdown) {
        let mut y = 100.0;
        
        // Sous-total HT
        layer.use_text("Sous-total HT", 10.0, Mm(140.0), Mm(y), &self.fonts.regular);
        layer.use_text(&format!("{:.2} €", breakdown.subtotal), 10.0, Mm(180.0), Mm(y), &self.fonts.regular);
        y -= 8.0;
        
        // TVA par taux
        for line in &breakdown.lines {
            layer.use_text(&format!("TVA {}%", line.rate * Decimal::from(100)), 10.0, Mm(140.0), Mm(y), &self.fonts.regular);
            layer.use_text(&format!("{:.2} €", line.vat), 10.0, Mm(180.0), Mm(y), &self.fonts.regular);
            y -= 8.0;
        }
        
        // Total TTC
        layer.use_text("Total TTC", 12.0, Mm(140.0), Mm(y), &self.fonts.bold);
        layer.use_text(&format!("{:.2} €", breakdown.total), 12.0, Mm(180.0), Mm(y), &self.fonts.bold);
    }
}
```

## Templates personnalisables

```rust
pub struct PdfTemplate {
    pub header_template: String,
    pub footer_template: String,
    pub item_row_template: String,
}

impl PdfTemplate {
    pub fn default() -> Self {
        Self {
            header_template: include_str!("templates/header.html").to_string(),
            footer_template: include_str!("templates/footer.html").to_string(),
            item_row_template: include_str!("templates/item_row.html").to_string(),
        }
    }
    
    pub fn render(&self, invoice: &Invoice, customer: &Customer) -> String {
        // Remplacer les placeholders dans les templates
        // {invoice_number}, {customer_name}, etc.
        // ...
    }
}
```

## Mise en page professionnelle

```rust
impl InvoicePdfGenerator {
    fn draw_header(&self, layer: &PdfLayerReference) {
        // Logo (si disponible)
        // Nom de l'entreprise
        // Coordonnées
        // SIRET, TVA intracommunautaire
    }
    
    fn draw_invoice_info(&self, layer: &PdfLayerReference, invoice: &Invoice) {
        layer.use_text(
            &format!("FACTURE N° {}", invoice.number),
            20.0,
            Mm(20.0), Mm(250.0),
            &self.fonts.bold,
        );
        
        layer.use_text(
            &format!("Date d'émission: {}", invoice.issue_date.format("%d/%m/%Y")),
            10.0,
            Mm(20.0), Mm(240.0),
            &self.fonts.regular,
        );
        
        layer.use_text(
            &format!("Date d'échéance: {}", invoice.due_date.format("%d/%m/%Y")),
            10.0,
            Mm(20.0), Mm(230.0),
            &self.fonts.regular,
        );
    }
    
    fn draw_customer_info(&self, layer: &PdfLayerReference, customer: &Customer) {
        layer.use_text("Facturé à:", 10.0, Mm(120.0), Mm(250.0), &self.fonts.bold);
        layer.use_text(&customer.name, 10.0, Mm(120.0), Mm(240.0), &self.fonts.regular);
        // Adresse complète...
    }
}
```

## Résumé

- **Génération complète** : Toutes les sections de la facture
- **Templates** : Personnalisation possible
- **Mise en page** : Format professionnel A4
- **Calculs** : Totaux et TVA automatiques
