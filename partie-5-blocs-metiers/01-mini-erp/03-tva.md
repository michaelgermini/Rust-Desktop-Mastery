# 20.3 TVA

## Calcul de TVA

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

## Gestion des taux

```rust
pub struct VatRateManager {
    rates: HashMap<String, Decimal>,
    default_rate: Decimal,
}

impl VatRateManager {
    pub fn new() -> Self {
        let mut rates = HashMap::new();
        
        // Taux standards français
        rates.insert("standard".to_string(), Decimal::new(20, 2));  // 20%
        rates.insert("reduced".to_string(), Decimal::new(55, 3));   // 5.5%
        rates.insert("super_reduced".to_string(), Decimal::new(25, 3)); // 2.1%
        rates.insert("zero".to_string(), Decimal::ZERO);
        
        Self {
            rates,
            default_rate: Decimal::new(20, 2),
        }
    }
    
    pub fn get_rate(&self, key: &str) -> Decimal {
        self.rates.get(key).copied().unwrap_or(self.default_rate)
    }
    
    pub fn add_rate(&mut self, key: String, rate: Decimal) {
        self.rates.insert(key, rate);
    }
}
```

## Conformité légale

```rust
impl Invoice {
    /// Valider la conformité légale d'une facture
    pub fn validate_legal_compliance(&self) -> Result<(), Vec<ComplianceError>> {
        let mut errors = Vec::new();
        
        // Numéro obligatoire
        if self.number.is_empty() {
            errors.push(ComplianceError::MissingNumber);
        }
        
        // Date d'émission obligatoire
        // (déjà dans le modèle)
        
        // Au moins un item
        if self.items.is_empty() {
            errors.push(ComplianceError::EmptyInvoice);
        }
        
        // Montants positifs
        for (i, item) in self.items.iter().enumerate() {
            if item.quantity <= Decimal::ZERO {
                errors.push(ComplianceError::InvalidQuantity(i));
            }
            if item.unit_price < Decimal::ZERO {
                errors.push(ComplianceError::NegativePrice(i));
            }
        }
        
        // TVA cohérente
        let breakdown = VatCalculator::new(self.vat_rate).calculate(self);
        if breakdown.total_vat < Decimal::ZERO {
            errors.push(ComplianceError::NegativeVat);
        }
        
        if errors.is_empty() {
            Ok(())
        } else {
            Err(errors)
        }
    }
}

#[derive(Debug, Clone)]
pub enum ComplianceError {
    MissingNumber,
    EmptyInvoice,
    InvalidQuantity(usize),
    NegativePrice(usize),
    NegativeVat,
}
```

## Résumé

- **Calcul flexible** : Taux par facture ou par ligne
- **Gestion des taux** : Plusieurs taux disponibles
- **Conformité** : Validation légale des factures
- **Breakdown** : Détail par taux de TVA
