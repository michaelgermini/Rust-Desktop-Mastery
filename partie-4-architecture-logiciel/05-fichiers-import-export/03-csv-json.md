# 18.3 CSV et JSON

## Export CSV

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

## Import CSV

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

## Import/Export JSON

```rust
use serde_json;

pub fn export_to_json<T: Serialize>(data: &T, path: &Path) -> Result<()> {
    let json = serde_json::to_string_pretty(data)?;
    std::fs::write(path, json)?;
    Ok(())
}

pub fn import_from_json<T: DeserializeOwned>(path: &Path) -> Result<T> {
    let content = std::fs::read_to_string(path)?;
    let data: T = serde_json::from_str(&content)?;
    Ok(data)
}
```

## Résumé

- **CSV** : Pour Excel et autres outils
- **JSON** : Pour échanges de données structurées
- **Validation** : Vérifier les données importées
- **Erreurs** : Rapporter les lignes problématiques
