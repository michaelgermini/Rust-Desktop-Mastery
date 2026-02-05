# Chapitre 16 : Gestion d'État Robuste

## Introduction

La gestion de l'état est cruciale pour une application desktop. Un état mal géré cause des bugs, des incohérences et une UX dégradée.

---

## 16.1 Store Global

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
}
```

---

## 16.2 État Local vs Global

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
}

impl InvoiceEditor {
    pub fn new(invoice: Invoice) -> Self {
        Self {
            local_state: InvoiceEditorState {
                draft: invoice,
                validation_errors: Vec::new(),
                show_preview: false,
            },
        }
    }
}
```

---

## 16.3 Cache et Undo Stack

```rust
/// Cache LRU simple
pub struct Cache<K, V> {
    entries: HashMap<K, CacheEntry<V>>,
    max_size: usize,
}

impl<K: Hash + Eq, V: Clone> Cache<K, V> {
    pub fn get(&mut self, key: &K) -> Option<&V> {
        self.entries.get_mut(key).map(|e| {
            e.last_access = Instant::now();
            &e.value
        })
    }
    
    pub fn insert(&mut self, key: K, value: V) {
        if self.entries.len() >= self.max_size {
            self.evict_oldest();
        }
        self.entries.insert(key, CacheEntry {
            value,
            last_access: Instant::now(),
        });
    }
}

/// Undo Stack
pub struct UndoStack<T: Clone> {
    history: Vec<T>,
    position: usize,
}

impl<T: Clone> UndoStack<T> {
    pub fn push(&mut self, state: T) {
        self.history.truncate(self.position + 1);
        self.history.push(state);
        self.position = self.history.len() - 1;
    }
    
    pub fn undo(&mut self) -> Option<&T> {
        if self.position > 0 {
            self.position -= 1;
            Some(&self.history[self.position])
        } else {
            None
        }
    }
    
    pub fn redo(&mut self) -> Option<&T> {
        if self.position < self.history.len() - 1 {
            self.position += 1;
            Some(&self.history[self.position])
        } else {
            None
        }
    }
}
```

---

## Résumé

- **Store global** : État partagé avec ArcSwap pour performance
- **État local** : Propre à chaque écran/composant
- **Cache** : Éviter les recalculs coûteux
- **Undo Stack** : Historique des modifications
