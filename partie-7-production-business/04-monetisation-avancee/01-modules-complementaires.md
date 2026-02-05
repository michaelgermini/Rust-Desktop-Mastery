# 32.1 Modules Complémentaires

## Système de plugins payants

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
    pub icon: Option<Vec<u8>>,
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
}
```

## Marketplace d'addons

```rust
impl AddonManager {
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
                            
                            // Liste des fonctionnalités
                            ui.add_space(tokens.spacing.xs);
                            for feature in &addon.features {
                                ui.label(format!("  ✓ {}", feature));
                            }
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

## Gestion des licences

```rust
impl AddonManager {
    pub fn install_addon(&mut self, addon_id: &str, license_key: &str) -> Result<()> {
        // Valider la licence
        let validator = LicenseValidator::new();
        let license_data = validator.validate(license_key)?;
        
        // Vérifier que la licence correspond à l'addon
        if license_data.addon_id != addon_id {
            return Err(anyhow!("Licence invalide pour cet addon"));
        }
        
        // Installer
        let installed = InstalledAddon {
            addon_id: addon_id.to_string(),
            license_key: license_key.to_string(),
            installed_at: Utc::now(),
        };
        
        self.installed_addons.push(installed);
        self.save_installed_addons()?;
        
        Ok(())
    }
    
    fn save_installed_addons(&self) -> Result<()> {
        let path = Self::addons_config_path();
        let content = toml::to_string_pretty(&self.installed_addons)?;
        std::fs::write(path, content)?;
        Ok(())
    }
}
```

## Résumé

- **Plugins payants** : Modules complémentaires avec licence
- **Marketplace** : Catalogue d'addons disponibles
- **Licences** : Validation et gestion des licences d'addons
- **Features** : Déblocage de fonctionnalités via addons
