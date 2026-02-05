# 12.5 Autosave

## Sauvegarde automatique intelligente

```rust
pub struct AutoSave {
    last_content_hash: u64,
    last_save_time: Instant,
    save_delay: Duration,
    pending_save: bool,
}

impl AutoSave {
    pub fn new(save_delay: Duration) -> Self {
        Self {
            last_content_hash: 0,
            last_save_time: Instant::now(),
            save_delay,
            pending_save: false,
        }
    }
    
    /// Appelé quand le contenu change
    pub fn content_changed(&mut self, content: &impl std::hash::Hash) {
        use std::hash::{Hash, Hasher};
        use std::collections::hash_map::DefaultHasher;
        
        let mut hasher = DefaultHasher::new();
        content.hash(&mut hasher);
        let new_hash = hasher.finish();
        
        if new_hash != self.last_content_hash {
            self.last_content_hash = new_hash;
            self.pending_save = true;
        }
    }
    
    /// Retourne true si on doit sauvegarder maintenant
    pub fn should_save(&self) -> bool {
        self.pending_save && self.last_save_time.elapsed() >= self.save_delay
    }
    
    /// Marquer comme sauvegardé
    pub fn mark_saved(&mut self) {
        self.pending_save = false;
        self.last_save_time = Instant::now();
    }
    
    /// Indicateur pour l'UI
    pub fn status(&self) -> AutoSaveStatus {
        if self.pending_save {
            AutoSaveStatus::Pending
        } else {
            AutoSaveStatus::Saved(self.last_save_time)
        }
    }
}

pub enum AutoSaveStatus {
    Saved(Instant),
    Pending,
    Saving,
    Error(String),
}

impl AutoSaveStatus {
    pub fn render(&self, ui: &mut Ui, tokens: &DesignTokens) {
        let (icon, text, color) = match self {
            AutoSaveStatus::Saved(time) => {
                let elapsed = time.elapsed();
                let text = if elapsed < Duration::from_secs(5) {
                    "Sauvegardé".to_string()
                } else if elapsed < Duration::from_secs(60) {
                    format!("Sauvegardé il y a {}s", elapsed.as_secs())
                } else {
                    format!("Sauvegardé il y a {}m", elapsed.as_secs() / 60)
                };
                ("✓", text, tokens.colors.success)
            }
            AutoSaveStatus::Pending => {
                ("○", "Modifications non sauvegardées".to_string(), tokens.colors.warning)
            }
            AutoSaveStatus::Saving => {
                ("↻", "Sauvegarde...".to_string(), tokens.colors.text_muted)
            }
            AutoSaveStatus::Error(msg) => {
                ("✗", format!("Erreur: {}", msg), tokens.colors.error)
            }
        };
        
        ui.horizontal(|ui| {
            ui.label(
                egui::RichText::new(icon)
                    .size(tokens.typography.size_sm)
                    .color(color)
            );
            ui.label(
                egui::RichText::new(text)
                    .size(tokens.typography.size_xs)
                    .color(tokens.colors.text_muted)
            );
        });
    }
}
```

## Usage

```rust
let mut autosave = AutoSave::new(Duration::from_secs(2));

// Quand le contenu change
autosave.content_changed(&document);

// Dans la boucle principale
if autosave.should_save() {
    match save_document().await {
        Ok(_) => autosave.mark_saved(),
        Err(e) => {
            // Gérer l'erreur
        }
    }
}

// Afficher le statut dans l'UI
autosave.status().render(ui, &tokens);
```

## Résumé

- **Détection de changement** : Hash du contenu
- **Délai** : Sauvegarde après 2 secondes d'inactivité
- **Indicateur visuel** : Statut dans la barre de statut
- **Gestion d'erreur** : Afficher les erreurs de sauvegarde
