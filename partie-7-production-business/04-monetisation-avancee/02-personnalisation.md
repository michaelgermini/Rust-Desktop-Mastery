# 32.2 Personnalisation et Branding

## White-label

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
        tokens.colors.secondary = self.secondary_color;
    }
    
    pub fn render_logo(&self, ui: &mut Ui) {
        if let Some(logo_data) = &self.logo {
            // Afficher le logo personnalisé
            if let Ok(texture) = ui.ctx().load_texture("white_label_logo", logo_data.clone()) {
                ui.image(texture);
            }
        }
    }
}
```

## Customisation pour revendeurs

```rust
/// Offre de personnalisation
pub struct CustomizationOffer {
    pub base_price: Decimal,
    pub options: Vec<CustomizationOption>,
}

pub struct CustomizationOption {
    pub name: String,
    pub description: String,
    pub price: Decimal,
    pub category: CustomizationCategory,
}

#[derive(Clone)]
pub enum CustomizationCategory {
    Branding,      // Logo, couleurs
    Features,      // Fonctionnalités spécifiques
    Integration,   // Intégrations tierces
    Support,       // Support personnalisé
}

impl CustomizationOffer {
    pub fn calculate_total(&self, selected_options: &[String]) -> Decimal {
        let mut total = self.base_price;
        
        for option_id in selected_options {
            if let Some(option) = self.options.iter().find(|o| o.name == *option_id) {
                total += option.price;
            }
        }
        
        total
    }
}
```

## Branding personnalisé

```rust
impl App {
    pub fn apply_white_label(&mut self) {
        if let Some(config) = WhiteLabelConfig::load() {
            // Appliquer le branding
            config.apply_to_tokens(&mut self.design_tokens);
            
            // Remplacer le nom de l'app
            self.app_name = config.app_name.clone();
            
            // Afficher le logo personnalisé
            self.logo = config.logo.clone();
        }
    }
    
    pub fn render_header(&mut self, ui: &mut Ui) {
        ui.horizontal(|ui| {
            // Logo personnalisé ou par défaut
            if let Some(config) = WhiteLabelConfig::load() {
                config.render_logo(ui);
            } else {
                ui.label("Mon App");
            }
            
            ui.with_layout(egui::Layout::right_to_left(egui::Align::Center), |ui| {
                // Menu
            });
        });
    }
}
```

## Résumé

- **White-label** : Personnalisation complète pour revendeurs
- **Branding** : Logo, couleurs, nom personnalisés
- **Customisation** : Options de personnalisation payantes
- **Revendeurs** : Solution pour partenaires et intégrateurs
