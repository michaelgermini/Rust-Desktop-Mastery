# 17.2 Migrations

## Système de migrations

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
                println!("Applying migration: {}", name);
                self.conn.execute_batch(sql)?;
                self.conn.execute(
                    "INSERT INTO _migrations (name, applied_at) VALUES (?1, datetime('now'))",
                    [name],
                )?;
            }
        }
        
        Ok(())
    }
}
```

## Versioning du schéma

### Fichiers de migration

```sql
-- migrations/001_initial.sql
CREATE TABLE customers (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    email TEXT UNIQUE,
    created_at TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- migrations/002_add_customers.sql
CREATE TABLE invoices (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    customer_id INTEGER NOT NULL,
    number TEXT NOT NULL UNIQUE,
    total REAL NOT NULL,
    FOREIGN KEY (customer_id) REFERENCES customers(id)
);

-- migrations/003_add_invoices.sql
ALTER TABLE invoices ADD COLUMN status TEXT DEFAULT 'draft';
CREATE INDEX idx_invoices_customer ON invoices(customer_id);
```

## Rollback et forward

```rust
pub struct Migration {
    pub name: String,
    pub up: String,
    pub down: String,  // Script de rollback
}

impl Database {
    pub fn rollback(&self, migration_name: &str) -> Result<()> {
        // Trouver la migration
        let migration = MIGRATIONS.iter()
            .find(|(name, _)| *name == migration_name)
            .ok_or("Migration not found")?;
        
        // Exécuter le script down
        // (nécessite de stocker les scripts down)
        
        // Retirer de la table _migrations
        self.conn.execute(
            "DELETE FROM _migrations WHERE name = ?1",
            [migration_name],
        )?;
        
        Ok(())
    }
}
```

## Résumé

- **Table _migrations** : Suivi des migrations appliquées
- **Versioning** : Fichiers SQL numérotés
- **Idempotent** : Vérifier avant d'appliquer
- **Rollback** : Scripts de retour en arrière (optionnel)
