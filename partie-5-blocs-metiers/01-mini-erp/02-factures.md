# 20.2 Factures

## Modèle de facture

```rust
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
    pub created_at: DateTime<Utc>,
    pub updated_at: DateTime<Utc>,
}

#[derive(Debug, Clone, Copy, PartialEq, Eq, Serialize, Deserialize)]
pub enum InvoiceStatus {
    Draft,
    Sent,
    Paid,
    Cancelled,
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

impl Invoice {
    pub fn new(customer_id: CustomerId, vat_rate: Decimal) -> Self {
        let now = Utc::now().date_naive();
        Self {
            id: InvoiceId(0),
            number: String::new(),  // Sera généré
            customer_id,
            items: Vec::new(),
            status: InvoiceStatus::Draft,
            issue_date: now,
            due_date: now + chrono::Duration::days(30),
            vat_rate,
            notes: None,
            created_at: Utc::now(),
            updated_at: Utc::now(),
        }
    }
    
    pub fn subtotal(&self) -> Decimal {
        self.items.iter().map(|i| i.subtotal()).sum()
    }
    
    pub fn total_vat(&self) -> Decimal {
        self.items.iter().map(|i| i.vat(self.vat_rate)).sum()
    }
    
    pub fn total(&self) -> Decimal {
        self.subtotal() + self.total_vat()
    }
}
```

## Gestion des lignes

```rust
impl Invoice {
    pub fn add_item(&mut self, item: InvoiceItem) {
        self.items.push(item);
        self.updated_at = Utc::now();
    }
    
    pub fn remove_item(&mut self, index: usize) -> Result<(), AppError> {
        if index >= self.items.len() {
            return Err(AppError::InvalidItemIndex);
        }
        self.items.remove(index);
        self.updated_at = Utc::now();
        Ok(())
    }
    
    pub fn update_item(&mut self, index: usize, item: InvoiceItem) -> Result<(), AppError> {
        if index >= self.items.len() {
            return Err(AppError::InvalidItemIndex);
        }
        self.items[index] = item;
        self.updated_at = Utc::now();
        Ok(())
    }
}
```

## Numérotation automatique

```rust
pub struct InvoiceNumberGenerator {
    prefix: String,
    year: i32,
    sequence: i64,
}

impl InvoiceNumberGenerator {
    pub fn new(prefix: String) -> Self {
        let year = chrono::Utc::now().year();
        Self {
            prefix,
            year,
            sequence: 0,
        }
    }
    
    pub fn generate(&mut self) -> String {
        self.sequence += 1;
        format!("{}-{}-{:04}", self.prefix, self.year, self.sequence)
    }
    
    pub fn from_existing_number(number: &str) -> Result<Self, AppError> {
        // Parser un numéro existant pour continuer la séquence
        // Format attendu: PREFIX-YYYY-NNNN
        let parts: Vec<&str> = number.split('-').collect();
        if parts.len() != 3 {
            return Err(AppError::InvalidInvoiceNumber);
        }
        
        let prefix = parts[0].to_string();
        let year: i32 = parts[1].parse()?;
        let sequence: i64 = parts[2].parse()?;
        
        Ok(Self {
            prefix,
            year,
            sequence,
        })
    }
}

// Dans le repository
impl InvoiceRepository {
    pub fn next_number(&self) -> Result<String, AppError> {
        // Récupérer le dernier numéro
        let last_number = self.get_last_number()?;
        
        let mut generator = if let Some(num) = last_number {
            InvoiceNumberGenerator::from_existing_number(&num)?
        } else {
            InvoiceNumberGenerator::new("INV".to_string())
        };
        
        Ok(generator.generate())
    }
}
```

## Résumé

- **Modèle complet** : Facture avec items, statut, dates
- **Gestion des lignes** : Ajouter, modifier, supprimer
- **Numérotation** : Génération automatique séquentielle
- **Calculs** : Sous-total, TVA, total automatiques
