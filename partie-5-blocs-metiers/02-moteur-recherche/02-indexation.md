# 21.2 Indexation

## Indexation initiale

```rust
impl SearchEngine {
    pub fn index_all_documents(&self, documents: &[Document]) -> Result<()> {
        let mut writer = self.index.writer(50_000_000)?;
        
        for doc in documents {
            let mut tantivy_doc = Document::new();
            tantivy_doc.add_text(self.id_field, &doc.id);
            tantivy_doc.add_text(self.title_field, &doc.title);
            tantivy_doc.add_text(self.body_field, &doc.content);
            
            writer.add_document(tantivy_doc)?;
        }
        
        writer.commit()?;
        Ok(())
    }
}
```

## Indexation incrémentale

```rust
pub struct IncrementalIndexer {
    engine: Arc<SearchEngine>,
    last_indexed: DateTime<Utc>,
}

impl IncrementalIndexer {
    pub fn index_new_documents(&mut self, documents: &[Document]) -> Result<()> {
        let new_docs: Vec<_> = documents
            .iter()
            .filter(|doc| doc.updated_at > self.last_indexed)
            .collect();
        
        for doc in new_docs {
            self.engine.index_document(&doc.id, &doc.title, &doc.content)?;
        }
        
        if let Some(latest) = documents.iter().map(|d| d.updated_at).max() {
            self.last_indexed = latest;
        }
        
        Ok(())
    }
}
```

## Indexation en arrière-plan

```rust
use std::sync::mpsc;

pub struct BackgroundIndexer {
    engine: Arc<SearchEngine>,
    tx: mpsc::Sender<IndexCommand>,
}

enum IndexCommand {
    Index { id: String, title: String, body: String },
    Delete { id: String },
    Rebuild,
}

impl BackgroundIndexer {
    pub fn new(engine: Arc<SearchEngine>) -> Self {
        let (tx, rx) = mpsc::channel(1000);
        let engine_clone = Arc::clone(&engine);
        
        std::thread::spawn(move || {
            while let Ok(cmd) = rx.recv() {
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
    
    pub fn queue_delete(&self, id: String) {
        self.tx.send(IndexCommand::Delete { id }).ok();
    }
}
```

## Résumé

- **Initiale** : Indexer tous les documents au démarrage
- **Incrémentale** : Indexer seulement les nouveaux/modifiés
- **Arrière-plan** : Thread dédié pour ne pas bloquer l'UI
- **Queue** : Commandes mises en file d'attente
