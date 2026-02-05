# Chapitre 32 : Monétisation Avancée

## Introduction

Au-delà de la licence de base, plusieurs stratégies permettent d'augmenter les revenus tout en apportant de la valeur aux utilisateurs.

---

## 32.1 Modules Complémentaires

```rust
/// Système de modules optionnels payants
pub struct AddonManager {
    available_addons: Vec<Addon>,
    installed_addons: Vec<InstalledAddon>,
}

pub struct Addon {
    pub id: String,
    pub name: String,
    pub description: String,
    pub price: Decimal,
    pub features: Vec<Feature>,
}

pub struct InstalledAddon {
    pub addon_id: String,
    pub license_key: String,
    pub installed_at: DateTime<Utc>,
}

impl AddonManager {
    pub fn is_feature_unlocked(&self, feature: Feature) -> bool {
        // Vérifier si un addon installé débloque cette feature
        self.installed_addons.iter().any(|installed| {
            self.available_addons.iter()
                .find(|a| a.id == installed.addon_id)
                .map(|addon| addon.features.contains(&feature))
                .unwrap_or(false)
        })
    }
    
    pub fn show_addon_store(&self, ui: &mut Ui, tokens: &DesignTokens) {
        ui.heading("Modules complémentaires");
        ui.add_space(tokens.spacing.md);
        
        for addon in &self.available_addons {
            let is_installed = self.installed_addons.iter()
                .any(|i| i.addon_id == addon.id);
            
            egui::Frame::none()
                .fill(tokens.colors.bg_secondary)
                .rounding(tokens.radius.md)
                .inner_margin(tokens.spacing.md)
                .show(ui, |ui| {
                    ui.horizontal(|ui| {
                        ui.vertical(|ui| {
                            ui.strong(&addon.name);
                            ui.label(&addon.description);
                        });
                        
                        ui.with_layout(egui::Layout::right_to_left(egui::Align::Center), |ui| {
                            if is_installed {
                                ui.colored_label(tokens.colors.success, "✓ Installé");
                            } else {
                                ui.label(format!("{}€", addon.price));
                                if ui.button("Acheter").clicked() {
                                    open::that(format!(
                                        "https://example.com/buy/{}",
                                        addon.id
                                    )).ok();
                                }
                            }
                        });
                    });
                });
            
            ui.add_space(tokens.spacing.sm);
        }
    }
}
```

---

## 32.2 Personnalisation et Branding

```rust
/// White-label pour revendeurs
pub struct WhiteLabelConfig {
    pub company_name: String,
    pub app_name: String,
    pub logo: Option<Vec<u8>>,
    pub primary_color: Color32,
    pub secondary_color: Color32,
    pub contact_email: String,
    pub support_url: String,
}

impl WhiteLabelConfig {
    pub fn load() -> Option<Self> {
        let config_path = get_config_dir().join("branding.toml");
        if config_path.exists() {
            let content = std::fs::read_to_string(&config_path).ok()?;
            toml::from_str(&content).ok()
        } else {
            None
        }
    }
    
    pub fn apply_to_tokens(&self, tokens: &mut DesignTokens) {
        tokens.colors.primary = self.primary_color;
        tokens.colors.primary_hover = lighten(self.primary_color, 0.1);
    }
}

/// Offre de personnalisation
pub struct CustomizationOffer {
    pub base_price: Decimal,
    pub options: Vec<CustomizationOption>,
}

pub struct CustomizationOption {
    pub name: String,
    pub description: String,
    pub price: Decimal,
}
```

---

## 32.3 Services Associés

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

pub enum TrainingFormat {
    Online,
    OnSite,
    Hybrid,
}
```

---

## 32.4 Programme d'Affiliation

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
}
```

---

## Résumé

| Source de revenus | Marge | Effort |
|-------------------|-------|--------|
| Modules payants | Élevée | Moyen |
| White-label | Très élevée | Élevé |
| Support premium | Élevée | Récurrent |
| Formation | Variable | Ponctuel |
| Affiliation | Faible | Minimal |

- **Modules** : Fonctionnalités optionnelles payantes
- **Personnalisation** : White-label pour revendeurs
- **Services** : Support, formation, consulting
- **Affiliation** : Programme de parrainage
