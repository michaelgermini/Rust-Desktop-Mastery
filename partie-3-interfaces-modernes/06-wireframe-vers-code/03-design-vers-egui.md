# 13.3 Design vers egui

## Conversion des propriétés CSS

### Couleurs

```rust
// SVG: fill="#3b82f6"
// Rust:
let color = egui::Color32::from_rgb(59, 130, 246);

// SVG: fill="rgba(59, 130, 246, 0.5)"
// Rust:
let color = egui::Color32::from_rgba_unmultiplied(59, 130, 246, 128);
```

### Espacement

```rust
// SVG: x="280" y="80" width="300" height="40"
// Rust:
let rect = egui::Rect::from_min_size(
    egui::pos2(280.0, 80.0),
    egui::vec2(300.0, 40.0)
);
```

### Bordures arrondies

```rust
// SVG: rx="8"
// Rust:
let rounding = egui::Rounding::same(8.0);
```

## Layouts équivalents

### Flexbox → egui Layout

```rust
// CSS: display: flex; flex-direction: column;
// Rust:
ui.vertical(|ui| {
    // ...
});

// CSS: display: flex; flex-direction: row;
// Rust:
ui.horizontal(|ui| {
    // ...
});
```

### Grid → egui Grid

```rust
// CSS: display: grid; grid-template-columns: repeat(3, 1fr);
// Rust:
egui::Grid::new("my_grid")
    .num_columns(3)
    .show(ui, |ui| {
        // ...
    });
```

## Styles et couleurs

### Utiliser les tokens de design

```rust
// Au lieu de valeurs hardcodées
let color = tokens.colors.primary;
let spacing = tokens.spacing.md;
let radius = tokens.radius.sm;
```

## Résumé

- **Couleurs** : Conversion hex → Color32
- **Layouts** : Flexbox → egui Layout
- **Espacement** : Positions SVG → egui Rect
- **Tokens** : Utiliser le design system plutôt que valeurs hardcodées
