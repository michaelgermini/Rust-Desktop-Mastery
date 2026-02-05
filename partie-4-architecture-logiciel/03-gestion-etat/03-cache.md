# 16.3 Cache

## Stratégies de cache

### Cache LRU simple

```rust
use std::collections::HashMap;
use std::hash::Hash;
use std::time::Instant;

struct CacheEntry<V> {
    value: V,
    last_access: Instant,
}

/// Cache LRU simple
pub struct Cache<K, V> {
    entries: HashMap<K, CacheEntry<V>>,
    max_size: usize,
}

impl<K: Hash + Eq + Clone, V: Clone> Cache<K, V> {
    pub fn new(max_size: usize) -> Self {
        Self {
            entries: HashMap::new(),
            max_size,
        }
    }
    
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
    
    fn evict_oldest(&mut self) {
        let oldest_key = self.entries.iter()
            .min_by_key(|(_, entry)| entry.last_access)
            .map(|(key, _)| key.clone());
        
        if let Some(key) = oldest_key {
            self.entries.remove(&key);
        }
    }
    
    pub fn clear(&mut self) {
        self.entries.clear();
    }
}
```

## Cache avec TTL (Time To Live)

```rust
use std::time::Duration;

struct CacheEntry<V> {
    value: V,
    created_at: Instant,
    ttl: Duration,
}

impl<V> CacheEntry<V> {
    fn is_expired(&self) -> bool {
        self.created_at.elapsed() > self.ttl
    }
}

pub struct TtlCache<K, V> {
    entries: HashMap<K, CacheEntry<V>>,
}

impl<K: Hash + Eq, V> TtlCache<K, V> {
    pub fn get(&mut self, key: &K) -> Option<&V> {
        if let Some(entry) = self.entries.get(key) {
            if entry.is_expired() {
                self.entries.remove(key);
                return None;
            }
            Some(&entry.value)
        } else {
            None
        }
    }
    
    pub fn insert(&mut self, key: K, value: V, ttl: Duration) {
        self.entries.insert(key, CacheEntry {
            value,
            created_at: Instant::now(),
            ttl,
        });
    }
}
```

## Invalidation

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
        self.entries.retain(|_, entry| !entry.is_expired());
    }
}
```

## Usage

```rust
// Cache pour les résultats de recherche
let mut search_cache = Cache::new(100);

fn search_with_cache(query: &str, cache: &mut Cache<String, Vec<SearchResult>>) -> Vec<SearchResult> {
    if let Some(results) = cache.get(query) {
        return results.clone();
    }
    
    let results = perform_search(query);
    cache.insert(query.to_string(), results.clone());
    results
}
```

## Résumé

- **LRU** : Éviction des entrées les moins récemment utilisées
- **TTL** : Expiration automatique après un délai
- **Invalidation** : Suppression manuelle ou conditionnelle
- **Performance** : Éviter les recalculs coûteux
