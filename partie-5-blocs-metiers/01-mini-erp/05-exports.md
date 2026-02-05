# 20.5 Exports

## Export CSV

```rust
use csv::Writer;

pub fn export_invoices_csv(invoices: &[Invoice], path: &Path) -> Result<()> {
    let file = File::create(path)?;
    let mut writer = Writer::from_writer(file);
    
    // En-têtes
    writer.write_record(&[
        "Numéro",
        "Client",
        "Date émission",
        "Date échéance",
        "Montant HT",
        "TVA",
        "Total TTC",
        "Statut",
    ])?;
    
    for inv in invoices {
        let breakdown = VatCalculator::new(inv.vat_rate).calculate(inv);
        
        writer.write_record(&[
            &inv.number,
            &inv.customer_name,
            &inv.issue_date.format("%d/%m/%Y").to_string(),
            &inv.due_date.format("%d/%m/%Y").to_string(),
            &format!("{:.2}", breakdown.subtotal),
            &format!("{:.2}", breakdown.total_vat),
            &format!("{:.2}", breakdown.total),
            &format!("{:?}", inv.status),
        ])?;
    }
    
    writer.flush()?;
    Ok(())
}
```

## Export Excel

```rust
use calamine::{Writer, Data, DataType};

pub fn export_invoices_excel(invoices: &[Invoice], path: &Path) -> Result<()> {
    let mut workbook = Writer::new(path)?;
    
    // Créer une feuille
    workbook.push_worksheet("Factures")?;
    
    // En-têtes
    workbook.write(0, 0, Data::String("Numéro".to_string()))?;
    workbook.write(0, 1, Data::String("Client".to_string()))?;
    workbook.write(0, 2, Data::String("Date".to_string()))?;
    workbook.write(0, 3, Data::String("Montant".to_string()))?;
    
    // Données
    for (row, inv) in invoices.iter().enumerate() {
        let breakdown = VatCalculator::new(inv.vat_rate).calculate(inv);
        
        workbook.write((row + 1) as u32, 0, Data::String(inv.number.clone()))?;
        workbook.write((row + 1) as u32, 1, Data::String(inv.customer_name.clone()))?;
        workbook.write((row + 1) as u32, 2, Data::DateTime(inv.issue_date))?;
        workbook.write((row + 1) as u32, 3, Data::Float(breakdown.total.to_f64().unwrap_or(0.0)))?;
    }
    
    workbook.close()?;
    Ok(())
}
```

## Export comptable

```rust
pub struct AccountingExport {
    format: AccountingFormat,
}

pub enum AccountingFormat {
    FEC,  // Fichier des Écritures Comptables (France)
    CSV,
    OFX,
}

impl AccountingExport {
    pub fn export_fec(&self, invoices: &[Invoice], path: &Path) -> Result<()> {
        let file = File::create(path)?;
        let mut writer = Writer::from_writer(file);
        
        // En-têtes FEC (format français)
        writer.write_record(&[
            "JournalCode",
            "JournalLib",
            "EcritureNum",
            "EcritureDate",
            "CompteNum",
            "CompteLib",
            "CompAuxNum",
            "CompAuxLib",
            "PieceRef",
            "PieceDate",
            "EcritureLib",
            "Debit",
            "Credit",
            "EcritureLet",
            "DateLet",
            "ValidDate",
            "Montantdevise",
            "Idevise",
        ])?;
        
        for inv in invoices {
            if inv.status == InvoiceStatus::Paid {
                // Écriture de vente
                writer.write_record(&[
                    "VT",  // Ventes
                    "Ventes",
                    &inv.number,
                    &inv.issue_date.format("%Y%m%d").to_string(),
                    "701000",  // Compte de vente
                    "Ventes de produits",
                    "",
                    "",
                    &inv.number,
                    &inv.issue_date.format("%Y%m%d").to_string(),
                    &format!("Facture {}", inv.number),
                    "",
                    &format!("{:.2}", inv.total()),
                    "",
                    "",
                    "",
                    "",
                    "",
                ])?;
                
                // Écriture TVA
                // ...
                
                // Écriture client
                // ...
            }
        }
        
        writer.flush()?;
        Ok(())
    }
}
```

## Résumé

- **CSV** : Format universel pour Excel et autres outils
- **Excel** : Format natif .xlsx avec calamine
- **Comptable** : Export FEC pour la comptabilité française
- **Flexibilité** : Plusieurs formats selon les besoins
