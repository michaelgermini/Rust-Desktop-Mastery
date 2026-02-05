# 7.1 Tracing : Logs Structurés

## Pourquoi tracing plutôt que println! ?

```rust
// ❌ println! : difficile à filtrer, pas de contexte
println!("Processing user {}", user_id);
println!("Query took {}ms", duration);

// ✅ tracing : structuré, filtrable, contextuel
tracing::info!(user_id = %user_id, "Processing user");
tracing::debug!(duration_ms = duration, query = %sql, "Query executed");
```

## Configuration de base

```toml
# Cargo.toml
[dependencies]
tracing = "0.1"
tracing-subscriber = { version = "0.3", features = ["env-filter", "json"] }
```

```rust
use tracing::{info, debug, warn, error, instrument, Level};
use tracing_subscriber::{fmt, prelude::*, EnvFilter};

fn setup_logging() {
    // Configuration avec filtre par variable d'environnement
    // RUST_LOG=debug cargo run
    // RUST_LOG=my_app=debug,sqlx=warn cargo run
    
    tracing_subscriber::registry()
        .with(fmt::layer()
            .with_target(true)      // Afficher le module source
            .with_thread_ids(true)   // Afficher le thread
            .with_file(true)         // Afficher le fichier source
            .with_line_number(true)  // Afficher la ligne
        )
        .with(EnvFilter::from_default_env()
            .add_directive(Level::INFO.into())  // Niveau par défaut
        )
        .init();
}

fn main() {
    setup_logging();
    
    info!("Application démarrée");
    
    // Le reste de l'application...
}
```

## Niveaux de log

```rust
use tracing::{trace, debug, info, warn, error};

fn process_document(doc: &Document) -> Result<()> {
    // TRACE : très verbeux, pour le debugging fin
    trace!(doc_id = %doc.id, "Entering process_document");
    
    // DEBUG : informations utiles au développement
    debug!(doc_size = doc.content.len(), "Processing document");
    
    // INFO : événements normaux importants
    info!(doc_id = %doc.id, "Document processed successfully");
    
    // WARN : situations anormales mais gérées
    if doc.content.len() > 1_000_000 {
        warn!(doc_id = %doc.id, size = doc.content.len(), 
              "Document exceptionnellement large");
    }
    
    // ERROR : erreurs qui impactent le fonctionnement
    if let Err(e) = validate_document(doc) {
        error!(doc_id = %doc.id, error = %e, "Document validation failed");
        return Err(e);
    }
    
    Ok(())
}
```

## Spans : contexte hiérarchique

```rust
use tracing::{instrument, info_span, Span};

// Instrumentation automatique avec #[instrument]
#[instrument(skip(db), fields(user_id = %user_id))]
async fn fetch_user_data(db: &Database, user_id: i64) -> Result<UserData> {
    // Tout ce qui est loggué ici inclut automatiquement user_id
    let user = db.get_user(user_id).await?;
    debug!("User found: {:?}", user.name);
    
    let orders = db.get_orders(user_id).await?;
    debug!(order_count = orders.len(), "Orders loaded");
    
    Ok(UserData { user, orders })
}

// Spans manuels pour plus de contrôle
fn process_batch(items: &[Item]) {
    let span = info_span!("process_batch", item_count = items.len());
    let _guard = span.enter();
    
    for (i, item) in items.iter().enumerate() {
        let item_span = info_span!("process_item", index = i, item_id = %item.id);
        let _guard = item_span.enter();
        
        // Logs ici incluent process_batch > process_item comme contexte
        process_single_item(item);
    }
}
```

## Logging vers fichier

```rust
use tracing_subscriber::fmt::writer::MakeWriterExt;
use std::fs::File;

fn setup_file_logging() {
    let file = File::create("app.log").expect("Unable to create log file");
    
    tracing_subscriber::registry()
        .with(fmt::layer()
            .with_writer(file)
            .with_ansi(false)  // Pas de couleurs dans le fichier
            .json()             // Format JSON pour parsing
        )
        .with(fmt::layer()
            .with_writer(std::io::stdout)
            .with_ansi(true)    // Couleurs dans le terminal
        )
        .with(EnvFilter::from_default_env())
        .init();
}
```

## Résumé

- **tracing** : Logging structuré avec métadonnées
- **Niveaux** : trace < debug < info < warn < error
- **Spans** : Contexte hiérarchique automatique
- **Filtrage** : Via RUST_LOG pour contrôler la verbosité
