# 24.3 Design System

## Design tokens

```rust
// Utiliser le design system créé dans la Partie III
use notevault_ui::design_system::DesignTokens;

pub struct App {
    theme: ThemeManager,
    // ...
}

impl App {
    fn setup_theme(&mut self) {
        self.theme = ThemeManager::new(ThemeMode::System);
    }
    
    fn render(&mut self, ctx: &egui::Context) {
        // Appliquer le thème
        apply_theme_to_egui(ctx, self.theme.tokens());
        
        // Rendre l'UI avec les tokens
        // ...
    }
}
```

## Composants

```rust
// Réutiliser les composants de la Partie III
use notevault_ui::components::{Button, TextInput, Table, Modal};

impl App {
    fn render_ui(&mut self, ui: &mut Ui) {
        let tokens = self.theme.tokens();
        
        // Utiliser les composants réutilisables
        if Button::new("Nouvelle note")
            .variant(ButtonVariant::Primary)
            .show(ui, tokens)
            .clicked()
        {
            self.create_note();
        }
        
        // Table avec design system
        Table::new(&self.notes)
            .column("Titre", |note, ui, _| {
                ui.label(&note.title);
            })
            .show(ui, tokens);
    }
}
```

## Thèmes

```rust
impl App {
    fn render_settings(&mut self, ctx: &egui::Context) {
        Modal::new("Paramètres", &mut self.show_settings)
            .show(ctx, self.theme.tokens(), |ui, tokens| {
                ui.label("Thème:");
                
                ui.radio_value(&mut self.theme.mode, ThemeMode::Light, "Clair");
                ui.radio_value(&mut self.theme.mode, ThemeMode::Dark, "Sombre");
                ui.radio_value(&mut self.theme.mode, ThemeMode::System, "Système");
                
                if ui.button("Appliquer").clicked() {
                    self.theme.set_mode(self.theme.mode);
                    apply_theme_to_egui(ctx, self.theme.tokens());
                }
            });
    }
}
```

## Résumé

- **Design tokens** : Cohérence visuelle partout
- **Composants** : Réutilisation des composants de la Partie III
- **Thèmes** : Light/Dark/System
- **Cohérence** : Même apparence dans toute l'application
