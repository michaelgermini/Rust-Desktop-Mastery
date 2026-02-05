# 1.5 Offline-First

## Philosophie Offline-First

Une application offline-first fonctionne parfaitement sans connexion internet, et utilise le réseau comme amélioration optionnelle.

```rust
/// Hiérarchie des sources de données
pub enum DataSource {
    /// Source principale : données locales
    Local(LocalStorage),
    
    /// Source secondaire : cache
    Cache(CacheLayer),
    
    /// Source optionnelle : synchronisation cloud
    Remote(Option<RemoteSync>),
}

impl DataSource {
    /// Toujours commencer par le local
    pub async fn fetch(&self, query: &Query) -> Result<Data> {
        // 1. Chercher localement d'abord
        if let Some(data) = self.local.get(query)? {
            return Ok(data);
        }
        
        // 2. Vérifier le cache
        if let Some(cached) = self.cache.get(query)? {
            return Ok(cached);
        }
        
        // 3. Seulement si nécessaire, aller en ligne
        if let Some(remote) = &self.remote {
            if remote.is_available().await {
                let data = remote.fetch(query).await?;
                self.local.save(&data)?;  // Sauvegarder localement
                return Ok(data);
            }
        }
        
        Err(Error::NotFound)
    }
}
```

## Avantages de l'approche locale

### Disponibilité 100%

```rust
pub struct AvailabilityComparison {
    pub saas_uptime: f64,      // 99.9% = 8.7h de downtime/an
    pub local_uptime: f64,     // 100% (tant que le PC fonctionne)
}

// Avec une app locale :
// - Pas d'impact des pannes AWS/Azure/GCP
// - Pas de "Service temporairement indisponible"
// - Pas de latence réseau
// - Fonctionne dans l'avion, le train, la campagne
```

### Latence zéro

```rust
use std::time::Duration;

pub struct LatencyComparison {
    pub operation: &'static str,
    pub saas_latency: Duration,
    pub local_latency: Duration,
}

const LATENCY_EXAMPLES: &[LatencyComparison] = &[
    LatencyComparison {
        operation: "Ouvrir un document",
        saas_latency: Duration::from_millis(500),   // Requête HTTP
        local_latency: Duration::from_micros(100),  // Lecture disque
    },
    LatencyComparison {
        operation: "Recherche",
        saas_latency: Duration::from_millis(200),
        local_latency: Duration::from_millis(5),
    },
    LatencyComparison {
        operation: "Sauvegarde",
        saas_latency: Duration::from_millis(300),
        local_latency: Duration::from_micros(500),
    },
];
```

### Coûts prévisibles

```rust
pub struct CostComparison {
    pub saas_monthly: Decimal,
    pub local_one_time: Decimal,
    pub break_even_months: u32,
}

impl CostComparison {
    pub fn example() -> Self {
        Self {
            saas_monthly: Decimal::new(2999, 2),     // 29.99€/mois
            local_one_time: Decimal::new(14900, 2), // 149€ une fois
            break_even_months: 5,                    // Rentabilisé en 5 mois
        }
    }
}
```

## Architecture Offline-First

### Stockage local robuste

```rust
use rusqlite::Connection;

pub struct LocalDatabase {
    conn: Connection,
}

impl LocalDatabase {
    pub fn new(path: &Path) -> Result<Self> {
        let conn = Connection::open(path)?;
        
        // Configuration pour la robustesse
        conn.execute_batch("
            PRAGMA journal_mode = WAL;
            PRAGMA synchronous = NORMAL;
            PRAGMA foreign_keys = ON;
            PRAGMA auto_vacuum = INCREMENTAL;
        ")?;
        
        Ok(Self { conn })
    }
    
    /// Sauvegarde automatique
    pub fn backup(&self, backup_path: &Path) -> Result<()> {
        let backup = Connection::open(backup_path)?;
        self.conn.backup(rusqlite::DatabaseName::Main, &backup, None)?;
        Ok(())
    }
}
```

### File d'attente de synchronisation

```rust
use std::collections::VecDeque;
use chrono::{DateTime, Utc};

/// File d'attente pour sync quand connexion disponible
pub struct SyncQueue {
    pending: VecDeque<SyncOperation>,
    last_sync: Option<DateTime<Utc>>,
}

pub struct SyncOperation {
    pub id: Uuid,
    pub operation_type: OperationType,
    pub data: Vec<u8>,
    pub created_at: DateTime<Utc>,
    pub retry_count: u32,
}

impl SyncQueue {
    /// Ajouter une opération à synchroniser plus tard
    pub fn enqueue(&mut self, op: SyncOperation) {
        self.pending.push_back(op);
        self.persist_queue();  // Sauvegarder sur disque
    }
    
    /// Synchroniser quand connexion disponible
    pub async fn sync_when_online(&mut self, client: &HttpClient) -> Result<SyncResult> {
        if !client.is_online().await {
            return Ok(SyncResult::Offline);
        }
        
        let mut synced = 0;
        let mut failed = 0;
        
        while let Some(mut op) = self.pending.pop_front() {
            match client.sync(&op).await {
                Ok(_) => synced += 1,
                Err(e) => {
                    op.retry_count += 1;
                    if op.retry_count < 3 {
                        self.pending.push_back(op);
                    }
                    failed += 1;
                }
            }
        }
        
        self.last_sync = Some(Utc::now());
        Ok(SyncResult::Completed { synced, failed })
    }
}
```

### Résolution de conflits

```rust
/// Stratégie de résolution de conflits
pub enum ConflictResolution {
    /// Le plus récent gagne
    LastWriteWins,
    
    /// Le local a priorité
    LocalFirst,
    
    /// Le serveur a priorité
    ServerFirst,
    
    /// Demander à l'utilisateur
    AskUser,
    
    /// Fusion automatique si possible
    AutoMerge,
}

pub struct ConflictResolver {
    strategy: ConflictResolution,
}

impl ConflictResolver {
    pub fn resolve(&self, local: &Document, remote: &Document) -> ResolvedDocument {
        match self.strategy {
            ConflictResolution::LastWriteWins => {
                if local.updated_at > remote.updated_at {
                    ResolvedDocument::UseLocal(local.clone())
                } else {
                    ResolvedDocument::UseRemote(remote.clone())
                }
            }
            ConflictResolution::AutoMerge => {
                self.attempt_merge(local, remote)
            }
            ConflictResolution::AskUser => {
                ResolvedDocument::NeedsUserInput {
                    local: local.clone(),
                    remote: remote.clone(),
                }
            }
            // ...
        }
    }
}
```

## Patterns d'UI Offline

### Indicateur de statut

```rust
pub fn show_connection_status(ui: &mut Ui, status: &ConnectionStatus) {
    let (icon, color, text) = match status {
        ConnectionStatus::Online => ("🟢", Color32::GREEN, "En ligne"),
        ConnectionStatus::Offline => ("🔴", Color32::RED, "Hors ligne"),
        ConnectionStatus::Syncing => ("🔄", Color32::YELLOW, "Synchronisation..."),
        ConnectionStatus::SyncPending(count) => {
            ("⏳", Color32::ORANGE, &format!("{} en attente", count))
        }
    };
    
    ui.horizontal(|ui| {
        ui.label(icon);
        ui.colored_label(color, text);
    });
}
```

### Actions disponibles hors ligne

```rust
pub fn show_action_availability(ui: &mut Ui, is_online: bool) {
    ui.horizontal(|ui| {
        // Toujours disponible
        if ui.button("📝 Créer").clicked() {
            // Créer localement
        }
        
        if ui.button("💾 Sauvegarder").clicked() {
            // Sauvegarder localement
        }
        
        // Nécessite connexion
        ui.add_enabled(is_online, egui::Button::new("☁️ Sync"))
            .on_disabled_hover_text("Nécessite une connexion internet");
    });
}
```

## Conclusion

L'approche offline-first garantit :

- **Disponibilité** : L'app fonctionne toujours
- **Performance** : Données locales = latence minimale
- **Résilience** : Pas de dépendance réseau
- **Autonomie** : L'utilisateur garde le contrôle
- **Économies** : Pas d'abonnement cloud obligatoire
