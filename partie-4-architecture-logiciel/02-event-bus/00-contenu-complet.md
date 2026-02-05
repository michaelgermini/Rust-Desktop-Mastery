# Chapitre 15 : Event Bus et Communication Interne

## Introduction

Dans une application desktop, différents composants doivent communiquer : l'UI doit notifier les workers, les workers doivent mettre à jour l'état. Ce chapitre présente les patterns de communication efficaces.

---

## 15.1 Channels : La Base

```rust
use crossbeam_channel::{bounded, Sender, Receiver};

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

---

## 15.2 Pub/Sub Pattern

```rust
use std::collections::HashMap;
use std::any::{Any, TypeId};

pub trait Message: Clone + Send + 'static {}

pub struct PubSub {
    subscribers: HashMap<TypeId, Vec<Box<dyn Fn(&dyn Any) + Send + Sync>>>,
}

impl PubSub {
    pub fn new() -> Self {
        Self { subscribers: HashMap::new() }
    }
    
    pub fn subscribe<M: Message>(&mut self, handler: impl Fn(&M) + Send + Sync + 'static) {
        let type_id = TypeId::of::<M>();
        let boxed_handler = Box::new(move |msg: &dyn Any| {
            if let Some(typed_msg) = msg.downcast_ref::<M>() {
                handler(typed_msg);
            }
        });
        self.subscribers.entry(type_id).or_default().push(boxed_handler);
    }
    
    pub fn publish<M: Message>(&self, message: M) {
        let type_id = TypeId::of::<M>();
        if let Some(handlers) = self.subscribers.get(&type_id) {
            for handler in handlers {
                handler(&message);
            }
        }
    }
}
```

---

## 15.3 Architecture Complète

```rust
pub struct App {
    event_bus: EventBus,
    state: AppState,
    workers: WorkerPool,
}

impl eframe::App for App {
    fn update(&mut self, ctx: &egui::Context, _frame: &mut eframe::Frame) {
        // 1. Traiter les événements
        for event in self.event_bus.poll_events() {
            self.handle_event(event);
        }
        
        // 2. Rendu UI
        self.render(ctx);
        
        // 3. Demander repaint si workers actifs
        if self.workers.has_active_tasks() {
            ctx.request_repaint();
        }
    }
}

impl App {
    fn handle_event(&mut self, event: Event) {
        match event {
            Event::DataLoaded { data } => {
                self.state.data = LoadState::Loaded(data);
            }
            Event::Progress { task, percent } => {
                self.state.progress.insert(task, percent);
            }
            Event::Error { message } => {
                self.state.toasts.error(message);
            }
            // ... autres événements
        }
    }
    
    fn send_command(&self, cmd: Command) {
        self.event_bus.command_sender().send(cmd).ok();
    }
}
```

---

## Résumé

- **Channels** : Communication unidirectionnelle simple
- **Pub/Sub** : Découplage total entre émetteurs et récepteurs
- **Event Bus** : Centralise la communication de l'application
