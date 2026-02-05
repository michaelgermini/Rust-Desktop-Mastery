# Chapitre 6 : Async et Concurrence Desktop

## Introduction

Une application desktop doit rester réactive même pendant des opérations longues. Ce chapitre explique comment utiliser la concurrence en Rust pour éviter les freezes d'interface tout en gérant correctement les ressources partagées.

---

## 6.1 Le Problème du Thread UI

### Pourquoi l'interface freeze

```rust
// ❌ MAUVAIS : Tout sur le thread UI
fn handle_export_button(ui: &mut Ui, data: &AppData) {
    if ui.button("Exporter").clicked() {
        // Cette opération bloque le thread UI pendant 5 secondes
        let report = generate_large_report(data);  // 3 secondes
        save_to_file(&report);                      // 2 secondes
        
        // Pendant ce temps : l'interface est gelée
        // L'utilisateur ne peut pas annuler
        // L'OS peut afficher "Application ne répond pas"
    }
}
```

### La règle d'or

```
Règle : Le thread UI ne doit JAMAIS bloquer plus de 16ms
        (1 frame à 60 FPS)

Opérations > 16ms :
- Requêtes base de données
- Lecture/écriture fichiers
- Calculs lourds
- Requêtes réseau

Ces opérations doivent être déplacées sur des threads workers.
```

### Architecture thread UI / Workers

```
┌─────────────────────────────────────────────────────┐
│                    Thread UI                         │
│  - Rendu de l'interface (60 FPS)                    │
│  - Gestion des événements (clics, clavier)          │
│  - Mise à jour de l'affichage                       │
│  - Envoi de commandes aux workers                   │
│  - Réception des résultats                          │
└─────────────────┬───────────────────────────────────┘
                  │ Channels (non-bloquants)
┌─────────────────▼───────────────────────────────────┐
│                 Workers (Pool de threads)            │
│  - Calculs lourds                                   │
│  - I/O fichiers                                     │
│  - Requêtes base de données                         │
│  - Requêtes réseau                                  │
└─────────────────────────────────────────────────────┘
```

---

## 6.2 Tokio pour l'Async

### Configuration minimale

```toml
# Cargo.toml
[dependencies]
tokio = { version = "1.0", features = ["rt-multi-thread", "sync", "time", "fs"] }
```

### Runtime Tokio dans une app desktop

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

### Intégration avec egui

```rust
impl eframe::App for App {
    fn update(&mut self, ctx: &egui::Context, _frame: &mut eframe::Frame) {
        // 1. Traiter les événements des workers
        for event in self.poll_events() {
            match event {
                AppEvent::Progress { percent, message } => {
                    self.progress = Some((percent, message));
                }
                AppEvent::ExportComplete { path } => {
                    self.progress = None;
                    self.show_success(&format!("Export terminé: {:?}", path));
                }
                AppEvent::ExportError { error } => {
                    self.progress = None;
                    self.show_error(&error);
                }
                // ...
            }
        }
        
        // 2. Rendu UI
        egui::CentralPanel::default().show(ctx, |ui| {
            if let Some((percent, message)) = &self.progress {
                // Afficher la progression
                ui.add(egui::ProgressBar::new(*percent / 100.0).text(message));
                
                // Demander un repaint pour mettre à jour la progression
                ctx.request_repaint();
            } else {
                // UI normale
                if ui.button("Exporter PDF").clicked() {
                    self.send_command(AppCommand::ExportReport { 
                        format: ExportFormat::Pdf 
                    });
                }
            }
        });
    }
}
```

---

## 6.3 Channels avec Crossbeam

### Pourquoi crossbeam ?

Les channels de la std sont limités. Crossbeam offre :
- Channels multi-producteur multi-consommateur (MPMC)
- Meilleure performance
- Select sur plusieurs channels
- Channels bornés et non-bornés

```toml
# Cargo.toml
[dependencies]
crossbeam-channel = "0.5"
```

### Types de channels

```rust
use crossbeam_channel::{bounded, unbounded, select, Receiver, Sender};

// Channel borné : bloque si plein (backpressure)
let (tx, rx): (Sender<Message>, Receiver<Message>) = bounded(100);

// Channel non-borné : ne bloque jamais (attention à la mémoire)
let (tx, rx) = unbounded();

// Envoyer
tx.send(Message::Data(vec![1, 2, 3]))?;

// Recevoir (bloquant)
let msg = rx.recv()?;

// Recevoir (non-bloquant) - pour le thread UI
match rx.try_recv() {
    Ok(msg) => handle_message(msg),
    Err(crossbeam_channel::TryRecvError::Empty) => { /* rien à faire */ }
    Err(crossbeam_channel::TryRecvError::Disconnected) => { /* channel fermé */ }
}
```

### Select : attendre sur plusieurs channels

```rust
use crossbeam_channel::{select, bounded, unbounded};

let (tx_ui, rx_ui) = bounded(10);      // Commandes UI
let (tx_net, rx_net) = unbounded();    // Réponses réseau
let (tx_timer, rx_timer) = bounded(1); // Timer

// Dans le worker
loop {
    select! {
        recv(rx_ui) -> msg => {
            match msg {
                Ok(UiCommand::Quit) => break,
                Ok(UiCommand::Refresh) => refresh_data(),
                Err(_) => break,  // Channel fermé
            }
        }
        recv(rx_net) -> msg => {
            if let Ok(response) = msg {
                process_network_response(response);
            }
        }
        recv(rx_timer) -> _ => {
            // Tick périodique
            check_for_updates();
        }
        default(Duration::from_millis(100)) => {
            // Timeout : rien reçu
            idle_work();
        }
    }
}
```

---

## 6.4 Arc et ArcSwap pour l'État Partagé

### Le problème de l'état partagé

```rust
// ❌ Ce code ne compile pas
fn main() {
    let data = vec![1, 2, 3];
    
    std::thread::spawn(|| {
        println!("{:?}", data);  // Erreur : `data` n'a pas la bonne lifetime
    });
}
```

### Arc : Atomic Reference Counting

```rust
use std::sync::Arc;

fn main() {
    let data = Arc::new(vec![1, 2, 3]);
    
    let data_clone = Arc::clone(&data);
    std::thread::spawn(move || {
        println!("{:?}", data_clone);  // ✅ OK
    });
    
    println!("{:?}", data);  // ✅ OK aussi
}
```

### Arc + Mutex : lecture ET écriture partagée

```rust
use std::sync::{Arc, Mutex};

#[derive(Default)]
pub struct SharedState {
    pub counter: i32,
    pub messages: Vec<String>,
}

pub struct App {
    state: Arc<Mutex<SharedState>>,
}

impl App {
    pub fn new() -> Self {
        Self {
            state: Arc::new(Mutex::new(SharedState::default())),
        }
    }
    
    pub fn increment(&self) {
        let mut state = self.state.lock().unwrap();
        state.counter += 1;
    }
    
    pub fn spawn_worker(&self) {
        let state = Arc::clone(&self.state);
        
        std::thread::spawn(move || {
            // Le worker peut modifier l'état
            let mut state = state.lock().unwrap();
            state.messages.push("Message from worker".to_string());
        });
    }
}
```

### ArcSwap : lecture sans lock

Quand les lectures sont fréquentes et les écritures rares, `ArcSwap` est plus performant :

```toml
# Cargo.toml
[dependencies]
arc-swap = "1.6"
```

```rust
use arc_swap::ArcSwap;
use std::sync::Arc;

#[derive(Clone)]
pub struct AppData {
    pub items: Vec<Item>,
    pub settings: Settings,
}

pub struct App {
    // Les lectures ne nécessitent pas de lock
    data: Arc<ArcSwap<AppData>>,
}

impl App {
    pub fn new(initial_data: AppData) -> Self {
        Self {
            data: Arc::new(ArcSwap::new(Arc::new(initial_data))),
        }
    }
    
    /// Lecture rapide (pas de lock)
    pub fn get_items(&self) -> Vec<Item> {
        // load() retourne un Guard qui déréférence vers Arc<AppData>
        self.data.load().items.clone()
    }
    
    /// Écriture atomique (remplace tout l'état)
    pub fn update_items(&self, new_items: Vec<Item>) {
        // Charger l'état actuel
        let current = self.data.load();
        
        // Créer un nouvel état avec les modifications
        let mut new_data = (**current).clone();
        new_data.items = new_items;
        
        // Remplacer atomiquement
        self.data.store(Arc::new(new_data));
    }
    
    /// Worker qui peut lire l'état
    pub fn spawn_reader(&self) {
        let data = Arc::clone(&self.data);
        
        std::thread::spawn(move || {
            loop {
                // Lecture très rapide, pas de contention
                let items = data.load().items.len();
                println!("Current item count: {}", items);
                std::thread::sleep(Duration::from_secs(1));
            }
        });
    }
}
```

### Choisir le bon outil

| Situation | Utiliser |
|-----------|----------|
| Pas de partage entre threads | Pas besoin d'Arc |
| Lecture seule partagée | `Arc<T>` |
| Lectures fréquentes, écritures rares | `Arc<ArcSwap<T>>` |
| Lectures et écritures fréquentes | `Arc<Mutex<T>>` ou `Arc<RwLock<T>>` |
| Haute performance avec écritures | `Arc<RwLock<T>>` ou channels |

---

## 6.5 Éviter les Freezes

### Pattern 1 : Chunked Processing

```rust
/// Traiter de grandes quantités de données par chunks
pub async fn process_large_dataset(
    data: Vec<Item>,
    progress_tx: mpsc::Sender<f32>,
) -> Result<ProcessedData> {
    let total = data.len();
    let chunk_size = 100;
    let mut results = Vec::with_capacity(total);
    
    for (i, chunk) in data.chunks(chunk_size).enumerate() {
        // Traiter un chunk
        for item in chunk {
            results.push(process_item(item)?);
        }
        
        // Envoyer la progression
        let progress = (i * chunk_size) as f32 / total as f32;
        progress_tx.send(progress).await.ok();
        
        // Yield pour permettre à d'autres tâches de s'exécuter
        tokio::task::yield_now().await;
    }
    
    Ok(ProcessedData { results })
}
```

### Pattern 2 : Annulation cooperative

```rust
use tokio::sync::watch;
use tokio_util::sync::CancellationToken;

pub struct CancellableTask {
    cancel_token: CancellationToken,
}

impl CancellableTask {
    pub fn new() -> Self {
        Self {
            cancel_token: CancellationToken::new(),
        }
    }
    
    pub fn cancel(&self) {
        self.cancel_token.cancel();
    }
    
    pub async fn run_export(&self, data: &[Item]) -> Result<Option<Export>> {
        let mut export = Export::new();
        
        for (i, item) in data.iter().enumerate() {
            // Vérifier si annulé
            if self.cancel_token.is_cancelled() {
                return Ok(None);  // Annulé proprement
            }
            
            export.add_item(item)?;
            
            // Permettre l'annulation entre les items
            tokio::select! {
                _ = self.cancel_token.cancelled() => {
                    return Ok(None);
                }
                _ = tokio::time::sleep(Duration::from_millis(1)) => {
                    // Continue
                }
            }
        }
        
        Ok(Some(export))
    }
}

// Dans l'UI
if ui.button("Annuler").clicked() {
    self.current_task.as_ref().map(|t| t.cancel());
}
```

### Pattern 3 : Debouncing pour les recherches

```rust
use tokio::time::{sleep, Duration, Instant};

pub struct SearchDebouncer {
    last_query: String,
    last_change: Instant,
    debounce_duration: Duration,
}

impl SearchDebouncer {
    pub fn new(debounce_ms: u64) -> Self {
        Self {
            last_query: String::new(),
            last_change: Instant::now(),
            debounce_duration: Duration::from_millis(debounce_ms),
        }
    }
    
    /// Retourne Some(query) si on doit lancer la recherche
    pub fn update(&mut self, query: &str) -> Option<String> {
        if query != self.last_query {
            self.last_query = query.to_string();
            self.last_change = Instant::now();
            None  // Attendre le debounce
        } else if self.last_change.elapsed() >= self.debounce_duration {
            Some(self.last_query.clone())
        } else {
            None
        }
    }
}

// Usage dans l'UI
fn update(&mut self, ctx: &egui::Context, _frame: &mut eframe::Frame) {
    let response = ui.text_edit_singleline(&mut self.search_query);
    
    if response.changed() {
        if let Some(query) = self.debouncer.update(&self.search_query) {
            // Lancer la recherche seulement après 300ms d'inactivité
            self.send_command(AppCommand::Search { query });
        }
        
        // Demander un repaint pour vérifier le debounce
        ctx.request_repaint_after(Duration::from_millis(50));
    }
}
```

### Pattern 4 : Loading states

```rust
#[derive(Clone)]
pub enum LoadingState<T> {
    Idle,
    Loading { message: String, progress: Option<f32> },
    Loaded(T),
    Error(String),
}

impl<T> LoadingState<T> {
    pub fn is_loading(&self) -> bool {
        matches!(self, LoadingState::Loading { .. })
    }
    
    pub fn render(&self, ui: &mut egui::Ui, render_loaded: impl FnOnce(&mut egui::Ui, &T)) {
        match self {
            LoadingState::Idle => {
                ui.label("Prêt");
            }
            LoadingState::Loading { message, progress } => {
                ui.horizontal(|ui| {
                    ui.spinner();
                    ui.label(message);
                });
                if let Some(p) = progress {
                    ui.add(egui::ProgressBar::new(*p));
                }
            }
            LoadingState::Loaded(data) => {
                render_loaded(ui, data);
            }
            LoadingState::Error(msg) => {
                ui.colored_label(egui::Color32::RED, format!("Erreur: {}", msg));
            }
        }
    }
}
```

---

## Résumé du Chapitre

### Architecture recommandée

```
Thread UI (main)
│
├── Rendu à 60 FPS
├── Gestion des inputs
├── Poll des channels (non-bloquant)
│
└── Communication via channels ←→ Workers (tokio runtime)
                                   │
                                   ├── I/O fichiers
                                   ├── Base de données
                                   ├── Calculs lourds
                                   └── Réseau
```

### Checklist anti-freeze

- [ ] Aucune opération I/O sur le thread UI
- [ ] Aucun calcul > 16ms sur le thread UI
- [ ] Channels non-bloquants pour communiquer
- [ ] Progress feedback pour les opérations longues
- [ ] Possibilité d'annuler les tâches longues
- [ ] Debouncing pour les recherches/filtres temps réel

---

**Dans le prochain chapitre, nous verrons comment ajouter des logs professionnels et profiler l'application pour optimiser les performances.**
