# 9.4 Gestion Propre de l'État

## State local vs global

### État local (dans le composant)

```rust
struct MyScreen {
    // État local à ce screen
    input_text: String,
    selected_index: Option<usize>,
    show_details: bool,
}

impl MyScreen {
    fn render(&mut self, ui: &mut egui::Ui) {
        // Utiliser l'état local directement
        ui.text_edit_singleline(&mut self.input_text);
        
        if ui.button("Afficher détails").clicked() {
            self.show_details = !self.show_details;
        }
    }
}
```

### État global (partagé)

```rust
use std::sync::Arc;
use arc_swap::ArcSwap;

pub struct AppState {
    pub current_user: Option<User>,
    pub settings: Settings,
    pub data: Vec<DataItem>,
}

pub struct App {
    // État global partagé
    state: Arc<ArcSwap<AppState>>,
}

impl App {
    fn render(&mut self, ui: &mut egui::Ui) {
        let state = self.state.load();
        
        // Lire l'état global
        if let Some(user) = &state.current_user {
            ui.label(format!("Utilisateur: {}", user.name));
        }
    }
    
    fn update_state(&self, new_state: AppState) {
        self.state.store(Arc::new(new_state));
    }
}
```

## Patterns de state management

### Pattern 1 : État dans la struct App

```rust
#[derive(Default)]
struct MyApp {
    // État de l'application
    counter: i32,
    name: String,
    items: Vec<String>,
}

impl eframe::App for MyApp {
    fn update(&mut self, ctx: &egui::Context, _frame: &mut eframe::Frame) {
        // Accès direct à self
        egui::CentralPanel::default().show(ctx, |ui| {
            ui.label(format!("Counter: {}", self.counter));
            
            if ui.button("Increment").clicked() {
                self.counter += 1;
            }
        });
    }
}
```

### Pattern 2 : État séparé par écran

```rust
struct App {
    // État global
    current_user: Option<User>,
    
    // État par écran
    dashboard: DashboardState,
    settings: SettingsState,
    invoices: InvoiceState,
    
    // Écran actuel
    current_screen: Screen,
}

enum Screen {
    Dashboard,
    Settings,
    Invoices,
}

impl App {
    fn render(&mut self, ctx: &egui::Context) {
        match self.current_screen {
            Screen::Dashboard => self.dashboard.render(ctx, &mut self.dashboard),
            Screen::Settings => self.settings.render(ctx, &mut self.settings),
            Screen::Invoices => self.invoices.render(ctx, &mut self.invoices),
        }
    }
}
```

### Pattern 3 : Store centralisé

```rust
use std::sync::Arc;
use arc_swap::ArcSwap;

pub struct Store {
    state: Arc<ArcSwap<AppState>>,
}

impl Store {
    pub fn new() -> Self {
        Self {
            state: Arc::new(ArcSwap::from_pointee(AppState::default())),
        }
    }
    
    pub fn read(&self) -> Arc<AppState> {
        self.state.load_full()
    }
    
    pub fn update<F>(&self, f: F)
    where
        F: FnOnce(&AppState) -> AppState,
    {
        self.state.rcu(|state| Arc::new(f(state)));
    }
}

pub struct App {
    store: Store,
}

impl App {
    fn render(&mut self, ctx: &egui::Context) {
        let state = self.store.read();
        
        // Utiliser state pour l'affichage
        egui::CentralPanel::default().show(ctx, |ui| {
            ui.label(&state.current_message);
        });
        
        // Mettre à jour si nécessaire
        if some_condition {
            self.store.update(|state| {
                let mut new = state.clone();
                new.current_message = "Nouveau message".to_string();
                new
            });
        }
    }
}
```

## Persistence de l'état UI

### Sauvegarder les préférences UI

```rust
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize)]
pub struct UiPreferences {
    pub sidebar_width: f32,
    pub theme: Theme,
    pub window_size: (f32, f32),
    pub window_pos: (f32, f32),
}

impl UiPreferences {
    pub fn load() -> Self {
        let path = dirs::config_dir()
            .unwrap()
            .join("mon-app")
            .join("ui_prefs.json");
        
        if path.exists() {
            if let Ok(content) = std::fs::read_to_string(&path) {
                if let Ok(prefs) = serde_json::from_str(&content) {
                    return prefs;
                }
            }
        }
        
        Self::default()
    }
    
    pub fn save(&self) -> Result<()> {
        let path = dirs::config_dir()
            .unwrap()
            .join("mon-app")
            .join("ui_prefs.json");
        
        std::fs::create_dir_all(path.parent().unwrap())?;
        let content = serde_json::to_string_pretty(self)?;
        std::fs::write(path, content)?;
        Ok(())
    }
}
```

### Restaurer la taille/position de la fenêtre

```rust
impl eframe::App for MyApp {
    fn save(&mut self, storage: &mut dyn eframe::Storage) {
        // Sauvegarder automatiquement par eframe
        eframe::set_value(storage, "app_state", &self.state);
    }
    
    fn setup(&mut self, ctx: &egui::Context, _frame: &mut eframe::Frame, storage: Option<&dyn eframe::Storage>) {
        // Restaurer l'état sauvegardé
        if let Some(storage) = storage {
            if let Some(state) = eframe::get_value(storage, "app_state") {
                self.state = state;
            }
        }
    }
}
```

## Résumé

- **État local** : Pour l'état spécifique à un écran/composant
- **État global** : Pour les données partagées (ArcSwap)
- **Store centralisé** : Pattern pour applications complexes
- **Persistence** : Sauvegarder les préférences UI et l'état de l'application
