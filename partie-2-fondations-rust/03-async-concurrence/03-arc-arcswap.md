# 6.3 Arc et ArcSwap

## Arc : Atomic Reference Counting

### Partage de données immutables

```rust
use std::sync::Arc;

fn main() {
    let data = Arc::new(vec![1, 2, 3]);
    
    let data_clone = Arc::clone(&data);
    std::thread::spawn(move || {
        println!("{:?}", data_clone);  // ✅ OK
    });
    
    println!("{:?}", data);  // ✅ OK aussi
}
```

### Arc + Mutex : lecture ET écriture partagée

```rust
use std::sync::{Arc, Mutex};

#[derive(Default)]
pub struct SharedState {
    pub counter: i32,
    pub messages: Vec<String>,
}

pub struct App {
    state: Arc<Mutex<SharedState>>,
}

impl App {
    pub fn new() -> Self {
        Self {
            state: Arc::new(Mutex::new(SharedState::default())),
        }
    }
    
    pub fn increment(&self) {
        let mut state = self.state.lock().unwrap();
        state.counter += 1;
    }
    
    pub fn spawn_worker(&self) {
        let state = Arc::clone(&self.state);
        
        std::thread::spawn(move || {
            // Le worker peut modifier l'état
            let mut state = state.lock().unwrap();
            state.messages.push("Message from worker".to_string());
        });
    }
}
```

## ArcSwap : échange atomique sans lock

### Le problème de Mutex

```rust
// ❌ Mutex bloque même pour la lecture
let state = Arc<Mutex<AppState>>;
let state = state.lock().unwrap();  // Bloque si quelqu'un écrit
let value = state.some_field;       // Lecture simple mais bloquée
```

### Solution : ArcSwap

```toml
# Cargo.toml
[dependencies]
arc-swap = "1.7"
```

```rust
use arc_swap::ArcSwap;
use std::sync::Arc;

pub struct AppState {
    pub users: Vec<User>,
    pub settings: Settings,
}

pub struct App {
    // État partagé, lecture sans lock
    state: Arc<ArcSwap<AppState>>,
}

impl App {
    pub fn new() -> Self {
        Self {
            state: Arc::new(ArcSwap::from_pointee(AppState {
                users: vec![],
                settings: Settings::default(),
            })),
        }
    }
    
    /// Lecture sans lock (très rapide)
    pub fn get_users(&self) -> Vec<User> {
        let state = self.state.load();
        state.users.clone()  // Clone de la référence Arc
    }
    
    /// Mise à jour atomique
    pub fn update_state(&self, new_state: AppState) {
        self.state.store(Arc::new(new_state));
    }
    
    /// Mise à jour avec transformation
    pub fn add_user(&self, user: User) {
        self.state.rcu(|state| {
            let mut new_state = (**state).clone();
            new_state.users.push(user);
            Arc::new(new_state)
        });
    }
}
```

### Pattern : Store global avec ArcSwap

```rust
use arc_swap::ArcSwap;
use std::sync::Arc;

pub struct GlobalStore {
    state: Arc<ArcSwap<AppState>>,
}

impl GlobalStore {
    pub fn new() -> Self {
        Self {
            state: Arc::new(ArcSwap::from_pointee(AppState::default())),
        }
    }
    
    /// Lecture rapide depuis n'importe quel thread
    pub fn read(&self) -> Arc<AppState> {
        self.state.load_full()
    }
    
    /// Écriture atomique
    pub fn write(&self, new_state: AppState) {
        self.state.store(Arc::new(new_state));
    }
    
    /// Mise à jour fonctionnelle
    pub fn update<F>(&self, f: F)
    where
        F: FnOnce(&AppState) -> AppState,
    {
        self.state.rcu(|state| Arc::new(f(state)));
    }
}
```

## Comparaison des approches

| Approche | Lecture | Écriture | Performance |
|----------|---------|----------|-------------|
| `Arc<Mutex<T>>` | Lock requis | Lock requis | Plus lent |
| `Arc<ArcSwap<T>>` | Pas de lock | Atomic swap | Plus rapide |
| `Arc<RwLock<T>>` | Lock partagé | Lock exclusif | Moyen |

## Résumé

- **Arc** : Partage de données entre threads
- **ArcSwap** : Échange atomique sans lock pour lecture
- **RCU (Read-Copy-Update)** : Pattern pour mises à jour fonctionnelles
- **Idéal pour** : État global lu fréquemment, écrit occasionnellement
