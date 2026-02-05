# 21.4 Performance

## Optimisation des requêtes

```rust
impl SearchEngine {
    pub fn optimize_index(&self) -> Result<()> {
        // Fusionner les segments pour améliorer les performances
        let mut writer = self.index.writer(50_000_000)?;
        writer.merge_segments()?;
        writer.commit()?;
        Ok(())
    }
    
    pub fn search_fast(&self, query: &str, limit: usize) -> Result<Vec<SearchResult>> {
        // Utiliser un reader réutilisable (pool de readers)
        let reader = self.index.reader()?;
        let searcher = reader.searcher();
        
        // Parser la requête une seule fois
        let query_parser = QueryParser::for_index(
            &self.index,
            vec![self.title_field, self.body_field],
        );
        
        let query = query_parser.parse_query(query)?;
        
        // Limiter le nombre de résultats
        let top_docs = searcher.search(&query, &TopDocs::with_limit(limit.min(100)))?;
        
        // ...
    }
}
```

## Cache des résultats

```rust
use std::collections::HashMap;
use std::time::{Duration, Instant};

pub struct CachedSearchEngine {
    engine: SearchEngine,
    cache: HashMap<String, (Vec<SearchResult>, Instant)>,
    cache_ttl: Duration,
}

impl CachedSearchEngine {
    pub fn new(engine: SearchEngine) -> Self {
        Self {
            engine,
            cache: HashMap::new(),
            cache_ttl: Duration::from_secs(60),
        }
    }
    
    pub fn search(&mut self, query: &str, limit: usize) -> Result<Vec<SearchResult>> {
        let cache_key = format!("{}:{}", query, limit);
        
        // Vérifier le cache
        if let Some((results, cached_at)) = self.cache.get(&cache_key) {
            if cached_at.elapsed() < self.cache_ttl {
                return Ok(results.clone());
            }
        }
        
        // Recherche réelle
        let results = self.engine.search(query, limit)?;
        
        // Mettre en cache
        self.cache.insert(cache_key, (results.clone(), Instant::now()));
        
        // Nettoyer le cache expiré
        self.cache.retain(|_, (_, cached_at)| cached_at.elapsed() < self.cache_ttl);
        
        Ok(results)
    }
}
```

## Indexation asynchrone

```rust
use tokio::sync::mpsc;

pub struct AsyncIndexer {
    engine: Arc<SearchEngine>,
    tx: mpsc::UnboundedSender<IndexCommand>,
}

impl AsyncIndexer {
    pub fn new(engine: Arc<SearchEngine>) -> Self {
        let (tx, mut rx) = mpsc::unbounded_channel();
        let engine_clone = Arc::clone(&engine);
        
        tokio::spawn(async move {
            while let Some(cmd) = rx.recv().await {
                match cmd {
                    IndexCommand::Index { id, title, body } => {
                        engine_clone.index_document(&id, &title, &body).ok();
                    }
                    IndexCommand::Delete { id } => {
                        engine_clone.delete_document(&id).ok();
                    }
                    IndexCommand::Rebuild => {
                        engine_clone.rebuild_index().ok();
                    }
                }
            }
        });
        
        Self { engine, tx }
    }
    
    pub fn queue_index(&self, id: String, title: String, body: String) {
        self.tx.send(IndexCommand::Index { id, title, body }).ok();
    }
}
```

## Résumé

- **Optimisation** : Fusionner les segments régulièrement
- **Cache** : Mettre en cache les résultats fréquents
- **Asynchrone** : Indexation non bloquante
- **Limites** : Limiter le nombre de résultats pour la performance
