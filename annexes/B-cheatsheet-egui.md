# Annexe B : Cheatsheet egui

Référence rapide des widgets et patterns egui les plus utilisés.

---

## Widgets de Base

### Labels et Texte

```rust
// Label simple
ui.label("Texte");

// Label coloré
ui.colored_label(Color32::RED, "Erreur");

// Texte fort (gras)
ui.strong("Important");

// Texte faible (grisé)
ui.weak("Secondaire");

// Texte monospace
ui.monospace("code");

// Heading
ui.heading("Titre");

// Petit texte
ui.small("Petit texte");
```

### Boutons

```rust
// Bouton simple
if ui.button("Cliquer").clicked() {
    // action
}

// Bouton avec taille
ui.add_sized([100.0, 40.0], egui::Button::new("Grand"));

// Bouton désactivé
ui.add_enabled(false, egui::Button::new("Désactivé"));

// Lien hypertexte
if ui.link("Voir plus").clicked() {
    // action
}
```

### Champs de Saisie

```rust
// Texte single line
ui.text_edit_singleline(&mut my_string);

// Texte multiline
ui.text_edit_multiline(&mut my_string);

// Avec placeholder
ui.add(egui::TextEdit::singleline(&mut my_string)
    .hint_text("Placeholder"));

// Mot de passe
ui.add(egui::TextEdit::singleline(&mut password)
    .password(true));

// Largeur personnalisée
ui.add(egui::TextEdit::singleline(&mut text)
    .desired_width(200.0));
```

### Sélecteurs

```rust
// Checkbox
ui.checkbox(&mut my_bool, "Option");

// Radio buttons
ui.radio_value(&mut selected, Value::A, "Option A");
ui.radio_value(&mut selected, Value::B, "Option B");

// Combo box / Dropdown
egui::ComboBox::from_label("Choisir")
    .selected_text(&selected)
    .show_ui(ui, |ui| {
        ui.selectable_value(&mut selected, "A", "Option A");
        ui.selectable_value(&mut selected, "B", "Option B");
    });

// Slider
ui.add(egui::Slider::new(&mut value, 0.0..=100.0));

// Slider avec texte
ui.add(egui::Slider::new(&mut value, 0..=100).text("Volume"));
```

---

## Layouts

### Horizontal

```rust
ui.horizontal(|ui| {
    ui.label("Label");
    ui.text_edit_singleline(&mut text);
    if ui.button("OK").clicked() {}
});
```

### Vertical

```rust
ui.vertical(|ui| {
    ui.label("Ligne 1");
    ui.label("Ligne 2");
    ui.label("Ligne 3");
});
```

### Centré

```rust
ui.vertical_centered(|ui| {
    ui.heading("Centré");
});

ui.horizontal_centered(|ui| {
    ui.label("Centré horizontalement");
});
```

### Alignement

```rust
ui.with_layout(egui::Layout::right_to_left(egui::Align::Center), |ui| {
    ui.button("À droite");
});

ui.with_layout(egui::Layout::top_down(egui::Align::Center), |ui| {
    ui.label("Centré verticalement");
});
```

### Espacement

```rust
// Espace fixe
ui.add_space(16.0);

// Séparateur
ui.separator();

// Remplir l'espace disponible
ui.allocate_space(ui.available_size());
```

---

## Conteneurs

### Frame (cadre avec style)

```rust
egui::Frame::none()
    .fill(Color32::from_gray(30))
    .rounding(8.0)
    .inner_margin(16.0)
    .stroke(egui::Stroke::new(1.0, Color32::GRAY))
    .show(ui, |ui| {
        ui.label("Contenu");
    });
```

### ScrollArea

```rust
egui::ScrollArea::vertical()
    .max_height(300.0)
    .show(ui, |ui| {
        for i in 0..100 {
            ui.label(format!("Ligne {}", i));
        }
    });

// Horizontal
egui::ScrollArea::horizontal().show(ui, |ui| {
    ui.horizontal(|ui| {
        for i in 0..50 {
            ui.button(format!("Btn {}", i));
        }
    });
});
```

### Collapsible (pliable)

```rust
ui.collapsing("Voir plus", |ui| {
    ui.label("Contenu caché");
});

// Avec état contrôlé
egui::CollapsingHeader::new("Section")
    .default_open(true)
    .show(ui, |ui| {
        ui.label("Contenu");
    });
```

### Window

```rust
egui::Window::new("Ma fenêtre")
    .resizable(true)
    .collapsible(true)
    .show(ctx, |ui| {
        ui.label("Contenu de la fenêtre");
    });
```

### Modal

```rust
if show_modal {
    egui::Window::new("Confirmation")
        .collapsible(false)
        .resizable(false)
        .anchor(egui::Align2::CENTER_CENTER, [0.0, 0.0])
        .show(ctx, |ui| {
            ui.label("Êtes-vous sûr ?");
            ui.horizontal(|ui| {
                if ui.button("Annuler").clicked() {
                    show_modal = false;
                }
                if ui.button("Confirmer").clicked() {
                    // action
                    show_modal = false;
                }
            });
        });
}
```

---

## Panels

### SidePanel

```rust
egui::SidePanel::left("sidebar")
    .resizable(true)
    .default_width(200.0)
    .show(ctx, |ui| {
        ui.heading("Menu");
    });
```

### TopBottomPanel

```rust
egui::TopBottomPanel::top("header").show(ctx, |ui| {
    ui.heading("App Title");
});

egui::TopBottomPanel::bottom("footer").show(ctx, |ui| {
    ui.label("Status: Ready");
});
```

### CentralPanel

```rust
egui::CentralPanel::default().show(ctx, |ui| {
    ui.label("Contenu principal");
});
```

---

## Grille et Tables

### Grid

```rust
egui::Grid::new("my_grid")
    .num_columns(2)
    .spacing([40.0, 4.0])
    .striped(true)
    .show(ui, |ui| {
        ui.label("Nom");
        ui.text_edit_singleline(&mut name);
        ui.end_row();

        ui.label("Email");
        ui.text_edit_singleline(&mut email);
        ui.end_row();
    });
```

### Table avec TableBuilder

```rust
use egui_extras::{TableBuilder, Column};

TableBuilder::new(ui)
    .striped(true)
    .cell_layout(egui::Layout::left_to_right(egui::Align::Center))
    .column(Column::initial(100.0).resizable(true))
    .column(Column::remainder())
    .header(20.0, |mut header| {
        header.col(|ui| { ui.strong("Nom"); });
        header.col(|ui| { ui.strong("Valeur"); });
    })
    .body(|mut body| {
        body.row(18.0, |mut row| {
            row.col(|ui| { ui.label("Item 1"); });
            row.col(|ui| { ui.label("100"); });
        });
    });
```

---

## Interactions

### Réponse aux clics

```rust
let response = ui.button("Click me");

if response.clicked() { /* clic gauche */ }
if response.secondary_clicked() { /* clic droit */ }
if response.double_clicked() { /* double clic */ }
if response.hovered() { /* survol */ }
```

### Drag and Drop

```rust
let response = ui.label("Drag me");
if response.dragged() {
    // En cours de glissement
}
```

### Raccourcis clavier

```rust
if ctx.input(|i| i.key_pressed(egui::Key::Escape)) {
    // Escape pressé
}

if ctx.input(|i| i.modifiers.command && i.key_pressed(egui::Key::S)) {
    // Ctrl+S / Cmd+S
}
```

---

## Styles

### Couleurs

```rust
// Couleurs prédéfinies
Color32::RED
Color32::GREEN
Color32::from_rgb(100, 150, 200)
Color32::from_rgba_unmultiplied(100, 150, 200, 128)
Color32::from_gray(128)
```

### Visuals

```rust
// Mode sombre
ctx.set_visuals(egui::Visuals::dark());

// Mode clair
ctx.set_visuals(egui::Visuals::light());

// Personnalisation
let mut visuals = egui::Visuals::dark();
visuals.widgets.noninteractive.bg_fill = Color32::from_gray(20);
ctx.set_visuals(visuals);
```

---

## Astuces

### Forcer le rafraîchissement

```rust
ctx.request_repaint();
```

### ID unique

```rust
ui.push_id(unique_id, |ui| {
    // widgets avec ID unique
});
```

### Sens (interactivité)

```rust
// Zone cliquable personnalisée
let (rect, response) = ui.allocate_exact_size(
    egui::vec2(100.0, 50.0),
    egui::Sense::click()
);

if response.clicked() {
    // action
}

ui.painter().rect_filled(rect, 4.0, Color32::BLUE);
```
