# 32.3 Services Associés

## Support premium

```rust
/// Services de support premium
pub enum SupportTier {
    Community,  // Gratuit - Forum/Discord
    Standard,   // Inclus avec licence - Email 48h
    Premium,    // Payant - Email 4h + Chat
    Enterprise, // Sur mesure - Téléphone + SLA
}

pub struct SupportService {
    pub tier: SupportTier,
    pub tickets: Vec<SupportTicket>,
}

impl SupportService {
    pub fn max_response_time(&self) -> Duration {
        match self.tier {
            SupportTier::Community => Duration::from_secs(0),  // Pas de garantie
            SupportTier::Standard => Duration::from_secs(48 * 3600),
            SupportTier::Premium => Duration::from_secs(4 * 3600),
            SupportTier::Enterprise => Duration::from_secs(1 * 3600),
        }
    }
    
    pub fn show_support_options(&self, ui: &mut Ui) {
        match self.tier {
            SupportTier::Community | SupportTier::Standard => {
                ui.label("💡 Passez au support Premium pour des réponses en 4h");
                if ui.button("En savoir plus").clicked() {
                    open::that("https://example.com/support-premium").ok();
                }
            }
            _ => {}
        }
    }
}
```

## Formation et consulting

```rust
/// Formation et consulting
pub struct TrainingService {
    pub sessions: Vec<TrainingSession>,
}

pub struct TrainingSession {
    pub title: String,
    pub description: String,
    pub duration_hours: u32,
    pub price: Decimal,
    pub format: TrainingFormat,
}

#[derive(Clone)]
pub enum TrainingFormat {
    Online,
    OnSite,
    Hybrid,
}

impl TrainingService {
    pub fn show_training_options(&self, ui: &mut Ui, tokens: &DesignTokens) {
        ui.heading("Formation et Consulting");
        
        for session in &self.sessions {
            egui::Frame::none()
                .fill(tokens.colors.bg_secondary)
                .rounding(tokens.radius.md)
                .inner_margin(tokens.spacing.md)
                .show(ui, |ui| {
                    ui.strong(&session.title);
                    ui.label(&session.description);
                    ui.label(format!("Durée : {} heures", session.duration_hours));
                    ui.label(format!("Format : {:?}", session.format));
                    ui.label(format!("Prix : {}€", session.price));
                    
                    if ui.button("Réserver").clicked() {
                        open::that(format!("https://example.com/training/{}", session.title)).ok();
                    }
                });
        }
    }
}
```

## Services professionnels

```rust
pub struct ProfessionalServices {
    pub offerings: Vec<ServiceOffering>,
}

pub struct ServiceOffering {
    pub name: String,
    pub description: String,
    pub price_range: PriceRange,
    pub duration: String,
}

pub struct PriceRange {
    pub min: Decimal,
    pub max: Decimal,
}

impl ProfessionalServices {
    pub fn show_offerings(&self, ui: &mut Ui) {
        ui.heading("Services Professionnels");
        
        ui.label("• Intégration sur mesure");
        ui.label("• Migration de données");
        ui.label("• Développement de fonctionnalités spécifiques");
        ui.label("• Audit et optimisation");
        
        if ui.button("Demander un devis").clicked() {
            open::that("https://example.com/contact").ok();
        }
    }
}
```

## Résumé

- **Support premium** : Niveaux de support avec garanties de réponse
- **Formation** : Sessions de formation payantes
- **Consulting** : Services professionnels sur mesure
- **Revenus récurrents** : Services complémentaires à la licence
