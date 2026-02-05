# 17.1 SQLite

## Configuration SQLite

```rust
use rusqlite::{Connection, params};

pub struct Database {
    conn: Connection,
}

impl Database {
    pub fn open(path: &str) -> Result<Self> {
        let conn = Connection::open(path)?;
        
        // Optimisations SQLite pour desktop
        conn.execute_batch("
            PRAGMA journal_mode = WAL;
            PRAGMA synchronous = NORMAL;
            PRAGMA foreign_keys = ON;
            PRAGMA cache_size = -64000;
            PRAGMA temp_store = MEMORY;
        ")?;
        
        Ok(Self { conn })
    }
    
    pub fn in_memory() -> Result<Self> {
        let conn = Connection::open_in_memory()?;
        Ok(Self { conn })
    }
}
```

## Optimisations pour desktop

### WAL Mode (Write-Ahead Logging)

```rust
// WAL permet des lectures concurrentes pendant les écritures
conn.execute("PRAGMA journal_mode = WAL", [])?;
```

**Avantages** :
- Lectures non bloquantes pendant les écritures
- Meilleures performances
- Récupération plus rapide après crash

### Cache size

```rust
// Cache de 64 MB (en pages de 1 KB)
conn.execute("PRAGMA cache_size = -64000", [])?;
```

### Foreign keys

```rust
// Activer la vérification des clés étrangères
conn.execute("PRAGMA foreign_keys = ON", [])?;
```

## Full-text search (FTS5)

```rust
// Créer une table FTS5
conn.execute("
    CREATE VIRTUAL TABLE documents_fts USING fts5(
        title,
        content,
        content_rowid='rowid'
    )
", [])?;

// Indexer un document
conn.execute("
    INSERT INTO documents_fts(rowid, title, content)
    VALUES (?1, ?2, ?3)
", params![doc_id, title, content])?;

// Rechercher
let mut stmt = conn.prepare("
    SELECT rowid, title, content
    FROM documents_fts
    WHERE documents_fts MATCH ?1
    ORDER BY rank
")?;

let results = stmt.query_map([query], |row| {
    Ok(SearchResult {
        id: row.get(0)?,
        title: row.get(1)?,
        content: row.get(2)?,
    })
})?;
```

## Transactions

```rust
impl Database {
    pub fn transaction<F, T>(&mut self, f: F) -> Result<T>
    where
        F: FnOnce(&mut Transaction) -> Result<T>,
    {
        let tx = self.conn.transaction()?;
        let result = f(&mut tx)?;
        tx.commit()?;
        Ok(result)
    }
}

// Usage
db.transaction(|tx| {
    tx.execute("INSERT INTO invoices (...) VALUES (...)", [])?;
    tx.execute("UPDATE customers SET ...", [])?;
    Ok(())
})?;
```

## Résumé

- **WAL mode** : Meilleures performances et concurrence
- **Cache** : Optimiser la taille du cache
- **FTS5** : Recherche full-text intégrée
- **Transactions** : Atomicité des opérations
