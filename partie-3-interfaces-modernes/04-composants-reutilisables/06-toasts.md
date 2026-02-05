# 11.6 Toasts

## Notifications toast

```rust
#[derive(Clone)]
pub enum ToastLevel {
    Info,
    Success,
    Warning,
    Error,
}

#[derive(Clone)]
pub struct Toast {
    pub id: u64,
    pub message: String,
    pub level: ToastLevel,
    pub created_at: std::time::Instant,
    pub duration: std::time::Duration,
}

pub struct ToastManager {
    toasts: Vec<Toast>,
    next_id: u64,
}

impl ToastManager {
    pub fn new() -> Self {
        Self {
            toasts: Vec::new(),
            next_id: 0,
        }
    }
    
    pub fn show(&mut self, message: impl Into<String>, level: ToastLevel) {
        self.toasts.push(Toast {
            id: self.next_id,
            message: message.into(),
            level,
            created_at: std::time::Instant::now(),
            duration: std::time::Duration::from_secs(5),
        });
        self.next_id += 1;
    }
    
    pub fn success(&mut self, message: impl Into<String>) {
        self.show(message, ToastLevel::Success);
    }
    
    pub fn error(&mut self, message: impl Into<String>) {
        self.show(message, ToastLevel::Error);
    }
    
    pub fn render(&mut self, ctx: &egui::Context, tokens: &DesignTokens) {
        // Supprimer les toasts expirés et les afficher
        // Voir 00-contenu-complet.md pour l'implémentation complète
    }
}
```

## Usage

```rust
let mut toasts = ToastManager::new();

// Afficher un toast
toasts.success("Document sauvegardé");
toasts.error("Erreur de connexion");
toasts.show("Information", ToastLevel::Info);

// Rendre dans l'UI
toasts.render(ctx, &tokens);
```

## Features

- **Auto-dismiss** : Disparaît après 5 secondes
- **Niveaux** : Info, Success, Warning, Error
- **Empilement** : Plusieurs toasts simultanés
- **Position** : Coin inférieur droit
