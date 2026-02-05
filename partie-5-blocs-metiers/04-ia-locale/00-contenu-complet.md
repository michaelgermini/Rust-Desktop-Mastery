# Chapitre 23 : IA Locale dans une App Rust

## Introduction

L'IA locale permet d'ajouter des fonctionnalités intelligentes sans envoyer de données au cloud. Ce chapitre couvre les embeddings et la recherche sémantique.

---

## 23.1 Embeddings avec Candle

```rust
use candle_core::{Device, Tensor};
use candle_transformers::models::bert::{BertModel, Config};
use tokenizers::Tokenizer;

pub struct EmbeddingModel {
    model: BertModel,
    tokenizer: Tokenizer,
    device: Device,
}

impl EmbeddingModel {
    pub fn load(model_path: &Path) -> Result<Self> {
        let device = Device::Cpu;  // Ou Device::cuda_if_available()
        
        let tokenizer = Tokenizer::from_file(model_path.join("tokenizer.json"))?;
        
        let config: Config = serde_json::from_str(
            &std::fs::read_to_string(model_path.join("config.json"))?
        )?;
        
        let weights = candle_core::safetensors::load(
            model_path.join("model.safetensors"),
            &device,
        )?;
        
        let vb = candle_nn::VarBuilder::from_tensors(weights, candle_core::DType::F32, &device);
        let model = BertModel::load(vb, &config)?;
        
        Ok(Self { model, tokenizer, device })
    }
    
    pub fn embed(&self, text: &str) -> Result<Vec<f32>> {
        // Tokeniser
        let encoding = self.tokenizer.encode(text, true)?;
        let tokens = encoding.get_ids().to_vec();
        
        // Créer le tensor
        let input_ids = Tensor::new(&tokens[..], &self.device)?.unsqueeze(0)?;
        let token_type_ids = input_ids.zeros_like()?;
        
        // Inférence
        let output = self.model.forward(&input_ids, &token_type_ids)?;
        
        // Mean pooling
        let embeddings = output.mean(1)?;
        let vec: Vec<f32> = embeddings.squeeze(0)?.to_vec1()?;
        
        Ok(vec)
    }
}
```

---

## 23.2 Recherche Sémantique

```rust
pub struct SemanticSearch {
    model: EmbeddingModel,
    index: Vec<IndexedDocument>,
}

pub struct IndexedDocument {
    pub id: String,
    pub title: String,
    pub embedding: Vec<f32>,
}

impl SemanticSearch {
    pub fn index_document(&mut self, id: &str, title: &str, content: &str) -> Result<()> {
        let text = format!("{} {}", title, content);
        let embedding = self.model.embed(&text)?;
        
        self.index.push(IndexedDocument {
            id: id.to_string(),
            title: title.to_string(),
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
        scored.sort_by(|a, b| b.1.partial_cmp(&a.1).unwrap_or(std::cmp::Ordering::Equal));
        
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

fn cosine_similarity(a: &[f32], b: &[f32]) -> f32 {
    let dot: f32 = a.iter().zip(b.iter()).map(|(x, y)| x * y).sum();
    let norm_a: f32 = a.iter().map(|x| x * x).sum::<f32>().sqrt();
    let norm_b: f32 = b.iter().map(|x| x * x).sum::<f32>().sqrt();
    dot / (norm_a * norm_b)
}
```

---

## 23.3 Intégration UI

```rust
pub struct SemanticSearchBox {
    query: String,
    results: Vec<SearchResult>,
    loading: bool,
}

impl SemanticSearchBox {
    pub fn render(
        &mut self,
        ui: &mut Ui,
        search: &SemanticSearch,
        tokens: &DesignTokens,
    ) -> Option<String> {
        let mut selected_id = None;
        
        ui.horizontal(|ui| {
            ui.label("🧠");  // Indique IA
            
            let response = ui.add(
                egui::TextEdit::singleline(&mut self.query)
                    .hint_text("Recherche sémantique...")
            );
            
            if response.lost_focus() && ui.input(|i| i.key_pressed(egui::Key::Enter)) {
                self.loading = true;
                
                // Recherche (idéalement en async)
                match search.search(&self.query, 5) {
                    Ok(results) => {
                        self.results = results;
                        self.loading = false;
                    }
                    Err(_) => {
                        self.loading = false;
                    }
                }
            }
        });
        
        // Résultats
        if self.loading {
            ui.spinner();
        } else if !self.results.is_empty() {
            for result in &self.results {
                ui.horizontal(|ui| {
                    // Score de similarité
                    let score_color = if result.score > 0.8 {
                        tokens.colors.success
                    } else if result.score > 0.5 {
                        tokens.colors.warning
                    } else {
                        tokens.colors.text_muted
                    };
                    
                    ui.label(
                        egui::RichText::new(format!("{:.0}%", result.score * 100.0))
                            .size(tokens.typography.size_xs)
                            .color(score_color)
                    );
                    
                    if ui.link(&result.title).clicked() {
                        selected_id = Some(result.id.clone());
                    }
                });
            }
        }
        
        selected_id
    }
}
```

---

## Résumé

- **Candle** : Framework ML en Rust pur
- **Embeddings** : Représentations vectorielles du texte
- **Recherche sémantique** : Trouver par sens, pas par mots-clés
- **100% local** : Aucune donnée envoyée au cloud
