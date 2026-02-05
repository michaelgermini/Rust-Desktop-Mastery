# 11.5 Modals

## Dialogues modaux

```rust
pub struct Modal<'a> {
    title: &'a str,
    open: &'a mut bool,
    width: f32,
}

impl<'a> Modal<'a> {
    pub fn new(title: &'a str, open: &'a mut bool) -> Self {
        Self {
            title,
            open,
            width: 400.0,
        }
    }
    
    pub fn width(mut self, width: f32) -> Self {
        self.width = width;
        self
    }
    
    pub fn show(
        self,
        ctx: &egui::Context,
        tokens: &DesignTokens,
        content: impl FnOnce(&mut Ui, &DesignTokens),
    ) {
        // Overlay sombre + fenêtre modale
        // Voir 00-contenu-complet.md pour l'implémentation complète
    }
}
```

## Usage

```rust
Modal::new("Confirmation", &mut show_confirm)
    .show(ctx, &tokens, |ui, tokens| {
        ui.label("Êtes-vous sûr de vouloir supprimer ?");
        ui.add_space(tokens.spacing.md);
        ui.horizontal(|ui| {
            if Button::new("Annuler")
                .variant(ButtonVariant::Ghost)
                .show(ui, tokens)
                .clicked()
            {
                show_confirm = false;
            }
            if Button::new("Supprimer")
                .variant(ButtonVariant::Danger)
                .show(ui, tokens)
                .clicked()
            {
                delete_item();
                show_confirm = false;
            }
        });
    });
```

## Features

- **Overlay** : Fond sombre semi-transparent
- **Fermeture** : Clic sur overlay ou Escape
- **Centré** : Positionnement automatique
- **Taille custom** : Largeur configurable
