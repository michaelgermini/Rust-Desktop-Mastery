# 26.4 Tailles et Lisibilité

## Tailles minimales

```rust
/// Tailles minimales recommandées pour l'accessibilité
pub struct AccessibleSizes {
    /// Zone de clic minimale (44x44 sur mobile, 24x24 desktop)
    pub min_touch_target: f32,
    /// Taille de texte minimale
    pub min_font_size: f32,
    /// Espacement minimal entre éléments interactifs
    pub min_spacing: f32,
    /// Hauteur de ligne minimale (line-height)
    pub min_line_height: f32,
}

impl Default for AccessibleSizes {
    fn default() -> Self {
        Self {
            min_touch_target: 32.0,  // Desktop (24px minimum recommandé, 32px pour confort)
            min_font_size: 14.0,      // 14px minimum pour lisibilité
            min_spacing: 8.0,         // 8px entre éléments interactifs
            min_line_height: 1.5,     // 1.5x la taille de police
        }
    }
}
```

## Zones de clic accessibles

```rust
/// Wrapper pour garantir une zone de clic minimale
pub fn accessible_button(ui: &mut Ui, text: &str, min_size: f32) -> Response {
    let desired_size = egui::vec2(
        ui.available_width().min(200.0).max(min_size),
        min_size,
    );
    
    ui.add_sized(desired_size, egui::Button::new(text))
}

/// Bouton avec zone de clic garantie
pub fn render_accessible_button(
    ui: &mut Ui,
    text: &str,
    sizes: &AccessibleSizes,
) -> Response {
    let button_size = egui::vec2(
        text.len() as f32 * 8.0 + 16.0,  // Largeur estimée
        sizes.min_touch_target,
    );
    
    ui.add_sized(button_size, egui::Button::new(text))
}
```

## Typographie lisible

```rust
impl TypographyTokens {
    pub fn ensure_readable(&self) -> Self {
        let mut tokens = self.clone();
        
        // Garantir des tailles minimales
        tokens.size_xs = tokens.size_xs.max(12.0);
        tokens.size_sm = tokens.size_sm.max(14.0);
        tokens.size_md = tokens.size_md.max(16.0);
        
        // Garantir des line-heights appropriés
        tokens.line_height_sm = 1.5;
        tokens.line_height_md = 1.6;
        tokens.line_height_lg = 1.7;
        
        tokens
    }
    
    pub fn apply_to_text(&self, text: &str, size: f32, colors: &ColorTokens) -> egui::RichText {
        let readable_size = size.max(14.0);  // Minimum 14px
        
        egui::RichText::new(text)
            .size(readable_size)
            .color(colors.text_primary)
    }
}
```

## Résumé

- **Tailles minimales** : 32px pour zones de clic, 14px pour texte
- **Zones de clic** : Garantir une taille minimale pour tous les boutons
- **Typographie** : Tailles et line-heights appropriés
- **Espacement** : Suffisamment d'espace entre éléments interactifs
