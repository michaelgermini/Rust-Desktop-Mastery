# 15.4 Flux de Données Clair

## Unidirectional data flow

Le flux de données doit être unidirectionnel et prévisible.

```
┌─────────────┐
│    State    │
│  (Source)   │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│     UI      │
│  (Render)   │
└──────┬──────┘
       │
       │ User Action
       ▼
┌─────────────┐
│  Commands   │
│  (Actions)  │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│   Workers   │
│  (Process)  │
└──────┬──────┘
       │
       │ Events
       ▼
┌─────────────┐
│    State    │
│  (Update)   │
└─────────────┘
```

## Source of truth unique

```rust
pub struct AppState {
    // Une seule source de vérité
    invoices: Arc<ArcSwap<Vec<Invoice>>>,
    customers: Arc<ArcSwap<Vec<Customer>>>,
    ui_state: UiState,
}

impl AppState {
    pub fn update_invoices<F>(&self, f: F)
    where
        F: FnOnce(&Vec<Invoice>) -> Vec<Invoice>,
    {
        self.invoices.rcu(|invoices| Arc::new(f(invoices)));
    }
    
    pub fn get_invoices(&self) -> Arc<Vec<Invoice>> {
        self.invoices.load_full()
    }
}
```

## Dérivation de l'état

```rust
impl AppState {
    // État dérivé : calculé depuis l'état source
    pub fn total_revenue(&self) -> Decimal {
        self.get_invoices()
            .iter()
            .filter(|inv| inv.status == InvoiceStatus::Paid)
            .map(|inv| inv.total(vat_rate))
            .sum()
    }
    
    pub fn pending_invoices(&self) -> Vec<Invoice> {
        self.get_invoices()
            .iter()
            .filter(|inv| inv.status == InvoiceStatus::Sent)
            .cloned()
            .collect()
    }
    
    pub fn customer_invoices(&self, customer_id: CustomerId) -> Vec<Invoice> {
        self.get_invoices()
            .iter()
            .filter(|inv| inv.customer_id == customer_id)
            .cloned()
            .collect()
    }
}
```

## Architecture complète

```rust
pub struct App {
    event_bus: EventBus,
    state: AppState,
    workers: WorkerPool,
}

impl eframe::App for App {
    fn update(&mut self, ctx: &egui::Context, _frame: &mut eframe::Frame) {
        // 1. Traiter les événements (mise à jour de l'état)
        for event in self.event_bus.poll_events() {
            self.handle_event(event);
        }
        
        // 2. Rendu UI (lecture de l'état)
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
                self.state.update_invoices(|_| data.invoices);
            }
            Event::InvoiceCreated { invoice } => {
                self.state.update_invoices(|invoices| {
                    let mut new = (*invoices).clone();
                    new.push(invoice);
                    new
                });
            }
            Event::Progress { task, percent } => {
                self.state.progress.insert(task, percent);
            }
            Event::Error { message } => {
                self.state.toasts.error(message);
            }
        }
    }
    
    fn render(&mut self, ctx: &egui::Context) {
        // Lire l'état (jamais le modifier directement)
        let invoices = self.state.get_invoices();
        let total_revenue = self.state.total_revenue();
        
        egui::CentralPanel::default().show(ctx, |ui| {
            ui.label(format!("Total revenue: {}", total_revenue));
            
            // Actions utilisateur → Commands
            if ui.button("Create Invoice").clicked() {
                self.send_command(Command::CreateInvoice { /* ... */ });
            }
        });
    }
    
    fn send_command(&self, cmd: Command) {
        self.event_bus.command_sender().send(cmd).ok();
    }
}
```

## Résumé

- **Unidirectional** : State → UI → Commands → Workers → Events → State
- **Source unique** : Une seule source de vérité
- **Dérivation** : État calculé depuis l'état source
- **Prévisibilité** : Flux clair et traçable
