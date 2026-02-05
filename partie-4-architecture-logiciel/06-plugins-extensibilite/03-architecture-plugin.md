# 19.3 Architecture Plugin

## Trait Plugin

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
    pub event_bus: Arc<EventBus>,
    pub storage: Box<dyn PluginStorage>,
    pub ui_registry: Arc<Mutex<UiRegistry>>,
}

/// Gestionnaire de plugins
pub struct PluginManager {
    plugins: Vec<Box<dyn Plugin>>,
    context: PluginContext,
}

impl PluginManager {
    pub fn new(event_bus: Arc<EventBus>) -> Self {
        Self {
            plugins: Vec::new(),
            context: PluginContext {
                event_bus,
                storage: Box::new(MemoryPluginStorage::new()),
                ui_registry: Arc::new(Mutex::new(UiRegistry::new())),
            },
        }
    }
    
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

## PluginManager

```rust
impl PluginManager {
    pub fn load_from_directory(&mut self, dir: &Path) -> Result<()> {
        for entry in std::fs::read_dir(dir)? {
            let entry = entry?;
            let path = entry.path();
            
            if path.extension() == Some(std::ffi::OsStr::new("plugin")) {
                let plugin = self.load_plugin_file(&path)?;
                self.load_plugin(plugin)?;
            }
        }
        Ok(())
    }
}
```

## Points d'extension (hooks)

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

## Exemple de plugin

```rust
pub struct MyPlugin {
    name: String,
}

impl Plugin for MyPlugin {
    fn name(&self) -> &str {
        &self.name
    }
    
    fn version(&self) -> &str {
        "1.0.0"
    }
    
    fn initialize(&mut self, ctx: &mut PluginContext) -> Result<()> {
        // S'abonner aux événements
        ctx.event_bus.subscribe(|event: &InvoiceCreated| {
            println!("Plugin: Invoice created!");
        });
        
        // Enregistrer une extension UI
        let registry = ctx.ui_registry.lock().unwrap();
        registry.register_toolbar(Box::new(MyToolbarExtension));
        
        Ok(())
    }
    
    fn shutdown(&mut self) -> Result<()> {
        println!("Plugin {} shutting down", self.name);
        Ok(())
    }
}
```

## Résumé

- **Trait Plugin** : Interface claire pour les extensions
- **PluginManager** : Chargement et gestion des plugins
- **Points d'extension** : Hooks pour intégrer dans l'UI
- **Découplage** : Les plugins n'affectent pas le code core
