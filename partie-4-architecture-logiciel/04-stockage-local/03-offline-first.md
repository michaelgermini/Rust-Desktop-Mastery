# 17.3 Offline-First

## Architecture offline-first

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

#[derive(Serialize, Deserialize)]
pub enum Operation {
    Create { entity_type: String, data: serde_json::Value },
    Update { entity_type: String, id: String, data: serde_json::Value },
    Delete { entity_type: String, id: String },
}

impl OfflineFirstStore {
    /// Sauvegarder localement et marquer pour sync
    pub fn save(&mut self, entity: &impl Syncable) -> Result<()> {
        // 1. Sauvegarder localement immédiatement
        self.local_db.save(entity)?;
        
        // 2. Ajouter à la queue de sync
        self.pending_sync.push(SyncOperation {
            id: Uuid::new_v4(),
            operation: Operation::Create {
                entity_type: entity.entity_type(),
                data: entity.to_json(),
            },
            created_at: Utc::now(),
            synced: false,
        });
        
        Ok(())
    }
}
```

## Sync queue

```rust
impl OfflineFirstStore {
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
                Err(e) => {
                    eprintln!("Sync failed: {}", e);
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

## Résolution de conflits

```rust
pub enum ConflictResolution {
    LocalWins,
    RemoteWins,
    Merge,
    Manual,
}

impl OfflineFirstStore {
    pub fn resolve_conflict(
        &mut self,
        local: &Entity,
        remote: &Entity,
        strategy: ConflictResolution,
    ) -> Result<Entity> {
        match strategy {
            ConflictResolution::LocalWins => Ok(local.clone()),
            ConflictResolution::RemoteWins => Ok(remote.clone()),
            ConflictResolution::Merge => {
                // Logique de merge personnalisée
                self.merge_entities(local, remote)
            }
            ConflictResolution::Manual => {
                // Retourner les deux pour résolution manuelle
                Err("Manual resolution required".into())
            }
        }
    }
}
```

## Résumé

- **Local-first** : Sauvegarder localement immédiatement
- **Sync queue** : Opérations en attente de synchronisation
- **Conflits** : Stratégies de résolution
- **Résilience** : Fonctionne sans réseau
