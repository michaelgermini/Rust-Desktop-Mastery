# Chapitre 25 : Performance Perçue

## Introduction

La performance perçue est ce que l'utilisateur ressent, pas ce que les benchmarks mesurent. Une application peut être techniquement rapide mais sembler lente si elle ne donne pas de feedback.

---

## 25.1 Démarrage Instantané

```rust
/// Stratégie de chargement différé
pub struct LazyApp {
    // Chargé immédiatement (essentiel pour le premier écran)
    ui_state: UiState,
    theme: Theme,
    
    // Chargé en arrière-plan
    data: LoadState<AppData>,
    search_index: LoadState<SearchIndex>,
}

impl LazyApp {
    pub fn new() -> Self {
        Self {
            // L'UI est prête immédiatement
            ui_state: UiState::default(),
            theme: Theme::default(),
            
            // Les données seront chargées async
            data: LoadState::Loading { started_at: Instant::now() },
            search_index: LoadState::Idle,
        }
    }
    
    pub fn start_background_loading(&self, runtime: &Runtime) {
        let data_tx = self.data_channel.clone();
        
        runtime.spawn(async move {
            // Charger les données essentielles d'abord
            let recent_docs = load_recent_documents().await;
            data_tx.send(DataEvent::RecentLoaded(recent_docs)).ok();
            
            // Puis les données secondaires
            let all_docs = load_all_documents().await;
            data_tx.send(DataEvent::AllLoaded(all_docs)).ok();
        });
    }
}
```

---

## 25.2 Skeleton Loading

```rust
pub fn render_with_skeleton<T>(
    ui: &mut Ui,
    state: &LoadState<T>,
    tokens: &DesignTokens,
    render_skeleton: impl FnOnce(&mut Ui),
    render_content: impl FnOnce(&mut Ui, &T),
) {
    match state {
        LoadState::Loading { started_at } => {
            // Afficher skeleton seulement après un court délai
            if started_at.elapsed() > Duration::from_millis(100) {
                render_skeleton(ui);
                ui.ctx().request_repaint();
            }
        }
        LoadState::Loaded(data) => {
            render_content(ui, data);
        }
        LoadState::Error(msg) => {
            ui.colored_label(tokens.colors.error, msg);
        }
        LoadState::Idle => {}
    }
}

pub fn skeleton_list(ui: &mut Ui, count: usize, tokens: &DesignTokens) {
    for _ in 0..count {
        skeleton_row(ui, tokens);
        ui.add_space(tokens.spacing.xs);
    }
}

pub fn skeleton_row(ui: &mut Ui, tokens: &DesignTokens) {
    ui.horizontal(|ui| {
        // Avatar skeleton
        skeleton_circle(ui, 32.0, tokens);
        ui.add_space(tokens.spacing.sm);
        
        // Text skeletons
        ui.vertical(|ui| {
            skeleton_rect(ui, 120.0, 14.0, tokens);
            ui.add_space(4.0);
            skeleton_rect(ui, 80.0, 12.0, tokens);
        });
    });
}

fn skeleton_rect(ui: &mut Ui, width: f32, height: f32, tokens: &DesignTokens) {
    let (rect, _) = ui.allocate_exact_size(egui::vec2(width, height), egui::Sense::hover());
    
    // Animation de pulsation
    let t = ui.ctx().input(|i| i.time) as f32;
    let alpha = 0.6 + 0.4 * (t * 2.0).sin();
    
    ui.painter().rect_filled(
        rect,
        tokens.radius.sm,
        tokens.colors.bg_tertiary.linear_multiply(alpha),
    );
}
```

---

## 25.3 Caching Intelligent

```rust
use std::collections::HashMap;
use std::time::{Duration, Instant};

pub struct Cache<K, V> {
    entries: HashMap<K, CacheEntry<V>>,
    ttl: Duration,
    max_size: usize,
}

struct CacheEntry<V> {
    value: V,
    created_at: Instant,
    last_accessed: Instant,
}

impl<K: Hash + Eq, V: Clone> Cache<K, V> {
    pub fn new(ttl: Duration, max_size: usize) -> Self {
        Self {
            entries: HashMap::new(),
            ttl,
            max_size,
        }
    }
    
    pub fn get(&mut self, key: &K) -> Option<V> {
        if let Some(entry) = self.entries.get_mut(key) {
            if entry.created_at.elapsed() < self.ttl {
                entry.last_accessed = Instant::now();
                return Some(entry.value.clone());
            }
            // Expiré, supprimer
            self.entries.remove(key);
        }
        None
    }
    
    pub fn insert(&mut self, key: K, value: V) {
        // Éviction si nécessaire
        if self.entries.len() >= self.max_size {
            self.evict_lru();
        }
        
        self.entries.insert(key, CacheEntry {
            value,
            created_at: Instant::now(),
            last_accessed: Instant::now(),
        });
    }
    
    fn evict_lru(&mut self) {
        if let Some(oldest_key) = self.entries.iter()
            .min_by_key(|(_, v)| v.last_accessed)
            .map(|(k, _)| k.clone())
        {
            self.entries.remove(&oldest_key);
        }
    }
}
```

---

## 25.4 Optimistic Updates

```rust
/// Appliquer immédiatement, rollback si erreur
pub struct OptimisticUpdater<T: Clone> {
    pending: HashMap<Uuid, PendingUpdate<T>>,
}

struct PendingUpdate<T> {
    original: T,
    optimistic: T,
    rollback_fn: Box<dyn FnOnce()>,
}

impl<T: Clone> OptimisticUpdater<T> {
    pub async fn update<F, Fut>(
        &mut self,
        state: &mut T,
        apply: impl FnOnce(&mut T),
        persist: F,
    ) -> Result<(), Error>
    where
        F: FnOnce(T) -> Fut,
        Fut: Future<Output = Result<(), Error>>,
    {
        let update_id = Uuid::new_v4();
        let original = state.clone();
        
        // 1. Appliquer immédiatement (optimistic)
        apply(state);
        let optimistic = state.clone();
        
        // 2. Persister en arrière-plan
        match persist(optimistic.clone()).await {
            Ok(_) => {
                // Succès, rien à faire
                Ok(())
            }
            Err(e) => {
                // Échec, rollback
                *state = original;
                Err(e)
            }
        }
    }
}
```

---

## Résumé

- **Démarrage différé** : UI visible immédiatement, données en arrière-plan
- **Skeleton loading** : Feedback visuel pendant le chargement
- **Cache LRU** : Éviter les rechargements inutiles
- **Optimistic updates** : Feedback instantané, rollback si erreur
