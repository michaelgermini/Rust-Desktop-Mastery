# 7.2 Logs Visuels pour Desktop

## Logger vers l'UI

### Buffer de logs

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
}
```

### Intégration avec tracing

```rust
use tracing_subscriber::layer::SubscriberExt;
use tracing_subscriber::Registry;

pub struct UiLogLayer {
    buffer: Arc<UiLogBuffer>,
}

impl<S> tracing_subscriber::Layer<S> for UiLogLayer
where
    S: tracing::Subscriber,
{
    fn on_event(
        &self,
        event: &tracing::Event<'_>,
        _ctx: &tracing_subscriber::layer::Context<'_, S>,
    ) {
        let level = match *event.metadata().level() {
            tracing::Level::TRACE => LogLevel::Trace,
            tracing::Level::DEBUG => LogLevel::Debug,
            tracing::Level::INFO => LogLevel::Info,
            tracing::Level::WARN => LogLevel::Warn,
            tracing::Level::ERROR => LogLevel::Error,
        };
        
        let message = format!("{}", event);
        
        self.buffer.push(LogEntry {
            timestamp: chrono::Local::now(),
            level,
            message,
            target: event.metadata().target().to_string(),
        });
    }
}

fn setup_ui_logging(buffer: Arc<UiLogBuffer>) {
    let ui_layer = UiLogLayer { buffer };
    
    tracing_subscriber::registry()
        .with(ui_layer)
        .with(fmt::layer())  // Aussi dans le terminal
        .init();
}
```

### Affichage dans l'UI

```rust
pub fn show_log_panel(ui: &mut Ui, log_buffer: &UiLogBuffer, tokens: &DesignTokens) {
    egui::Window::new("Logs")
        .resizable(true)
        .show(ui.ctx(), |ui| {
            // Filtres
            ui.horizontal(|ui| {
                ui.label("Niveau:");
                ui.selectable_value(&mut filter_level, LogLevel::All, "Tous");
                ui.selectable_value(&mut filter_level, LogLevel::Info, "Info+");
                ui.selectable_value(&mut filter_level, LogLevel::Warn, "Warn+");
                ui.selectable_value(&mut filter_level, LogLevel::Error, "Erreurs");
            });
            
            ui.separator();
            
            // Liste des logs
            egui::ScrollArea::vertical()
                .auto_shrink([false; 2])
                .show(ui, |ui| {
                    for entry in log_buffer.get_entries() {
                        if should_show(&entry, &filter_level) {
                            let color = match entry.level {
                                LogLevel::Error => tokens.colors.error,
                                LogLevel::Warn => tokens.colors.warning,
                                LogLevel::Info => tokens.colors.info,
                                _ => tokens.colors.text_secondary,
                            };
                            
                            ui.horizontal(|ui| {
                                ui.colored_label(color, &format!("{:?}", entry.level));
                                ui.label(&entry.timestamp.format("%H:%M:%S").to_string());
                                ui.label(&entry.message);
                            });
                        }
                    }
                });
        });
}
```

## Résumé

- **Buffer** : Stocker les logs dans un VecDeque thread-safe
- **Layer custom** : Intégrer avec tracing
- **UI** : Afficher dans un panneau avec filtres
- **Performance** : Limiter le nombre d'entrées pour éviter les ralentissements
