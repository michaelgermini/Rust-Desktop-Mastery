# Chapitre 13 : Du Wireframe SVG au Code Rust

## Introduction

Ce chapitre présente un workflow efficace pour traduire des maquettes visuelles en code egui. L'objectif est de réduire le temps entre la conception et l'implémentation tout en maintenant la fidélité au design.

---

## 13.1 Maquettes SVG comme Source de Vérité

### Pourquoi SVG ?

- **Éditable** : Figma, Sketch, Inkscape exportent en SVG
- **Versionnable** : Texte, donc compatible Git
- **Mesurable** : Dimensions et positions exactes
- **Inspectable** : Structure hiérarchique visible

### Convention de nommage des IDs

```xml
<!-- Maquette SVG avec IDs sémantiques -->
<svg viewBox="0 0 1200 800">
  <!-- Layout principal -->
  <g id="layout-sidebar">
    <rect id="sidebar-bg" x="0" y="0" width="240" height="800"/>
    <g id="sidebar-nav">
      <g id="nav-item-dashboard" class="nav-item">
        <rect x="16" y="80" width="208" height="40"/>
        <text x="56" y="105">Dashboard</text>
      </g>
      <g id="nav-item-invoices" class="nav-item">
        <rect x="16" y="128" width="208" height="40"/>
        <text x="56" y="153">Factures</text>
      </g>
    </g>
  </g>
  
  <g id="layout-main">
    <g id="header">
      <text id="header-title" x="280" y="40">Tableau de bord</text>
      <g id="header-actions">
        <rect id="btn-new-invoice" x="1050" y="16" width="120" height="36"/>
      </g>
    </g>
    
    <g id="content">
      <g id="stats-grid">
        <g id="stat-card-revenue" class="stat-card">
          <!-- ... -->
        </g>
      </g>
    </g>
  </g>
</svg>
```

### Convention de nommage

```
Format: [type]-[zone]-[element]-[variant]

Exemples:
- layout-sidebar       : Zone de layout
- nav-item-dashboard   : Item de navigation
- btn-new-invoice      : Bouton d'action
- card-stat-revenue    : Card de statistique
- input-search-main    : Champ de recherche
- modal-confirm-delete : Modale de confirmation
```

---

## 13.2 Mapping IDs vers Composants

### Fichier de mapping

```rust
// src/ui/mapping.rs

use std::collections::HashMap;

/// Mapping entre IDs SVG et composants Rust
pub struct UiMapping {
    components: HashMap<String, ComponentSpec>,
}

pub struct ComponentSpec {
    pub component_type: ComponentType,
    pub props: HashMap<String, String>,
    pub rect: Rect,
}

pub enum ComponentType {
    // Layout
    Sidebar,
    Header,
    MainContent,
    
    // Navigation
    NavItem,
    NavGroup,
    
    // Cards
    StatCard,
    ContentCard,
    
    // Inputs
    Button,
    TextInput,
    SearchBox,
    
    // Data
    Table,
    List,
    
    // Feedback
    Modal,
    Toast,
}

impl UiMapping {
    pub fn from_svg(svg_content: &str) -> Self {
        let doc = roxmltree::Document::parse(svg_content).unwrap();
        let mut components = HashMap::new();
        
        // Parcourir et extraire les composants
        for node in doc.descendants() {
            if let Some(id) = node.attribute("id") {
                if let Some(spec) = Self::parse_component(id, &node) {
                    components.insert(id.to_string(), spec);
                }
            }
        }
        
        Self { components }
    }
    
    fn parse_component(id: &str, node: &roxmltree::Node) -> Option<ComponentSpec> {
        let parts: Vec<&str> = id.split('-').collect();
        
        let component_type = match parts.get(0)? {
            &"layout" => match parts.get(1)? {
                &"sidebar" => ComponentType::Sidebar,
                &"header" => ComponentType::Header,
                &"main" => ComponentType::MainContent,
                _ => return None,
            },
            &"nav" => ComponentType::NavItem,
            &"btn" => ComponentType::Button,
            &"input" => ComponentType::TextInput,
            &"card" => match parts.get(1)? {
                &"stat" => ComponentType::StatCard,
                _ => ComponentType::ContentCard,
            },
            &"modal" => ComponentType::Modal,
            _ => return None,
        };
        
        // Extraire les dimensions
        let rect = Self::extract_rect(node)?;
        
        Some(ComponentSpec {
            component_type,
            props: HashMap::new(),
            rect,
        })
    }
    
    fn extract_rect(node: &roxmltree::Node) -> Option<Rect> {
        // Chercher un rect enfant ou les attributs de positionnement
        if let Some(rect_node) = node.children().find(|n| n.tag_name().name() == "rect") {
            let x = rect_node.attribute("x")?.parse().ok()?;
            let y = rect_node.attribute("y")?.parse().ok()?;
            let width = rect_node.attribute("width")?.parse().ok()?;
            let height = rect_node.attribute("height")?.parse().ok()?;
            return Some(Rect { x, y, width, height });
        }
        None
    }
}

#[derive(Clone, Copy)]
pub struct Rect {
    pub x: f32,
    pub y: f32,
    pub width: f32,
    pub height: f32,
}
```

---

## 13.3 Générateur de Code

### Script de génération

```rust
// tools/generate_ui.rs

use std::fs;
use std::path::Path;

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
    code.push_str("// Component stubs\n");
    for (id, spec) in &mapping.components {
        if matches!(spec.component_type, ComponentType::NavItem | ComponentType::StatCard | ComponentType::Button) {
            code.push_str(&generate_component_stub(id, spec));
        }
    }
    
    fs::write(output_dir.join("generated_ui.rs"), code).unwrap();
}

fn generate_component_stub(id: &str, spec: &ComponentSpec) -> String {
    let fn_name = id.replace('-', "_");
    
    match spec.component_type {
        ComponentType::Button => format!(
            r#"
pub fn {fn_name}(ui: &mut Ui, tokens: &DesignTokens) -> egui::Response {{
    // Generated from: {id}
    // Size: {w:.0}x{h:.0}
    crate::components::Button::new("{label}")
        .show(ui, tokens)
}}
"#,
            fn_name = fn_name,
            id = id,
            w = spec.rect.width,
            h = spec.rect.height,
            label = id.split('-').last().unwrap_or("Button"),
        ),
        
        ComponentType::StatCard => format!(
            r#"
pub fn {fn_name}(ui: &mut Ui, value: &str, label: &str, tokens: &DesignTokens) {{
    // Generated from: {id}
    // Size: {w:.0}x{h:.0}
    crate::components::StatCard::new(value, label)
        .show(ui, tokens);
}}
"#,
            fn_name = fn_name,
            id = id,
            w = spec.rect.width,
            h = spec.rect.height,
        ),
        
        _ => String::new(),
    }
}
```

---

## 13.4 Workflow Design vers Code

### Pipeline de travail

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

### Script d'automatisation

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

### Préserver le code custom

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

---

## 13.5 Exemple Complet

### Wireframe source

```xml
<!-- wireframes/invoice_list.svg -->
<svg viewBox="0 0 1200 800" xmlns="http://www.w3.org/2000/svg">
  <!-- Sidebar -->
  <g id="layout-sidebar">
    <rect fill="#f3f4f6" x="0" y="0" width="240" height="800"/>
    <g id="nav-items">
      <g id="nav-item-dashboard"><rect x="16" y="80" width="208" height="40" rx="8"/></g>
      <g id="nav-item-invoices" class="active"><rect x="16" y="128" width="208" height="40" rx="8"/></g>
      <g id="nav-item-clients"><rect x="16" y="176" width="208" height="40" rx="8"/></g>
    </g>
  </g>
  
  <!-- Main content -->
  <g id="layout-main">
    <!-- Header -->
    <g id="header">
      <text id="header-title" x="280" y="48" font-size="24" font-weight="bold">Factures</text>
      <g id="header-actions">
        <rect id="btn-new-invoice" x="1050" y="20" width="120" height="36" rx="8" fill="#3b82f6"/>
      </g>
    </g>
    
    <!-- Filters -->
    <g id="filters">
      <rect id="input-search" x="280" y="80" width="300" height="40" rx="8" fill="#fff" stroke="#e5e7eb"/>
      <rect id="filter-status" x="600" y="80" width="150" height="40" rx="8" fill="#fff" stroke="#e5e7eb"/>
      <rect id="filter-date" x="770" y="80" width="150" height="40" rx="8" fill="#fff" stroke="#e5e7eb"/>
    </g>
    
    <!-- Table -->
    <g id="table-invoices">
      <rect x="280" y="140" width="890" height="600" rx="8" fill="#fff" stroke="#e5e7eb"/>
      <!-- Header row -->
      <g id="table-header">
        <text x="300" y="170" fill="#6b7280">N° Facture</text>
        <text x="450" y="170" fill="#6b7280">Client</text>
        <text x="700" y="170" fill="#6b7280">Date</text>
        <text x="850" y="170" fill="#6b7280">Montant</text>
        <text x="1000" y="170" fill="#6b7280">Statut</text>
      </g>
    </g>
  </g>
</svg>
```

### Code généré

```rust
// src/ui/generated/invoice_list.rs
// AUTO-GENERATED - DO NOT EDIT

pub mod layout {
    pub const SIDEBAR_WIDTH: f32 = 240.0;
    pub const MAIN_X: f32 = 280.0;
    pub const HEADER_HEIGHT: f32 = 60.0;
    pub const FILTERS_Y: f32 = 80.0;
    pub const TABLE_Y: f32 = 140.0;
}

pub mod components {
    pub const BTN_NEW_INVOICE: (f32, f32, f32, f32) = (1050.0, 20.0, 120.0, 36.0);
    pub const INPUT_SEARCH: (f32, f32, f32, f32) = (280.0, 80.0, 300.0, 40.0);
    pub const FILTER_STATUS: (f32, f32, f32, f32) = (600.0, 80.0, 150.0, 40.0);
    pub const FILTER_DATE: (f32, f32, f32, f32) = (770.0, 80.0, 150.0, 40.0);
}
```

### Implémentation finale

```rust
// src/ui/screens/invoice_list.rs

mod generated {
    include!("../generated/invoice_list.rs");
}

use generated::{layout, components};

pub struct InvoiceListScreen {
    search_query: String,
    status_filter: Option<InvoiceStatus>,
    invoices: Vec<Invoice>,
    selected: Option<usize>,
}

impl InvoiceListScreen {
    pub fn render(&mut self, ctx: &egui::Context, tokens: &DesignTokens) {
        // Sidebar (utilise les dimensions générées)
        egui::SidePanel::left("sidebar")
            .exact_width(layout::SIDEBAR_WIDTH)
            .show(ctx, |ui| {
                self.render_nav(ui, tokens);
            });
        
        // Main content
        egui::CentralPanel::default().show(ctx, |ui| {
            // Header
            ui.horizontal(|ui| {
                ui.heading("Factures");
                
                ui.with_layout(egui::Layout::right_to_left(egui::Align::Center), |ui| {
                    if Button::new("+ Nouvelle facture")
                        .variant(ButtonVariant::Primary)
                        .show(ui, tokens)
                        .clicked()
                    {
                        self.create_new_invoice();
                    }
                });
            });
            
            ui.add_space(tokens.spacing.md);
            
            // Filters (positions from wireframe)
            ui.horizontal(|ui| {
                // Search - largeur du wireframe
                ui.add_sized(
                    [components::INPUT_SEARCH.2, components::INPUT_SEARCH.3],
                    egui::TextEdit::singleline(&mut self.search_query)
                        .hint_text("🔍 Rechercher...")
                );
                
                // Status filter
                egui::ComboBox::from_id_source("status_filter")
                    .width(components::FILTER_STATUS.2)
                    .selected_text(self.status_filter.map(|s| format!("{:?}", s)).unwrap_or("Tous".into()))
                    .show_ui(ui, |ui| {
                        ui.selectable_value(&mut self.status_filter, None, "Tous");
                        ui.selectable_value(&mut self.status_filter, Some(InvoiceStatus::Draft), "Brouillon");
                        ui.selectable_value(&mut self.status_filter, Some(InvoiceStatus::Sent), "Envoyée");
                        ui.selectable_value(&mut self.status_filter, Some(InvoiceStatus::Paid), "Payée");
                    });
            });
            
            ui.add_space(tokens.spacing.md);
            
            // Table
            Table::new(&self.filtered_invoices())
                .column("N°", |inv, ui, _| { ui.label(&inv.number); })
                .column("Client", |inv, ui, _| { ui.label(&inv.client_name); })
                .column("Date", |inv, ui, _| { ui.label(inv.date.format("%d/%m/%Y").to_string()); })
                .column("Montant", |inv, ui, tokens| {
                    ui.label(egui::RichText::new(format!("{:.2} €", inv.total))
                        .color(tokens.colors.text_primary));
                })
                .column("Statut", |inv, ui, tokens| {
                    self.render_status_badge(ui, inv.status, tokens);
                })
                .selectable(&mut self.selected)
                .show(ui, tokens);
        });
    }
}
```

---

## Résumé du Chapitre

### Workflow optimisé

1. **Design** → SVG avec IDs sémantiques
2. **Parse** → Extraire les dimensions et composants
3. **Generate** → Code Rust avec constantes
4. **Implement** → Compléter avec logique custom
5. **Iterate** → Régénérer sans perdre le custom

### Avantages

- **Fidélité** : Dimensions exactes du design
- **Rapidité** : Moins de code à écrire
- **Cohérence** : Single source of truth
- **Maintenabilité** : Mise à jour facile

---

**Dans la prochaine partie, nous aborderons l'architecture d'applications desktop complètes.**
