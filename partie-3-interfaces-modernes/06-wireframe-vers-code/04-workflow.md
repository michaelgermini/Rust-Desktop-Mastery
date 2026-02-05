# 13.4 Workflow Rapide Design vers Code

## Pipeline de travail

```
1. DESIGN (Figma/Sketch)
   └── Export SVG avec IDs sémantiques
   
2. PARSING
   └── cargo run --bin parse-wireframe wireframe.svg
   └── Génère: ui_mapping.json
   
3. GÉNÉRATION
   └── cargo run --bin generate-ui
   └── Génère: src/ui/generated_ui.rs
   
4. IMPLÉMENTATION
   └── Compléter les composants générés
   └── Connecter à l'état de l'application
   
5. ITÉRATION
   └── Modifier le design
   └── Régénérer
   └── Le code custom est préservé
```

## Script d'automatisation

```bash
#!/bin/bash
# scripts/sync-design.sh

echo "🎨 Synchronizing design..."

# 1. Télécharger le dernier export Figma (optionnel)
# curl -o wireframes/latest.svg "https://api.figma.com/..."

# 2. Parser les wireframes
echo "📐 Parsing wireframes..."
cargo run --bin parse-wireframe -- wireframes/*.svg -o src/ui/mapping.json

# 3. Générer le code
echo "⚙️ Generating UI code..."
cargo run --bin generate-ui -- src/ui/mapping.json -o src/ui/generated/

# 4. Formatter
echo "🧹 Formatting..."
cargo fmt

# 5. Vérifier la compilation
echo "🔨 Checking build..."
cargo check

echo "✅ Design sync complete!"
```

## Préserver le code custom

```rust
// src/ui/screens/dashboard.rs

// ===== GENERATED SECTION - DO NOT EDIT =====
include!("generated/dashboard_layout.rs");
// ===== END GENERATED SECTION =====

// ===== CUSTOM IMPLEMENTATION =====
impl DashboardScreen {
    pub fn new(/* ... */) -> Self {
        // Code custom préservé lors de la régénération
    }
    
    pub fn render(&mut self, ui: &mut Ui, tokens: &DesignTokens) {
        // Utiliser les constantes générées
        let sidebar_rect = egui::Rect::from_min_size(
            egui::pos2(layout::LAYOUT_SIDEBAR.0, layout::LAYOUT_SIDEBAR.1),
            egui::vec2(layout::LAYOUT_SIDEBAR.2, layout::LAYOUT_SIDEBAR.3),
        );
        
        // Implémentation custom
        self.render_sidebar(ui, sidebar_rect, tokens);
        self.render_content(ui, tokens);
    }
}
```

## Générateur de code

```rust
// tools/generate_ui.rs

pub fn generate_ui_code(mapping: &UiMapping, output_dir: &Path) {
    let mut code = String::new();
    
    code.push_str("// AUTO-GENERATED from wireframe.svg\n");
    code.push_str("// Do not edit manually - regenerate with: cargo run --bin generate-ui\n\n");
    
    code.push_str("use egui::Ui;\n");
    code.push_str("use crate::theme::DesignTokens;\n\n");
    
    // Générer les constantes de layout
    code.push_str("// Layout constants\n");
    code.push_str("pub mod layout {\n");
    
    for (id, spec) in &mapping.components {
        if matches!(spec.component_type, ComponentType::Sidebar | ComponentType::Header | ComponentType::MainContent) {
            code.push_str(&format!(
                "    pub const {}: (f32, f32, f32, f32) = ({:.1}, {:.1}, {:.1}, {:.1});\n",
                id.to_uppercase().replace('-', "_"),
                spec.rect.x, spec.rect.y, spec.rect.width, spec.rect.height
            ));
        }
    }
    
    code.push_str("}\n\n");
    
    // Générer les stubs de composants
    // ...
    
    fs::write(output_dir.join("generated_ui.rs"), code).unwrap();
}
```

## Résumé

- **Pipeline automatisé** : Design → Parse → Generate → Implement
- **Scripts** : Automatisation complète du workflow
- **Code custom préservé** : Sections générées vs custom
- **Itération rapide** : Modifier design et régénérer facilement
