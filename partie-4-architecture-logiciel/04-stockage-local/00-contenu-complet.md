# Chapitre 17 : Stockage Local Souverain

## Introduction

Le stockage local est au cœur de l'approche "local-first". Ce chapitre couvre SQLite, les migrations, le chiffrement et les stratégies offline-first.

---

## 17.1 SQLite avec rusqlite

```rust
use rusqlite::{Connection, params};

pub struct Database {
    conn: Connection,
}

impl Database {
    pub fn open(path: &str) -> Result<Self> {
        let conn = Connection::open(path)?;
        
        // Optimisations SQLite
        conn.execute_batch("
            PRAGMA journal_mode = WAL;
            PRAGMA synchronous = NORMAL;
            PRAGMA foreign_keys = ON;
            PRAGMA cache_size = -64000;
        ")?;
        
        Ok(Self { conn })
    }
    
    pub fn in_memory() -> Result<Self> {
        let conn = Connection::open_in_memory()?;
        Ok(Self { conn })
    }
}
```

---

## 17.2 Migrations

```rust
const MIGRATIONS: &[(&str, &str)] = &[
    ("001_initial", include_str!("../migrations/001_initial.sql")),
    ("002_add_customers", include_str!("../migrations/002_add_customers.sql")),
    ("003_add_invoices", include_str!("../migrations/003_add_invoices.sql")),
];

impl Database {
    pub fn run_migrations(&self) -> Result<()> {
        // Créer la table de migrations
        self.conn.execute(
            "CREATE TABLE IF NOT EXISTS _migrations (
                name TEXT PRIMARY KEY,
                applied_at TEXT NOT NULL
            )",
            [],
        )?;
        
        for (name, sql) in MIGRATIONS {
            let already_applied: bool = self.conn.query_row(
                "SELECT EXISTS(SELECT 1 FROM _migrations WHERE name = ?1)",
                [name],
                |row| row.get(0),
            )?;
            
            if !already_applied {
                self.conn.execute_batch(sql)?;
                self.conn.execute(
                    "INSERT INTO _migrations (name, applied_at) VALUES (?1, datetime('now'))",
                    [name],
                )?;
                println!("Applied migration: {}", name);
            }
        }
        
        Ok(())
    }
}
```

---

## 17.3 Chiffrement

```rust
use aes_gcm::{Aes256Gcm, Key, Nonce, aead::{Aead, NewAead}};
use argon2::{Argon2, password_hash::SaltString};

pub struct EncryptedStorage {
    cipher: Aes256Gcm,
}

impl EncryptedStorage {
    pub fn new(password: &str, salt: &[u8]) -> Self {
        // Dériver une clé du mot de passe
        let mut key_bytes = [0u8; 32];
        Argon2::default()
            .hash_password_into(password.as_bytes(), salt, &mut key_bytes)
            .expect("Key derivation failed");
        
        let key = Key::from_slice(&key_bytes);
        let cipher = Aes256Gcm::new(key);
        
        Self { cipher }
    }
    
    pub fn encrypt(&self, data: &[u8]) -> Vec<u8> {
        let nonce = Nonce::from_slice(b"unique nonce"); // Générer aléatoirement en prod
        self.cipher.encrypt(nonce, data).expect("Encryption failed")
    }
    
    pub fn decrypt(&self, encrypted: &[u8]) -> Vec<u8> {
        let nonce = Nonce::from_slice(b"unique nonce");
        self.cipher.decrypt(nonce, encrypted).expect("Decryption failed")
    }
}
```

---

## 17.4 Offline-First Pattern

```rust
pub struct OfflineFirstStore {
    local_db: Database,
    pending_sync: Vec<SyncOperation>,
}

#[derive(Serialize, Deserialize)]
pub struct SyncOperation {
    pub id: Uuid,
    pub operation: Operation,
    pub created_at: DateTime<Utc>,
    pub synced: bool,
}

impl OfflineFirstStore {
    /// Sauvegarder localement et marquer pour sync
    pub fn save(&mut self, entity: &impl Syncable) -> Result<()> {
        // 1. Sauvegarder localement
        self.local_db.save(entity)?;
        
        // 2. Ajouter à la queue de sync
        self.pending_sync.push(SyncOperation {
            id: Uuid::new_v4(),
            operation: Operation::Upsert(entity.to_json()),
            created_at: Utc::now(),
            synced: false,
        });
        
        Ok(())
    }
    
    /// Synchroniser quand connexion disponible
    pub async fn sync(&mut self, api: &ApiClient) -> Result<SyncResult> {
        let mut synced = 0;
        let mut failed = 0;
        
        for op in &mut self.pending_sync {
            if op.synced {
                continue;
            }
            
            match api.sync_operation(op).await {
                Ok(_) => {
                    op.synced = true;
                    synced += 1;
                }
                Err(_) => {
                    failed += 1;
                }
            }
        }
        
        // Nettoyer les opérations synchronisées
        self.pending_sync.retain(|op| !op.synced);
        
        Ok(SyncResult { synced, failed })
    }
}
```

---

## Résumé

- **SQLite** : Base de données embarquée fiable
- **Migrations** : Évolution du schéma versionnée
- **Chiffrement** : Protection des données sensibles
- **Offline-first** : Fonctionne sans réseau, sync quand possible
