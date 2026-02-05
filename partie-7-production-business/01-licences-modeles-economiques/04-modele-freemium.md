# 29.4 Modèle Freemium

## Limites du free tier

```rust
pub struct FreemiumManager {
    current_license: Option<License>,
    free_tier_limits: FreeTierLimits,
}

pub struct FreeTierLimits {
    pub max_documents: usize,
    pub max_customers: usize,
    pub max_invoices_per_month: usize,
    pub features_enabled: Vec<Feature>,
    pub watermark_enabled: bool,
}

#[derive(Clone, PartialEq, Eq, Hash)]
pub enum Feature {
    PdfExport,
    CsvExport,
    CustomBranding,
    AdvancedReports,
    ApiAccess,
    PrioritySupport,
}

impl FreemiumManager {
    pub fn can_create_document(&self, current_count: usize) -> bool {
        if self.is_premium() {
            true
        } else {
            current_count < self.free_tier_limits.max_documents
        }
    }
    
    pub fn can_create_invoice(&self, current_month_count: usize) -> bool {
        if self.is_premium() {
            true
        } else {
            current_month_count < self.free_tier_limits.max_invoices_per_month
        }
    }
    
    pub fn is_feature_available(&self, feature: Feature) -> bool {
        if self.is_premium() {
            true
        } else {
            self.free_tier_limits.features_enabled.contains(&feature)
        }
    }
    
    pub fn is_premium(&self) -> bool {
        self.current_license
            .as_ref()
            .map(|l| l.is_active())
            .unwrap_or(false)
    }
}
```

## Prompts d'upgrade

```rust
impl FreemiumManager {
    pub fn show_upgrade_prompt(&self, ui: &mut Ui, tokens: &DesignTokens, reason: UpgradeReason) {
        if self.is_premium() {
            return;
        }
        
        egui::Frame::none()
            .fill(tokens.colors.bg_secondary)
            .rounding(tokens.radius.md)
            .inner_margin(tokens.spacing.md)
            .show(ui, |ui| {
                ui.horizontal(|ui| {
                    ui.colored_label(tokens.colors.warning, "⭐");
                    
                    ui.vertical(|ui| {
                        ui.strong("Passez à la version Pro");
                        
                        let message = match reason {
                            UpgradeReason::LimitReached { limit_type } => {
                                format!("Vous avez atteint la limite de {}", limit_type)
                            }
                            UpgradeReason::FeatureLocked { feature } => {
                                format!("Cette fonctionnalité nécessite la version Pro")
                            }
                            UpgradeReason::Watermark => {
                                "Supprimez le watermark avec la version Pro".to_string()
                            }
                        };
                        
                        ui.label(message);
                    });
                    
                    ui.with_layout(egui::Layout::right_to_left(egui::Align::Center), |ui| {
                        if ui.button("Mettre à niveau").clicked() {
                            open::that("https://example.com/upgrade").ok();
                        }
                    });
                });
            });
    }
}

#[derive(Clone)]
pub enum UpgradeReason {
    LimitReached { limit_type: String },
    FeatureLocked { feature: Feature },
    Watermark,
}
```

## Conversion

```rust
impl FreemiumManager {
    /// Tracker les événements de conversion
    pub fn track_conversion_event(&self, event: ConversionEvent) {
        // Envoyer à l'analytics
        match event {
            ConversionEvent::UpgradePromptShown { reason } => {
                // Log
            }
            ConversionEvent::UpgradeButtonClicked { reason } => {
                // Log
            }
            ConversionEvent::UpgradeCompleted => {
                // Log
            }
        }
    }
    
    /// Afficher un prompt contextuel selon l'usage
    pub fn show_contextual_upgrade(&self, ui: &mut Ui, tokens: &DesignTokens) {
        // Ne pas spammer l'utilisateur
        if self.upgrade_prompts_shown_today() > 3 {
            return;
        }
        
        // Analyser l'usage pour proposer l'upgrade au bon moment
        if self.is_heavy_user() {
            self.show_upgrade_prompt(ui, tokens, UpgradeReason::LimitReached {
                limit_type: "documents".to_string(),
            });
        }
    }
    
    fn is_heavy_user(&self) -> bool {
        // Utilisateur qui approche des limites
        // ...
        false
    }
    
    fn upgrade_prompts_shown_today(&self) -> u32 {
        // Compter les prompts affichés aujourd'hui
        // ...
        0
    }
}

#[derive(Clone)]
pub enum ConversionEvent {
    UpgradePromptShown { reason: UpgradeReason },
    UpgradeButtonClicked { reason: UpgradeReason },
    UpgradeCompleted,
}
```

## Résumé

- **Limites** : Restrictions sur documents, fonctionnalités, exports
- **Prompts** : Messages contextuels pour encourager l'upgrade
- **Conversion** : Tracking des événements de conversion
- **Équilibre** : Limites suffisantes pour valeur, mais incitant à l'upgrade
