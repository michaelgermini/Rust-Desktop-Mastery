# 12.1 Loading States

## Les trois états d'une ressource

```rust
#[derive(Clone)]
pub enum LoadState<T> {
    /// Pas encore chargé
    Idle,
    /// Chargement en cours
    Loading { 
        started_at: Instant,
        message: Option<String>,
    },
    /// Chargé avec succès
    Loaded(T),
    /// Erreur de chargement
    Error { 
        message: String,
        retry_count: u32,
    },
}

impl<T> LoadState<T> {
    pub fn is_loading(&self) -> bool {
        matches!(self, LoadState::Loading { .. })
    }
    
    pub fn as_loaded(&self) -> Option<&T> {
        match self {
            LoadState::Loaded(data) => Some(data),
            _ => None,
        }
    }
}
```

## Skeleton loading

```rust
pub fn render_skeleton_row(ui: &mut Ui, tokens: &DesignTokens) {
    ui.horizontal(|ui| {
        // Avatar skeleton
        let (rect, _) = ui.allocate_exact_size(
            egui::vec2(32.0, 32.0),
            egui::Sense::hover(),
        );
        ui.painter().rect_filled(
            rect,
            tokens.radius.full,
            pulse_color(tokens.colors.bg_tertiary),
        );
        
        ui.add_space(tokens.spacing.sm);
        
        // Text skeletons
        ui.vertical(|ui| {
            skeleton_line(ui, 150.0, tokens);
            ui.add_space(4.0);
            skeleton_line(ui, 100.0, tokens);
        });
    });
}

fn skeleton_line(ui: &mut Ui, width: f32, tokens: &DesignTokens) {
    let (rect, _) = ui.allocate_exact_size(
        egui::vec2(width, 12.0),
        egui::Sense::hover(),
    );
    ui.painter().rect_filled(
        rect,
        tokens.radius.sm,
        pulse_color(tokens.colors.bg_tertiary),
    );
}

fn pulse_color(base: Color32) -> Color32 {
    // Animation de pulsation
    let t = (std::time::SystemTime::now()
        .duration_since(std::time::UNIX_EPOCH)
        .unwrap()
        .as_millis() % 1000) as f32 / 1000.0;
    
    let pulse = (t * std::f32::consts::PI * 2.0).sin() * 0.5 + 0.5;
    ColorTokens::lighten(base, pulse * 0.1)
}
```

## Pattern de chargement complet

```rust
pub fn render_with_loading<T>(
    ui: &mut Ui,
    state: &LoadState<T>,
    tokens: &DesignTokens,
    render_loaded: impl FnOnce(&mut Ui, &T),
    render_skeleton: impl FnOnce(&mut Ui),
) {
    match state {
        LoadState::Idle => {
            ui.centered_and_justified(|ui| {
                ui.label("Prêt à charger");
            });
        }
        
        LoadState::Loading { started_at, message } => {
            ui.vertical_centered(|ui| {
                ui.add_space(tokens.spacing.xl);
                
                // Afficher skeleton si chargement court
                if started_at.elapsed() < Duration::from_millis(300) {
                    render_skeleton(ui);
                } else {
                    // Spinner pour chargement long
                    ui.spinner();
                    if let Some(msg) = message {
                        ui.label(msg);
                    }
                }
            });
            
            // Demander repaint pour animation
            ui.ctx().request_repaint();
        }
        
        LoadState::Loaded(data) => {
            render_loaded(ui, data);
        }
        
        LoadState::Error { message, retry_count } => {
            ui.vertical_centered(|ui| {
                ui.add_space(tokens.spacing.xl);
                
                ui.label(
                    egui::RichText::new("❌")
                        .size(tokens.typography.size_4xl)
                );
                
                ui.label(
                    egui::RichText::new("Erreur de chargement")
                        .size(tokens.typography.size_lg)
                        .color(tokens.colors.error)
                );
                
                ui.label(
                    egui::RichText::new(message)
                        .size(tokens.typography.size_sm)
                        .color(tokens.colors.text_muted)
                );
                
                ui.add_space(tokens.spacing.md);
                
                if Button::new("Réessayer")
                    .variant(ButtonVariant::Primary)
                    .show(ui, tokens)
                    .clicked()
                {
                    // Trigger retry
                }
            });
        }
    }
}
```

## Résumé

- **États clairs** : Idle, Loading, Loaded, Error
- **Skeleton loading** : Pour chargements rapides (<300ms)
- **Spinner** : Pour chargements longs
- **Retry** : Possibilité de réessayer en cas d'erreur
