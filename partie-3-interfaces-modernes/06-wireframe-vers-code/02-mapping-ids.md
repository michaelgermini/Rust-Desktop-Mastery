# 13.2 Mapping IDs vers Composants

## Fichier de mapping

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

## Résumé

- **Parsing SVG** : Extraire les IDs et dimensions
- **Mapping automatique** : Détection du type de composant
- **Structure de données** : ComponentSpec avec type et rect
- **Base pour génération** : Prêt pour créer le code Rust
