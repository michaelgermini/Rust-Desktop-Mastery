# 16.2 État Local

## Quand utiliser l'état local

L'état local est approprié pour :
- État temporaire d'un composant (input en cours de saisie)
- État UI spécifique (menu ouvert/fermé)
- État de formulaire non sauvegardé

```rust
/// État global : partagé entre tous les écrans
pub struct GlobalState {
    pub current_user: Option<User>,
    pub theme: ThemeMode,
    pub recent_files: Vec<PathBuf>,
}

/// État local : propre à un écran
pub struct InvoiceEditorState {
    pub draft: Invoice,
    pub validation_errors: Vec<String>,
    pub show_preview: bool,
    pub selected_tab: InvoiceTab,
}

pub struct InvoiceEditor {
    local_state: InvoiceEditorState,
    global_state: Arc<Store>,
}

impl InvoiceEditor {
    pub fn new(invoice: Invoice, global_state: Arc<Store>) -> Self {
        Self {
            local_state: InvoiceEditorState {
                draft: invoice,
                validation_errors: Vec::new(),
                show_preview: false,
                selected_tab: InvoiceTab::Items,
            },
            global_state,
        }
    }
    
    pub fn render(&mut self, ui: &mut egui::Ui) {
        // Utiliser l'état local pour l'UI
        ui.text_edit_multiline(&mut self.local_state.draft.notes);
        
        // Valider localement
        if ui.button("Valider").clicked() {
            self.validate();
        }
        
        // Sauvegarder dans l'état global
        if ui.button("Sauvegarder").clicked() {
            self.save_to_global();
        }
    }
    
    fn validate(&mut self) {
        self.local_state.validation_errors.clear();
        if self.local_state.draft.items.is_empty() {
            self.local_state.validation_errors.push("Au moins un item requis".to_string());
        }
    }
    
    fn save_to_global(&self) {
        self.global_state.update(|state| {
            // Trouver et mettre à jour la facture
            if let Some(pos) = state.invoices.iter().position(|i| i.id == self.local_state.draft.id) {
                state.invoices[pos] = self.local_state.draft.clone();
            } else {
                state.invoices.push(self.local_state.draft.clone());
            }
        });
    }
}
```

## Lifting state up

Quand plusieurs composants ont besoin du même état local, remonter l'état vers le parent commun.

```rust
// ❌ État dupliqué
struct ComponentA {
    search_query: String,
}

struct ComponentB {
    search_query: String,  // Duplication
}

// ✅ État remonté
struct ParentComponent {
    search_query: String,  // État partagé
}

impl ParentComponent {
    fn render(&mut self, ui: &mut egui::Ui) {
        ui.horizontal(|ui| {
            ComponentA::render(ui, &mut self.search_query);
            ComponentB::render(ui, &self.search_query);
        });
    }
}
```

## Props drilling

Quand l'état doit passer par plusieurs niveaux, considérer le contexte ou le store global.

```rust
// ❌ Props drilling
struct App {
    user: Option<User>,
}

struct Screen {
    user: Option<User>,  // Passé depuis App
}

struct Component {
    user: Option<User>,  // Passé depuis Screen
}

// ✅ Utiliser le store global
struct App {
    store: Arc<Store>,
}

struct Component {
    store: Arc<Store>,  // Accès direct au store
}

impl Component {
    fn render(&self, ui: &mut egui::Ui) {
        let state = self.store.get();
        if let Some(user) = &state.user {
            ui.label(&user.name);
        }
    }
}
```

## Résumé

- **État local** : Pour état temporaire/UI spécifique
- **Lifting state up** : Partager l'état entre composants
- **Props drilling** : Éviter en utilisant le store global
- **Règle** : État local si pas besoin ailleurs, global sinon
