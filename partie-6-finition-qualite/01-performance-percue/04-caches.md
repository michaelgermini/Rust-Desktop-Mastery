# 25.4 Caches

## Stratégies de cache

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

## LRU cache

```rust
use std::collections::HashMap;
use std::hash::Hash;

pub struct LRUCache<K, V> {
    entries: HashMap<K, (V, Instant)>,
    max_size: usize,
}

impl<K: Hash + Eq + Clone, V: Clone> LRUCache<K, V> {
    pub fn new(max_size: usize) -> Self {
        Self {
            entries: HashMap::new(),
            max_size,
        }
    }
    
    pub fn get(&mut self, key: &K) -> Option<V> {
        if let Some((value, _)) = self.entries.remove(key) {
            // Remettre en fin (most recently used)
            self.entries.insert(key.clone(), (value.clone(), Instant::now()));
            Some(value)
        } else {
            None
        }
    }
    
    pub fn insert(&mut self, key: K, value: V) {
        // Si la clé existe déjà, la mettre à jour
        if self.entries.contains_key(&key) {
            self.entries.insert(key, (value, Instant::now()));
            return;
        }
        
        // Si le cache est plein, évincer le moins récemment utilisé
        if self.entries.len() >= self.max_size {
            self.evict_lru();
        }
        
        self.entries.insert(key, (value, Instant::now()));
    }
    
    fn evict_lru(&mut self) {
        // Trouver l'entrée la plus ancienne
        if let Some(oldest_key) = self.entries.iter()
            .min_by_key(|(_, (_, time))| time)
            .map(|(k, _)| k.clone())
        {
            self.entries.remove(&oldest_key);
        }
    }
}
```

## Invalidation intelligente

```rust
impl<K: Hash + Eq, V> Cache<K, V> {
    /// Invalider une clé spécifique
    pub fn invalidate(&mut self, key: &K) {
        self.entries.remove(key);
    }
    
    /// Invalider toutes les clés correspondant à un prédicat
    pub fn invalidate_if<F>(&mut self, predicate: F)
    where
        F: Fn(&K) -> bool,
    {
        self.entries.retain(|k, _| !predicate(k));
    }
    
    /// Invalider toutes les entrées expirées
    pub fn cleanup_expired(&mut self) {
        self.entries.retain(|_, entry| entry.created_at.elapsed() < self.ttl);
    }
    
    /// Invalider par pattern (ex: tous les IDs d'un certain type)
    pub fn invalidate_pattern(&mut self, pattern: impl Fn(&K) -> bool) {
        self.entries.retain(|k, _| !pattern(k));
    }
}
```

## Résumé

- **TTL** : Expiration automatique après un délai
- **LRU** : Éviction des entrées les moins récemment utilisées
- **Invalidation** : Suppression manuelle ou conditionnelle
- **Performance** : Éviter les recalculs coûteux
