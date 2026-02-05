# Chapitre 7 : Logs, Debug et Profiling

## Introduction

Un logiciel professionnel nécessite une observabilité sérieuse : savoir ce qui se passe, diagnostiquer les problèmes, et optimiser les performances. Ce chapitre couvre les outils et techniques pour y parvenir en Rust.

---

## 7.1 Tracing : Logs Structurés

### Pourquoi tracing plutôt que println! ?

```rust
// ❌ println! : difficile à filtrer, pas de contexte
println!("Processing user {}", user_id);
println!("Query took {}ms", duration);

// ✅ tracing : structuré, filtrable, contextuel
tracing::info!(user_id = %user_id, "Processing user");
tracing::debug!(duration_ms = duration, query = %sql, "Query executed");
```

### Configuration de base

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
            .with_thread_ids(true)  // Afficher le thread
            .with_file(true)        // Afficher le fichier source
            .with_line_number(true) // Afficher la ligne
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

### Niveaux de log

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

### Spans : contexte hiérarchique

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

### Logging vers fichier

```rust
use tracing_subscriber::fmt::writer::MakeWriterExt;
use std::fs::File;

fn setup_file_logging() {
    let file = File::create("app.log").expect("Unable to create log file");
    
    tracing_subscriber::registry()
        .with(fmt::layer()
            .with_writer(file)
            .with_ansi(false)  // Pas de couleurs dans le fichier
            .json()            // Format JSON pour parsing
        )
        .with(fmt::layer()
            .with_writer(std::io::stdout)
            .with_ansi(true)   // Couleurs dans le terminal
        )
        .with(EnvFilter::from_default_env())
        .init();
}
```

---

## 7.2 Logs Visuels pour Desktop

### Logger vers l'UI

```rust
use std::sync::{Arc, Mutex};
use std::collections::VecDeque;

#[derive(Clone)]
pub struct LogEntry {
    pub timestamp: chrono::DateTime<chrono::Local>,
    pub level: LogLevel,
    pub message: String,
    pub target: String,
}

#[derive(Clone)]
pub enum LogLevel {
    Trace,
    Debug,
    Info,
    Warn,
    Error,
}

/// Buffer de logs accessible depuis l'UI
pub struct UiLogBuffer {
    entries: Arc<Mutex<VecDeque<LogEntry>>>,
    max_entries: usize,
}

impl UiLogBuffer {
    pub fn new(max_entries: usize) -> Self {
        Self {
            entries: Arc::new(Mutex::new(VecDeque::with_capacity(max_entries))),
            max_entries,
        }
    }
    
    pub fn push(&self, entry: LogEntry) {
        let mut entries = self.entries.lock().unwrap();
        if entries.len() >= self.max_entries {
            entries.pop_front();
        }
        entries.push_back(entry);
    }
    
    pub fn get_entries(&self) -> Vec<LogEntry> {
        self.entries.lock().unwrap().iter().cloned().collect()
    }
    
    pub fn clear(&self) {
        self.entries.lock().unwrap().clear();
    }
}
```

### Panneau de logs dans egui

```rust
pub struct LogPanel {
    buffer: UiLogBuffer,
    filter_level: LogLevel,
    filter_text: String,
    auto_scroll: bool,
}

impl LogPanel {
    pub fn new(buffer: UiLogBuffer) -> Self {
        Self {
            buffer,
            filter_level: LogLevel::Debug,
            filter_text: String::new(),
            auto_scroll: true,
        }
    }
    
    pub fn render(&mut self, ui: &mut egui::Ui) {
        // Contrôles de filtrage
        ui.horizontal(|ui| {
            ui.label("Niveau:");
            egui::ComboBox::from_id_source("log_level")
                .selected_text(format!("{:?}", self.filter_level))
                .show_ui(ui, |ui| {
                    ui.selectable_value(&mut self.filter_level, LogLevel::Trace, "Trace");
                    ui.selectable_value(&mut self.filter_level, LogLevel::Debug, "Debug");
                    ui.selectable_value(&mut self.filter_level, LogLevel::Info, "Info");
                    ui.selectable_value(&mut self.filter_level, LogLevel::Warn, "Warn");
                    ui.selectable_value(&mut self.filter_level, LogLevel::Error, "Error");
                });
            
            ui.text_edit_singleline(&mut self.filter_text);
            
            ui.checkbox(&mut self.auto_scroll, "Auto-scroll");
            
            if ui.button("Effacer").clicked() {
                self.buffer.clear();
            }
        });
        
        ui.separator();
        
        // Liste des logs
        egui::ScrollArea::vertical()
            .auto_shrink([false, false])
            .stick_to_bottom(self.auto_scroll)
            .show(ui, |ui| {
                for entry in self.buffer.get_entries() {
                    if !self.should_show(&entry) {
                        continue;
                    }
                    
                    let color = match entry.level {
                        LogLevel::Trace => egui::Color32::GRAY,
                        LogLevel::Debug => egui::Color32::LIGHT_GRAY,
                        LogLevel::Info => egui::Color32::WHITE,
                        LogLevel::Warn => egui::Color32::YELLOW,
                        LogLevel::Error => egui::Color32::RED,
                    };
                    
                    ui.horizontal(|ui| {
                        ui.label(
                            egui::RichText::new(entry.timestamp.format("%H:%M:%S%.3f").to_string())
                                .monospace()
                                .color(egui::Color32::DARK_GRAY)
                        );
                        ui.label(
                            egui::RichText::new(format!("[{:5}]", format!("{:?}", entry.level)))
                                .monospace()
                                .color(color)
                        );
                        ui.label(&entry.message);
                    });
                }
            });
    }
    
    fn should_show(&self, entry: &LogEntry) -> bool {
        // Filtrer par niveau
        let level_ok = match (&self.filter_level, &entry.level) {
            (LogLevel::Trace, _) => true,
            (LogLevel::Debug, LogLevel::Trace) => false,
            (LogLevel::Debug, _) => true,
            (LogLevel::Info, LogLevel::Trace | LogLevel::Debug) => false,
            (LogLevel::Info, _) => true,
            (LogLevel::Warn, LogLevel::Error | LogLevel::Warn) => true,
            (LogLevel::Warn, _) => false,
            (LogLevel::Error, LogLevel::Error) => true,
            (LogLevel::Error, _) => false,
        };
        
        // Filtrer par texte
        let text_ok = self.filter_text.is_empty() 
            || entry.message.to_lowercase().contains(&self.filter_text.to_lowercase());
        
        level_ok && text_ok
    }
}
```

---

## 7.3 Flamegraphs et Profiling

### Profiling avec cargo-flamegraph

```bash
# Installation
cargo install flamegraph

# Sur Linux, installer perf
sudo apt install linux-perf

# Générer un flamegraph
cargo flamegraph --bin mon_app

# Le résultat est dans flamegraph.svg
```

### Instrumentation manuelle pour profiling

```rust
use std::time::Instant;

/// Macro pour mesurer le temps d'exécution
macro_rules! timed {
    ($name:expr, $block:expr) => {{
        let start = std::time::Instant::now();
        let result = $block;
        let elapsed = start.elapsed();
        tracing::debug!(
            target: "profiling",
            name = $name,
            duration_ms = elapsed.as_secs_f64() * 1000.0,
            "Timing"
        );
        result
    }};
}

fn process_data(data: &[Item]) -> ProcessedData {
    let validated = timed!("validation", {
        validate_items(data)
    });
    
    let transformed = timed!("transformation", {
        transform_items(&validated)
    });
    
    let indexed = timed!("indexation", {
        index_items(&transformed)
    });
    
    ProcessedData { items: indexed }
}
```

### Profiler avec tracing

```toml
# Cargo.toml
[dependencies]
tracing-chrome = "0.7"  # Export au format Chrome DevTools
```

```rust
use tracing_chrome::ChromeLayerBuilder;
use tracing_subscriber::prelude::*;

fn setup_profiling() {
    let (chrome_layer, guard) = ChromeLayerBuilder::new()
        .file("trace.json")
        .build();
    
    tracing_subscriber::registry()
        .with(chrome_layer)
        .init();
    
    // guard doit rester vivant pour que le fichier soit écrit
    // Stocker dans l'App ou utiliser un scope global
}

// Les spans seront visualisables dans chrome://tracing
#[instrument]
fn expensive_operation() {
    // ...
}
```

### Benchmarking intégré

```rust
/// Mesurer les performances en release
#[cfg(not(debug_assertions))]
fn bench_search() {
    use std::time::Instant;
    
    let data = load_test_data();
    let queries = ["rust", "performance", "desktop", "application"];
    
    let iterations = 1000;
    let start = Instant::now();
    
    for _ in 0..iterations {
        for query in &queries {
            let _ = search(&data, query);
        }
    }
    
    let elapsed = start.elapsed();
    let per_query = elapsed / (iterations * queries.len() as u32);
    
    println!("Search benchmark:");
    println!("  Total: {:?}", elapsed);
    println!("  Per query: {:?}", per_query);
    println!("  Queries/sec: {:.0}", 1.0 / per_query.as_secs_f64());
}
```

---

## 7.4 Optimiser Sans Se Mentir

### Les pièges de l'optimisation prématurée

```rust
// ❌ "Optimisation" contre-productive
fn bad_optimization(items: &[Item]) -> Vec<ProcessedItem> {
    // "Les allocations sont coûteuses, je réutilise un buffer"
    static mut BUFFER: Vec<ProcessedItem> = Vec::new();
    
    unsafe {
        BUFFER.clear();
        for item in items {
            BUFFER.push(process(item));
        }
        BUFFER.clone()  // On clone quand même !
    }
    // Résultat : code dangereux, pas plus rapide, bugs potentiels
}

// ✅ Code simple et rapide
fn good_code(items: &[Item]) -> Vec<ProcessedItem> {
    items.iter().map(process).collect()
    // Rust optimise très bien ce pattern
}
```

### Méthodologie de profiling

```
1. MESURER d'abord
   - Établir une baseline
   - Identifier les vrais goulots d'étranglement
   - Ne pas deviner

2. OPTIMISER ensuite
   - Cibler les 20% qui causent 80% du temps
   - Un changement à la fois
   - Mesurer après chaque changement

3. VÉRIFIER toujours
   - Comparer avec la baseline
   - S'assurer que le comportement est identique
   - Documenter les gains
```

### Outils de mesure intégrés

```rust
pub struct PerformanceMonitor {
    frame_times: VecDeque<Duration>,
    max_samples: usize,
}

impl PerformanceMonitor {
    pub fn new(max_samples: usize) -> Self {
        Self {
            frame_times: VecDeque::with_capacity(max_samples),
            max_samples,
        }
    }
    
    pub fn record_frame(&mut self, duration: Duration) {
        if self.frame_times.len() >= self.max_samples {
            self.frame_times.pop_front();
        }
        self.frame_times.push_back(duration);
    }
    
    pub fn average_fps(&self) -> f64 {
        if self.frame_times.is_empty() {
            return 0.0;
        }
        
        let avg_duration: Duration = self.frame_times.iter().sum::<Duration>() 
            / self.frame_times.len() as u32;
        
        1.0 / avg_duration.as_secs_f64()
    }
    
    pub fn percentile_frame_time(&self, percentile: f64) -> Duration {
        let mut sorted: Vec<_> = self.frame_times.iter().copied().collect();
        sorted.sort();
        
        let index = ((sorted.len() as f64 * percentile) / 100.0) as usize;
        sorted.get(index).copied().unwrap_or_default()
    }
    
    pub fn render_overlay(&self, ui: &mut egui::Ui) {
        ui.label(format!("FPS: {:.1}", self.average_fps()));
        ui.label(format!("Frame time (avg): {:.2}ms", 
            self.frame_times.iter().sum::<Duration>().as_secs_f64() * 1000.0 
            / self.frame_times.len() as f64));
        ui.label(format!("Frame time (p99): {:.2}ms", 
            self.percentile_frame_time(99.0).as_secs_f64() * 1000.0));
    }
}
```

### Optimisations courantes validées

```rust
// 1. Pré-allocation quand la taille est connue
let mut results = Vec::with_capacity(items.len());

// 2. Éviter les clones inutiles
fn process(item: &Item) -> ProcessedItem  // &Item plutôt que Item
fn process(item: Item) -> ProcessedItem   // Si on a besoin de la propriété

// 3. Utiliser des itérateurs
items.iter()
    .filter(|i| i.is_valid())
    .map(|i| transform(i))
    .collect()

// 4. String : éviter les allocations
// ❌ Beaucoup d'allocations
let result = "Hello ".to_string() + &name + "!";

// ✅ Une seule allocation
let result = format!("Hello {}!", name);

// 5. Cow pour les modifications conditionnelles
use std::borrow::Cow;

fn maybe_transform(s: &str) -> Cow<str> {
    if s.contains("bad") {
        Cow::Owned(s.replace("bad", "good"))
    } else {
        Cow::Borrowed(s)  // Pas d'allocation
    }
}
```

---

## Résumé du Chapitre

### Configuration logging recommandée

```rust
// Production
tracing_subscriber::registry()
    .with(fmt::layer().json())  // JSON pour parsing
    .with(EnvFilter::new("info,my_app=debug"))
    .init();

// Développement
tracing_subscriber::registry()
    .with(fmt::layer().pretty())  // Lisible humainement
    .with(EnvFilter::new("debug"))
    .init();
```

### Checklist profiling

- [ ] Mesurer avant d'optimiser
- [ ] Utiliser des outils (flamegraph, tracing-chrome)
- [ ] Cibler les vrais goulots d'étranglement
- [ ] Mesurer après chaque changement
- [ ] Documenter les gains obtenus

### Métriques à surveiller

| Métrique | Cible | Critique |
|----------|-------|----------|
| FPS UI | > 60 | < 30 |
| Frame time p99 | < 16ms | > 33ms |
| Démarrage | < 1s | > 3s |
| Mémoire au repos | Stable | Croissance continue |

---

**Dans la prochaine partie, nous aborderons la construction d'interfaces modernes avec les frameworks UI Rust.**
