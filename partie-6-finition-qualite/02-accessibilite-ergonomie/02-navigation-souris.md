# 26.2 Navigation Souris

## Navigation au clavier

```rust
pub struct FocusManager {
    focusable_items: Vec<FocusableId>,
    current_focus: Option<usize>,
}

#[derive(Clone, Copy, PartialEq, Eq)]
pub struct FocusableId(usize);

impl FocusManager {
    pub fn new() -> Self {
        Self {
            focusable_items: Vec::new(),
            current_focus: None,
        }
    }
    
    pub fn register_focusable(&mut self) -> FocusableId {
        let id = FocusableId(self.focusable_items.len());
        self.focusable_items.push(id);
        id
    }
    
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
    
    pub fn is_focused(&self, id: FocusableId) -> bool {
        self.current_focus == Some(id.0)
    }
}
```

## Focus management

```rust
pub fn render_focusable_input(
    ui: &mut Ui,
    focus_manager: &FocusManager,
    id: FocusableId,
    value: &mut String,
) -> Response {
    let is_focused = focus_manager.is_focused(id);
    
    let response = ui.add(
        egui::TextEdit::singleline(value)
            .desired_width(200.0)
    );
    
    if is_focused {
        response.request_focus();
    }
    
    response
}
```

## Tab order

```rust
impl App {
    fn render_with_tab_order(&mut self, ui: &mut Ui) {
        // L'ordre dans le code définit l'ordre de tabulation
        let id1 = self.focus_manager.register_focusable();
        render_focusable_input(ui, &self.focus_manager, id1, &mut self.field1);
        
        let id2 = self.focus_manager.register_focusable();
        render_focusable_input(ui, &self.focus_manager, id2, &mut self.field2);
        
        let id3 = self.focus_manager.register_focusable();
        if Button::new("Submit").show(ui).clicked() {
            // ...
        }
        
        // Gérer Tab/Shift+Tab
        self.focus_manager.handle_tab_navigation(ui.ctx());
    }
}
```

## Résumé

- **Tab navigation** : Parcours logique avec Tab/Shift+Tab
- **Focus management** : Suivi de l'élément actif
- **Tab order** : Ordre déterminé par l'ordre dans le code
- **Accessibilité** : Navigation complète au clavier
