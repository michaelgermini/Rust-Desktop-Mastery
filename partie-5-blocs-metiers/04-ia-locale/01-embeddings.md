# 23.1 Embeddings

## Génération d'embeddings

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
    
    pub fn embed_batch(&self, texts: &[String]) -> Result<Vec<Vec<f32>>> {
        texts.iter().map(|text| self.embed(text)).collect()
    }
}
```

## Modèles disponibles

### Modèles BERT

```rust
pub enum EmbeddingModelType {
    // Modèles multilingues
    MultilingualBert,
    
    // Modèles français spécialisés
    CamemBert,
    FlauBert,
    
    // Modèles légers
    DistilBert,
}

impl EmbeddingModel {
    pub fn load_preset(model_type: EmbeddingModelType) -> Result<Self> {
        let model_path = match model_type {
            EmbeddingModelType::MultilingualBert => {
                download_model("bert-base-multilingual-cased")?
            }
            EmbeddingModelType::CamemBert => {
                download_model("camembert-base")?
            }
            EmbeddingModelType::DistilBert => {
                download_model("distilbert-base-multilingual-cased")?
            }
            _ => return Err("Model not available".into()),
        };
        
        Self::load(&model_path)
    }
}
```

## Stockage des embeddings

```rust
pub struct EmbeddingStore {
    db: Database,
}

impl EmbeddingStore {
    pub fn save_embedding(&self, document_id: &str, embedding: &[f32]) -> Result<()> {
        // Stocker dans SQLite avec BLOB
        self.db.execute(
            "INSERT OR REPLACE INTO embeddings (id, embedding) VALUES (?1, ?2)",
            params![document_id, embedding],
        )?;
        Ok(())
    }
    
    pub fn load_embedding(&self, document_id: &str) -> Result<Option<Vec<f32>>> {
        self.db.query_row(
            "SELECT embedding FROM embeddings WHERE id = ?1",
            [document_id],
            |row| {
                let blob: Vec<u8> = row.get(0)?;
                // Convertir le BLOB en Vec<f32>
                let embedding: Vec<f32> = unsafe {
                    std::slice::from_raw_parts(
                        blob.as_ptr() as *const f32,
                        blob.len() / 4,
                    ).to_vec()
                };
                Ok(embedding)
            },
        ).optional()
    }
}
```

## Résumé

- **Candle** : Framework ML en Rust pur
- **BERT** : Modèles de transformers pour embeddings
- **Batch processing** : Traiter plusieurs textes à la fois
- **Stockage** : Sauvegarder les embeddings pour réutilisation
