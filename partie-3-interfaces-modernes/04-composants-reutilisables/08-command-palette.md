# 11.8 Command Palette

## Palette de commandes avec recherche

```rust
pub struct CommandPalette<'a> {
    open: &'a mut bool,
    query: &'a mut String,
    commands: Vec<Command<'a>>,
    selected_index: usize,
}

pub struct Command<'a> {
    pub id: &'a str,
    pub label: &'a str,
    pub shortcut: Option<&'a str>,
    pub icon: Option<&'a str>,
}

impl<'a> CommandPalette<'a> {
    pub fn new(open: &'a mut bool, query: &'a mut String) -> Self {
        Self {
            open,
            query,
            commands: Vec::new(),
            selected_index: 0,
        }
    }
    
    pub fn command(mut self, id: &'a str, label: &'a str) -> Self {
        self.commands.push(Command {
            id,
            label,
            shortcut: None,
            icon: None,
        });
        self
    }
    
    pub fn command_with_shortcut(
        mut self,
        id: &'a str,
        label: &'a str,
        shortcut: &'a str,
    ) -> Self {
        self.commands.push(Command {
            id,
            label,
            shortcut: Some(shortcut),
            icon: None,
        });
        self
    }
    
    pub fn show(mut self, ctx: &egui::Context, tokens: &DesignTokens) -> Option<&'a str> {
        // Afficher la palette avec recherche et navigation clavier
        // Voir 00-contenu-complet.md pour l'implémentation complète
    }
}
```

## Usage

```rust
let mut show_palette = false;
let mut query = String::new();

// Ouvrir avec Cmd+K (ou Ctrl+K)
if ctx.input(|i| i.modifiers.command && i.key_pressed(egui::Key::K)) {
    show_palette = true;
}

// Afficher la palette
if let Some(command_id) = CommandPalette::new(&mut show_palette, &mut query)
    .command("new-doc", "Nouveau document")
    .command_with_shortcut("save", "Sauvegarder", "Cmd+S")
    .command("settings", "Paramètres")
    .show(ctx, &tokens)
{
    match command_id {
        "new-doc" => create_new_document(),
        "save" => save_document(),
        "settings" => open_settings(),
        _ => {}
    }
}
```

## Features

- **Recherche** : Filtrage en temps réel
- **Navigation clavier** : Flèches haut/bas, Enter
- **Raccourcis** : Afficher les raccourcis clavier
- **Icônes** : Support pour icônes visuelles
- **Fuzzy search** : Recherche approximative (optionnel)
