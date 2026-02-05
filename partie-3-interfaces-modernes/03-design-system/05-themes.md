# 10.5 Thèmes Dark et Light

## Gestionnaire de thème

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

## Appliquer le thème à egui

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

## Transition entre thèmes

```rust
impl ThemeManager {
    /// Basculer le thème avec animation (optionnel)
    pub fn toggle(&mut self) {
        let new_mode = match self.current_mode {
            ThemeMode::Light => ThemeMode::Dark,
            ThemeMode::Dark => ThemeMode::Light,
            ThemeMode::System => {
                if Self::system_is_dark() {
                    ThemeMode::Light
                } else {
                    ThemeMode::Dark
                }
            }
        };
        self.set_mode(new_mode);
    }
}
```

## Résumé

- **Trois modes** : Light, Dark, System
- **Détection système** : Suit les préférences OS
- **Application automatique** : Style egui mis à jour
- **Transition facile** : Basculer entre thèmes
