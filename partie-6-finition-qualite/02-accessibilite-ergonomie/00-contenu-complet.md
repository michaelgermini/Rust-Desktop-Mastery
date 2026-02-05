# Chapitre 26 : Accessibilité et Ergonomie

## Introduction

Une application accessible et ergonomique peut être utilisée confortablement pendant des heures par tous les utilisateurs.

---

## 26.1 Raccourcis Clavier

```rust
pub struct ShortcutManager {
    shortcuts: HashMap<KeyCombo, ShortcutAction>,
}

#[derive(Hash, Eq, PartialEq, Clone)]
pub struct KeyCombo {
    pub key: egui::Key,
    pub modifiers: egui::Modifiers,
}

impl ShortcutManager {
    pub fn new() -> Self {
        let mut manager = Self { shortcuts: HashMap::new() };
        
        // Raccourcis standard
        manager.register(KeyCombo::cmd(egui::Key::N), ShortcutAction::NewDocument);
        manager.register(KeyCombo::cmd(egui::Key::O), ShortcutAction::OpenDocument);
        manager.register(KeyCombo::cmd(egui::Key::S), ShortcutAction::Save);
        manager.register(KeyCombo::cmd_shift(egui::Key::S), ShortcutAction::SaveAs);
        manager.register(KeyCombo::cmd(egui::Key::Z), ShortcutAction::Undo);
        manager.register(KeyCombo::cmd_shift(egui::Key::Z), ShortcutAction::Redo);
        manager.register(KeyCombo::cmd(egui::Key::F), ShortcutAction::Search);
        manager.register(KeyCombo::cmd(egui::Key::Comma), ShortcutAction::Settings);
        manager.register(KeyCombo::new(egui::Key::Escape), ShortcutAction::Cancel);
        
        manager
    }
    
    pub fn handle(&self, ctx: &egui::Context) -> Option<ShortcutAction> {
        ctx.input(|i| {
            for (combo, action) in &self.shortcuts {
                if i.modifiers == combo.modifiers && i.key_pressed(combo.key) {
                    return Some(action.clone());
                }
            }
            None
        })
    }
}

impl KeyCombo {
    pub fn new(key: egui::Key) -> Self {
        Self { key, modifiers: egui::Modifiers::NONE }
    }
    
    pub fn cmd(key: egui::Key) -> Self {
        Self { key, modifiers: egui::Modifiers::COMMAND }
    }
    
    pub fn cmd_shift(key: egui::Key) -> Self {
        Self { 
            key, 
            modifiers: egui::Modifiers::COMMAND | egui::Modifiers::SHIFT 
        }
    }
}
```

---

## 26.2 Navigation au Clavier

```rust
pub struct FocusManager {
    focusable_items: Vec<FocusableId>,
    current_focus: Option<usize>,
}

impl FocusManager {
    pub fn handle_tab_navigation(&mut self, ctx: &egui::Context) {
        ctx.input(|i| {
            if i.key_pressed(egui::Key::Tab) {
                if i.modifiers.shift {
                    self.focus_previous();
                } else {
                    self.focus_next();
                }
            }
        });
    }
    
    fn focus_next(&mut self) {
        self.current_focus = Some(
            self.current_focus
                .map(|i| (i + 1) % self.focusable_items.len())
                .unwrap_or(0)
        );
    }
    
    fn focus_previous(&mut self) {
        self.current_focus = Some(
            self.current_focus
                .map(|i| i.checked_sub(1).unwrap_or(self.focusable_items.len() - 1))
                .unwrap_or(self.focusable_items.len() - 1)
        );
    }
}
```

---

## 26.3 Contraste et Lisibilité

```rust
pub struct AccessibilityChecker;

impl AccessibilityChecker {
    /// Vérifier le ratio de contraste (WCAG 2.1)
    pub fn contrast_ratio(fg: Color32, bg: Color32) -> f32 {
        let l1 = Self::relative_luminance(fg);
        let l2 = Self::relative_luminance(bg);
        
        let lighter = l1.max(l2);
        let darker = l1.min(l2);
        
        (lighter + 0.05) / (darker + 0.05)
    }
    
    fn relative_luminance(color: Color32) -> f32 {
        let r = Self::linearize(color.r() as f32 / 255.0);
        let g = Self::linearize(color.g() as f32 / 255.0);
        let b = Self::linearize(color.b() as f32 / 255.0);
        
        0.2126 * r + 0.7152 * g + 0.0722 * b
    }
    
    fn linearize(value: f32) -> f32 {
        if value <= 0.03928 {
            value / 12.92
        } else {
            ((value + 0.055) / 1.055).powf(2.4)
        }
    }
    
    /// Vérifier si le contraste est suffisant
    pub fn meets_wcag_aa(fg: Color32, bg: Color32, large_text: bool) -> bool {
        let ratio = Self::contrast_ratio(fg, bg);
        if large_text {
            ratio >= 3.0  // AA pour grand texte
        } else {
            ratio >= 4.5  // AA pour texte normal
        }
    }
    
    pub fn meets_wcag_aaa(fg: Color32, bg: Color32, large_text: bool) -> bool {
        let ratio = Self::contrast_ratio(fg, bg);
        if large_text {
            ratio >= 4.5  // AAA pour grand texte
        } else {
            ratio >= 7.0  // AAA pour texte normal
        }
    }
}
```

---

## 26.4 Tailles et Zones de Clic

```rust
/// Tailles minimales recommandées
pub struct AccessibleSizes {
    /// Zone de clic minimale (44x44 sur mobile, 24x24 desktop)
    pub min_touch_target: f32,
    /// Taille de texte minimale
    pub min_font_size: f32,
    /// Espacement minimal entre éléments interactifs
    pub min_spacing: f32,
}

impl Default for AccessibleSizes {
    fn default() -> Self {
        Self {
            min_touch_target: 32.0,  // Desktop
            min_font_size: 14.0,
            min_spacing: 8.0,
        }
    }
}

/// Wrapper pour garantir une zone de clic minimale
pub fn accessible_button(ui: &mut Ui, text: &str, min_size: f32) -> Response {
    let desired_size = egui::vec2(
        ui.available_width().min(200.0).max(min_size),
        min_size,
    );
    
    ui.add_sized(desired_size, egui::Button::new(text))
}
```

---

## Résumé

- **Raccourcis clavier** : Actions rapides pour power users
- **Navigation Tab** : Parcours logique au clavier
- **Contraste WCAG** : Lisibilité garantie
- **Zones de clic** : Tailles minimales respectées
