# 9.2 Layouts

## Vertical et Horizontal

### Layout vertical (par défaut)

```rust
fn render_vertical(ui: &mut egui::Ui) {
    // Vertical (par défaut)
    ui.vertical(|ui| {
        ui.label("Ligne 1");
        ui.label("Ligne 2");
        ui.label("Ligne 3");
    });
}
```

### Layout horizontal

```rust
fn render_horizontal(ui: &mut egui::Ui) {
    // Horizontal
    ui.horizontal(|ui| {
        ui.label("Gauche");
        ui.label("Centre");
        ui.label("Droite");
    });
    
    // Horizontal avec wrapping
    ui.horizontal_wrapped(|ui| {
        for i in 0..20 {
            ui.label(format!("Item {}", i));
        }
    });
}
```

## Centrage et alignement

### Centrage vertical

```rust
fn render_centered(ui: &mut egui::Ui) {
    // Centrer horizontalement
    ui.vertical_centered(|ui| {
        ui.heading("Titre Centré");
        ui.label("Contenu centré aussi");
    });
    
    // Centrer avec justification
    ui.vertical_centered_justified(|ui| {
        if ui.button("Bouton pleine largeur").clicked() {
            // ...
        }
    });
}
```

### Alignement custom

```rust
fn render_custom_alignment(ui: &mut egui::Ui) {
    // Alignement à droite
    ui.with_layout(
        egui::Layout::right_to_left(egui::Align::Center),
        |ui| {
            ui.label("Aligné à droite");
            ui.button("Action");
        }
    );
    
    // Alignement en bas
    ui.with_layout(
        egui::Layout::bottom_up(egui::Align::Center),
        |ui| {
            ui.button("En bas");
        }
    );
}
```

## Grilles

### Grid simple

```rust
fn render_grid(ui: &mut egui::Ui) {
    egui::Grid::new("my_grid")
        .num_columns(3)
        .spacing([20.0, 10.0])
        .striped(true)
        .show(ui, |ui| {
            // Ligne 1
            ui.label("Nom");
            ui.label("Age");
            ui.label("Email");
            ui.end_row();
            
            // Ligne 2
            ui.label("Alice");
            ui.label("25");
            ui.label("alice@example.com");
            ui.end_row();
        });
}
```

### Grid avec colonnes de largeur variable

```rust
fn render_flexible_grid(ui: &mut egui::Ui) {
    egui::Grid::new("flexible_grid")
        .num_columns(2)
        .spacing([40.0, 4.0])
        .show(ui, |ui| {
            // Colonne 1 : label fixe
            ui.label("Nom:");
            // Colonne 2 : input prend l'espace restant
            ui.text_edit_singleline(&mut self.name);
            ui.end_row();
            
            ui.label("Email:");
            ui.text_edit_singleline(&mut self.email);
            ui.end_row();
        });
}
```

## Panels et zones

### SidePanel (panneau latéral)

```rust
fn render_side_panels(ctx: &egui::Context) {
    // Panel latéral gauche
    egui::SidePanel::left("left_panel")
        .default_width(200.0)
        .resizable(true)
        .show(ctx, |ui| {
            ui.heading("Navigation");
            if ui.button("Dashboard").clicked() {}
            if ui.button("Settings").clicked() {}
        });
    
    // Panel latéral droit
    egui::SidePanel::right("right_panel")
        .default_width(300.0)
        .show(ctx, |ui| {
            ui.heading("Propriétés");
            // ...
        });
}
```

### TopBottomPanel (menu bar, status bar)

```rust
fn render_top_bottom_panels(ctx: &egui::Context) {
    // Panel du haut (menu bar)
    egui::TopBottomPanel::top("top_panel").show(ctx, |ui| {
        egui::menu::bar(ui, |ui| {
            ui.menu_button("Fichier", |ui| {
                if ui.button("Nouveau").clicked() { /* ... */ }
                if ui.button("Ouvrir").clicked() { /* ... */ }
                ui.separator();
                if ui.button("Quitter").clicked() { /* ... */ }
            });
            
            ui.menu_button("Édition", |ui| {
                if ui.button("Annuler").clicked() { /* ... */ }
                if ui.button("Refaire").clicked() { /* ... */ }
            });
        });
    });
    
    // Panel du bas (status bar)
    egui::TopBottomPanel::bottom("bottom_panel").show(ctx, |ui| {
        ui.horizontal(|ui| {
            ui.label("Prêt");
            ui.separator();
            ui.label("Ligne 1, Colonne 1");
        });
    });
}
```

### CentralPanel (contenu principal)

```rust
fn render_central_panel(ctx: &egui::Context) {
    // Panel central (ce qui reste après les autres panels)
    egui::CentralPanel::default().show(ctx, |ui| {
        ui.heading("Contenu principal");
        // Le reste de votre UI...
    });
}
```

## ScrollArea

### Scroll vertical

```rust
fn render_scroll(ui: &mut egui::Ui, items: &[String]) {
    // Scroll vertical
    egui::ScrollArea::vertical()
        .max_height(300.0)
        .auto_shrink([false, false])
        .show(ui, |ui| {
            for item in items {
                ui.label(item);
            }
        });
}
```

### Scroll avec stick to bottom

```rust
fn render_logs(ui: &mut egui::Ui, logs: &[String]) {
    // Scroll avec stick to bottom (pour logs)
    egui::ScrollArea::vertical()
        .stick_to_bottom(true)
        .show(ui, |ui| {
            for log in logs {
                ui.label(log);
            }
        });
}
```

### Scroll horizontal et vertical

```rust
fn render_both_scroll(ui: &mut egui::Ui) {
    // Scroll horizontal et vertical
    egui::ScrollArea::both()
        .show(ui, |ui| {
            // Grand contenu qui dépasse dans les deux directions...
        });
}
```

## Résumé

- **Vertical/Horizontal** : Layouts de base
- **Grid** : Tableaux et formulaires
- **Panels** : Structure de l'application (sidebar, menu, content)
- **ScrollArea** : Contenu qui dépasse
- **Centrage** : Alignement avec `vertical_centered` ou `with_layout`
