# 15.1 Channels

## Communication via channels

Les channels permettent une communication thread-safe entre composants.

```rust
use crossbeam_channel::{bounded, unbounded, Sender, Receiver};

/// Messages de l'UI vers les workers
pub enum Command {
    LoadData { path: PathBuf },
    SaveDocument { doc: Document },
    Search { query: String },
    Export { format: ExportFormat },
    Quit,
}

/// Messages des workers vers l'UI
pub enum Event {
    DataLoaded { data: AppData },
    DocumentSaved { path: PathBuf },
    SearchResults { results: Vec<SearchResult> },
    ExportComplete { path: PathBuf },
    Progress { task: String, percent: f32 },
    Error { message: String },
}

pub struct EventBus {
    command_tx: Sender<Command>,
    command_rx: Receiver<Command>,
    event_tx: Sender<Event>,
    event_rx: Receiver<Event>,
}

impl EventBus {
    pub fn new() -> Self {
        let (command_tx, command_rx) = bounded(100);
        let (event_tx, event_rx) = bounded(100);
        Self { command_tx, command_rx, event_tx, event_rx }
    }
    
    pub fn command_sender(&self) -> Sender<Command> {
        self.command_tx.clone()
    }
    
    pub fn event_sender(&self) -> Sender<Event> {
        self.event_tx.clone()
    }
    
    pub fn poll_events(&self) -> Vec<Event> {
        let mut events = Vec::new();
        while let Ok(event) = self.event_rx.try_recv() {
            events.push(event);
        }
        events
    }
}
```

## Types de messages

### Commands (UI → Workers)

```rust
pub enum Command {
    // Actions utilisateur
    LoadData { path: PathBuf },
    SaveDocument { doc: Document },
    
    // Recherche
    Search { query: String },
    
    // Export
    Export { format: ExportFormat, path: PathBuf },
    
    // Contrôle
    Quit,
    CancelTask { task_id: TaskId },
}
```

### Events (Workers → UI)

```rust
pub enum Event {
    // Résultats
    DataLoaded { data: AppData },
    DocumentSaved { path: PathBuf },
    SearchResults { results: Vec<SearchResult> },
    
    // Progression
    Progress { task: String, percent: f32 },
    
    // État
    TaskStarted { task: String },
    TaskComplete { task: String },
    
    // Erreurs
    Error { message: String, task: Option<String> },
}
```

## Patterns de routing

### Pattern 1 : Single receiver

```rust
// Un seul worker écoute les commands
let command_rx = event_bus.command_receiver();
std::thread::spawn(move || {
    while let Ok(cmd) = command_rx.recv() {
        match cmd {
            Command::LoadData { path } => {
                // Traiter...
            }
            Command::Quit => break,
            _ => {}
        }
    }
});
```

### Pattern 2 : Multiple receivers (broadcast)

```rust
use crossbeam_channel::select;

// Plusieurs workers peuvent écouter
let command_rx1 = event_bus.command_receiver();
let command_rx2 = event_bus.command_receiver();

loop {
    select! {
        recv(command_rx1) -> cmd => {
            // Worker 1 traite
        }
        recv(command_rx2) -> cmd => {
            // Worker 2 traite
        }
    }
}
```

## Bounded vs Unbounded

### Bounded (taille limitée)

```rust
let (tx, rx) = bounded(100);  // Buffer de 100 messages

// Si le buffer est plein, send() bloque
tx.send(msg)?;  // Peut bloquer
```

**Avantages** : Contrôle de la mémoire, backpressure automatique  
**Inconvénients** : Peut bloquer si le receiver est lent

### Unbounded (illimité)

```rust
let (tx, rx) = unbounded();

// Ne bloque jamais
tx.send(msg)?;  // Ne bloque jamais
```

**Avantages** : Ne bloque jamais  
**Inconvénients** : Peut consommer beaucoup de mémoire

## Résumé

- **Channels** : Communication thread-safe entre threads
- **Commands** : UI → Workers
- **Events** : Workers → UI
- **Bounded** : Contrôle de la mémoire
- **Unbounded** : Performance maximale
