# 6.5 Éviter les Freezes

## Détection des opérations bloquantes

### Checklist des opérations suspectes

```rust
/// Opérations qui doivent être async
pub enum BlockingOperation {
    /// Lecture/écriture fichiers
    FileIO { path: PathBuf },
    
    /// Requêtes base de données
    DatabaseQuery { sql: String },
    
    /// Requêtes réseau
    NetworkRequest { url: String },
    
    /// Calculs lourds
    HeavyComputation { data_size: usize },
    
    /// Parsing de gros fichiers
    FileParsing { file_size: usize },
}
```

### Pattern : Wrapper non-bloquant

```rust
pub struct NonBlocking<T> {
    value: Arc<Mutex<Option<T>>>,
    loading: Arc<AtomicBool>,
}

impl<T: Send + 'static> NonBlocking<T> {
    pub fn load_async<F>(&self, loader: F)
    where
        F: FnOnce() -> T + Send + 'static,
    {
        self.loading.store(true, Ordering::Relaxed);
        
        let value = Arc::clone(&self.value);
        let loading = Arc::clone(&self.loading);
        
        std::thread::spawn(move || {
            let result = loader();
            *value.lock().unwrap() = Some(result);
            loading.store(false, Ordering::Relaxed);
        });
    }
    
    pub fn get(&self) -> Option<T>
    where
        T: Clone,
    {
        self.value.lock().unwrap().clone()
    }
    
    pub fn is_loading(&self) -> bool {
        self.loading.load(Ordering::Relaxed)
    }
}
```

## Patterns de déblocage

### 1. Chunked processing

```rust
pub fn process_large_dataset_chunked(
    data: &[Item],
    chunk_size: usize,
    progress: &Sender<Progress>,
) {
    for (i, chunk) in data.chunks(chunk_size).enumerate() {
        // Traiter un chunk
        process_chunk(chunk);
        
        // Signaler la progression
        progress.send(Progress {
            current: i * chunk_size,
            total: data.len(),
        }).ok();
        
        // Laisser respirer le thread UI
        std::thread::yield_now();
    }
}
```

### 2. Debouncing

```rust
use std::time::{Duration, Instant};

pub struct Debouncer {
    last_call: Arc<Mutex<Instant>>,
    delay: Duration,
}

impl Debouncer {
    pub fn new(delay: Duration) -> Self {
        Self {
            last_call: Arc::new(Mutex::new(Instant::now())),
            delay,
        }
    }
    
    pub fn debounce<F: FnOnce()>(&self, f: F) {
        let mut last = self.last_call.lock().unwrap();
        if last.elapsed() > self.delay {
            f();
            *last = Instant::now();
        }
    }
}
```

### 3. Cancellation token

```rust
use std::sync::atomic::{AtomicBool, Ordering};

pub struct CancellationToken {
    cancelled: Arc<AtomicBool>,
}

impl CancellationToken {
    pub fn new() -> Self {
        Self {
            cancelled: Arc::new(AtomicBool::new(false)),
        }
    }
    
    pub fn cancel(&self) {
        self.cancelled.store(true, Ordering::Relaxed);
    }
    
    pub fn is_cancelled(&self) -> bool {
        self.cancelled.load(Ordering::Relaxed)
    }
}

pub fn long_operation(token: &CancellationToken) -> Result<()> {
    for i in 0..1000 {
        if token.is_cancelled() {
            return Err(Error::Cancelled);
        }
        
        // Travail...
        process_item(i);
    }
    Ok(())
}
```

## Feedback utilisateur pendant les opérations

```rust
pub fn show_progress_ui(ui: &mut Ui, progress: &ProgressState) {
    if progress.is_active() {
        egui::Window::new("Traitement en cours")
            .collapsible(false)
            .resizable(false)
            .show(ui.ctx(), |ui| {
                ui.label(&progress.message);
                ui.add(egui::ProgressBar::new(progress.percent / 100.0));
                
                if ui.button("Annuler").clicked() {
                    progress.cancel();
                }
            });
    }
}
```

## Résumé

- **Détecter** : Identifier les opérations > 16ms
- **Déplacer** : Envoyer aux workers via channels
- **Chunked** : Traiter par petits morceaux
- **Feedback** : Toujours informer l'utilisateur
- **Annulation** : Permettre d'annuler les opérations longues
