# 25.1 Skeleton Loading

## Skeleton loading

Le skeleton loading affiche une structure similaire au contenu final pendant le chargement, donnant l'impression que l'application est rapide.

```rust
pub fn render_with_skeleton<T>(
    ui: &mut Ui,
    state: &LoadState<T>,
    tokens: &DesignTokens,
    render_skeleton: impl FnOnce(&mut Ui),
    render_content: impl FnOnce(&mut Ui, &T),
) {
    match state {
        LoadState::Loading { started_at } => {
            // Afficher skeleton seulement après un court délai
            if started_at.elapsed() > Duration::from_millis(100) {
                render_skeleton(ui);
                ui.ctx().request_repaint();
            }
        }
        LoadState::Loaded(data) => {
            render_content(ui, data);
        }
        LoadState::Error(msg) => {
            ui.colored_label(tokens.colors.error, msg);
        }
        LoadState::Idle => {}
    }
}
```

## Placeholders animés

```rust
pub fn skeleton_list(ui: &mut Ui, count: usize, tokens: &DesignTokens) {
    for _ in 0..count {
        skeleton_row(ui, tokens);
        ui.add_space(tokens.spacing.xs);
    }
}

pub fn skeleton_row(ui: &mut Ui, tokens: &DesignTokens) {
    ui.horizontal(|ui| {
        // Avatar skeleton
        skeleton_circle(ui, 32.0, tokens);
        ui.add_space(tokens.spacing.sm);
        
        // Text skeletons
        ui.vertical(|ui| {
            skeleton_rect(ui, 120.0, 14.0, tokens);
            ui.add_space(4.0);
            skeleton_rect(ui, 80.0, 12.0, tokens);
        });
    });
}

fn skeleton_rect(ui: &mut Ui, width: f32, height: f32, tokens: &DesignTokens) {
    let (rect, _) = ui.allocate_exact_size(egui::vec2(width, height), egui::Sense::hover());
    
    // Animation de pulsation
    let t = ui.ctx().input(|i| i.time) as f32;
    let alpha = 0.6 + 0.4 * (t * 2.0).sin();
    
    ui.painter().rect_filled(
        rect,
        tokens.radius.sm,
        tokens.colors.bg_tertiary.linear_multiply(alpha),
    );
}

fn skeleton_circle(ui: &mut Ui, size: f32, tokens: &DesignTokens) {
    let (rect, _) = ui.allocate_exact_size(egui::vec2(size, size), egui::Sense::hover());
    
    let t = ui.ctx().input(|i| i.time) as f32;
    let alpha = 0.6 + 0.4 * (t * 2.0).sin();
    
    ui.painter().circle_filled(
        rect.center(),
        size / 2.0,
        tokens.colors.bg_tertiary.linear_multiply(alpha),
    );
}
```

## Feedback visuel

```rust
pub fn render_loading_with_feedback(
    ui: &mut Ui,
    state: &LoadState<Data>,
    tokens: &DesignTokens,
) {
    match state {
        LoadState::Loading { started_at } => {
            let elapsed = started_at.elapsed();
            
            if elapsed < Duration::from_millis(100) {
                // Trop rapide, ne rien afficher
            } else if elapsed < Duration::from_millis(500) {
                // Skeleton pour chargement court
                render_skeleton(ui, tokens);
            } else {
                // Spinner pour chargement long
                ui.vertical_centered(|ui| {
                    ui.spinner();
                    ui.label(format!("Chargement... ({:.0}s)", elapsed.as_secs_f32()));
                });
            }
            
            ui.ctx().request_repaint();
        }
        LoadState::Loaded(data) => {
            render_content(ui, data);
        }
        _ => {}
    }
}
```

## Résumé

- **Skeleton** : Structure similaire au contenu final
- **Animation** : Pulsation pour indiquer le chargement
- **Délai** : Afficher seulement si chargement > 100ms
- **Feedback** : Message de progression pour chargements longs
