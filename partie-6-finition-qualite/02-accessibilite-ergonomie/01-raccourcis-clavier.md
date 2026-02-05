# 26.1 Raccourcis Clavier

## Système de raccourcis

```rust
use std::collections::HashMap;

pub struct ShortcutManager {
    shortcuts: HashMap<KeyCombo, ShortcutAction>,
}

#[derive(Hash, Eq, PartialEq, Clone)]
pub struct KeyCombo {
    pub key: egui::Key,
    pub modifiers: egui::Modifiers,
}

#[derive(Clone)]
pub enum ShortcutAction {
    NewDocument,
    OpenDocument,
    Save,
    SaveAs,
    Undo,
    Redo,
    Search,
    Settings,
    Cancel,
    Quit,
}

impl ShortcutManager {
    pub fn new() -> Self {
        let mut manager = Self { shortcuts: HashMap::new() };
        
        // Raccourcis standard
        manager.register(KeyCombo::cmd(egui::Key::N), ShortcutAction::NewDocument);
        manager.register(KeyCombo::cmd(egui::Key::O), ShortcutAction::OpenDocument);
        manager.register(KeyCombo::cmd(egui::Key::S), ShortcutAction::Save);
        manager.register(KeyCombo::cmd_shift(egui::Key::S), ShortcutAction::SaveAs);
        manager.register(KeyCombo::cmd(egui::Key::Z), ShortcutAction::Undo);
        manager.register(KeyCombo::cmd_shift(egui::Key::Z), ShortcutAction::Redo);
        manager.register(KeyCombo::cmd(egui::Key::F), ShortcutAction::Search);
        manager.register(KeyCombo::cmd(egui::Key::Comma), ShortcutAction::Settings);
        manager.register(KeyCombo::new(egui::Key::Escape), ShortcutAction::Cancel);
        manager.register(KeyCombo::cmd(egui::Key::Q), ShortcutAction::Quit);
        
        manager
    }
    
    pub fn register(&mut self, combo: KeyCombo, action: ShortcutAction) {
        self.shortcuts.insert(combo, action);
    }
    
    pub fn handle(&self, ctx: &egui::Context) -> Option<ShortcutAction> {
        ctx.input(|i| {
            for (combo, action) in &self.shortcuts {
                if i.modifiers == combo.modifiers && i.key_pressed(combo.key) {
                    return Some(action.clone());
                }
            }
            None
        })
    }
}

impl KeyCombo {
    pub fn new(key: egui::Key) -> Self {
        Self { key, modifiers: egui::Modifiers::NONE }
    }
    
    pub fn cmd(key: egui::Key) -> Self {
        Self { key, modifiers: egui::Modifiers::COMMAND }
    }
    
    pub fn cmd_shift(key: egui::Key) -> Self {
        Self { 
            key, 
            modifiers: egui::Modifiers::COMMAND | egui::Modifiers::SHIFT 
        }
    }
    
    pub fn ctrl_alt(key: egui::Key) -> Self {
        Self {
            key,
            modifiers: egui::Modifiers::CTRL | egui::Modifiers::ALT,
        }
    }
}
```

## Raccourcis globaux

```rust
impl App {
    fn handle_global_shortcuts(&mut self, ctx: &egui::Context) {
        if let Some(action) = self.shortcut_manager.handle(ctx) {
            match action {
                ShortcutAction::NewDocument => self.create_new_document(),
                ShortcutAction::Save => self.save_current_document(),
                ShortcutAction::Search => self.focus_search(),
                ShortcutAction::Settings => self.show_settings = true,
                ShortcutAction::Quit => ctx.send_viewport_cmd(egui::ViewportCommand::Close),
                _ => {}
            }
        }
    }
}
```

## Personnalisation

```rust
pub struct CustomizableShortcuts {
    shortcuts: HashMap<String, KeyCombo>,
    defaults: HashMap<String, KeyCombo>,
}

impl CustomizableShortcuts {
    pub fn load_from_config(&mut self, path: &Path) -> Result<()> {
        if path.exists() {
            let content = std::fs::read_to_string(path)?;
            let config: HashMap<String, String> = toml::from_str(&content)?;
            
            for (action, combo_str) in config {
                if let Ok(combo) = self.parse_key_combo(&combo_str) {
                    self.shortcuts.insert(action, combo);
                }
            }
        }
        Ok(())
    }
    
    pub fn save_to_config(&self, path: &Path) -> Result<()> {
        let config: HashMap<String, String> = self.shortcuts.iter()
            .map(|(action, combo)| (action.clone(), self.format_key_combo(combo)))
            .collect();
        
        let content = toml::to_string_pretty(&config)?;
        std::fs::write(path, content)?;
        Ok(())
    }
    
    fn parse_key_combo(&self, s: &str) -> Result<KeyCombo> {
        // Parser "Cmd+S" ou "Ctrl+Shift+N"
        // ...
    }
}
```

## Résumé

- **Système centralisé** : Tous les raccourcis au même endroit
- **Raccourcis globaux** : Disponibles partout dans l'app
- **Personnalisation** : Sauvegarder les préférences utilisateur
- **Standards** : Respecter les conventions de chaque OS
