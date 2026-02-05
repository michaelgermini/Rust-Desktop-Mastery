# 11.4 Sidebar

## Navigation sidebar avec badges

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
        // Rendu de la sidebar
    }
}
```

## Usage

```rust
Sidebar::new(&mut selected_tab)
    .item("📊", "Dashboard")
    .item("📄", "Factures")
    .item_with_badge("🔔", "Notifications", 5)
    .collapsed(false)
    .show(ctx, &tokens);
```

## Features

- **Icônes** : Emoji ou caractères
- **Badges** : Compteurs de notifications
- **Collapsible** : Mode réduit
- **Sélection visuelle** : Indicateur de l'item actif
