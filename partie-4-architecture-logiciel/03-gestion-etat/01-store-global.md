# 16.1 Store Global

## État global avec ArcSwap

```rust
use std::sync::Arc;
use arc_swap::ArcSwap;

/// État global de l'application
#[derive(Clone, Default)]
pub struct AppState {
    pub user: Option<User>,
    pub documents: Vec<Document>,
    pub settings: Settings,
    pub ui: UiState,
}

/// Store avec accès thread-safe
pub struct Store {
    state: Arc<ArcSwap<AppState>>,
}

impl Store {
    pub fn new() -> Self {
        Self {
            state: Arc::new(ArcSwap::new(Arc::new(AppState::default()))),
        }
    }
    
    /// Lecture (très rapide, sans lock)
    pub fn get(&self) -> Arc<AppState> {
        self.state.load_full()
    }
    
    /// Mise à jour avec une fonction
    pub fn update(&self, f: impl FnOnce(&mut AppState)) {
        let current = self.state.load();
        let mut new_state = (**current).clone();
        f(&mut new_state);
        self.state.store(Arc::new(new_state));
    }
    
    /// Mise à jour atomique (RCU - Read-Copy-Update)
    pub fn update_rcu<F>(&self, f: F)
    where
        F: FnOnce(&AppState) -> AppState,
    {
        self.state.rcu(|state| Arc::new(f(state)));
    }
}
```

## Accès thread-safe

```rust
// Plusieurs threads peuvent lire simultanément
let store = Arc::new(Store::new());

// Thread 1
let store1 = Arc::clone(&store);
std::thread::spawn(move || {
    let state = store1.get();
    println!("User: {:?}", state.user);
});

// Thread 2
let store2 = Arc::clone(&store);
std::thread::spawn(move || {
    let state = store2.get();
    println!("Documents: {}", state.documents.len());
});

// Thread UI peut mettre à jour
store.update(|state| {
    state.user = Some(new_user);
});
```

## Patterns de mise à jour

### Pattern 1 : Update simple

```rust
store.update(|state| {
    state.documents.push(new_document);
});
```

### Pattern 2 : Update conditionnel

```rust
store.update(|state| {
    if let Some(user) = &mut state.user {
        user.last_login = Utc::now();
    }
});
```

### Pattern 3 : Update avec rollback

```rust
let old_state = store.get();
store.update(|state| {
    state.settings.theme = ThemeMode::Dark;
});

// Si erreur, rollback
if error_occurred {
    store.state.store(old_state);
}
```

## Résumé

- **ArcSwap** : Lecture rapide sans lock, écriture atomique
- **Thread-safe** : Plusieurs threads peuvent lire simultanément
- **RCU pattern** : Mise à jour atomique sans bloquer les lecteurs
- **Performance** : Pas de Mutex pour les lectures
