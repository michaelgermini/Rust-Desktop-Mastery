# 18.1 Rapports

## Génération de rapports

```rust
use printpdf::*;
use std::fs::File;
use std::io::BufWriter;

pub struct ReportGenerator {
    fonts: PdfFonts,
}

impl ReportGenerator {
    pub fn generate_report(&self, data: &ReportData) -> Vec<u8> {
        let (doc, page1, layer1) = PdfDocument::new(
            "Rapport",
            Mm(210.0), Mm(297.0), // A4
            "Layer 1",
        );
        
        let current_layer = doc.get_page(page1).get_layer(layer1);
        
        // En-tête
        self.render_header(&current_layer, &data.title);
        
        // Contenu
        self.render_content(&current_layer, &data.sections);
        
        // Pied de page
        self.render_footer(&current_layer);
        
        doc.save_to_bytes().unwrap()
    }
    
    fn render_header(&self, layer: &PdfLayerReference, title: &str) {
        layer.use_text(
            title,
            24.0,
            Mm(20.0), Mm(270.0),
            &self.fonts.bold,
        );
    }
}
```

## Templates

```rust
pub struct ReportTemplate {
    pub header: TemplateSection,
    pub body: TemplateSection,
    pub footer: TemplateSection,
}

pub struct TemplateSection {
    pub elements: Vec<TemplateElement>,
}

pub enum TemplateElement {
    Text { content: String, font_size: f64, x: f64, y: f64 },
    Table { columns: Vec<String>, rows: Vec<Vec<String>> },
    Image { path: PathBuf, x: f64, y: f64 },
}
```

## Formatage

```rust
impl ReportGenerator {
    fn format_currency(&self, amount: Decimal) -> String {
        format!("{:.2} €", amount)
    }
    
    fn format_date(&self, date: DateTime<Utc>) -> String {
        date.format("%d/%m/%Y").to_string()
    }
    
    fn format_percentage(&self, value: f64) -> String {
        format!("{:.1}%", value * 100.0)
    }
}
```

## Résumé

- **PDF** : Génération avec printpdf
- **Templates** : Structure réutilisable
- **Formatage** : Fonctions utilitaires pour les formats
