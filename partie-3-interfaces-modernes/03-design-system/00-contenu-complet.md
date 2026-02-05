# Chapitre 10 : Créer son Design System

## Introduction

Un Design System est un ensemble de règles, composants et tokens qui garantissent la cohérence visuelle d'une application. Ce chapitre montre comment construire un système de design complet en Rust pour egui.

---

## 10.1 Tokens de Design

### Qu'est-ce qu'un token ?

Les tokens sont les valeurs atomiques de votre design : couleurs, espacements, tailles de police. Ils permettent de modifier l'apparence globale depuis un seul endroit.

```rust
/// Tokens de design - les valeurs atomiques du système
pub struct DesignTokens {
    pub colors: ColorTokens,
    pub spacing: SpacingTokens,
    pub typography: TypographyTokens,
    pub radius: RadiusTokens,
    pub shadows: ShadowTokens,
}

impl DesignTokens {
    /// Tokens par défaut (thème light)
    pub fn light() -> Self {
        Self {
            colors: ColorTokens::light(),
            spacing: SpacingTokens::default(),
            typography: TypographyTokens::default(),
            radius: RadiusTokens::default(),
            shadows: ShadowTokens::light(),
        }
    }
    
    /// Tokens pour le thème dark
    pub fn dark() -> Self {
        Self {
            colors: ColorTokens::dark(),
            spacing: SpacingTokens::default(),
            typography: TypographyTokens::default(),
            radius: RadiusTokens::default(),
            shadows: ShadowTokens::dark(),
        }
    }
}
```

---

## 10.2 Couleurs

### Palette de couleurs

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

### Utilitaires de couleur

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

---

## 10.3 Spacing

### Système d'espacement

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

---

## 10.4 Typographie

### Système typographique

```rust
pub struct TypographyTokens {
    // Tailles de police
    pub size_xs: f32,    // 12px
    pub size_sm: f32,    // 14px
    pub size_md: f32,    // 16px
    pub size_lg: f32,    // 18px
    pub size_xl: f32,    // 20px
    pub size_2xl: f32,   // 24px
    pub size_3xl: f32,   // 30px
    pub size_4xl: f32,   // 36px
    
    // Familles de polices
    pub font_family_sans: egui::FontFamily,
    pub font_family_mono: egui::FontFamily,
}

impl Default for TypographyTokens {
    fn default() -> Self {
        Self {
            size_xs: 12.0,
            size_sm: 14.0,
            size_md: 16.0,
            size_lg: 18.0,
            size_xl: 20.0,
            size_2xl: 24.0,
            size_3xl: 30.0,
            size_4xl: 36.0,
            font_family_sans: egui::FontFamily::Proportional,
            font_family_mono: egui::FontFamily::Monospace,
        }
    }
}

impl TypographyTokens {
    /// Styles de texte prédéfinis
    pub fn heading_1(&self, colors: &ColorTokens) -> egui::RichText {
        egui::RichText::new("")
            .size(self.size_4xl)
            .color(colors.text_primary)
            .strong()
    }
    
    pub fn heading_2(&self, colors: &ColorTokens) -> egui::RichText {
        egui::RichText::new("")
            .size(self.size_3xl)
            .color(colors.text_primary)
            .strong()
    }
    
    pub fn heading_3(&self, colors: &ColorTokens) -> egui::RichText {
        egui::RichText::new("")
            .size(self.size_2xl)
            .color(colors.text_primary)
            .strong()
    }
    
    pub fn body(&self, colors: &ColorTokens) -> egui::RichText {
        egui::RichText::new("")
            .size(self.size_md)
            .color(colors.text_primary)
    }
    
    pub fn body_small(&self, colors: &ColorTokens) -> egui::RichText {
        egui::RichText::new("")
            .size(self.size_sm)
            .color(colors.text_secondary)
    }
    
    pub fn caption(&self, colors: &ColorTokens) -> egui::RichText {
        egui::RichText::new("")
            .size(self.size_xs)
            .color(colors.text_muted)
    }
    
    pub fn code(&self, colors: &ColorTokens) -> egui::RichText {
        egui::RichText::new("")
            .size(self.size_sm)
            .color(colors.text_primary)
            .family(self.font_family_mono.clone())
    }
}
```

### Installer des polices custom

```rust
fn setup_fonts(ctx: &egui::Context) {
    let mut fonts = egui::FontDefinitions::default();
    
    // Charger une police custom
    fonts.font_data.insert(
        "Inter".to_owned(),
        egui::FontData::from_static(include_bytes!("../assets/fonts/Inter-Regular.ttf")),
    );
    
    fonts.font_data.insert(
        "Inter-Bold".to_owned(),
        egui::FontData::from_static(include_bytes!("../assets/fonts/Inter-Bold.ttf")),
    );
    
    fonts.font_data.insert(
        "JetBrainsMono".to_owned(),
        egui::FontData::from_static(include_bytes!("../assets/fonts/JetBrainsMono-Regular.ttf")),
    );
    
    // Configurer les familles
    fonts.families.entry(egui::FontFamily::Proportional)
        .or_default()
        .insert(0, "Inter".to_owned());
    
    fonts.families.entry(egui::FontFamily::Monospace)
        .or_default()
        .insert(0, "JetBrainsMono".to_owned());
    
    ctx.set_fonts(fonts);
}
```

---

## 10.5 Thèmes Dark et Light

### Gestionnaire de thème

```rust
#[derive(Clone, Copy, PartialEq, Debug)]
pub enum ThemeMode {
    Light,
    Dark,
    System,
}

pub struct ThemeManager {
    current_mode: ThemeMode,
    tokens: DesignTokens,
}

impl ThemeManager {
    pub fn new(mode: ThemeMode) -> Self {
        let tokens = match mode {
            ThemeMode::Light => DesignTokens::light(),
            ThemeMode::Dark => DesignTokens::dark(),
            ThemeMode::System => {
                if Self::system_is_dark() {
                    DesignTokens::dark()
                } else {
                    DesignTokens::light()
                }
            }
        };
        
        Self { current_mode: mode, tokens }
    }
    
    pub fn set_mode(&mut self, mode: ThemeMode) {
        self.current_mode = mode;
        self.tokens = match mode {
            ThemeMode::Light => DesignTokens::light(),
            ThemeMode::Dark => DesignTokens::dark(),
            ThemeMode::System => {
                if Self::system_is_dark() {
                    DesignTokens::dark()
                } else {
                    DesignTokens::light()
                }
            }
        };
    }
    
    pub fn tokens(&self) -> &DesignTokens {
        &self.tokens
    }
    
    pub fn is_dark(&self) -> bool {
        match self.current_mode {
            ThemeMode::Dark => true,
            ThemeMode::Light => false,
            ThemeMode::System => Self::system_is_dark(),
        }
    }
    
    #[cfg(target_os = "windows")]
    fn system_is_dark() -> bool {
        // Lire la registry Windows
        use winreg::RegKey;
        use winreg::enums::HKEY_CURRENT_USER;
        
        let hkcu = RegKey::predef(HKEY_CURRENT_USER);
        if let Ok(key) = hkcu.open_subkey("Software\\Microsoft\\Windows\\CurrentVersion\\Themes\\Personalize") {
            if let Ok(value) = key.get_value::<u32, _>("AppsUseLightTheme") {
                return value == 0;
            }
        }
        false
    }
    
    #[cfg(target_os = "macos")]
    fn system_is_dark() -> bool {
        // Utiliser l'API macOS
        dark_light::detect() == dark_light::Mode::Dark
    }
    
    #[cfg(target_os = "linux")]
    fn system_is_dark() -> bool {
        dark_light::detect() == dark_light::Mode::Dark
    }
}
```

### Appliquer le thème à egui

```rust
fn apply_theme_to_egui(ctx: &egui::Context, tokens: &DesignTokens) {
    let mut style = (*ctx.style()).clone();
    
    // Couleurs des widgets
    style.visuals.widgets.noninteractive.bg_fill = tokens.colors.bg_secondary;
    style.visuals.widgets.noninteractive.fg_stroke = egui::Stroke::new(1.0, tokens.colors.text_primary);
    
    style.visuals.widgets.inactive.bg_fill = tokens.colors.bg_tertiary;
    style.visuals.widgets.inactive.fg_stroke = egui::Stroke::new(1.0, tokens.colors.text_primary);
    
    style.visuals.widgets.hovered.bg_fill = tokens.colors.primary_muted;
    style.visuals.widgets.hovered.fg_stroke = egui::Stroke::new(1.0, tokens.colors.primary);
    
    style.visuals.widgets.active.bg_fill = tokens.colors.primary;
    style.visuals.widgets.active.fg_stroke = egui::Stroke::new(1.0, tokens.colors.text_inverse);
    
    // Couleurs de fond
    style.visuals.panel_fill = tokens.colors.bg_primary;
    style.visuals.window_fill = tokens.colors.bg_elevated;
    style.visuals.extreme_bg_color = tokens.colors.bg_tertiary;
    
    // Sélection
    style.visuals.selection.bg_fill = ColorTokens::with_alpha(tokens.colors.primary, 80);
    style.visuals.selection.stroke = egui::Stroke::new(1.0, tokens.colors.primary);
    
    // Espacement
    style.spacing.item_spacing = egui::vec2(tokens.spacing.sm, tokens.spacing.xs);
    style.spacing.button_padding = egui::vec2(tokens.spacing.md, tokens.spacing.sm);
    
    // Rayons de bordure
    style.visuals.window_rounding = egui::Rounding::same(tokens.radius.md);
    style.visuals.widgets.noninteractive.rounding = egui::Rounding::same(tokens.radius.sm);
    style.visuals.widgets.inactive.rounding = egui::Rounding::same(tokens.radius.sm);
    style.visuals.widgets.hovered.rounding = egui::Rounding::same(tokens.radius.sm);
    style.visuals.widgets.active.rounding = egui::Rounding::same(tokens.radius.sm);
    
    ctx.set_style(style);
}
```

---

## 10.6 DPI Scaling

### Gérer les écrans haute résolution

```rust
pub struct DpiManager {
    scale_factor: f32,
    base_scale: f32,
}

impl DpiManager {
    pub fn new(native_scale: f32) -> Self {
        Self {
            scale_factor: native_scale,
            base_scale: 1.0,
        }
    }
    
    /// Facteur de scale combiné (système + préférence utilisateur)
    pub fn effective_scale(&self) -> f32 {
        self.scale_factor * self.base_scale
    }
    
    /// Définir une préférence utilisateur de zoom
    pub fn set_user_scale(&mut self, scale: f32) {
        self.base_scale = scale.clamp(0.75, 2.0);
    }
    
    /// Convertir une taille en pixels logiques vers pixels physiques
    pub fn to_physical(&self, logical: f32) -> f32 {
        logical * self.effective_scale()
    }
    
    /// Convertir une taille en pixels physiques vers pixels logiques
    pub fn to_logical(&self, physical: f32) -> f32 {
        physical / self.effective_scale()
    }
}

/// Tokens de design avec DPI awareness
impl DesignTokens {
    pub fn scaled(&self, dpi: &DpiManager) -> ScaledTokens {
        let scale = dpi.effective_scale();
        
        ScaledTokens {
            spacing: ScaledSpacing {
                xs: self.spacing.xs * scale,
                sm: self.spacing.sm * scale,
                md: self.spacing.md * scale,
                lg: self.spacing.lg * scale,
                xl: self.spacing.xl * scale,
            },
            typography: ScaledTypography {
                size_sm: self.typography.size_sm * scale,
                size_md: self.typography.size_md * scale,
                size_lg: self.typography.size_lg * scale,
                size_xl: self.typography.size_xl * scale,
            },
            radius: ScaledRadius {
                sm: self.radius.sm * scale,
                md: self.radius.md * scale,
                lg: self.radius.lg * scale,
            },
        }
    }
}
```

### Configuration egui pour HiDPI

```rust
fn configure_egui_for_hidpi(ctx: &egui::Context, scale: f32) {
    // egui gère le DPI automatiquement, mais on peut ajuster
    ctx.set_pixels_per_point(scale);
    
    // Ou laisser egui détecter automatiquement
    // ctx.set_pixels_per_point(ctx.native_pixels_per_point().unwrap_or(1.0));
}
```

---

## 10.7 Radius et Shadows

### Tokens de bordure arrondie

```rust
pub struct RadiusTokens {
    pub none: f32,
    pub sm: f32,    // 4px
    pub md: f32,    // 8px
    pub lg: f32,    // 12px
    pub xl: f32,    // 16px
    pub full: f32,  // 9999px (cercle)
}

impl Default for RadiusTokens {
    fn default() -> Self {
        Self {
            none: 0.0,
            sm: 4.0,
            md: 8.0,
            lg: 12.0,
            xl: 16.0,
            full: 9999.0,
        }
    }
}
```

### Tokens d'ombre

```rust
pub struct ShadowTokens {
    pub sm: Shadow,
    pub md: Shadow,
    pub lg: Shadow,
    pub xl: Shadow,
}

pub struct Shadow {
    pub offset: egui::Vec2,
    pub blur: f32,
    pub spread: f32,
    pub color: Color32,
}

impl ShadowTokens {
    pub fn light() -> Self {
        let shadow_color = Color32::from_rgba_unmultiplied(0, 0, 0, 25);
        
        Self {
            sm: Shadow {
                offset: egui::vec2(0.0, 1.0),
                blur: 2.0,
                spread: 0.0,
                color: shadow_color,
            },
            md: Shadow {
                offset: egui::vec2(0.0, 4.0),
                blur: 6.0,
                spread: -1.0,
                color: shadow_color,
            },
            lg: Shadow {
                offset: egui::vec2(0.0, 10.0),
                blur: 15.0,
                spread: -3.0,
                color: shadow_color,
            },
            xl: Shadow {
                offset: egui::vec2(0.0, 20.0),
                blur: 25.0,
                spread: -5.0,
                color: shadow_color,
            },
        }
    }
    
    pub fn dark() -> Self {
        // Ombres plus subtiles en mode sombre
        let shadow_color = Color32::from_rgba_unmultiplied(0, 0, 0, 50);
        
        Self {
            sm: Shadow {
                offset: egui::vec2(0.0, 1.0),
                blur: 3.0,
                spread: 0.0,
                color: shadow_color,
            },
            // ... etc
            md: Shadow { offset: egui::vec2(0.0, 4.0), blur: 8.0, spread: -1.0, color: shadow_color },
            lg: Shadow { offset: egui::vec2(0.0, 10.0), blur: 20.0, spread: -3.0, color: shadow_color },
            xl: Shadow { offset: egui::vec2(0.0, 20.0), blur: 30.0, spread: -5.0, color: shadow_color },
        }
    }
}

impl Shadow {
    /// Appliquer une ombre à un painter egui
    pub fn paint(&self, painter: &egui::Painter, rect: egui::Rect, rounding: f32) {
        let shadow_rect = rect.translate(self.offset).expand(self.spread);
        painter.rect_filled(
            shadow_rect.expand(self.blur),
            rounding + self.blur,
            self.color,
        );
    }
}
```

---

## Résumé du Chapitre

### Structure du Design System

```
DesignTokens
├── colors: ColorTokens
│   ├── primary, secondary, ...
│   ├── text_primary, text_secondary, ...
│   └── bg_primary, bg_secondary, ...
├── spacing: SpacingTokens
│   └── xs, sm, md, lg, xl, ...
├── typography: TypographyTokens
│   ├── size_sm, size_md, size_lg, ...
│   └── font_family_sans, font_family_mono
├── radius: RadiusTokens
│   └── sm, md, lg, full
└── shadows: ShadowTokens
    └── sm, md, lg, xl
```

### Avantages d'un Design System

- **Cohérence** : Même apparence partout
- **Maintenabilité** : Changement global depuis un seul fichier
- **Thèmes** : Light/Dark mode simple à implémenter
- **Collaboration** : Vocabulaire partagé avec les designers

---

**Dans le prochain chapitre, nous utiliserons ce design system pour créer des composants réutilisables de qualité professionnelle.**
