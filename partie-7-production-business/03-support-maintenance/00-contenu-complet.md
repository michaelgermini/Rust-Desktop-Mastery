# Chapitre 31 : Support et Maintenance

## Introduction

Le support et la maintenance sont essentiels pour fidéliser les utilisateurs et améliorer continuellement le produit.

---

## 31.1 Documentation Utilisateur

```rust
/// Structure de documentation intégrée
pub struct InAppDocumentation {
    articles: HashMap<String, HelpArticle>,
    search_index: SearchIndex,
}

pub struct HelpArticle {
    pub id: String,
    pub title: String,
    pub content: String,  // Markdown
    pub category: HelpCategory,
    pub keywords: Vec<String>,
}

impl InAppDocumentation {
    pub fn show_help_panel(&self, ui: &mut Ui, context: &str) {
        // Afficher l'aide contextuelle
        if let Some(article) = self.get_contextual_help(context) {
            egui::ScrollArea::vertical().show(ui, |ui| {
                ui.heading(&article.title);
                ui.separator();
                
                // Render markdown
                render_markdown(ui, &article.content);
            });
        }
    }
    
    pub fn show_search(&mut self, ui: &mut Ui, query: &mut String) {
        ui.horizontal(|ui| {
            ui.label("🔍");
            ui.text_edit_singleline(query);
        });
        
        if !query.is_empty() {
            let results = self.search_index.search(query);
            for article in results.iter().take(5) {
                if ui.link(&article.title).clicked() {
                    // Ouvrir l'article
                }
            }
        }
    }
}
```

### Structure de la Documentation

```
docs/
├── getting-started/
│   ├── installation.md
│   ├── first-steps.md
│   └── basic-workflow.md
├── features/
│   ├── customers.md
│   ├── invoices.md
│   ├── reports.md
│   └── export.md
├── advanced/
│   ├── keyboard-shortcuts.md
│   ├── backup-restore.md
│   └── customization.md
├── troubleshooting/
│   ├── common-issues.md
│   └── error-messages.md
└── changelog.md
```

---

## 31.2 Feedback In-App

```rust
pub struct FeedbackWidget {
    is_open: bool,
    feedback_type: FeedbackType,
    message: String,
    email: String,
    include_logs: bool,
    screenshot: Option<Vec<u8>>,
}

#[derive(Clone, PartialEq)]
pub enum FeedbackType {
    Bug,
    Feature,
    Question,
    Other,
}

impl FeedbackWidget {
    pub fn show(&mut self, ui: &mut Ui, tokens: &DesignTokens) {
        if !self.is_open {
            // Bouton flottant
            egui::Area::new(egui::Id::new("feedback_button"))
                .anchor(egui::Align2::RIGHT_BOTTOM, egui::vec2(-20.0, -20.0))
                .show(ui.ctx(), |ui| {
                    if ui.button("💬 Feedback").clicked() {
                        self.is_open = true;
                    }
                });
            return;
        }
        
        egui::Window::new("Envoyer un feedback")
            .resizable(false)
            .show(ui.ctx(), |ui| {
                // Type de feedback
                ui.horizontal(|ui| {
                    ui.selectable_value(&mut self.feedback_type, FeedbackType::Bug, "🐛 Bug");
                    ui.selectable_value(&mut self.feedback_type, FeedbackType::Feature, "💡 Idée");
                    ui.selectable_value(&mut self.feedback_type, FeedbackType::Question, "❓ Question");
                });
                
                ui.add_space(tokens.spacing.sm);
                
                // Message
                ui.label("Votre message :");
                ui.add(egui::TextEdit::multiline(&mut self.message)
                    .desired_rows(5)
                    .desired_width(f32::INFINITY));
                
                // Email (optionnel)
                ui.label("Email (optionnel, pour vous répondre) :");
                ui.text_edit_singleline(&mut self.email);
                
                // Options
                ui.checkbox(&mut self.include_logs, "Inclure les logs de diagnostic");
                
                ui.add_space(tokens.spacing.md);
                
                // Actions
                ui.horizontal(|ui| {
                    if ui.button("Annuler").clicked() {
                        self.is_open = false;
                        self.reset();
                    }
                    
                    if ui.button("Envoyer").clicked() {
                        self.submit();
                        self.is_open = false;
                        self.reset();
                    }
                });
            });
    }
    
    fn submit(&self) {
        let feedback = FeedbackSubmission {
            feedback_type: self.feedback_type.clone(),
            message: self.message.clone(),
            email: if self.email.is_empty() { None } else { Some(self.email.clone()) },
            logs: if self.include_logs { Some(collect_logs()) } else { None },
            app_version: env!("CARGO_PKG_VERSION").to_string(),
            os: std::env::consts::OS.to_string(),
        };
        
        // Envoyer en arrière-plan
        std::thread::spawn(move || {
            send_feedback(&feedback).ok();
        });
    }
}
```

---

## 31.3 Télémétrie Respectueuse

```rust
/// Télémétrie opt-in avec données anonymisées
pub struct TelemetryManager {
    enabled: bool,
    session_id: String,  // Pas d'identifiant permanent
    events: Vec<TelemetryEvent>,
}

#[derive(Serialize)]
pub struct TelemetryEvent {
    pub event_type: String,
    pub timestamp: DateTime<Utc>,
    pub properties: HashMap<String, String>,
    // PAS de données personnelles
}

impl TelemetryManager {
    pub fn new() -> Self {
        Self {
            enabled: false,  // Opt-in par défaut
            session_id: Uuid::new_v4().to_string(),
            events: Vec::new(),
        }
    }
    
    /// Demander le consentement au premier lancement
    pub fn show_consent_dialog(ui: &mut Ui) -> Option<bool> {
        let mut result = None;
        
        egui::Window::new("Améliorer Mon App")
            .collapsible(false)
            .resizable(false)
            .show(ui.ctx(), |ui| {
                ui.label("Nous collectons des données anonymes pour améliorer l'application.");
                ui.label("");
                ui.label("Données collectées :");
                ui.label("• Fonctionnalités utilisées (sans contenu)");
                ui.label("• Erreurs rencontrées");
                ui.label("• Performances");
                ui.label("");
                ui.colored_label(egui::Color32::GRAY, 
                    "Aucune donnée personnelle n'est collectée.");
                
                ui.add_space(16.0);
                
                ui.horizontal(|ui| {
                    if ui.button("Refuser").clicked() {
                        result = Some(false);
                    }
                    if ui.button("Accepter").clicked() {
                        result = Some(true);
                    }
                });
            });
        
        result
    }
    
    /// Tracker un événement (uniquement si consentement)
    pub fn track(&mut self, event_type: &str, properties: HashMap<String, String>) {
        if !self.enabled {
            return;
        }
        
        self.events.push(TelemetryEvent {
            event_type: event_type.to_string(),
            timestamp: Utc::now(),
            properties,
        });
        
        // Envoyer par batch de 10
        if self.events.len() >= 10 {
            self.flush();
        }
    }
    
    fn flush(&mut self) {
        if self.events.is_empty() {
            return;
        }
        
        let events = std::mem::take(&mut self.events);
        std::thread::spawn(move || {
            // Envoyer au serveur
            send_telemetry(&events).ok();
        });
    }
}
```

---

## 31.4 Gestion des Bugs

```rust
/// Capture automatique des erreurs
pub fn setup_panic_handler() {
    std::panic::set_hook(Box::new(|panic_info| {
        let location = panic_info.location().map(|l| {
            format!("{}:{}:{}", l.file(), l.line(), l.column())
        }).unwrap_or_default();
        
        let message = panic_info.payload()
            .downcast_ref::<&str>()
            .map(|s| s.to_string())
            .or_else(|| panic_info.payload()
                .downcast_ref::<String>()
                .cloned())
            .unwrap_or_else(|| "Unknown panic".to_string());
        
        let report = CrashReport {
            message,
            location,
            backtrace: std::backtrace::Backtrace::capture().to_string(),
            app_version: env!("CARGO_PKG_VERSION").to_string(),
            os: std::env::consts::OS.to_string(),
        };
        
        // Sauvegarder localement
        save_crash_report(&report).ok();
    }));
}
```

---

## Résumé

- **Documentation** : Intégrée, contextuelle, searchable
- **Feedback** : Widget in-app avec diagnostic optionnel
- **Télémétrie** : Opt-in, anonyme, respectueuse
- **Crash reports** : Capture automatique pour amélioration
