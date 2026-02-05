# Chapitre 9 : Construire une UI avec egui

## Introduction

egui utilise le paradigme "immediate mode" : l'UI est reconstruite à chaque frame. Ce modèle, différent du "retained mode" traditionnel, offre une simplicité remarquable une fois compris.

---

## 9.1 Immediate Mode Mental Model

### Retained Mode vs Immediate Mode

**Retained Mode (Qt, GTK, React...)**
```rust
// Créer les widgets une fois
let button = Button::new("Click me");
button.set_on_click(|_| {
    println!("Clicked!");
});
container.add(button);

// Les widgets existent comme objets persistants
// Problème : synchronisation état ↔ UI
```

**Immediate Mode (egui)**
```rust
// Reconstruire à chaque frame (60 FPS)
fn update(&mut self, ctx: &egui::Context, _frame: &mut eframe::Frame) {
    egui::CentralPanel::default().show(ctx, |ui| {
        // Le bouton est "créé" à chaque frame
        // Mais egui gère l'identité en interne
        if ui.button("Click me").clicked() {
            println!("Clicked!");
        }
    });
}

// Avantage : l'UI reflète TOUJOURS l'état actuel
// Pas de désynchronisation possible
```

### Le flux egui

```
┌─────────────────────────────────────────────────────────┐
│                    Boucle principale                     │
│                                                         │
│  1. Début de frame                                      │
│     └─ egui prépare le contexte                        │
│                                                         │
│  2. Construction UI (votre code)                        │
│     └─ ui.button(), ui.label(), etc.                   │
│     └─ egui collecte les "shapes" à dessiner           │
│     └─ egui détecte les interactions                   │
│                                                         │
│  3. Rendu                                               │
│     └─ egui envoie les shapes au GPU                   │
│     └─ Rendu à l'écran                                 │
│                                                         │
│  4. Fin de frame → retour à 1                           │
└─────────────────────────────────────────────────────────┘
```

### L'astuce de l'identité

```rust
// Comment egui sait que c'est le "même" bouton ?
// → Par son texte et sa position dans l'arbre UI

fn update(&mut self, ctx: &egui::Context, _frame: &mut eframe::Frame) {
    egui::CentralPanel::default().show(ctx, |ui| {
        // Ces deux boutons ont des identités différentes
        if ui.button("Bouton A").clicked() { /* ... */ }
        if ui.button("Bouton B").clicked() { /* ... */ }
        
        // Pour des boutons avec le même texte, utiliser push_id
        for i in 0..3 {
            ui.push_id(i, |ui| {
                if ui.button("Action").clicked() {
                    println!("Bouton {} cliqué", i);
                }
            });
        }
    });
}
```

---

## 9.2 Layouts

### Vertical et Horizontal

```rust
fn render_layouts(ui: &mut egui::Ui) {
    // Vertical (par défaut)
    ui.vertical(|ui| {
        ui.label("Ligne 1");
        ui.label("Ligne 2");
        ui.label("Ligne 3");
    });
    
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

### Centrage et alignement

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
    
    // Alignement custom
    ui.with_layout(
        egui::Layout::right_to_left(egui::Align::Center),
        |ui| {
            ui.label("Aligné à droite");
        }
    );
}
```

### Grilles

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
            
            // Ligne 3
            ui.label("Bob");
            ui.label("30");
            ui.label("bob@example.com");
            ui.end_row();
        });
}
```

### Panels et zones

```rust
fn render_panels(ctx: &egui::Context) {
    // Panel latéral gauche
    egui::SidePanel::left("left_panel")
        .default_width(200.0)
        .resizable(true)
        .show(ctx, |ui| {
            ui.heading("Navigation");
            // ...
        });
    
    // Panel latéral droit
    egui::SidePanel::right("right_panel")
        .default_width(300.0)
        .show(ctx, |ui| {
            ui.heading("Propriétés");
            // ...
        });
    
    // Panel du haut
    egui::TopBottomPanel::top("top_panel").show(ctx, |ui| {
        egui::menu::bar(ui, |ui| {
            ui.menu_button("Fichier", |ui| {
                if ui.button("Nouveau").clicked() { /* ... */ }
                if ui.button("Ouvrir").clicked() { /* ... */ }
                ui.separator();
                if ui.button("Quitter").clicked() { /* ... */ }
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
    
    // Panel central (ce qui reste)
    egui::CentralPanel::default().show(ctx, |ui| {
        ui.heading("Contenu principal");
    });
}
```

### ScrollArea

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
    
    // Scroll horizontal et vertical
    egui::ScrollArea::both()
        .show(ui, |ui| {
            // Grand contenu...
        });
    
    // Scroll avec stick to bottom (pour logs)
    egui::ScrollArea::vertical()
        .stick_to_bottom(true)
        .show(ui, |ui| {
            for log in &self.logs {
                ui.label(log);
            }
        });
}
```

---

## 9.3 Widgets

### Widgets de base

```rust
fn render_basic_widgets(ui: &mut egui::Ui, state: &mut AppState) {
    // Labels
    ui.label("Label simple");
    ui.heading("Titre");
    ui.monospace("Code monospace");
    ui.small("Petit texte");
    
    // Label avec style
    ui.label(egui::RichText::new("Coloré").color(egui::Color32::RED).size(20.0));
    
    // Boutons
    if ui.button("Bouton").clicked() {
        // Action
    }
    
    // Bouton avec icône (emoji ou caractère)
    if ui.button("📁 Ouvrir").clicked() {
        // ...
    }
    
    // Bouton toggle
    ui.toggle_value(&mut state.enabled, "Activer");
    
    // Checkbox
    ui.checkbox(&mut state.checked, "Option");
    
    // Radio buttons
    ui.radio_value(&mut state.choice, Choice::A, "Option A");
    ui.radio_value(&mut state.choice, Choice::B, "Option B");
    ui.radio_value(&mut state.choice, Choice::C, "Option C");
}
```

### Inputs

```rust
fn render_inputs(ui: &mut egui::Ui, state: &mut AppState) {
    // Text input single line
    ui.text_edit_singleline(&mut state.text);
    
    // Avec placeholder
    let response = ui.add(
        egui::TextEdit::singleline(&mut state.search)
            .hint_text("Rechercher...")
    );
    
    // Focus automatique
    if state.should_focus_search {
        response.request_focus();
        state.should_focus_search = false;
    }
    
    // Text input multiline
    ui.text_edit_multiline(&mut state.notes);
    
    // Avec taille fixe
    ui.add(
        egui::TextEdit::multiline(&mut state.code)
            .font(egui::TextStyle::Monospace)
            .desired_width(f32::INFINITY)
            .desired_rows(10)
    );
    
    // Password
    ui.add(egui::TextEdit::singleline(&mut state.password).password(true));
    
    // Sliders
    ui.add(egui::Slider::new(&mut state.value, 0.0..=100.0).text("Volume"));
    
    // Drag value
    ui.add(egui::DragValue::new(&mut state.number).speed(0.1));
    
    // Color picker
    ui.color_edit_button_srgba(&mut state.color);
}
```

### ComboBox et sélection

```rust
fn render_selection(ui: &mut egui::Ui, state: &mut AppState) {
    // ComboBox
    egui::ComboBox::from_label("Catégorie")
        .selected_text(format!("{:?}", state.category))
        .show_ui(ui, |ui| {
            ui.selectable_value(&mut state.category, Category::Work, "Travail");
            ui.selectable_value(&mut state.category, Category::Personal, "Personnel");
            ui.selectable_value(&mut state.category, Category::Other, "Autre");
        });
    
    // ComboBox avec recherche
    egui::ComboBox::from_id_source("country_selector")
        .selected_text(&state.selected_country)
        .show_ui(ui, |ui| {
            ui.text_edit_singleline(&mut state.country_filter);
            ui.separator();
            
            for country in COUNTRIES.iter()
                .filter(|c| c.to_lowercase().contains(&state.country_filter.to_lowercase()))
            {
                if ui.selectable_label(
                    state.selected_country == *country,
                    *country
                ).clicked() {
                    state.selected_country = country.to_string();
                }
            }
        });
}
```

### Progress et loading

```rust
fn render_progress(ui: &mut egui::Ui, state: &AppState) {
    // Progress bar
    ui.add(egui::ProgressBar::new(state.progress).text("Chargement..."));
    
    // Progress bar avec animation
    ui.add(egui::ProgressBar::new(state.progress)
        .animate(true)
        .text(format!("{:.0}%", state.progress * 100.0)));
    
    // Spinner
    ui.spinner();
    
    // Custom loading
    if state.is_loading {
        ui.horizontal(|ui| {
            ui.spinner();
            ui.label("Chargement en cours...");
        });
    }
}
```

---

## 9.4 Gestion Propre de l'État

### Structure d'état

```rust
/// État principal de l'application
pub struct AppState {
    // Données
    pub documents: Vec<Document>,
    pub selected_document: Option<usize>,
    
    // UI State
    pub ui: UiState,
    
    // Paramètres
    pub settings: Settings,
}

/// État spécifique à l'UI
pub struct UiState {
    pub search_query: String,
    pub show_settings: bool,
    pub show_about: bool,
    pub sidebar_expanded: bool,
    pub active_tab: Tab,
}

impl Default for UiState {
    fn default() -> Self {
        Self {
            search_query: String::new(),
            show_settings: false,
            show_about: false,
            sidebar_expanded: true,
            active_tab: Tab::Documents,
        }
    }
}
```

### Séparation logique/rendu

```rust
impl eframe::App for App {
    fn update(&mut self, ctx: &egui::Context, _frame: &mut eframe::Frame) {
        // 1. Traiter les événements async
        self.process_background_events();
        
        // 2. Gérer les raccourcis globaux
        self.handle_keyboard_shortcuts(ctx);
        
        // 3. Rendu UI
        self.render_ui(ctx);
        
        // 4. Mettre à jour si nécessaire
        if self.needs_repaint() {
            ctx.request_repaint();
        }
    }
}

impl App {
    fn render_ui(&mut self, ctx: &egui::Context) {
        self.render_menu_bar(ctx);
        self.render_sidebar(ctx);
        self.render_main_content(ctx);
        self.render_dialogs(ctx);
    }
    
    fn render_main_content(&mut self, ctx: &egui::Context) {
        egui::CentralPanel::default().show(ctx, |ui| {
            match self.state.ui.active_tab {
                Tab::Documents => self.render_documents_tab(ui),
                Tab::Search => self.render_search_tab(ui),
                Tab::Settings => self.render_settings_tab(ui),
            }
        });
    }
}
```

### Pattern : Composants avec état local

```rust
/// Composant avec son propre état
pub struct SearchBox {
    query: String,
    focused: bool,
    results: Vec<SearchResult>,
}

impl SearchBox {
    pub fn new() -> Self {
        Self {
            query: String::new(),
            focused: false,
            results: Vec::new(),
        }
    }
    
    /// Retourne l'action demandée par l'utilisateur
    pub fn show(&mut self, ui: &mut egui::Ui) -> Option<SearchAction> {
        let mut action = None;
        
        ui.horizontal(|ui| {
            let response = ui.add(
                egui::TextEdit::singleline(&mut self.query)
                    .hint_text("🔍 Rechercher...")
            );
            
            self.focused = response.has_focus();
            
            if response.changed() {
                action = Some(SearchAction::QueryChanged(self.query.clone()));
            }
            
            if ui.input(|i| i.key_pressed(egui::Key::Escape)) && self.focused {
                self.query.clear();
                action = Some(SearchAction::Cancelled);
            }
            
            if ui.input(|i| i.key_pressed(egui::Key::Enter)) && self.focused {
                action = Some(SearchAction::Submitted(self.query.clone()));
            }
        });
        
        // Afficher les résultats si query non vide
        if !self.query.is_empty() && self.focused {
            self.render_results(ui, &mut action);
        }
        
        action
    }
    
    pub fn set_results(&mut self, results: Vec<SearchResult>) {
        self.results = results;
    }
    
    fn render_results(&self, ui: &mut egui::Ui, action: &mut Option<SearchAction>) {
        egui::Frame::popup(ui.style())
            .show(ui, |ui| {
                for (i, result) in self.results.iter().enumerate() {
                    if ui.selectable_label(false, &result.title).clicked() {
                        *action = Some(SearchAction::Selected(i));
                    }
                }
            });
    }
}

pub enum SearchAction {
    QueryChanged(String),
    Submitted(String),
    Selected(usize),
    Cancelled,
}
```

---

## Résumé du Chapitre

### Mental model immediate mode

- L'UI est reconstruite à chaque frame
- Pas de synchronisation état ↔ UI à gérer
- L'identité est basée sur le texte et la position

### Layouts essentiels

| Layout | Usage |
|--------|-------|
| `ui.vertical()` | Empiler verticalement |
| `ui.horizontal()` | Aligner horizontalement |
| `egui::Grid` | Tableaux et formulaires |
| `egui::SidePanel` | Barres latérales |
| `egui::ScrollArea` | Contenu scrollable |

### Organisation recommandée

1. État séparé de l'UI (`AppState`, `UiState`)
2. Méthodes de rendu dédiées (`render_*`)
3. Composants réutilisables avec leur propre état
4. Actions retournées plutôt que mutations directes

---

**Dans le prochain chapitre, nous créerons un Design System complet pour des interfaces cohérentes et professionnelles.**
