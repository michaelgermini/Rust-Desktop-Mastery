# Chapitre 11 : Composants Réutilisables

## Introduction

Ce chapitre présente une bibliothèque de composants UI réutilisables construits avec notre design system. Chaque composant est conçu pour être facile à utiliser, customisable et conforme aux standards UX modernes.

---

## 11.1 Button

### Implémentation complète

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
        let (padding, font_size) = match self.size {
            ButtonSize::Small => (egui::vec2(8.0, 4.0), tokens.typography.size_sm),
            ButtonSize::Medium => (egui::vec2(16.0, 8.0), tokens.typography.size_md),
            ButtonSize::Large => (egui::vec2(24.0, 12.0), tokens.typography.size_lg),
        };
        
        let (bg_color, text_color, hover_color) = match self.variant {
            ButtonVariant::Primary => (
                tokens.colors.primary,
                tokens.colors.text_inverse,
                tokens.colors.primary_hover,
            ),
            ButtonVariant::Secondary => (
                tokens.colors.bg_tertiary,
                tokens.colors.text_primary,
                tokens.colors.border_strong,
            ),
            ButtonVariant::Ghost => (
                Color32::TRANSPARENT,
                tokens.colors.text_primary,
                tokens.colors.bg_tertiary,
            ),
            ButtonVariant::Danger => (
                tokens.colors.error,
                tokens.colors.text_inverse,
                ColorTokens::darken(tokens.colors.error, 0.1),
            ),
        };
        
        let is_interactive = !self.disabled && !self.loading;
        
        // Calculer la taille
        let text_to_show = if self.loading { "..." } else { self.text };
        let galley = ui.fonts(|f| {
            f.layout_no_wrap(
                text_to_show.to_string(),
                egui::FontId::proportional(font_size),
                text_color,
            )
        });
        
        let icon_width = self.icon.map(|_| font_size + 4.0).unwrap_or(0.0);
        let desired_size = egui::vec2(
            if self.full_width { ui.available_width() } else { galley.size().x + icon_width + padding.x * 2.0 },
            galley.size().y + padding.y * 2.0,
        );
        
        let (rect, response) = ui.allocate_exact_size(desired_size, egui::Sense::click());
        
        if ui.is_rect_visible(rect) {
            let visuals = if !is_interactive {
                ui.style().visuals.widgets.inactive
            } else if response.hovered() {
                ui.style().visuals.widgets.hovered
            } else {
                ui.style().visuals.widgets.inactive
            };
            
            let bg = if self.disabled {
                ColorTokens::with_alpha(bg_color, 100)
            } else if response.hovered() {
                hover_color
            } else {
                bg_color
            };
            
            // Dessiner le fond
            ui.painter().rect_filled(
                rect,
                tokens.radius.sm,
                bg,
            );
            
            // Dessiner le contenu
            let content_rect = rect.shrink2(padding);
            
            if self.loading {
                // Spinner de chargement
                let spinner_rect = egui::Rect::from_center_size(
                    content_rect.center(),
                    egui::vec2(font_size, font_size),
                );
                egui::Spinner::new().paint_at(ui, spinner_rect);
            } else {
                let mut x = content_rect.left();
                
                // Icône
                if let Some(icon) = self.icon {
                    ui.painter().text(
                        egui::pos2(x, content_rect.center().y),
                        egui::Align2::LEFT_CENTER,
                        icon,
                        egui::FontId::proportional(font_size),
                        if self.disabled { ColorTokens::with_alpha(text_color, 100) } else { text_color },
                    );
                    x += font_size + 4.0;
                }
                
                // Texte
                ui.painter().text(
                    egui::pos2(x, content_rect.center().y),
                    egui::Align2::LEFT_CENTER,
                    self.text,
                    egui::FontId::proportional(font_size),
                    if self.disabled { ColorTokens::with_alpha(text_color, 100) } else { text_color },
                );
            }
        }
        
        if is_interactive { response } else { response.clone() }
    }
}

// Usage
// if Button::new("Sauvegarder")
//     .icon("💾")
//     .variant(ButtonVariant::Primary)
//     .show(ui, &tokens)
//     .clicked()
// {
//     save_document();
// }
```

---

## 11.2 Input

```rust
pub struct TextInput<'a> {
    value: &'a mut String,
    placeholder: &'a str,
    label: Option<&'a str>,
    error: Option<&'a str>,
    disabled: bool,
    password: bool,
}

impl<'a> TextInput<'a> {
    pub fn new(value: &'a mut String) -> Self {
        Self {
            value,
            placeholder: "",
            label: None,
            error: None,
            disabled: false,
            password: false,
        }
    }
    
    pub fn placeholder(mut self, placeholder: &'a str) -> Self {
        self.placeholder = placeholder;
        self
    }
    
    pub fn label(mut self, label: &'a str) -> Self {
        self.label = Some(label);
        self
    }
    
    pub fn error(mut self, error: &'a str) -> Self {
        self.error = Some(error);
        self
    }
    
    pub fn password(mut self) -> Self {
        self.password = true;
        self
    }
    
    pub fn show(self, ui: &mut Ui, tokens: &DesignTokens) -> Response {
        ui.vertical(|ui| {
            // Label
            if let Some(label) = self.label {
                ui.label(
                    egui::RichText::new(label)
                        .size(tokens.typography.size_sm)
                        .color(tokens.colors.text_secondary)
                );
                ui.add_space(tokens.spacing.xs);
            }
            
            // Input field
            let border_color = if self.error.is_some() {
                tokens.colors.error
            } else {
                tokens.colors.border_default
            };
            
            let response = egui::Frame::none()
                .fill(tokens.colors.bg_primary)
                .stroke(egui::Stroke::new(1.0, border_color))
                .rounding(tokens.radius.sm)
                .inner_margin(egui::Margin::symmetric(tokens.spacing.sm, tokens.spacing.xs))
                .show(ui, |ui| {
                    let mut text_edit = egui::TextEdit::singleline(self.value)
                        .hint_text(self.placeholder)
                        .frame(false);
                    
                    if self.password {
                        text_edit = text_edit.password(true);
                    }
                    
                    ui.add_enabled(!self.disabled, text_edit)
                })
                .inner;
            
            // Error message
            if let Some(error) = self.error {
                ui.add_space(tokens.spacing.xs);
                ui.label(
                    egui::RichText::new(error)
                        .size(tokens.typography.size_xs)
                        .color(tokens.colors.error)
                );
            }
            
            response
        })
        .inner
    }
}
```

---

## 11.3 Table

```rust
pub struct Table<'a, T> {
    items: &'a [T],
    columns: Vec<TableColumn<'a, T>>,
    selected: Option<&'a mut Option<usize>>,
    striped: bool,
}

pub struct TableColumn<'a, T> {
    header: &'a str,
    width: Option<f32>,
    render: Box<dyn Fn(&T, &mut Ui, &DesignTokens) + 'a>,
}

impl<'a, T> Table<'a, T> {
    pub fn new(items: &'a [T]) -> Self {
        Self {
            items,
            columns: Vec::new(),
            selected: None,
            striped: true,
        }
    }
    
    pub fn column(
        mut self,
        header: &'a str,
        render: impl Fn(&T, &mut Ui, &DesignTokens) + 'a,
    ) -> Self {
        self.columns.push(TableColumn {
            header,
            width: None,
            render: Box::new(render),
        });
        self
    }
    
    pub fn column_with_width(
        mut self,
        header: &'a str,
        width: f32,
        render: impl Fn(&T, &mut Ui, &DesignTokens) + 'a,
    ) -> Self {
        self.columns.push(TableColumn {
            header,
            width: Some(width),
            render: Box::new(render),
        });
        self
    }
    
    pub fn selectable(mut self, selected: &'a mut Option<usize>) -> Self {
        self.selected = Some(selected);
        self
    }
    
    pub fn show(self, ui: &mut Ui, tokens: &DesignTokens) -> Option<usize> {
        let mut clicked_row = None;
        
        egui::Frame::none()
            .fill(tokens.colors.bg_primary)
            .stroke(egui::Stroke::new(1.0, tokens.colors.border_default))
            .rounding(tokens.radius.sm)
            .show(ui, |ui| {
                egui::ScrollArea::horizontal().show(ui, |ui| {
                    egui::Grid::new("table_grid")
                        .num_columns(self.columns.len())
                        .striped(self.striped)
                        .show(ui, |ui| {
                            // Header
                            for col in &self.columns {
                                ui.label(
                                    egui::RichText::new(col.header)
                                        .strong()
                                        .color(tokens.colors.text_secondary)
                                        .size(tokens.typography.size_sm)
                                );
                            }
                            ui.end_row();
                            
                            // Rows
                            for (idx, item) in self.items.iter().enumerate() {
                                let is_selected = self.selected
                                    .as_ref()
                                    .map(|s| **s == Some(idx))
                                    .unwrap_or(false);
                                
                                for col in &self.columns {
                                    let response = ui.horizontal(|ui| {
                                        if is_selected {
                                            ui.visuals_mut().override_text_color = 
                                                Some(tokens.colors.primary);
                                        }
                                        (col.render)(item, ui, tokens);
                                    });
                                    
                                    if response.response.interact(egui::Sense::click()).clicked() {
                                        clicked_row = Some(idx);
                                    }
                                }
                                ui.end_row();
                            }
                        });
                });
            });
        
        // Mettre à jour la sélection
        if let (Some(selected), Some(idx)) = (self.selected, clicked_row) {
            *selected = Some(idx);
        }
        
        clicked_row
    }
}

// Usage:
// Table::new(&invoices)
//     .column("N°", |inv, ui, _| { ui.label(&inv.number); })
//     .column("Client", |inv, ui, _| { ui.label(&inv.client_name); })
//     .column_with_width("Montant", 100.0, |inv, ui, tokens| {
//         ui.label(
//             egui::RichText::new(format!("{:.2} €", inv.total))
//                 .color(tokens.colors.success)
//         );
//     })
//     .selectable(&mut selected_invoice)
//     .show(ui, &tokens);
```

---

## 11.4 Sidebar

```rust
pub struct Sidebar<'a> {
    items: Vec<SidebarItem<'a>>,
    selected: &'a mut usize,
    collapsed: bool,
    width: f32,
}

pub struct SidebarItem<'a> {
    icon: &'a str,
    label: &'a str,
    badge: Option<usize>,
}

impl<'a> Sidebar<'a> {
    pub fn new(selected: &'a mut usize) -> Self {
        Self {
            items: Vec::new(),
            selected,
            collapsed: false,
            width: 240.0,
        }
    }
    
    pub fn item(mut self, icon: &'a str, label: &'a str) -> Self {
        self.items.push(SidebarItem { icon, label, badge: None });
        self
    }
    
    pub fn item_with_badge(mut self, icon: &'a str, label: &'a str, badge: usize) -> Self {
        self.items.push(SidebarItem { icon, label, badge: Some(badge) });
        self
    }
    
    pub fn collapsed(mut self, collapsed: bool) -> Self {
        self.collapsed = collapsed;
        self
    }
    
    pub fn show(self, ctx: &egui::Context, tokens: &DesignTokens) {
        let width = if self.collapsed { 60.0 } else { self.width };
        
        egui::SidePanel::left("sidebar")
            .exact_width(width)
            .resizable(false)
            .frame(egui::Frame::none().fill(tokens.colors.bg_secondary))
            .show(ctx, |ui| {
                ui.add_space(tokens.spacing.md);
                
                for (idx, item) in self.items.iter().enumerate() {
                    let is_selected = *self.selected == idx;
                    
                    let response = ui.horizontal(|ui| {
                        // Indicateur de sélection
                        if is_selected {
                            let rect = ui.available_rect_before_wrap();
                            ui.painter().rect_filled(
                                egui::Rect::from_min_size(
                                    rect.min,
                                    egui::vec2(3.0, 32.0),
                                ),
                                0.0,
                                tokens.colors.primary,
                            );
                        }
                        
                        ui.add_space(tokens.spacing.sm);
                        
                        // Icône
                        ui.label(
                            egui::RichText::new(item.icon)
                                .size(tokens.typography.size_lg)
                                .color(if is_selected { 
                                    tokens.colors.primary 
                                } else { 
                                    tokens.colors.text_secondary 
                                })
                        );
                        
                        // Label (si non collapsed)
                        if !self.collapsed {
                            ui.add_space(tokens.spacing.sm);
                            ui.label(
                                egui::RichText::new(item.label)
                                    .size(tokens.typography.size_md)
                                    .color(if is_selected { 
                                        tokens.colors.text_primary 
                                    } else { 
                                        tokens.colors.text_secondary 
                                    })
                            );
                            
                            // Badge
                            if let Some(count) = item.badge {
                                ui.with_layout(
                                    egui::Layout::right_to_left(egui::Align::Center),
                                    |ui| {
                                        Badge::new(&count.to_string())
                                            .variant(BadgeVariant::Primary)
                                            .show(ui, tokens);
                                    }
                                );
                            }
                        }
                    });
                    
                    if response.response.interact(egui::Sense::click()).clicked() {
                        *self.selected = idx;
                    }
                    
                    ui.add_space(tokens.spacing.xs);
                }
            });
    }
}
```

---

## 11.5 Modal

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
        if !*self.open {
            return;
        }
        
        // Overlay sombre
        egui::Area::new("modal_overlay".into())
            .fixed_pos(egui::pos2(0.0, 0.0))
            .show(ctx, |ui| {
                let screen_rect = ctx.screen_rect();
                ui.painter().rect_filled(
                    screen_rect,
                    0.0,
                    Color32::from_rgba_unmultiplied(0, 0, 0, 150),
                );
                
                // Fermer en cliquant sur l'overlay
                if ui.interact(screen_rect, "overlay".into(), egui::Sense::click()).clicked() {
                    *self.open = false;
                }
            });
        
        // Fenêtre modale
        egui::Window::new(self.title)
            .collapsible(false)
            .resizable(false)
            .fixed_size([self.width, 0.0])
            .anchor(egui::Align2::CENTER_CENTER, [0.0, 0.0])
            .frame(egui::Frame::window(&ctx.style())
                .fill(tokens.colors.bg_elevated)
                .shadow(egui::epaint::Shadow {
                    offset: tokens.shadows.xl.offset,
                    blur: tokens.shadows.xl.blur,
                    spread: tokens.shadows.xl.spread,
                    color: tokens.shadows.xl.color,
                })
            )
            .show(ctx, |ui| {
                content(ui, tokens);
            });
        
        // Fermer avec Escape
        if ctx.input(|i| i.key_pressed(egui::Key::Escape)) {
            *self.open = false;
        }
    }
}

// Usage:
// Modal::new("Confirmation", &mut show_confirm)
//     .show(ctx, &tokens, |ui, tokens| {
//         ui.label("Êtes-vous sûr de vouloir supprimer ?");
//         ui.add_space(tokens.spacing.md);
//         ui.horizontal(|ui| {
//             if Button::new("Annuler")
//                 .variant(ButtonVariant::Ghost)
//                 .show(ui, tokens)
//                 .clicked()
//             {
//                 show_confirm = false;
//             }
//             if Button::new("Supprimer")
//                 .variant(ButtonVariant::Danger)
//                 .show(ui, tokens)
//                 .clicked()
//             {
//                 delete_item();
//                 show_confirm = false;
//             }
//         });
//     });
```

---

## 11.6 Toast

```rust
#[derive(Clone)]
pub enum ToastLevel {
    Info,
    Success,
    Warning,
    Error,
}

#[derive(Clone)]
pub struct Toast {
    pub id: u64,
    pub message: String,
    pub level: ToastLevel,
    pub created_at: std::time::Instant,
    pub duration: std::time::Duration,
}

pub struct ToastManager {
    toasts: Vec<Toast>,
    next_id: u64,
}

impl ToastManager {
    pub fn new() -> Self {
        Self {
            toasts: Vec::new(),
            next_id: 0,
        }
    }
    
    pub fn show(&mut self, message: impl Into<String>, level: ToastLevel) {
        self.toasts.push(Toast {
            id: self.next_id,
            message: message.into(),
            level,
            created_at: std::time::Instant::now(),
            duration: std::time::Duration::from_secs(5),
        });
        self.next_id += 1;
    }
    
    pub fn success(&mut self, message: impl Into<String>) {
        self.show(message, ToastLevel::Success);
    }
    
    pub fn error(&mut self, message: impl Into<String>) {
        self.show(message, ToastLevel::Error);
    }
    
    pub fn render(&mut self, ctx: &egui::Context, tokens: &DesignTokens) {
        // Supprimer les toasts expirés
        self.toasts.retain(|t| t.created_at.elapsed() < t.duration);
        
        if self.toasts.is_empty() {
            return;
        }
        
        egui::Area::new("toasts".into())
            .anchor(egui::Align2::RIGHT_BOTTOM, [-20.0, -20.0])
            .show(ctx, |ui| {
                ui.vertical(|ui| {
                    for toast in &self.toasts {
                        let (bg_color, icon) = match toast.level {
                            ToastLevel::Info => (tokens.colors.info, "ℹ️"),
                            ToastLevel::Success => (tokens.colors.success, "✅"),
                            ToastLevel::Warning => (tokens.colors.warning, "⚠️"),
                            ToastLevel::Error => (tokens.colors.error, "❌"),
                        };
                        
                        egui::Frame::none()
                            .fill(tokens.colors.bg_elevated)
                            .stroke(egui::Stroke::new(1.0, bg_color))
                            .rounding(tokens.radius.md)
                            .inner_margin(tokens.spacing.md)
                            .shadow(egui::epaint::Shadow {
                                offset: tokens.shadows.md.offset,
                                blur: tokens.shadows.md.blur,
                                spread: tokens.shadows.md.spread,
                                color: tokens.shadows.md.color,
                            })
                            .show(ui, |ui| {
                                ui.horizontal(|ui| {
                                    ui.label(icon);
                                    ui.label(&toast.message);
                                });
                            });
                        
                        ui.add_space(tokens.spacing.sm);
                    }
                });
            });
        
        // Demander un repaint pour l'animation de disparition
        ctx.request_repaint_after(std::time::Duration::from_millis(100));
    }
}
```

---

## 11.7 Command Palette

```rust
pub struct CommandPalette<'a> {
    open: &'a mut bool,
    query: &'a mut String,
    commands: Vec<Command<'a>>,
    selected_index: usize,
}

pub struct Command<'a> {
    pub id: &'a str,
    pub label: &'a str,
    pub shortcut: Option<&'a str>,
    pub icon: Option<&'a str>,
}

impl<'a> CommandPalette<'a> {
    pub fn new(open: &'a mut bool, query: &'a mut String) -> Self {
        Self {
            open,
            query,
            commands: Vec::new(),
            selected_index: 0,
        }
    }
    
    pub fn command(mut self, id: &'a str, label: &'a str) -> Self {
        self.commands.push(Command {
            id,
            label,
            shortcut: None,
            icon: None,
        });
        self
    }
    
    pub fn command_with_shortcut(
        mut self,
        id: &'a str,
        label: &'a str,
        shortcut: &'a str,
    ) -> Self {
        self.commands.push(Command {
            id,
            label,
            shortcut: Some(shortcut),
            icon: None,
        });
        self
    }
    
    pub fn show(mut self, ctx: &egui::Context, tokens: &DesignTokens) -> Option<&'a str> {
        if !*self.open {
            return None;
        }
        
        let mut executed_command = None;
        
        // Filtrer les commandes
        let filtered: Vec<_> = self.commands.iter()
            .filter(|c| {
                self.query.is_empty() || 
                c.label.to_lowercase().contains(&self.query.to_lowercase())
            })
            .collect();
        
        // Gérer la navigation clavier
        ctx.input(|i| {
            if i.key_pressed(egui::Key::ArrowDown) {
                self.selected_index = (self.selected_index + 1).min(filtered.len().saturating_sub(1));
            }
            if i.key_pressed(egui::Key::ArrowUp) {
                self.selected_index = self.selected_index.saturating_sub(1);
            }
            if i.key_pressed(egui::Key::Enter) && !filtered.is_empty() {
                executed_command = Some(filtered[self.selected_index].id);
                *self.open = false;
            }
            if i.key_pressed(egui::Key::Escape) {
                *self.open = false;
            }
        });
        
        // Overlay
        egui::Area::new("command_palette_overlay".into())
            .fixed_pos(egui::pos2(0.0, 0.0))
            .show(ctx, |ui| {
                let screen_rect = ctx.screen_rect();
                ui.painter().rect_filled(
                    screen_rect,
                    0.0,
                    Color32::from_rgba_unmultiplied(0, 0, 0, 100),
                );
            });
        
        // Palette
        egui::Window::new("Command Palette")
            .title_bar(false)
            .resizable(false)
            .fixed_size([500.0, 0.0])
            .anchor(egui::Align2::CENTER_TOP, [0.0, 100.0])
            .frame(egui::Frame::window(&ctx.style())
                .fill(tokens.colors.bg_elevated)
                .rounding(tokens.radius.lg)
            )
            .show(ctx, |ui| {
                // Search input
                let response = ui.add(
                    egui::TextEdit::singleline(self.query)
                        .hint_text("Rechercher une commande...")
                        .frame(false)
                        .font(egui::FontId::proportional(tokens.typography.size_lg))
                );
                response.request_focus();
                
                ui.separator();
                
                // Liste des commandes
                egui::ScrollArea::vertical()
                    .max_height(300.0)
                    .show(ui, |ui| {
                        for (idx, cmd) in filtered.iter().enumerate() {
                            let is_selected = idx == self.selected_index;
                            
                            let response = ui.horizontal(|ui| {
                                if is_selected {
                                    ui.painter().rect_filled(
                                        ui.available_rect_before_wrap(),
                                        tokens.radius.sm,
                                        tokens.colors.primary_muted,
                                    );
                                }
                                
                                ui.add_space(tokens.spacing.sm);
                                
                                if let Some(icon) = cmd.icon {
                                    ui.label(icon);
                                }
                                
                                ui.label(&cmd.label);
                                
                                if let Some(shortcut) = cmd.shortcut {
                                    ui.with_layout(
                                        egui::Layout::right_to_left(egui::Align::Center),
                                        |ui| {
                                            ui.label(
                                                egui::RichText::new(shortcut)
                                                    .size(tokens.typography.size_xs)
                                                    .color(tokens.colors.text_muted)
                                            );
                                        }
                                    );
                                }
                            });
                            
                            if response.response.interact(egui::Sense::click()).clicked() {
                                executed_command = Some(cmd.id);
                                *self.open = false;
                            }
                        }
                    });
            });
        
        executed_command
    }
}
```

---

## Résumé du Chapitre

### Bibliothèque de composants

| Composant | Usage |
|-----------|-------|
| `Button` | Actions utilisateur |
| `TextInput` | Saisie de texte |
| `Table` | Affichage de données |
| `Sidebar` | Navigation principale |
| `Modal` | Dialogues et confirmations |
| `Toast` | Notifications temporaires |
| `CommandPalette` | Accès rapide aux actions |

### Patterns de conception

1. **Builder pattern** : Configuration fluide
2. **Separation of concerns** : Logique vs rendu
3. **Design tokens** : Cohérence visuelle
4. **Accessibilité** : Navigation clavier

---

**Dans le prochain chapitre, nous explorerons les patterns UX avancés pour une expérience utilisateur professionnelle.**
