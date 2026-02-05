# 23.2 Recherche Sémantique

## Similarité cosinus

```rust
pub fn cosine_similarity(a: &[f32], b: &[f32]) -> f32 {
    if a.len() != b.len() {
        return 0.0;
    }
    
    let dot: f32 = a.iter().zip(b.iter()).map(|(x, y)| x * y).sum();
    let norm_a: f32 = a.iter().map(|x| x * x).sum::<f32>().sqrt();
    let norm_b: f32 = b.iter().map(|x| x * x).sum::<f32>().sqrt();
    
    if norm_a == 0.0 || norm_b == 0.0 {
        return 0.0;
    }
    
    dot / (norm_a * norm_b)
}
```

## Recherche par similarité

```rust
pub struct SemanticSearch {
    model: EmbeddingModel,
    index: Vec<IndexedDocument>,
}

pub struct IndexedDocument {
    pub id: String,
    pub title: String,
    pub content: String,
    pub embedding: Vec<f32>,
}

impl SemanticSearch {
    pub fn index_document(&mut self, id: &str, title: &str, content: &str) -> Result<()> {
        let text = format!("{} {}", title, content);
        let embedding = self.model.embed(&text)?;
        
        self.index.push(IndexedDocument {
            id: id.to_string(),
            title: title.to_string(),
            content: content.to_string(),
            embedding,
        });
        
        Ok(())
    }
    
    pub fn search(&self, query: &str, limit: usize) -> Result<Vec<SearchResult>> {
        let query_embedding = self.model.embed(query)?;
        
        let mut scored: Vec<_> = self.index.iter()
            .map(|doc| {
                let similarity = cosine_similarity(&query_embedding, &doc.embedding);
                (doc, similarity)
            })
            .collect();
        
        // Trier par similarité décroissante
        scored.sort_by(|a, b| {
            b.1.partial_cmp(&a.1).unwrap_or(std::cmp::Ordering::Equal)
        });
        
        Ok(scored.into_iter()
            .take(limit)
            .map(|(doc, score)| SearchResult {
                id: doc.id.clone(),
                title: doc.title.clone(),
                score,
            })
            .collect())
    }
}
```

## Ranking sémantique

```rust
pub struct HybridSearch {
    full_text: SearchEngine,
    semantic: SemanticSearch,
    weights: SearchWeights,
}

pub struct SearchWeights {
    pub full_text: f32,
    pub semantic: f32,
}

impl HybridSearch {
    pub fn search(&self, query: &str, limit: usize) -> Result<Vec<SearchResult>> {
        // Recherche full-text
        let full_text_results = self.full_text.search(query, limit * 2)?;
        
        // Recherche sémantique
        let semantic_results = self.semantic.search(query, limit * 2)?;
        
        // Combiner et reranker
        let mut combined: HashMap<String, CombinedScore> = HashMap::new();
        
        for result in full_text_results {
            let entry = combined.entry(result.id.clone()).or_insert(CombinedScore {
                id: result.id.clone(),
                title: result.title.clone(),
                full_text_score: 0.0,
                semantic_score: 0.0,
            });
            entry.full_text_score = result.score;
        }
        
        for result in semantic_results {
            let entry = combined.entry(result.id.clone()).or_insert(CombinedScore {
                id: result.id.clone(),
                title: result.title.clone(),
                full_text_score: 0.0,
                semantic_score: 0.0,
            });
            entry.semantic_score = result.score;
        }
        
        // Calculer le score combiné
        let mut final_results: Vec<_> = combined.into_values()
            .map(|mut score| {
                score.combined_score = 
                    score.full_text_score * self.weights.full_text +
                    score.semantic_score * self.weights.semantic;
                score
            })
            .collect();
        
        final_results.sort_by(|a, b| {
            b.combined_score.partial_cmp(&a.combined_score)
                .unwrap_or(std::cmp::Ordering::Equal)
        });
        
        Ok(final_results.into_iter()
            .take(limit)
            .map(|s| SearchResult {
                id: s.id,
                title: s.title,
                score: s.combined_score,
            })
            .collect())
    }
}
```

## Résumé

- **Similarité cosinus** : Mesure de similarité entre vecteurs
- **Recherche sémantique** : Trouver par sens, pas par mots-clés
- **Hybrid search** : Combiner full-text et sémantique
- **Ranking** : Score combiné pour meilleurs résultats
