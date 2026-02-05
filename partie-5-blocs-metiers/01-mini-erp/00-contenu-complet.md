# Chapitre 20 : Construire un Mini-ERP Rust

## Introduction

Ce chapitre guide la construction d'un système de gestion complet : clients, factures, TVA et exports.

---

## 20.1 Modèle de Données

```rust
// Entités principales

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct Customer {
    pub id: CustomerId,
    pub name: String,
    pub email: String,
    pub address: Address,
    pub vat_number: Option<String>,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct Invoice {
    pub id: InvoiceId,
    pub number: String,
    pub customer_id: CustomerId,
    pub items: Vec<InvoiceItem>,
    pub status: InvoiceStatus,
    pub issue_date: NaiveDate,
    pub due_date: NaiveDate,
    pub vat_rate: Decimal,
    pub notes: Option<String>,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct InvoiceItem {
    pub description: String,
    pub quantity: Decimal,
    pub unit_price: Decimal,
    pub vat_rate: Option<Decimal>, // Override par ligne
}

impl InvoiceItem {
    pub fn subtotal(&self) -> Decimal {
        self.quantity * self.unit_price
    }
    
    pub fn vat(&self, default_rate: Decimal) -> Decimal {
        let rate = self.vat_rate.unwrap_or(default_rate);
        self.subtotal() * rate
    }
    
    pub fn total(&self, default_rate: Decimal) -> Decimal {
        self.subtotal() + self.vat(default_rate)
    }
}
```

---

## 20.2 Calculs TVA

```rust
pub struct VatCalculator {
    default_rate: Decimal,
}

impl VatCalculator {
    pub fn new(default_rate: Decimal) -> Self {
        Self { default_rate }
    }
    
    pub fn calculate(&self, invoice: &Invoice) -> VatBreakdown {
        let mut by_rate: HashMap<Decimal, VatLine> = HashMap::new();
        
        for item in &invoice.items {
            let rate = item.vat_rate.unwrap_or(self.default_rate);
            let entry = by_rate.entry(rate).or_insert(VatLine {
                rate,
                base: Decimal::ZERO,
                vat: Decimal::ZERO,
            });
            
            let base = item.subtotal();
            entry.base += base;
            entry.vat += base * rate;
        }
        
        let lines: Vec<_> = by_rate.into_values().collect();
        let subtotal = lines.iter().map(|l| l.base).sum();
        let total_vat = lines.iter().map(|l| l.vat).sum();
        
        VatBreakdown {
            lines,
            subtotal,
            total_vat,
            total: subtotal + total_vat,
        }
    }
}

pub struct VatBreakdown {
    pub lines: Vec<VatLine>,
    pub subtotal: Decimal,
    pub total_vat: Decimal,
    pub total: Decimal,
}

pub struct VatLine {
    pub rate: Decimal,
    pub base: Decimal,
    pub vat: Decimal,
}
```

---

## 20.3 Génération de Facture PDF

```rust
pub struct InvoicePdfGenerator {
    company_info: CompanyInfo,
    fonts: PdfFonts,
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
}
```

---

## 20.4 Interface de Gestion

```rust
pub struct InvoiceListScreen {
    invoices: Vec<Invoice>,
    filter: InvoiceFilter,
    selected: Option<usize>,
}

impl InvoiceListScreen {
    pub fn render(&mut self, ui: &mut Ui, tokens: &DesignTokens) {
        // Toolbar
        ui.horizontal(|ui| {
            if Button::new("+ Nouvelle facture")
                .variant(ButtonVariant::Primary)
                .show(ui, tokens).clicked()
            {
                self.create_new();
            }
            
            ui.add_space(tokens.spacing.md);
            
            // Filtres
            egui::ComboBox::from_id_source("status_filter")
                .selected_text(self.filter.status_label())
                .show_ui(ui, |ui| {
                    ui.selectable_value(&mut self.filter.status, None, "Tous");
                    ui.selectable_value(&mut self.filter.status, Some(InvoiceStatus::Draft), "Brouillons");
                    ui.selectable_value(&mut self.filter.status, Some(InvoiceStatus::Sent), "Envoyées");
                    ui.selectable_value(&mut self.filter.status, Some(InvoiceStatus::Paid), "Payées");
                });
        });
        
        ui.add_space(tokens.spacing.md);
        
        // Table
        Table::new(&self.filtered_invoices())
            .column("N°", |inv, ui, _| { ui.label(&inv.number); })
            .column("Client", |inv, ui, _| { ui.label(&inv.customer_name); })
            .column("Date", |inv, ui, _| { ui.label(inv.issue_date.format("%d/%m/%Y").to_string()); })
            .column("Montant", |inv, ui, tokens| {
                ui.label(egui::RichText::new(format!("{:.2} €", inv.total))
                    .color(tokens.colors.text_primary));
            })
            .column("Statut", |inv, ui, tokens| {
                self.render_status_badge(ui, inv.status, tokens);
            })
            .column("Actions", |inv, ui, tokens| {
                ui.horizontal(|ui| {
                    if ui.small_button("📄").on_hover_text("Voir PDF").clicked() {
                        self.preview_pdf(inv.id);
                    }
                    if ui.small_button("✉️").on_hover_text("Envoyer").clicked() {
                        self.send_invoice(inv.id);
                    }
                });
            })
            .selectable(&mut self.selected)
            .show(ui, tokens);
    }
}
```

---

## Résumé

- **Modèle robuste** : Entités typées avec calculs intégrés
- **TVA flexible** : Taux par facture ou par ligne
- **PDF professionnel** : Génération complète avec mise en page
- **Interface intuitive** : Liste, filtres, actions rapides
