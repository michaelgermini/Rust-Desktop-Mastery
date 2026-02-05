# 6.1 Tokio pour l'Async

## Configuration minimale

```toml
# Cargo.toml
[dependencies]
tokio = { version = "1.0", features = ["rt-multi-thread", "sync", "time", "fs"] }
```

## Runtime Tokio dans une app desktop

### Création du runtime

```rust
use std::sync::Arc;
use tokio::runtime::Runtime;

pub struct App {
    runtime: Arc<Runtime>,
    // ... autres champs
}

impl App {
    pub fn new() -> Self {
        // Créer un runtime Tokio pour les tâches async
        let runtime = Runtime::new().expect("Failed to create Tokio runtime");
        
        Self {
            runtime: Arc::new(runtime),
        }
    }
    
    /// Exécuter une tâche async depuis le thread UI
    pub fn spawn_task<F>(&self, future: F)
    where
        F: std::future::Future<Output = ()> + Send + 'static,
    {
        self.runtime.spawn(future);
    }
}
```

### Pattern : Commande asynchrone avec callback

```rust
use tokio::sync::mpsc;

pub enum AppCommand {
    ExportReport { format: ExportFormat },
    LoadData { path: PathBuf },
    Search { query: String },
}

pub enum AppEvent {
    ExportComplete { path: PathBuf },
    ExportError { error: String },
    DataLoaded { data: AppData },
    SearchResults { results: Vec<SearchResult> },
    Progress { percent: f32, message: String },
}

pub struct App {
    runtime: Arc<Runtime>,
    command_tx: mpsc::Sender<AppCommand>,
    event_rx: mpsc::Receiver<AppEvent>,
}

impl App {
    pub fn new() -> Self {
        let runtime = Arc::new(Runtime::new().unwrap());
        let (command_tx, command_rx) = mpsc::channel(100);
        let (event_tx, event_rx) = mpsc::channel(100);
        
        // Spawn le worker qui traite les commandes
        runtime.spawn(Self::command_worker(command_rx, event_tx));
        
        Self { runtime, command_tx, event_rx }
    }
    
    async fn command_worker(
        mut rx: mpsc::Receiver<AppCommand>,
        tx: mpsc::Sender<AppEvent>,
    ) {
        while let Some(cmd) = rx.recv().await {
            match cmd {
                AppCommand::ExportReport { format } => {
                    // Signaler le début
                    tx.send(AppEvent::Progress { 
                        percent: 0.0, 
                        message: "Préparation...".into() 
                    }).await.ok();
                    
                    match Self::do_export(format, &tx).await {
                        Ok(path) => {
                            tx.send(AppEvent::ExportComplete { path }).await.ok();
                        }
                        Err(e) => {
                            tx.send(AppEvent::ExportError { 
                                error: e.to_string() 
                            }).await.ok();
                        }
                    }
                }
                // ... autres commandes
            }
        }
    }
    
    async fn do_export(
        format: ExportFormat, 
        progress: &mpsc::Sender<AppEvent>
    ) -> Result<PathBuf> {
        // Simulation d'une tâche longue avec progression
        for i in 0..100 {
            tokio::time::sleep(Duration::from_millis(50)).await;
            progress.send(AppEvent::Progress {
                percent: i as f32,
                message: format!("Export en cours... {}%", i),
            }).await.ok();
        }
        
        Ok(PathBuf::from("export.pdf"))
    }
    
    /// Appelé à chaque frame UI pour traiter les événements
    pub fn poll_events(&mut self) -> Vec<AppEvent> {
        let mut events = Vec::new();
        while let Ok(event) = self.event_rx.try_recv() {
            events.push(event);
        }
        events
    }
    
    /// Envoyer une commande (non-bloquant)
    pub fn send_command(&self, cmd: AppCommand) {
        self.command_tx.try_send(cmd).ok();
    }
}
```

## Utilisation dans l'UI

```rust
impl App {
    pub fn update(&mut self, ctx: &egui::Context) {
        // Traiter les événements reçus
        for event in self.poll_events() {
            match event {
                AppEvent::Progress { percent, message } => {
                    self.export_progress = Some((percent, message));
                    ctx.request_repaint();
                }
                AppEvent::ExportComplete { path } => {
                    self.export_progress = None;
                    self.show_success(format!("Export réussi : {:?}", path));
                }
                AppEvent::ExportError { error } => {
                    self.export_progress = None;
                    self.show_error(error);
                }
                _ => {}
            }
        }
        
        // Rendu UI...
    }
    
    fn render_export_button(&mut self, ui: &mut egui::Ui) {
        if let Some((percent, msg)) = &self.export_progress {
            ui.label(msg);
            ui.add(egui::ProgressBar::new(*percent / 100.0));
        } else if ui.button("Exporter").clicked() {
            // Non-bloquant : envoie une commande
            self.send_command(AppCommand::ExportReport { 
                format: ExportFormat::PDF 
            });
        }
    }
}
```

## Résumé

- **Runtime Tokio** : Créé une fois au démarrage, partagé via `Arc`
- **Channels** : Communication non-bloquante entre UI et workers
- **Spawn** : Tâches async exécutées en arrière-plan
- **Poll** : Vérifier les événements à chaque frame UI
