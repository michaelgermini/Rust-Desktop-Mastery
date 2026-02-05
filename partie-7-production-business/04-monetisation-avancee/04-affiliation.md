# 32.4 Programme d'Affiliation

## Système de parrainage

```rust
pub struct AffiliateProgram {
    pub affiliate_id: String,
    pub commission_rate: Decimal,  // Ex: 0.20 pour 20%
    pub referral_link: String,
    pub stats: AffiliateStats,
}

pub struct AffiliateStats {
    pub clicks: u32,
    pub conversions: u32,
    pub revenue: Decimal,
    pub commission_earned: Decimal,
}

impl AffiliateProgram {
    pub fn generate_link(&self, page: &str) -> String {
        format!(
            "https://example.com/{}?ref={}",
            page,
            self.affiliate_id
        )
    }
    
    pub fn track_click(&mut self) {
        self.stats.clicks += 1;
    }
    
    pub fn track_conversion(&mut self, revenue: Decimal) {
        self.stats.conversions += 1;
        self.stats.revenue += revenue;
        self.stats.commission_earned += revenue * self.commission_rate;
    }
}
```

## Commissions

```rust
impl AffiliateProgram {
    pub fn calculate_commission(&self, sale_amount: Decimal) -> Decimal {
        sale_amount * self.commission_rate
    }
    
    pub fn show_dashboard(&self, ui: &mut Ui, tokens: &DesignTokens) {
        ui.heading("Tableau de bord Affiliation");
        
        ui.horizontal(|ui| {
            ui.vertical(|ui| {
                ui.label("Clics");
                ui.strong(format!("{}", self.stats.clicks));
            });
            
            ui.vertical(|ui| {
                ui.label("Conversions");
                ui.strong(format!("{}", self.stats.conversions));
            });
            
            ui.vertical(|ui| {
                ui.label("Taux de conversion");
                let rate = if self.stats.clicks > 0 {
                    (self.stats.conversions as f32 / self.stats.clicks as f32) * 100.0
                } else {
                    0.0
                };
                ui.strong(format!("{:.1}%", rate));
            });
        });
        
        ui.separator();
        
        ui.label(format!("Revenus générés : {}€", self.stats.revenue));
        ui.label(format!("Commissions gagnées : {}€", self.stats.commission_earned));
        
        ui.separator();
        
        ui.label("Votre lien de parrainage :");
        ui.text_edit_singleline(&mut self.referral_link.clone());
        if ui.button("Copier").clicked() {
            // Copier dans le presse-papier
        }
    }
}
```

## Tracking

```rust
pub struct AffiliateTracker {
    pub referrals: HashMap<String, ReferralInfo>,
}

pub struct ReferralInfo {
    pub affiliate_id: String,
    pub clicked_at: DateTime<Utc>,
    pub converted_at: Option<DateTime<Utc>>,
    pub conversion_value: Option<Decimal>,
}

impl AffiliateTracker {
    pub fn track_referral(&mut self, affiliate_id: &str) {
        let referral = ReferralInfo {
            affiliate_id: affiliate_id.to_string(),
            clicked_at: Utc::now(),
            converted_at: None,
            conversion_value: None,
        };
        
        // Générer un cookie ou identifiant de session
        let session_id = Uuid::new_v4().to_string();
        self.referrals.insert(session_id, referral);
    }
    
    pub fn track_conversion(&mut self, session_id: &str, value: Decimal) {
        if let Some(referral) = self.referrals.get_mut(session_id) {
            referral.converted_at = Some(Utc::now());
            referral.conversion_value = Some(value);
        }
    }
}
```

## Résumé

- **Parrainage** : Système de références avec liens uniques
- **Commissions** : Pourcentage sur les ventes générées
- **Tracking** : Suivi des clics et conversions
- **Dashboard** : Tableau de bord pour les affiliés
