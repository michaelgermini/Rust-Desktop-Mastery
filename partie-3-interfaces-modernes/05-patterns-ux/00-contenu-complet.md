# Chapitre 12 : Patterns UX Professionnels

## Introduction

Ce chapitre présente les patterns d'expérience utilisateur qui distinguent une application amateur d'un produit professionnel. Ces patterns, souvent négligés, sont essentiels pour créer des logiciels que les utilisateurs apprécient.

---

## 12.1 Loading States

### Les trois états d'une ressource

```rust
#[derive(Clone)]
pub enum LoadState<T> {
    /// Pas encore chargé
    Idle,
    /// Chargement en cours
    Loading { 
        started_at: Instant,
        message: Option<String>,
    },
    /// Chargé avec succès
    Loaded(T),
    /// Erreur de chargement
    Error { 
        message: String,
        retry_count: u32,
    },
}

impl<T> LoadState<T> {
    pub fn is_loading(&self) -> bool {
        matches!(self, LoadState::Loading { .. })
    }
    
    pub fn as_loaded(&self) -> Option<&T> {
        match self {
            LoadState::Loaded(data) => Some(data),
            _ => None,
        }
    }
}
```

### Skeleton loading

```rust
pub fn render_skeleton_row(ui: &mut Ui, tokens: &DesignTokens) {
    ui.horizontal(|ui| {
        // Avatar skeleton
        let (rect, _) = ui.allocate_exact_size(
            egui::vec2(32.0, 32.0),
            egui::Sense::hover(),
        );
        ui.painter().rect_filled(
            rect,
            tokens.radius.full,
            pulse_color(tokens.colors.bg_tertiary),
        );
        
        ui.add_space(tokens.spacing.sm);
        
        // Text skeletons
        ui.vertical(|ui| {
            skeleton_line(ui, 150.0, tokens);
            ui.add_space(4.0);
            skeleton_line(ui, 100.0, tokens);
        });
    });
}

fn skeleton_line(ui: &mut Ui, width: f32, tokens: &DesignTokens) {
    let (rect, _) = ui.allocate_exact_size(
        egui::vec2(width, 12.0),
        egui::Sense::hover(),
    );
    ui.painter().rect_filled(
        rect,
        tokens.radius.sm,
        pulse_color(tokens.colors.bg_tertiary),
    );
}

fn pulse_color(base: Color32) -> Color32 {
    // Animation de pulsation
    let t = (std::time::SystemTime::now()
        .duration_since(std::time::UNIX_EPOCH)
        .unwrap()
        .as_millis() % 1000) as f32 / 1000.0;
    
    let pulse = (t * std::f32::consts::PI * 2.0).sin() * 0.5 + 0.5;
    ColorTokens::lighten(base, pulse * 0.1)
}
```

### Pattern de chargement complet

```rust
pub fn render_with_loading<T>(
    ui: &mut Ui,
    state: &LoadState<T>,
    tokens: &DesignTokens,
    render_loaded: impl FnOnce(&mut Ui, &T),
    render_skeleton: impl FnOnce(&mut Ui),
) {
    match state {
        LoadState::Idle => {
            ui.centered_and_justified(|ui| {
                ui.label("Prêt à charger");
            });
        }
        
        LoadState::Loading { started_at, message } => {
            ui.vertical_centered(|ui| {
                ui.add_space(tokens.spacing.xl);
                
                // Afficher skeleton si chargement court
                if started_at.elapsed() < Duration::from_millis(300) {
                    render_skeleton(ui);
                } else {
                    // Spinner pour chargement long
                    ui.spinner();
                    if let Some(msg) = message {
                        ui.label(msg);
                    }
                }
            });
            
            // Demander repaint pour animation
            ui.ctx().request_repaint();
        }
        
        LoadState::Loaded(data) => {
            render_loaded(ui, data);
        }
        
        LoadState::Error { message, retry_count } => {
            ui.vertical_centered(|ui| {
                ui.add_space(tokens.spacing.xl);
                
                ui.label(
                    egui::RichText::new("❌")
                        .size(tokens.typography.size_4xl)
                );
                
                ui.label(
                    egui::RichText::new("Erreur de chargement")
                        .size(tokens.typography.size_lg)
                        .color(tokens.colors.error)
                );
                
                ui.label(
                    egui::RichText::new(message)
                        .size(tokens.typography.size_sm)
                        .color(tokens.colors.text_muted)
                );
                
                ui.add_space(tokens.spacing.md);
                
                if Button::new("Réessayer")
                    .variant(ButtonVariant::Primary)
                    .show(ui, tokens)
                    .clicked()
                {
                    // Trigger retry
                }
                
                if *retry_count > 0 {
                    ui.label(
                        egui::RichText::new(format!("Tentative {}/3", retry_count))
                            .size(tokens.typography.size_xs)
                            .color(tokens.colors.text_muted)
                    );
                }
            });
        }
    }
}
```

---

## 12.2 Empty States

### Design d'états vides engageants

```rust
pub struct EmptyState<'a> {
    icon: &'a str,
    title: &'a str,
    description: &'a str,
    action_label: Option<&'a str>,
}

impl<'a> EmptyState<'a> {
    pub fn new(icon: &'a str, title: &'a str, description: &'a str) -> Self {
        Self {
            icon,
            title,
            description,
            action_label: None,
        }
    }
    
    pub fn with_action(mut self, label: &'a str) -> Self {
        self.action_label = Some(label);
        self
    }
    
    pub fn show(self, ui: &mut Ui, tokens: &DesignTokens) -> bool {
        let mut action_clicked = false;
        
        ui.vertical_centered(|ui| {
            ui.add_space(tokens.spacing.xxxl);
            
            // Icône grande
            ui.label(
                egui::RichText::new(self.icon)
                    .size(64.0)
            );
            
            ui.add_space(tokens.spacing.lg);
            
            // Titre
            ui.label(
                egui::RichText::new(self.title)
                    .size(tokens.typography.size_xl)
                    .color(tokens.colors.text_primary)
                    .strong()
            );
            
            ui.add_space(tokens.spacing.sm);
            
            // Description
            ui.label(
                egui::RichText::new(self.description)
                    .size(tokens.typography.size_md)
                    .color(tokens.colors.text_muted)
            );
            
            // Action
            if let Some(label) = self.action_label {
                ui.add_space(tokens.spacing.lg);
                
                if Button::new(label)
                    .variant(ButtonVariant::Primary)
                    .show(ui, tokens)
                    .clicked()
                {
                    action_clicked = true;
                }
            }
            
            ui.add_space(tokens.spacing.xxxl);
        });
        
        action_clicked
    }
}

// Usage
if documents.is_empty() {
    if EmptyState::new(
        "📄",
        "Aucun document",
        "Créez votre premier document pour commencer",
    )
    .with_action("Nouveau document")
    .show(ui, &tokens)
    {
        create_new_document();
    }
}
```

### Empty states contextuels

```rust
pub fn render_search_empty_state(
    ui: &mut Ui,
    query: &str,
    tokens: &DesignTokens,
) {
    EmptyState::new(
        "🔍",
        "Aucun résultat",
        &format!("Aucun résultat pour \"{}\"", query),
    )
    .show(ui, tokens);
    
    // Suggestions
    ui.vertical_centered(|ui| {
        ui.label(
            egui::RichText::new("Suggestions:")
                .size(tokens.typography.size_sm)
                .color(tokens.colors.text_muted)
        );
        
        ui.label("• Vérifiez l'orthographe");
        ui.label("• Essayez des termes plus généraux");
        ui.label("• Utilisez moins de mots-clés");
    });
}
```

---

## 12.3 Gestion des Erreurs UX

### Messages d'erreur utilisables

```rust
pub struct UserError {
    pub title: String,
    pub message: String,
    pub suggestion: Option<String>,
    pub technical_details: Option<String>,
    pub actions: Vec<ErrorAction>,
}

pub struct ErrorAction {
    pub label: String,
    pub action_type: ErrorActionType,
}

pub enum ErrorActionType {
    Retry,
    Dismiss,
    OpenSettings,
    ContactSupport,
    Custom(Box<dyn FnOnce()>),
}

impl UserError {
    pub fn from_io_error(error: std::io::Error, context: &str) -> Self {
        match error.kind() {
            std::io::ErrorKind::NotFound => Self {
                title: "Fichier introuvable".to_string(),
                message: format!("Le fichier {} n'existe plus.", context),
                suggestion: Some("Il a peut-être été déplacé ou supprimé.".to_string()),
                technical_details: Some(error.to_string()),
                actions: vec![
                    ErrorAction { label: "Parcourir".to_string(), action_type: ErrorActionType::Custom(Box::new(|| {})) },
                    ErrorAction { label: "Fermer".to_string(), action_type: ErrorActionType::Dismiss },
                ],
            },
            std::io::ErrorKind::PermissionDenied => Self {
                title: "Accès refusé".to_string(),
                message: format!("Impossible d'accéder à {}.", context),
                suggestion: Some("Vérifiez les permissions du fichier.".to_string()),
                technical_details: Some(error.to_string()),
                actions: vec![
                    ErrorAction { label: "Fermer".to_string(), action_type: ErrorActionType::Dismiss },
                ],
            },
            _ => Self {
                title: "Erreur".to_string(),
                message: format!("Une erreur est survenue avec {}.", context),
                suggestion: None,
                technical_details: Some(error.to_string()),
                actions: vec![
                    ErrorAction { label: "Réessayer".to_string(), action_type: ErrorActionType::Retry },
                    ErrorAction { label: "Fermer".to_string(), action_type: ErrorActionType::Dismiss },
                ],
            },
        }
    }
    
    pub fn render(&self, ui: &mut Ui, tokens: &DesignTokens) -> Option<ErrorActionType> {
        let mut result = None;
        
        egui::Frame::none()
            .fill(tokens.colors.bg_elevated)
            .stroke(egui::Stroke::new(1.0, tokens.colors.error))
            .rounding(tokens.radius.md)
            .inner_margin(tokens.spacing.md)
            .show(ui, |ui| {
                ui.horizontal(|ui| {
                    ui.label(
                        egui::RichText::new("⚠️")
                            .size(tokens.typography.size_2xl)
                    );
                    
                    ui.vertical(|ui| {
                        ui.label(
                            egui::RichText::new(&self.title)
                                .size(tokens.typography.size_lg)
                                .color(tokens.colors.error)
                                .strong()
                        );
                        
                        ui.label(&self.message);
                        
                        if let Some(suggestion) = &self.suggestion {
                            ui.add_space(tokens.spacing.xs);
                            ui.label(
                                egui::RichText::new(suggestion)
                                    .size(tokens.typography.size_sm)
                                    .color(tokens.colors.text_muted)
                                    .italics()
                            );
                        }
                    });
                });
                
                // Détails techniques (collapsible)
                if let Some(details) = &self.technical_details {
                    ui.add_space(tokens.spacing.sm);
                    egui::CollapsingHeader::new("Détails techniques")
                        .default_open(false)
                        .show(ui, |ui| {
                            ui.label(
                                egui::RichText::new(details)
                                    .size(tokens.typography.size_xs)
                                    .family(egui::FontFamily::Monospace)
                                    .color(tokens.colors.text_muted)
                            );
                        });
                }
                
                // Actions
                ui.add_space(tokens.spacing.md);
                ui.horizontal(|ui| {
                    for action in &self.actions {
                        let variant = match action.action_type {
                            ErrorActionType::Retry => ButtonVariant::Primary,
                            ErrorActionType::Dismiss => ButtonVariant::Ghost,
                            _ => ButtonVariant::Secondary,
                        };
                        
                        if Button::new(&action.label)
                            .variant(variant)
                            .show(ui, tokens)
                            .clicked()
                        {
                            result = Some(action.action_type.clone());
                        }
                    }
                });
            });
        
        result
    }
}
```

---

## 12.4 Undo et Redo

### Système d'historique

```rust
pub struct UndoStack<S: Clone> {
    history: Vec<S>,
    current_index: usize,
    max_size: usize,
}

impl<S: Clone> UndoStack<S> {
    pub fn new(initial_state: S, max_size: usize) -> Self {
        Self {
            history: vec![initial_state],
            current_index: 0,
            max_size,
        }
    }
    
    /// Enregistrer un nouvel état
    pub fn push(&mut self, state: S) {
        // Supprimer l'historique futur si on est pas à la fin
        self.history.truncate(self.current_index + 1);
        
        // Ajouter le nouvel état
        self.history.push(state);
        self.current_index += 1;
        
        // Limiter la taille
        if self.history.len() > self.max_size {
            self.history.remove(0);
            self.current_index -= 1;
        }
    }
    
    /// Annuler (undo)
    pub fn undo(&mut self) -> Option<&S> {
        if self.can_undo() {
            self.current_index -= 1;
            Some(&self.history[self.current_index])
        } else {
            None
        }
    }
    
    /// Refaire (redo)
    pub fn redo(&mut self) -> Option<&S> {
        if self.can_redo() {
            self.current_index += 1;
            Some(&self.history[self.current_index])
        } else {
            None
        }
    }
    
    pub fn can_undo(&self) -> bool {
        self.current_index > 0
    }
    
    pub fn can_redo(&self) -> bool {
        self.current_index < self.history.len() - 1
    }
    
    pub fn current(&self) -> &S {
        &self.history[self.current_index]
    }
}

// Intégration avec les raccourcis clavier
fn handle_undo_redo(ctx: &egui::Context, undo_stack: &mut UndoStack<AppState>) -> Option<AppState> {
    ctx.input(|i| {
        if i.modifiers.command && i.key_pressed(egui::Key::Z) {
            if i.modifiers.shift {
                // Cmd+Shift+Z = Redo
                return undo_stack.redo().cloned();
            } else {
                // Cmd+Z = Undo
                return undo_stack.undo().cloned();
            }
        }
        None
    })
}
```

---

## 12.5 Autosave

### Sauvegarde automatique intelligente

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

---

## 12.6 Feedback Instantané

### Optimistic Updates

```rust
/// Appliquer le changement immédiatement, puis confirmer/annuler
pub struct OptimisticUpdate<T> {
    pub original: T,
    pub optimistic: T,
    pub confirmed: bool,
}

impl<T: Clone> OptimisticUpdate<T> {
    pub fn new(original: T, optimistic: T) -> Self {
        Self {
            original,
            optimistic,
            confirmed: false,
        }
    }
    
    pub fn confirm(&mut self) {
        self.confirmed = true;
    }
    
    pub fn rollback(&mut self) -> T {
        self.original.clone()
    }
    
    pub fn current(&self) -> &T {
        &self.optimistic
    }
}

// Usage pattern
async fn update_item_optimistically(
    state: &mut AppState,
    item_id: ItemId,
    new_value: Value,
) {
    // 1. Appliquer immédiatement (optimistic)
    let original = state.items.get(&item_id).cloned();
    state.items.insert(item_id, new_value.clone());
    
    // 2. Envoyer au serveur/DB
    match save_to_database(item_id, &new_value).await {
        Ok(_) => {
            // Confirmé, rien à faire
        }
        Err(e) => {
            // Rollback
            if let Some(original) = original {
                state.items.insert(item_id, original);
            }
            state.toasts.error(format!("Erreur: {}", e));
        }
    }
}
```

### Micro-animations de feedback

```rust
pub struct FeedbackAnimation {
    start_time: Option<Instant>,
    duration: Duration,
    animation_type: AnimationType,
}

pub enum AnimationType {
    Success,
    Error,
    Pulse,
}

impl FeedbackAnimation {
    pub fn trigger(&mut self, animation_type: AnimationType) {
        self.start_time = Some(Instant::now());
        self.animation_type = animation_type;
    }
    
    pub fn is_active(&self) -> bool {
        self.start_time
            .map(|t| t.elapsed() < self.duration)
            .unwrap_or(false)
    }
    
    pub fn progress(&self) -> f32 {
        self.start_time
            .map(|t| (t.elapsed().as_secs_f32() / self.duration.as_secs_f32()).min(1.0))
            .unwrap_or(1.0)
    }
    
    pub fn apply_to_rect(&self, rect: egui::Rect, painter: &egui::Painter, tokens: &DesignTokens) {
        if !self.is_active() {
            return;
        }
        
        let progress = self.progress();
        let alpha = ((1.0 - progress) * 100.0) as u8;
        
        let color = match self.animation_type {
            AnimationType::Success => ColorTokens::with_alpha(tokens.colors.success, alpha),
            AnimationType::Error => ColorTokens::with_alpha(tokens.colors.error, alpha),
            AnimationType::Pulse => ColorTokens::with_alpha(tokens.colors.primary, alpha),
        };
        
        let expansion = progress * 4.0;
        painter.rect_stroke(
            rect.expand(expansion),
            tokens.radius.sm + expansion,
            egui::Stroke::new(2.0, color),
        );
    }
}
```

---

## Résumé du Chapitre

### Checklist UX professionnelle

- [ ] Loading states avec skeleton
- [ ] Empty states engageants avec CTA
- [ ] Messages d'erreur actionnables
- [ ] Undo/Redo fonctionnel
- [ ] Autosave intelligent
- [ ] Feedback visuel immédiat
- [ ] Optimistic updates

### Principes clés

1. **Toujours afficher quelque chose** - Jamais d'écran vide
2. **Expliquer les erreurs** - Pas juste "Erreur"
3. **Permettre la récupération** - Undo, retry, suggestions
4. **Feedback instantané** - L'utilisateur doit voir que son action a été prise en compte

---

**Dans le prochain chapitre, nous verrons comment traduire des maquettes SVG en code Rust efficacement.**
