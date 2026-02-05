# 9.3 Widgets

## Widgets de base

### Labels

```rust
fn render_labels(ui: &mut egui::Ui) {
    // Label simple
    ui.label("Label simple");
    
    // Heading (titre)
    ui.heading("Titre Principal");
    
    // Texte monospace
    ui.monospace("Code monospace");
    
    // Petit texte
    ui.small("Petit texte");
    
    // Label avec style
    ui.label(egui::RichText::new("Coloré")
        .color(egui::Color32::RED)
        .size(20.0)
        .strong());
}
```

### Boutons

```rust
fn render_buttons(ui: &mut egui::Ui, state: &mut AppState) {
    // Bouton simple
    if ui.button("Bouton").clicked() {
        // Action
    }
    
    // Bouton avec icône (emoji ou caractère)
    if ui.button("📁 Ouvrir").clicked() {
        // ...
    }
    
    // Bouton toggle
    ui.toggle_value(&mut state.enabled, "Activer");
    
    // Bouton avec taille personnalisée
    ui.add_sized(
        egui::vec2(200.0, 50.0),
        egui::Button::new("Grand bouton")
    );
}
```

### Checkbox et Radio

```rust
fn render_selections(ui: &mut egui::Ui, state: &mut AppState) {
    // Checkbox
    ui.checkbox(&mut state.checked, "Option");
    
    // Radio buttons
    ui.radio_value(&mut state.choice, Choice::A, "Option A");
    ui.radio_value(&mut state.choice, Choice::B, "Option B");
    ui.radio_value(&mut state.choice, Choice::C, "Option C");
}
```

## Inputs

### Text input

```rust
fn render_text_inputs(ui: &mut egui::Ui, state: &mut AppState) {
    // Single line
    ui.text_edit_singleline(&mut state.name);
    
    // Multiline
    ui.text_edit_multiline(&mut state.description);
    
    // Avec placeholder
    ui.add(egui::TextEdit::singleline(&mut state.search)
        .hint_text("Rechercher..."));
    
    // Mot de passe
    ui.add(egui::TextEdit::singleline(&mut state.password)
        .password(true));
    
    // Largeur désirée
    ui.add(egui::TextEdit::singleline(&mut state.email)
        .desired_width(300.0));
}
```

### Number input

```rust
fn render_number_inputs(ui: &mut egui::Ui, state: &mut AppState) {
    // Slider
    ui.add(egui::Slider::new(&mut state.value, 0..=100));
    
    // Slider avec texte
    ui.add(egui::Slider::new(&mut state.volume, 0..=100)
        .text("Volume"));
    
    // Slider vertical
    ui.add(egui::Slider::new(&mut state.brightness, 0.0..=1.0)
        .orientation(egui::SliderOrientation::Vertical));
    
    // Drag value (modifier en glissant)
    ui.add(egui::DragValue::new(&mut state.count));
}
```

### Combo box / Dropdown

```rust
fn render_dropdowns(ui: &mut egui::Ui, state: &mut AppState) {
    egui::ComboBox::from_label("Choisir")
        .selected_text(&state.selected)
        .show_ui(ui, |ui| {
            ui.selectable_value(&mut state.selected, "Option A".to_string(), "Option A");
            ui.selectable_value(&mut state.selected, "Option B".to_string(), "Option B");
            ui.selectable_value(&mut state.selected, "Option C".to_string(), "Option C");
        });
}
```

## Widgets avancés

### Color picker

```rust
fn render_color_picker(ui: &mut egui::Ui, color: &mut egui::Color32) {
    egui::color_picker::color_picker_color32(ui, color, egui::color_picker::Alpha::Opaque);
}
```

### Date picker

```rust
use chrono::{NaiveDate, Local};

fn render_date_picker(ui: &mut egui::Ui, date: &mut NaiveDate) {
    // Utiliser une crate comme egui-datepicker ou implémenter custom
    // Exemple simplifié :
    ui.horizontal(|ui| {
        ui.label(format!("Date: {}", date));
        if ui.button("Choisir").clicked() {
            // Ouvrir un calendrier...
        }
    });
}
```

### Progress bar

```rust
fn render_progress(ui: &mut egui::Ui, progress: f32) {
    ui.add(egui::ProgressBar::new(progress)
        .text(format!("{:.0}%", progress * 100.0)));
}
```

### Separator

```rust
fn render_separators(ui: &mut egui::Ui) {
    ui.label("Section 1");
    ui.separator();  // Ligne horizontale
    ui.label("Section 2");
}
```

## Custom widgets

### Créer un widget personnalisé

```rust
pub fn custom_button(ui: &mut egui::Ui, text: &str, color: egui::Color32) -> egui::Response {
    let button = egui::Button::new(
        egui::RichText::new(text).color(color)
    );
    
    ui.add(button)
}

// Utilisation
if custom_button(ui, "Action", egui::Color32::GREEN).clicked() {
    // ...
}
```

## Résumé

- **Labels** : Affichage de texte avec différents styles
- **Buttons** : Actions utilisateur
- **Inputs** : Saisie de texte et nombres
- **Selections** : Checkbox, radio, combo box
- **Avancés** : Color picker, progress bar, etc.
- **Custom** : Créer vos propres widgets en combinant les bases
