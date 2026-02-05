# 2.3 IA Locale, SQLite, Fichiers

## La révolution de l'IA locale

### Pourquoi l'IA locale devient viable

```rust
/// Évolution des capacités d'inférence locale
pub struct LocalAIEvolution {
    pub year: u32,
    pub typical_model_size: &'static str,
    pub inference_speed: &'static str,
    pub hardware_required: &'static str,
}

const AI_EVOLUTION: &[LocalAIEvolution] = &[
    LocalAIEvolution {
        year: 2020,
        typical_model_size: "GPT-2 (1.5B params)",
        inference_speed: "~10 tokens/sec sur GPU",
        hardware_required: "GPU dédié obligatoire",
    },
    LocalAIEvolution {
        year: 2023,
        typical_model_size: "Llama 2 7B quantized",
        inference_speed: "~20 tokens/sec sur CPU moderne",
        hardware_required: "CPU récent ou GPU intégré",
    },
    LocalAIEvolution {
        year: 2024,
        typical_model_size: "Phi-3 (3.8B) / Gemma 2B",
        inference_speed: "~50 tokens/sec sur CPU",
        hardware_required: "Laptop standard",
    },
];
```

### Applications pratiques de l'IA locale

```rust
/// Cas d'usage d'IA locale dans une app desktop
pub enum LocalAIUseCase {
    /// Recherche sémantique dans les documents
    SemanticSearch {
        model: &'static str,
        latency_ms: u32,
    },
    
    /// Classification automatique
    DocumentClassification {
        model: &'static str,
        accuracy: f32,
    },
    
    /// Extraction d'entités (noms, dates, montants)
    EntityExtraction {
        model: &'static str,
        entities: Vec<&'static str>,
    },
    
    /// Résumé automatique
    Summarization {
        model: &'static str,
        max_tokens: u32,
    },
    
    /// Complétion intelligente
    SmartCompletion {
        model: &'static str,
        context_window: u32,
    },
}

impl LocalAIUseCase {
    pub fn semantic_search_example() -> Self {
        Self::SemanticSearch {
            model: "all-MiniLM-L6-v2 (80MB)",
            latency_ms: 50,  // Par requête
        }
    }
}
```

### Implémentation avec Candle

```rust
use candle_core::{Device, Tensor};
use candle_nn::VarBuilder;
use candle_transformers::models::bert::BertModel;

pub struct LocalEmbeddings {
    model: BertModel,
    tokenizer: tokenizers::Tokenizer,
    device: Device,
}

impl LocalEmbeddings {
    pub fn new() -> Result<Self> {
        // Charger le modèle depuis les fichiers locaux
        let device = Device::Cpu;  // Ou Cuda si disponible
        
        let model = BertModel::load(
            "models/minilm-l6-v2.safetensors",
            &device,
        )?;
        
        let tokenizer = tokenizers::Tokenizer::from_file(
            "models/tokenizer.json"
        )?;
        
        Ok(Self { model, tokenizer, device })
    }
    
    /// Générer un embedding pour un texte
    pub fn embed(&self, text: &str) -> Result<Vec<f32>> {
        let tokens = self.tokenizer.encode(text, true)?;
        let input_ids = Tensor::new(tokens.get_ids(), &self.device)?;
        
        let embeddings = self.model.forward(&input_ids)?;
        
        // Mean pooling
        let pooled = embeddings.mean(1)?;
        
        Ok(pooled.to_vec1()?)
    }
    
    /// Recherche sémantique
    pub fn search(&self, query: &str, documents: &[Document]) -> Vec<SearchResult> {
        let query_embedding = self.embed(query).unwrap();
        
        let mut results: Vec<_> = documents.iter()
            .map(|doc| {
                let similarity = cosine_similarity(&query_embedding, &doc.embedding);
                SearchResult { document: doc.clone(), score: similarity }
            })
            .collect();
        
        results.sort_by(|a, b| b.score.partial_cmp(&a.score).unwrap());
        results
    }
}
```

## SQLite : la base de données parfaite pour le desktop

### Pourquoi SQLite

```rust
/// Avantages de SQLite pour les apps desktop
pub struct SQLiteAdvantages {
    pub advantages: Vec<Advantage>,
}

impl Default for SQLiteAdvantages {
    fn default() -> Self {
        Self {
            advantages: vec![
                Advantage {
                    name: "Zero configuration",
                    description: "Pas de serveur, pas d'installation",
                },
                Advantage {
                    name: "Un seul fichier",
                    description: "Toute la DB dans un .db portable",
                },
                Advantage {
                    name: "ACID complet",
                    description: "Transactions fiables même en cas de crash",
                },
                Advantage {
                    name: "Performance",
                    description: "Plus rapide que filesystem pour la plupart des cas",
                },
                Advantage {
                    name: "Requêtes SQL complètes",
                    description: "Joins, sous-requêtes, indexes, FTS5",
                },
                Advantage {
                    name: "Stable et testé",
                    description: "Utilisé par Firefox, Chrome, iOS, Android...",
                },
            ],
        }
    }
}
```

### Configuration optimale pour desktop

```rust
use rusqlite::{Connection, OpenFlags};

pub fn create_optimized_connection(path: &Path) -> Result<Connection> {
    let conn = Connection::open_with_flags(
        path,
        OpenFlags::SQLITE_OPEN_READ_WRITE
            | OpenFlags::SQLITE_OPEN_CREATE
            | OpenFlags::SQLITE_OPEN_NO_MUTEX,
    )?;
    
    // Optimisations pour desktop
    conn.execute_batch("
        -- Write-Ahead Logging pour performance et concurrence
        PRAGMA journal_mode = WAL;
        
        -- Synchronisation réduite (sûr avec WAL)
        PRAGMA synchronous = NORMAL;
        
        -- Cache en mémoire (64MB)
        PRAGMA cache_size = -64000;
        
        -- Stocker les temp en mémoire
        PRAGMA temp_store = MEMORY;
        
        -- Vérification des foreign keys
        PRAGMA foreign_keys = ON;
        
        -- Vacuum automatique
        PRAGMA auto_vacuum = INCREMENTAL;
    ")?;
    
    Ok(conn)
}
```

### Full-Text Search intégré

```rust
pub fn setup_fts5(conn: &Connection) -> Result<()> {
    // Créer une table FTS5 pour la recherche full-text
    conn.execute("
        CREATE VIRTUAL TABLE IF NOT EXISTS documents_fts USING fts5(
            title,
            content,
            content=documents,
            content_rowid=id
        );
    ", [])?;
    
    // Trigger pour maintenir l'index à jour
    conn.execute_batch("
        CREATE TRIGGER IF NOT EXISTS documents_ai AFTER INSERT ON documents BEGIN
            INSERT INTO documents_fts(rowid, title, content) 
            VALUES (new.id, new.title, new.content);
        END;
        
        CREATE TRIGGER IF NOT EXISTS documents_ad AFTER DELETE ON documents BEGIN
            INSERT INTO documents_fts(documents_fts, rowid, title, content) 
            VALUES ('delete', old.id, old.title, old.content);
        END;
        
        CREATE TRIGGER IF NOT EXISTS documents_au AFTER UPDATE ON documents BEGIN
            INSERT INTO documents_fts(documents_fts, rowid, title, content) 
            VALUES ('delete', old.id, old.title, old.content);
            INSERT INTO documents_fts(rowid, title, content) 
            VALUES (new.id, new.title, new.content);
        END;
    ")?;
    
    Ok(())
}

pub fn search_fts(conn: &Connection, query: &str) -> Result<Vec<SearchResult>> {
    let mut stmt = conn.prepare("
        SELECT d.id, d.title, snippet(documents_fts, 1, '<b>', '</b>', '...', 32) as snippet,
               bm25(documents_fts) as rank
        FROM documents_fts
        JOIN documents d ON d.id = documents_fts.rowid
        WHERE documents_fts MATCH ?
        ORDER BY rank
        LIMIT 20
    ")?;
    
    let results = stmt.query_map([query], |row| {
        Ok(SearchResult {
            id: row.get(0)?,
            title: row.get(1)?,
            snippet: row.get(2)?,
            score: row.get(3)?,
        })
    })?;
    
    results.collect()
}
```

## Gestion de fichiers

### Accès direct au système de fichiers

```rust
use std::path::PathBuf;
use notify::{Watcher, RecursiveMode, watcher};

/// Gestionnaire de fichiers avec watch
pub struct FileManager {
    watched_dirs: Vec<PathBuf>,
    watcher: notify::RecommendedWatcher,
}

impl FileManager {
    pub fn new() -> Result<Self> {
        let (tx, rx) = std::sync::mpsc::channel();
        
        let watcher = watcher(tx, Duration::from_secs(1))?;
        
        // Thread de surveillance
        std::thread::spawn(move || {
            for event in rx {
                match event {
                    Ok(notify::Event { kind, paths, .. }) => {
                        handle_file_event(kind, paths);
                    }
                    Err(e) => eprintln!("Watch error: {:?}", e),
                }
            }
        });
        
        Ok(Self {
            watched_dirs: Vec::new(),
            watcher,
        })
    }
    
    /// Surveiller un dossier pour les changements
    pub fn watch_directory(&mut self, path: &Path) -> Result<()> {
        self.watcher.watch(path, RecursiveMode::Recursive)?;
        self.watched_dirs.push(path.to_path_buf());
        Ok(())
    }
}

/// Dialogue de sélection de fichier natif
pub fn select_file(title: &str, filters: &[(&str, &[&str])]) -> Option<PathBuf> {
    native_dialog::FileDialog::new()
        .set_title(title)
        .add_filter(filters[0].0, filters[0].1)
        .show_open_single_file()
        .ok()
        .flatten()
}

/// Dialogue de sauvegarde
pub fn save_file(title: &str, default_name: &str) -> Option<PathBuf> {
    native_dialog::FileDialog::new()
        .set_title(title)
        .set_filename(default_name)
        .show_save_single_file()
        .ok()
        .flatten()
}
```

### Intégration avec les formats courants

```rust
/// Lecture de différents formats de fichiers
pub struct FileParser;

impl FileParser {
    /// Extraire le texte d'un PDF
    pub fn extract_pdf_text(path: &Path) -> Result<String> {
        let bytes = std::fs::read(path)?;
        let doc = pdf_extract::extract_text(&bytes)?;
        Ok(doc)
    }
    
    /// Parser un CSV
    pub fn parse_csv<T: DeserializeOwned>(path: &Path) -> Result<Vec<T>> {
        let mut reader = csv::Reader::from_path(path)?;
        reader.deserialize().collect()
    }
    
    /// Lire un JSON
    pub fn read_json<T: DeserializeOwned>(path: &Path) -> Result<T> {
        let content = std::fs::read_to_string(path)?;
        Ok(serde_json::from_str(&content)?)
    }
    
    /// Détecter le type et parser
    pub fn auto_parse(path: &Path) -> Result<ParsedContent> {
        match path.extension().and_then(|e| e.to_str()) {
            Some("pdf") => Ok(ParsedContent::Text(Self::extract_pdf_text(path)?)),
            Some("csv") => Ok(ParsedContent::Table(Self::parse_csv(path)?)),
            Some("json") => Ok(ParsedContent::Json(Self::read_json(path)?)),
            Some("md") | Some("txt") => {
                Ok(ParsedContent::Text(std::fs::read_to_string(path)?))
            }
            _ => Err(Error::UnsupportedFormat),
        }
    }
}
```

## Synergie des trois technologies

```rust
/// Application combinant IA locale, SQLite et fichiers
pub struct SmartDocumentApp {
    db: Connection,
    embeddings: LocalEmbeddings,
    file_manager: FileManager,
}

impl SmartDocumentApp {
    /// Import intelligent d'un fichier
    pub async fn import_file(&mut self, path: &Path) -> Result<DocumentId> {
        // 1. Parser le fichier
        let content = FileParser::auto_parse(path)?;
        
        // 2. Générer l'embedding pour la recherche sémantique
        let embedding = self.embeddings.embed(&content.text())?;
        
        // 3. Stocker dans SQLite
        let id = self.db.execute(
            "INSERT INTO documents (path, content, embedding) VALUES (?, ?, ?)",
            params![path.to_str(), content.text(), embedding.as_bytes()],
        )?;
        
        // 4. Indexer pour full-text search
        // (automatique via trigger FTS5)
        
        Ok(DocumentId(id))
    }
    
    /// Recherche hybride (sémantique + full-text)
    pub fn hybrid_search(&self, query: &str) -> Vec<SearchResult> {
        // Recherche sémantique
        let semantic_results = self.embeddings.search(query, &self.all_docs());
        
        // Recherche full-text
        let fts_results = search_fts(&self.db, query).unwrap_or_default();
        
        // Fusion des résultats avec re-ranking
        merge_and_rerank(semantic_results, fts_results)
    }
}
```

## Conclusion

La combinaison IA locale + SQLite + fichiers offre :

- **IA souveraine** : Embeddings et inférence sans cloud
- **Données structurées** : SQLite pour requêtes complexes
- **Données non-structurées** : Accès direct aux fichiers
- **Recherche puissante** : Full-text + sémantique
- **Offline complet** : Aucune dépendance réseau
