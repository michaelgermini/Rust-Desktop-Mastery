# 11.1 Button

## Implémentation complète

```rust
use egui::{Response, Ui, Widget};

#[derive(Clone)]
pub enum ButtonVariant {
    Primary,
    Secondary,
    Ghost,
    Danger,
}

#[derive(Clone)]
pub enum ButtonSize {
    Small,
    Medium,
    Large,
}

pub struct Button<'a> {
    text: &'a str,
    icon: Option<&'a str>,
    variant: ButtonVariant,
    size: ButtonSize,
    disabled: bool,
    loading: bool,
    full_width: bool,
}

impl<'a> Button<'a> {
    pub fn new(text: &'a str) -> Self {
        Self {
            text,
            icon: None,
            variant: ButtonVariant::Primary,
            size: ButtonSize::Medium,
            disabled: false,
            loading: false,
            full_width: false,
        }
    }
    
    pub fn icon(mut self, icon: &'a str) -> Self {
        self.icon = Some(icon);
        self
    }
    
    pub fn variant(mut self, variant: ButtonVariant) -> Self {
        self.variant = variant;
        self
    }
    
    pub fn size(mut self, size: ButtonSize) -> Self {
        self.size = size;
        self
    }
    
    pub fn disabled(mut self, disabled: bool) -> Self {
        self.disabled = disabled;
        self
    }
    
    pub fn loading(mut self, loading: bool) -> Self {
        self.loading = loading;
        self
    }
    
    pub fn full_width(mut self) -> Self {
        self.full_width = true;
        self
    }
    
    pub fn show(self, ui: &mut Ui, tokens: &DesignTokens) -> Response {
        // Implémentation complète avec styles selon variant et size
        // Voir le fichier 00-contenu-complet.md pour le code complet
    }
}
```

## Usage

```rust
// Bouton simple
if Button::new("Sauvegarder")
    .icon("💾")
    .variant(ButtonVariant::Primary)
    .show(ui, &tokens)
    .clicked()
{
    save_document();
}

// Bouton avec loading
Button::new("Charger")
    .loading(is_loading)
    .show(ui, &tokens);

// Bouton danger
if Button::new("Supprimer")
    .variant(ButtonVariant::Danger)
    .show(ui, &tokens)
    .clicked()
{
    delete_item();
}
```

## Variantes

- **Primary** : Action principale (bleu)
- **Secondary** : Action secondaire (gris)
- **Ghost** : Action discrète (transparent)
- **Danger** : Action destructive (rouge)

## États

- **Normal** : État par défaut
- **Hover** : Survol
- **Active** : Clic
- **Disabled** : Désactivé
- **Loading** : En cours de chargement
