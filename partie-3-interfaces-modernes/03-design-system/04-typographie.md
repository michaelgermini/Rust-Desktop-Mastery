# 10.4 Typographie

## Système typographique

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
```

## Styles prédéfinis

```rust
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

## Installer des polices custom

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

## Hiérarchie typographique

```rust
fn render_typography_hierarchy(ui: &mut egui::Ui, tokens: &DesignTokens) {
    // Heading 1
    ui.label(tokens.typography.heading_1(&tokens.colors).text("Titre Principal"));
    
    // Heading 2
    ui.label(tokens.typography.heading_2(&tokens.colors).text("Sous-titre"));
    
    // Body
    ui.label(tokens.typography.body(&tokens.colors).text("Texte de corps"));
    
    // Caption
    ui.label(tokens.typography.caption(&tokens.colors).text("Légende"));
    
    // Code
    ui.label(tokens.typography.code(&tokens.colors).text("let x = 42;"));
}
```

## Résumé

- **Échelle cohérente** : Tailles de xs à 4xl
- **Styles prédéfinis** : Headings, body, caption, code
- **Polices custom** : Support pour Inter, JetBrains Mono, etc.
- **Hiérarchie claire** : Distinction visuelle des niveaux
