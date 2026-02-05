# 19.2 Feature Flags

## Système de feature flags

```rust
use std::collections::HashSet;

pub struct FeatureFlags {
    enabled: HashSet<String>,
}

impl FeatureFlags {
    pub fn new() -> Self {
        Self { enabled: HashSet::new() }
    }
    
    pub fn enable(&mut self, feature: &str) {
        self.enabled.insert(feature.to_string());
    }
    
    pub fn disable(&mut self, feature: &str) {
        self.enabled.remove(feature);
    }
    
    pub fn is_enabled(&self, feature: &str) -> bool {
        self.enabled.contains(feature)
    }
    
    pub fn load_from_config(&mut self, path: &Path) -> Result<()> {
        if path.exists() {
            let content = std::fs::read_to_string(path)?;
            let config: FeatureConfig = toml::from_str(&content)?;
            self.enabled = config.enabled.into_iter().collect();
        }
        Ok(())
    }
}
```

## Activation/désactivation

```rust
// Dans le code
fn render_ui(ui: &mut Ui, features: &FeatureFlags) {
    ui.heading("Dashboard");
    
    if features.is_enabled("advanced_charts") {
        render_advanced_charts(ui);
    }
    
    if features.is_enabled("ai_assistant") {
        render_ai_panel(ui);
    }
    
    if features.is_enabled("export_pdf") {
        if ui.button("Export PDF").clicked() {
            export_pdf();
        }
    }
}
```

## A/B testing

```rust
impl FeatureFlags {
    pub fn is_in_group(&self, feature: &str, user_id: &str) -> bool {
        // Hash du user_id pour déterminer le groupe
        let hash = self.hash_user_id(user_id);
        let group = hash % 100;
        
        // 50% des utilisateurs dans le groupe A
        if group < 50 {
            self.is_enabled(&format!("{}_group_a", feature))
        } else {
            self.is_enabled(&format!("{}_group_b", feature))
        }
    }
    
    fn hash_user_id(&self, user_id: &str) -> u64 {
        use std::collections::hash_map::DefaultHasher;
        use std::hash::{Hash, Hasher};
        
        let mut hasher = DefaultHasher::new();
        user_id.hash(&mut hasher);
        hasher.finish()
    }
}
```

## Configuration

```toml
# features.toml
[features]
enabled = [
    "advanced_charts",
    "export_pdf",
]

disabled = [
    "ai_assistant",
    "beta_features",
]
```

## Résumé

- **Feature flags** : Activer/désactiver des fonctionnalités
- **Configuration** : Fichier de configuration pour les flags
- **A/B testing** : Répartition des utilisateurs en groupes
- **Déploiement** : Déployer sans activer immédiatement
