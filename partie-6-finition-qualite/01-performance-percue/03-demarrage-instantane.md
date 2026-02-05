# 25.3 Démarrage Instantané

## Optimisation du startup

```rust
pub struct FastStartupApp {
    // Charger immédiatement
    ui: UiState,
    theme: ThemeManager,
    
    // Charger en arrière-plan
    data: Arc<ArcSwap<LoadState<AppData>>>,
}

impl FastStartupApp {
    pub fn new() -> Self {
        // Initialisation minimale
        let ui = UiState::default();
        let theme = ThemeManager::new(ThemeMode::System);
        
        // Démarrer le chargement en arrière-plan
        let data = Arc::new(ArcSwap::from_pointee(LoadState::Loading {
            started_at: Instant::now(),
        }));
        
        let data_clone = Arc::clone(&data);
        std::thread::spawn(move || {
            // Charger les données
            match load_app_data() {
                Ok(app_data) => {
                    data_clone.store(Arc::new(LoadState::Loaded(app_data)));
                }
                Err(e) => {
                    data_clone.store(Arc::new(LoadState::Error(e.to_string())));
                }
            }
        });
        
        Self { ui, theme, data }
    }
}
```

## Chargement progressif

```rust
pub enum StartupPhase {
    /// UI de base visible
    UiReady,
    /// Données essentielles chargées
    EssentialDataLoaded,
    /// Toutes les données chargées
    FullyLoaded,
}

impl FastStartupApp {
    pub fn get_startup_phase(&self) -> StartupPhase {
        match self.data.load().as_ref() {
            LoadState::Idle | LoadState::Loading { .. } => StartupPhase::UiReady,
            LoadState::Loaded(data) if data.is_essential_only() => StartupPhase::EssentialDataLoaded,
            LoadState::Loaded(_) => StartupPhase::FullyLoaded,
            LoadState::Error(_) => StartupPhase::UiReady,
        }
    }
    
    pub fn render(&mut self, ctx: &egui::Context) {
        match self.get_startup_phase() {
            StartupPhase::UiReady => {
                // Afficher l'UI de base avec skeleton
                self.render_skeleton_ui(ctx);
            }
            StartupPhase::EssentialDataLoaded => {
                // Afficher les données essentielles
                self.render_with_essential_data(ctx);
            }
            StartupPhase::FullyLoaded => {
                // Afficher tout
                self.render_full_ui(ctx);
            }
        }
    }
}
```

## First paint rapide

```rust
impl FastStartupApp {
    pub fn minimize_startup_time() {
        // 1. Éviter les initialisations lourdes au démarrage
        // ❌ Ne pas faire ça :
        // let db = Database::open("huge.db").unwrap();  // Lent
        
        // ✅ Faire ça :
        // let db = Arc::new(Lazy::new(|| Database::open("huge.db").unwrap()));
        
        // 2. Charger les polices de manière asynchrone
        // 3. Initialiser les composants seulement quand nécessaires
        // 4. Utiliser des valeurs par défaut pour l'UI
    }
}
```

## Résumé

- **Optimisation** : Initialisation minimale au démarrage
- **Chargement progressif** : UI → Données essentielles → Tout
- **First paint** : Afficher quelque chose immédiatement
- **Arrière-plan** : Charger les données lourdes après le premier affichage
