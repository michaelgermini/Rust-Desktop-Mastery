# 23.3 Inférence CPU

## Modèles quantifiés

Les modèles quantifiés réduisent la précision (f32 → f16 ou int8) pour accélérer l'inférence sur CPU.

```rust
pub struct QuantizedModel {
    model: QuantizedBertModel,
    tokenizer: Tokenizer,
}

impl QuantizedModel {
    pub fn load_quantized(model_path: &Path) -> Result<Self> {
        // Charger un modèle quantifié (plus léger et plus rapide)
        // Les poids sont en int8 au lieu de f32
        // ...
    }
    
    pub fn embed(&self, text: &str) -> Result<Vec<f32>> {
        // Inférence avec modèle quantifié
        // Conversion automatique en f32 pour les résultats
        // ...
    }
}
```

## Performance CPU

```rust
use std::time::Instant;

impl EmbeddingModel {
    pub fn benchmark(&self, texts: &[String]) -> BenchmarkResult {
        let start = Instant::now();
        
        for text in texts {
            self.embed(text).ok();
        }
        
        let elapsed = start.elapsed();
        let avg_time = elapsed / texts.len() as u32;
        
        BenchmarkResult {
            total_time: elapsed,
            avg_time_per_text: avg_time,
            texts_per_second: texts.len() as f64 / elapsed.as_secs_f64(),
        }
    }
}

pub struct BenchmarkResult {
    pub total_time: Duration,
    pub avg_time_per_text: Duration,
    pub texts_per_second: f64,
}
```

## Optimisations

### Batch processing

```rust
impl EmbeddingModel {
    pub fn embed_batch_optimized(&self, texts: &[String], batch_size: usize) -> Result<Vec<Vec<f32>>> {
        let mut results = Vec::new();
        
        for chunk in texts.chunks(batch_size) {
            // Traiter par batch pour meilleure utilisation CPU
            let batch_embeddings = self.process_batch(chunk)?;
            results.extend(batch_embeddings);
        }
        
        Ok(results)
    }
    
    fn process_batch(&self, texts: &[String]) -> Result<Vec<Vec<f32>>> {
        // Tokeniser tous les textes
        let encodings: Vec<_> = texts.iter()
            .map(|t| self.tokenizer.encode(t, true))
            .collect::<Result<Vec<_>, _>>()?;
        
        // Créer un tensor batch
        let max_len = encodings.iter().map(|e| e.len()).max().unwrap_or(0);
        // Padding et création du batch tensor...
        
        // Inférence batch (plus efficace)
        // ...
    }
}
```

### Cache des embeddings

```rust
pub struct CachedEmbeddingModel {
    model: EmbeddingModel,
    cache: HashMap<String, Vec<f32>>,
}

impl CachedEmbeddingModel {
    pub fn embed(&mut self, text: &str) -> Result<Vec<f32>> {
        // Vérifier le cache
        if let Some(embedding) = self.cache.get(text) {
            return Ok(embedding.clone());
        }
        
        // Calculer et mettre en cache
        let embedding = self.model.embed(text)?;
        self.cache.insert(text.to_string(), embedding.clone());
        
        Ok(embedding)
    }
}
```

## Résumé

- **Modèles quantifiés** : Plus légers et plus rapides
- **Batch processing** : Traiter plusieurs textes ensemble
- **Cache** : Éviter les recalculs
- **Benchmark** : Mesurer les performances
