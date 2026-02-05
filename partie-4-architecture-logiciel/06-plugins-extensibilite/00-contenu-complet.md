# Chapitre 19 : Plugins et Extensibilité

## Introduction

Une architecture extensible permet d'ajouter des fonctionnalités sans modifier le code existant. Ce chapitre présente les patterns de plugins en Rust.

---

## 19.1 Architecture Plugin

```rust
/// Trait définissant un plugin
pub trait Plugin: Send + Sync {
    fn name(&self) -> &str;
    fn version(&self) -> &str;
    fn initialize(&mut self, ctx: &mut PluginContext) -> Result<()>;
    fn shutdown(&mut self) -> Result<()>;
}

/// Contexte fourni aux plugins
pub struct PluginContext {
    pub event_bus: EventBus,
    pub storage: Box<dyn PluginStorage>,
    pub ui_registry: UiRegistry,
}

/// Gestionnaire de plugins
pub struct PluginManager {
    plugins: Vec<Box<dyn Plugin>>,
    context: PluginContext,
}

impl PluginManager {
    pub fn load_plugin(&mut self, plugin: Box<dyn Plugin>) -> Result<()> {
        let mut plugin = plugin;
        plugin.initialize(&mut self.context)?;
        self.plugins.push(plugin);
        Ok(())
    }
    
    pub fn unload_all(&mut self) -> Result<()> {
        for plugin in &mut self.plugins {
            plugin.shutdown()?;
        }
        self.plugins.clear();
        Ok(())
    }
}
```

---

## 19.2 Feature Flags

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
}

// Usage
fn render_ui(ui: &mut Ui, features: &FeatureFlags) {
    ui.heading("Dashboard");
    
    if features.is_enabled("advanced_charts") {
        render_advanced_charts(ui);
    }
    
    if features.is_enabled("ai_assistant") {
        render_ai_panel(ui);
    }
}
```

---

## 19.3 Extension Points

```rust
/// Points d'extension pour les plugins
pub trait UiExtension: Send + Sync {
    fn render_toolbar(&self, ui: &mut Ui);
    fn render_sidebar(&self, ui: &mut Ui);
    fn render_context_menu(&self, ui: &mut Ui, context: &MenuContext);
}

/// Registry des extensions UI
pub struct UiRegistry {
    toolbar_extensions: Vec<Box<dyn UiExtension>>,
    menu_extensions: Vec<Box<dyn Fn(&mut Ui, &MenuContext)>>,
}

impl UiRegistry {
    pub fn register_toolbar(&mut self, extension: Box<dyn UiExtension>) {
        self.toolbar_extensions.push(extension);
    }
    
    pub fn render_all_toolbars(&self, ui: &mut Ui) {
        for ext in &self.toolbar_extensions {
            ext.render_toolbar(ui);
        }
    }
}
```

---

## Résumé

- **Plugin Trait** : Interface claire pour les extensions
- **Feature Flags** : Activer/désactiver des fonctionnalités
- **Extension Points** : Points d'accroche pour les plugins
- **Découplage** : Les plugins n'affectent pas le code core
