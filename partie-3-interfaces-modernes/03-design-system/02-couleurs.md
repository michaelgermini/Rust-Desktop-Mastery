# 10.2 Couleurs

## Palette de couleurs

```rust
use egui::Color32;

pub struct ColorTokens {
    // Couleurs primaires
    pub primary: Color32,
    pub primary_hover: Color32,
    pub primary_active: Color32,
    pub primary_muted: Color32,
    
    // Couleurs sémantiques
    pub success: Color32,
    pub warning: Color32,
    pub error: Color32,
    pub info: Color32,
    
    // Couleurs de texte
    pub text_primary: Color32,
    pub text_secondary: Color32,
    pub text_muted: Color32,
    pub text_inverse: Color32,
    
    // Couleurs de fond
    pub bg_primary: Color32,
    pub bg_secondary: Color32,
    pub bg_tertiary: Color32,
    pub bg_elevated: Color32,
    
    // Bordures
    pub border_default: Color32,
    pub border_strong: Color32,
    pub border_muted: Color32,
}

impl ColorTokens {
    pub fn light() -> Self {
        Self {
            // Primaire : bleu professionnel
            primary: Color32::from_rgb(59, 130, 246),
            primary_hover: Color32::from_rgb(37, 99, 235),
            primary_active: Color32::from_rgb(29, 78, 216),
            primary_muted: Color32::from_rgb(219, 234, 254),
            
            // Sémantiques
            success: Color32::from_rgb(34, 197, 94),
            warning: Color32::from_rgb(234, 179, 8),
            error: Color32::from_rgb(239, 68, 68),
            info: Color32::from_rgb(59, 130, 246),
            
            // Texte
            text_primary: Color32::from_rgb(17, 24, 39),
            text_secondary: Color32::from_rgb(75, 85, 99),
            text_muted: Color32::from_rgb(156, 163, 175),
            text_inverse: Color32::WHITE,
            
            // Fonds
            bg_primary: Color32::WHITE,
            bg_secondary: Color32::from_rgb(249, 250, 251),
            bg_tertiary: Color32::from_rgb(243, 244, 246),
            bg_elevated: Color32::WHITE,
            
            // Bordures
            border_default: Color32::from_rgb(229, 231, 235),
            border_strong: Color32::from_rgb(209, 213, 219),
            border_muted: Color32::from_rgb(243, 244, 246),
        }
    }
    
    pub fn dark() -> Self {
        Self {
            // Primaire
            primary: Color32::from_rgb(96, 165, 250),
            primary_hover: Color32::from_rgb(147, 197, 253),
            primary_active: Color32::from_rgb(59, 130, 246),
            primary_muted: Color32::from_rgb(30, 58, 138),
            
            // Sémantiques
            success: Color32::from_rgb(74, 222, 128),
            warning: Color32::from_rgb(250, 204, 21),
            error: Color32::from_rgb(248, 113, 113),
            info: Color32::from_rgb(96, 165, 250),
            
            // Texte
            text_primary: Color32::from_rgb(249, 250, 251),
            text_secondary: Color32::from_rgb(209, 213, 219),
            text_muted: Color32::from_rgb(107, 114, 128),
            text_inverse: Color32::from_rgb(17, 24, 39),
            
            // Fonds
            bg_primary: Color32::from_rgb(17, 24, 39),
            bg_secondary: Color32::from_rgb(31, 41, 55),
            bg_tertiary: Color32::from_rgb(55, 65, 81),
            bg_elevated: Color32::from_rgb(31, 41, 55),
            
            // Bordures
            border_default: Color32::from_rgb(55, 65, 81),
            border_strong: Color32::from_rgb(75, 85, 99),
            border_muted: Color32::from_rgb(31, 41, 55),
        }
    }
}
```

## Utilitaires de couleur

```rust
impl ColorTokens {
    /// Couleur avec opacité
    pub fn with_alpha(color: Color32, alpha: u8) -> Color32 {
        Color32::from_rgba_unmultiplied(color.r(), color.g(), color.b(), alpha)
    }
    
    /// Éclaircir une couleur
    pub fn lighten(color: Color32, amount: f32) -> Color32 {
        let [r, g, b, a] = color.to_array();
        Color32::from_rgba_unmultiplied(
            (r as f32 + (255.0 - r as f32) * amount) as u8,
            (g as f32 + (255.0 - g as f32) * amount) as u8,
            (b as f32 + (255.0 - b as f32) * amount) as u8,
            a,
        )
    }
    
    /// Assombrir une couleur
    pub fn darken(color: Color32, amount: f32) -> Color32 {
        let [r, g, b, a] = color.to_array();
        Color32::from_rgba_unmultiplied(
            (r as f32 * (1.0 - amount)) as u8,
            (g as f32 * (1.0 - amount)) as u8,
            (b as f32 * (1.0 - amount)) as u8,
            a,
        )
    }
}
```

## Accessibilité : Contrastes WCAG

```rust
/// Calculer le ratio de contraste (WCAG)
pub fn contrast_ratio(color1: Color32, color2: Color32) -> f32 {
    let luminance1 = relative_luminance(color1);
    let luminance2 = relative_luminance(color2);
    
    let lighter = luminance1.max(luminance2);
    let darker = luminance1.min(luminance2);
    
    (lighter + 0.05) / (darker + 0.05)
}

fn relative_luminance(color: Color32) -> f32 {
    let [r, g, b] = [color.r(), color.g(), color.b()];
    
    let r = (r as f32 / 255.0).powf(2.2);
    let g = (g as f32 / 255.0).powf(2.2);
    let b = (b as f32 / 255.0).powf(2.2);
    
    0.2126 * r + 0.7152 * g + 0.0722 * b
}

/// Vérifier si le contraste est suffisant (WCAG AA)
pub fn meets_wcag_aa(text: Color32, background: Color32) -> bool {
    contrast_ratio(text, background) >= 4.5
}

/// Vérifier si le contraste est suffisant (WCAG AAA)
pub fn meets_wcag_aaa(text: Color32, background: Color32) -> bool {
    contrast_ratio(text, background) >= 7.0
}
```

## Résumé

- **Palette structurée** : Primaire, sémantiques, texte, fonds, bordures
- **Thèmes** : Light et dark avec valeurs adaptées
- **Utilitaires** : Opacité, lighten, darken
- **Accessibilité** : Vérification des contrastes WCAG
