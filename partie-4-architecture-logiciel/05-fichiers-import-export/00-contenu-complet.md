# Chapitre 18 : Fichiers, Import, Export et PDF

## Introduction

Une application desktop doit interagir avec le système de fichiers, importer des données et générer des exports professionnels.

---

## 18.1 Génération PDF

```rust
use printpdf::*;
use std::fs::File;
use std::io::BufWriter;

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

---

## 18.2 Export CSV

```rust
use csv::Writer;

pub fn export_to_csv<T: Serialize>(data: &[T], path: &Path) -> Result<()> {
    let file = File::create(path)?;
    let mut writer = Writer::from_writer(file);
    
    for record in data {
        writer.serialize(record)?;
    }
    
    writer.flush()?;
    Ok(())
}

// Export avec colonnes custom
pub fn export_invoices_csv(invoices: &[Invoice], path: &Path) -> Result<()> {
    let file = File::create(path)?;
    let mut writer = Writer::from_writer(file);
    
    // En-têtes
    writer.write_record(&["Numéro", "Client", "Date", "Montant HT", "TVA", "Total TTC", "Statut"])?;
    
    for inv in invoices {
        let totals = calculate_totals(inv);
        writer.write_record(&[
            &inv.number,
            &inv.customer_name,
            &inv.issue_date.format("%d/%m/%Y").to_string(),
            &format!("{:.2}", totals.subtotal),
            &format!("{:.2}", totals.vat),
            &format!("{:.2}", totals.total),
            &format!("{:?}", inv.status),
        ])?;
    }
    
    writer.flush()?;
    Ok(())
}
```

---

## 18.3 Import de Données

```rust
use csv::Reader;

pub fn import_from_csv<T: DeserializeOwned>(path: &Path) -> Result<Vec<T>> {
    let file = File::open(path)?;
    let mut reader = Reader::from_reader(file);
    
    let mut records = Vec::new();
    for result in reader.deserialize() {
        let record: T = result?;
        records.push(record);
    }
    
    Ok(records)
}

// Import avec validation
pub fn import_customers_csv(path: &Path) -> Result<ImportResult<Customer>> {
    let mut successes = Vec::new();
    let mut errors = Vec::new();
    
    let file = File::open(path)?;
    let mut reader = Reader::from_reader(file);
    
    for (line, result) in reader.records().enumerate() {
        match result {
            Ok(record) => {
                match parse_customer(&record) {
                    Ok(customer) => successes.push(customer),
                    Err(e) => errors.push(ImportError { line: line + 2, message: e }),
                }
            }
            Err(e) => errors.push(ImportError { line: line + 2, message: e.to_string() }),
        }
    }
    
    Ok(ImportResult { successes, errors })
}
```

---

## 18.4 Intégration Système

```rust
use native_dialog::FileDialog;

/// Ouvrir un dialogue de sélection de fichier
pub fn pick_file(title: &str, filters: &[(&str, &[&str])]) -> Option<PathBuf> {
    let mut dialog = FileDialog::new().set_title(title);
    
    for (name, extensions) in filters {
        dialog = dialog.add_filter(name, extensions);
    }
    
    dialog.show_open_single_file().ok().flatten()
}

/// Sauvegarder avec dialogue
pub fn save_file_dialog(title: &str, default_name: &str, extension: &str) -> Option<PathBuf> {
    FileDialog::new()
        .set_title(title)
        .set_filename(default_name)
        .add_filter("Fichier", &[extension])
        .show_save_single_file()
        .ok()
        .flatten()
}

/// Ouvrir un fichier avec l'application par défaut
pub fn open_with_default_app(path: &Path) -> Result<()> {
    #[cfg(target_os = "windows")]
    std::process::Command::new("explorer").arg(path).spawn()?;
    
    #[cfg(target_os = "macos")]
    std::process::Command::new("open").arg(path).spawn()?;
    
    #[cfg(target_os = "linux")]
    std::process::Command::new("xdg-open").arg(path).spawn()?;
    
    Ok(())
}
```

---

## Résumé

- **PDF** : Génération de documents professionnels
- **CSV** : Import/export de données tabulaires
- **Dialogues natifs** : Intégration avec l'OS
- **Validation** : Import robuste avec gestion d'erreurs
