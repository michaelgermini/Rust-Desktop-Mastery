# Chapitre 21 : Moteur de Recherche Local

## Introduction

Tantivy est un moteur de recherche full-text écrit en Rust, inspiré de Lucene. Ce chapitre montre comment l'intégrer pour une recherche instantanée.

---

## 21.1 Configuration Tantivy

```rust
use tantivy::{schema::*, Index, IndexWriter, Document};
use tantivy::collector::TopDocs;
use tantivy::query::QueryParser;

pub struct SearchEngine {
    index: Index,
    schema: Schema,
    title_field: Field,
    body_field: Field,
    id_field: Field,
}

impl SearchEngine {
    pub fn new(index_path: &Path) -> Result<Self> {
        let mut schema_builder = Schema::builder();
        
        let id_field = schema_builder.add_text_field("id", STRING | STORED);
        let title_field = schema_builder.add_text_field("title", TEXT | STORED);
        let body_field = schema_builder.add_text_field("body", TEXT);
        
        let schema = schema_builder.build();
        
        let index = if index_path.exists() {
            Index::open_in_dir(index_path)?
        } else {
            std::fs::create_dir_all(index_path)?;
            Index::create_in_dir(index_path, schema.clone())?
        };
        
        Ok(Self { index, schema, title_field, body_field, id_field })
    }
    
    pub fn index_document(&self, id: &str, title: &str, body: &str) -> Result<()> {
        let mut writer = self.index.writer(50_000_000)?;
        
        let mut doc = Document::new();
        doc.add_text(self.id_field, id);
        doc.add_text(self.title_field, title);
        doc.add_text(self.body_field, body);
        
        writer.add_document(doc)?;
        writer.commit()?;
        
        Ok(())
    }
    
    pub fn search(&self, query: &str, limit: usize) -> Result<Vec<SearchResult>> {
        let reader = self.index.reader()?;
        let searcher = reader.searcher();
        
        let query_parser = QueryParser::for_index(
            &self.index,
            vec![self.title_field, self.body_field],
        );
        
        let query = query_parser.parse_query(query)?;
        let top_docs = searcher.search(&query, &TopDocs::with_limit(limit))?;
        
        let mut results = Vec::new();
        for (score, doc_address) in top_docs {
            let doc = searcher.doc(doc_address)?;
            
            let id = doc.get_first(self.id_field)
                .and_then(|v| v.as_text())
                .unwrap_or("")
                .to_string();
            
            let title = doc.get_first(self.title_field)
                .and_then(|v| v.as_text())
                .unwrap_or("")
                .to_string();
            
            results.push(SearchResult { id, title, score });
        }
        
        Ok(results)
    }
}

pub struct SearchResult {
    pub id: String,
    pub title: String,
    pub score: f32,
}
```

---

## 21.2 Indexation en Arrière-Plan

```rust
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
}
```

---

## 21.3 Interface de Recherche

```rust
pub struct SearchBox {
    query: String,
    results: Vec<SearchResult>,
    selected: usize,
    debouncer: Debouncer,
}

impl SearchBox {
    pub fn render(&mut self, ui: &mut Ui, engine: &SearchEngine, tokens: &DesignTokens) -> Option<String> {
        let mut selected_id = None;
        
        // Input de recherche
        let response = ui.add(
            egui::TextEdit::singleline(&mut self.query)
                .hint_text("🔍 Rechercher...")
        );
        
        // Rechercher avec debounce
        if response.changed() {
            if let Some(query) = self.debouncer.debounce(&self.query, Duration::from_millis(150)) {
                if query.len() >= 2 {
                    self.results = engine.search(&query, 10).unwrap_or_default();
                    self.selected = 0;
                } else {
                    self.results.clear();
                }
            }
        }
        
        // Navigation clavier
        if response.has_focus() {
            if ui.input(|i| i.key_pressed(egui::Key::ArrowDown)) {
                self.selected = (self.selected + 1).min(self.results.len().saturating_sub(1));
            }
            if ui.input(|i| i.key_pressed(egui::Key::ArrowUp)) {
                self.selected = self.selected.saturating_sub(1);
            }
            if ui.input(|i| i.key_pressed(egui::Key::Enter)) && !self.results.is_empty() {
                selected_id = Some(self.results[self.selected].id.clone());
            }
        }
        
        // Afficher les résultats
        if !self.results.is_empty() {
            egui::Frame::popup(ui.style())
                .show(ui, |ui| {
                    for (i, result) in self.results.iter().enumerate() {
                        let is_selected = i == self.selected;
                        
                        if ui.selectable_label(is_selected, &result.title).clicked() {
                            selected_id = Some(result.id.clone());
                        }
                    }
                });
        }
        
        selected_id
    }
}
```

---

## Résumé

- **Tantivy** : Moteur full-text performant en Rust
- **Indexation async** : Ne bloque pas l'UI
- **Recherche instantanée** : Résultats en millisecondes
- **UX fluide** : Debounce et navigation clavier
