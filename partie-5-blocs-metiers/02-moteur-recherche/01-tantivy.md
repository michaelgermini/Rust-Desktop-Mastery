# 21.1 Tantivy

## Introduction à Tantivy

Tantivy est un moteur de recherche full-text écrit en Rust, inspiré de Lucene. Il offre des performances excellentes pour la recherche locale.

## Configuration du schéma

```rust
use tantivy::{schema::*, Index, IndexWriter, Document};

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
        
        // Champ ID (stored pour récupération)
        let id_field = schema_builder.add_text_field("id", STRING | STORED);
        
        // Champ titre (indexed + stored)
        let title_field = schema_builder.add_text_field("title", TEXT | STORED);
        
        // Champ contenu (indexed seulement)
        let body_field = schema_builder.add_text_field("body", TEXT);
        
        let schema = schema_builder.build();
        
        let index = if index_path.exists() {
            Index::open_in_dir(index_path)?
        } else {
            std::fs::create_dir_all(index_path)?;
            Index::create_in_dir(index_path, schema.clone())?
        };
        
        Ok(Self {
            index,
            schema,
            title_field,
            body_field,
            id_field,
        })
    }
}
```

## Indexation de documents

```rust
impl SearchEngine {
    pub fn index_document(&self, id: &str, title: &str, body: &str) -> Result<()> {
        let mut writer = self.index.writer(50_000_000)?;  // Buffer de 50MB
        
        let mut doc = Document::new();
        doc.add_text(self.id_field, id);
        doc.add_text(self.title_field, title);
        doc.add_text(self.body_field, body);
        
        writer.add_document(doc)?;
        writer.commit()?;
        
        Ok(())
    }
    
    pub fn delete_document(&self, id: &str) -> Result<()> {
        let mut writer = self.index.writer(50_000_000)?;
        
        let term = Term::from_field_text(self.id_field, id);
        writer.delete_term(term);
        writer.commit()?;
        
        Ok(())
    }
}
```

## Recherche

```rust
use tantivy::collector::TopDocs;
use tantivy::query::QueryParser;

impl SearchEngine {
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
            
            results.push(SearchResult {
                id,
                title,
                score,
            });
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

## Résumé

- **Tantivy** : Moteur de recherche Rust performant
- **Schéma** : Définir les champs indexés
- **Indexation** : Ajouter des documents à l'index
- **Recherche** : Requêtes avec scoring
