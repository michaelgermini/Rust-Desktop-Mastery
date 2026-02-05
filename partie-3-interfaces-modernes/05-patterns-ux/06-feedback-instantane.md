# 12.6 Feedback Instantané

## Optimistic Updates

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
```

## Usage pattern

```rust
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

## Micro-animations de feedback

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

## Règle des 100ms

L'utilisateur doit voir une réaction dans les 100ms suivant son action :
- **0-100ms** : Feedback visuel immédiat (changement d'état du bouton)
- **100ms-1s** : Animation ou indicateur de chargement
- **1s+** : Message de progression

## Résumé

- **Optimistic updates** : Appliquer immédiatement, confirmer après
- **Animations** : Feedback visuel subtil
- **100ms** : Réaction immédiate requise
- **Rollback** : Annuler si l'opération échoue
