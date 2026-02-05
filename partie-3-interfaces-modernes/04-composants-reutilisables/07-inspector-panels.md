# 11.7 Inspector Panels

## Panneaux de debug et inspection

Les inspector panels sont des outils de développement intégrés pour inspecter l'état de l'application.

### Panneau d'inspection de l'état

```rust
pub struct StateInspector {
    expanded: bool,
    selected_path: Option<String>,
}

impl StateInspector {
    pub fn new() -> Self {
        Self {
            expanded: false,
            selected_path: None,
        }
    }
    
    pub fn show(&mut self, ui: &mut egui::Ui, state: &AppState) {
        egui::CollapsingHeader::new("🔍 State Inspector")
            .default_open(self.expanded)
            .show(ui, |ui| {
                self.render_state_tree(ui, state);
            });
    }
    
    fn render_state_tree(&mut self, ui: &mut egui::Ui, state: &AppState) {
        // Afficher l'arbre de l'état de manière explorable
        // Utiliser serde pour sérialiser et afficher
    }
}
```

### Panneau de performance

```rust
pub struct PerformancePanel {
    frame_times: Vec<f32>,
    max_samples: usize,
}

impl PerformancePanel {
    pub fn new() -> Self {
        Self {
            frame_times: Vec::new(),
            max_samples: 100,
        }
    }
    
    pub fn record_frame(&mut self, frame_time_ms: f32) {
        self.frame_times.push(frame_time_ms);
        if self.frame_times.len() > self.max_samples {
            self.frame_times.remove(0);
        }
    }
    
    pub fn show(&mut self, ui: &mut egui::Ui) {
        egui::CollapsingHeader::new("⚡ Performance")
            .show(ui, |ui| {
                let avg = self.frame_times.iter().sum::<f32>() / self.frame_times.len() as f32;
                ui.label(format!("Avg frame time: {:.2}ms", avg));
                ui.label(format!("FPS: {:.0}", 1000.0 / avg));
                
                // Graphique avec egui_plot
            });
    }
}
```

### Panneau de logs

```rust
pub struct LogPanel {
    logs: Vec<LogEntry>,
    max_logs: usize,
    filter_level: Option<LogLevel>,
}

pub struct LogEntry {
    level: LogLevel,
    message: String,
    timestamp: std::time::Instant,
}

impl LogPanel {
    pub fn new() -> Self {
        Self {
            logs: Vec::new(),
            max_logs: 1000,
            filter_level: None,
        }
    }
    
    pub fn add_log(&mut self, level: LogLevel, message: String) {
        self.logs.push(LogEntry {
            level,
            message,
            timestamp: std::time::Instant::now(),
        });
        
        if self.logs.len() > self.max_logs {
            self.logs.remove(0);
        }
    }
    
    pub fn show(&mut self, ui: &mut egui::Ui) {
        egui::CollapsingHeader::new("📋 Logs")
            .show(ui, |ui| {
                egui::ScrollArea::vertical()
                    .stick_to_bottom(true)
                    .show(ui, |ui| {
                        for log in &self.logs {
                            if self.filter_level.map(|l| l == log.level).unwrap_or(true) {
                                let color = match log.level {
                                    LogLevel::Error => egui::Color32::RED,
                                    LogLevel::Warning => egui::Color32::YELLOW,
                                    LogLevel::Info => egui::Color32::BLUE,
                                    LogLevel::Debug => egui::Color32::GRAY,
                                };
                                
                                ui.label(
                                    egui::RichText::new(&log.message)
                                        .color(color)
                                );
                            }
                        }
                    });
            });
    }
}
```

## Usage

```rust
let mut inspector = StateInspector::new();
let mut perf_panel = PerformancePanel::new();
let mut log_panel = LogPanel::new();

// Dans le rendu
egui::SidePanel::right("inspector")
    .show(ctx, |ui| {
        inspector.show(ui, &app_state);
        perf_panel.show(ui);
        log_panel.show(ui);
    });
```

## Features

- **State Inspector** : Explorer l'état de l'application
- **Performance Panel** : Mesurer les FPS et temps de frame
- **Log Panel** : Afficher les logs avec filtrage
- **Développement** : Outils intégrés pour le debug
