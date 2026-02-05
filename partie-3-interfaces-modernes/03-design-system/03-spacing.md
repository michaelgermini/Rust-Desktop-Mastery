# 10.3 Spacing

## Système d'espacement

```rust
/// Système d'espacement basé sur une grille de 4px
pub struct SpacingTokens {
    pub none: f32,
    pub xxs: f32,   // 2px
    pub xs: f32,    // 4px
    pub sm: f32,    // 8px
    pub md: f32,    // 16px
    pub lg: f32,    // 24px
    pub xl: f32,    // 32px
    pub xxl: f32,   // 48px
    pub xxxl: f32,  // 64px
}

impl Default for SpacingTokens {
    fn default() -> Self {
        Self {
            none: 0.0,
            xxs: 2.0,
            xs: 4.0,
            sm: 8.0,
            md: 16.0,
            lg: 24.0,
            xl: 32.0,
            xxl: 48.0,
            xxxl: 64.0,
        }
    }
}
```

## Méthodes utilitaires

```rust
impl SpacingTokens {
    /// Espacement pour les composants
    pub fn component_padding(&self) -> egui::Margin {
        egui::Margin::symmetric(self.sm, self.xs)
    }
    
    /// Espacement pour les cards
    pub fn card_padding(&self) -> egui::Margin {
        egui::Margin::same(self.md)
    }
    
    /// Espacement entre items d'une liste
    pub fn list_spacing(&self) -> f32 {
        self.xs
    }
    
    /// Espacement entre sections
    pub fn section_spacing(&self) -> f32 {
        self.lg
    }
}
```

## Utilisation pratique

```rust
fn render_with_spacing(ui: &mut egui::Ui, tokens: &DesignTokens) {
    // Espacement vertical
    ui.add_space(tokens.spacing.sm);
    
    // Padding d'un composant
    egui::Frame::none()
        .inner_margin(tokens.spacing.component_padding())
        .show(ui, |ui| {
            ui.label("Contenu");
        });
    
    // Espacement entre sections
    ui.add_space(tokens.spacing.section_spacing());
    
    // Espacement horizontal
    ui.horizontal(|ui| {
        ui.label("Gauche");
        ui.add_space(tokens.spacing.md);
        ui.label("Droite");
    });
}
```

## Grille de 4px

Le système est basé sur une grille de 4px pour :
- **Cohérence visuelle** : Alignements naturels
- **Simplicité** : Multiples faciles à retenir
- **Flexibilité** : Assez de granularité sans complexité

## Résumé

- **Grille de 4px** : Base du système
- **Échelle cohérente** : xxs à xxxl
- **Méthodes utilitaires** : Padding, spacing contextuels
- **Cohérence** : Même espacement partout
