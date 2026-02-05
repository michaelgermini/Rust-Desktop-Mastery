# 31.2 Feedback In-App

## Widget de feedback

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
    
    fn reset(&mut self) {
        self.message.clear();
        self.email.clear();
        self.include_logs = false;
        self.screenshot = None;
    }
}
```

## Collecte de logs

```rust
fn collect_logs() -> String {
    // Collecter les logs récents
    let log_dir = dirs::data_dir()
        .unwrap_or_else(|| PathBuf::from("."))
        .join("mon-app")
        .join("logs");
    
    // Lire les derniers logs
    let mut logs = String::new();
    if let Ok(entries) = std::fs::read_dir(&log_dir) {
        for entry in entries.flatten().take(5) {
            if let Ok(content) = std::fs::read_to_string(entry.path()) {
                logs.push_str(&format!("=== {} ===\n", entry.file_name().to_string_lossy()));
                logs.push_str(&content);
                logs.push_str("\n\n");
            }
        }
    }
    
    logs
}
```

## Intégration avec support

```rust
pub struct SupportIntegration {
    pub api_url: String,
    pub api_key: String,
}

impl SupportIntegration {
    pub fn send_feedback(&self, feedback: &FeedbackSubmission) -> Result<()> {
        let client = reqwest::Client::new();
        
        let response = client
            .post(&format!("{}/api/feedback", self.api_url))
            .header("Authorization", &format!("Bearer {}", self.api_key))
            .json(feedback)
            .send()?;
        
        response.error_for_status()?;
        Ok(())
    }
}
```

## Résumé

- **Widget** : Bouton flottant facilement accessible
- **Types** : Bug, Feature, Question
- **Logs** : Option d'inclure les logs de diagnostic
- **Intégration** : Envoi automatique au système de support
