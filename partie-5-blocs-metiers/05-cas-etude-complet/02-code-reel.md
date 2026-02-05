# 24.2 Code Réel

## Implémentation complète

### Point d'entrée

```rust
// src/main.rs
use notevault_ui::App;
use notevault_infrastructure::Database;
use std::sync::Arc;

fn main() -> eframe::Result<()> {
    // Configuration
    let data_dir = dirs::data_dir().unwrap().join("notevault");
    std::fs::create_dir_all(&data_dir).ok();
    
    // Base de données
    let db = Arc::new(
        Database::open(&data_dir.join("notes.db"))
            .expect("Failed to open database")
    );
    db.run_migrations().expect("Failed to run migrations");
    
    // Moteur de recherche
    let search = Arc::new(
        SearchEngine::new(&data_dir.join("search_index"))
            .expect("Failed to create search index")
    );
    
    // Application
    let options = eframe::NativeOptions {
        viewport: egui::ViewportBuilder::default()
            .with_inner_size([1200.0, 800.0])
            .with_min_inner_size([800.0, 600.0])
            .with_title("NoteVault"),
        ..Default::default()
    };
    
    eframe::run_native(
        "NoteVault",
        options,
        Box::new(|cc| {
            setup_fonts(&cc.egui_ctx);
            Ok(Box::new(App::new(db, search)))
        }),
    )
}
```

## Patterns utilisés

### Clean Architecture

```rust
// Core définit les entités
pub struct Note {
    pub id: NoteId,
    pub title: String,
    pub content: String,
    // ...
}

// Application définit les use cases
pub struct CreateNoteHandler {
    repository: Arc<dyn NoteRepository>,
}

// Infrastructure implémente
impl NoteRepository for SqliteNoteRepository {
    // ...
}
```

### Event Bus

```rust
pub struct App {
    event_bus: EventBus,
    // ...
}

impl App {
    fn handle_event(&mut self, event: Event) {
        match event {
            Event::NoteCreated { note } => {
                // Mettre à jour l'UI
            }
            // ...
        }
    }
}
```

### State Management

```rust
pub struct App {
    store: Arc<Store>,
    // ...
}

impl App {
    fn update_note(&self, note: Note) {
        self.store.update(|state| {
            state.notes.push(note);
        });
    }
}
```

## Bonnes pratiques

### Gestion d'erreurs

```rust
pub fn save_note(&self, note: &Note) -> Result<NoteId, AppError> {
    self.repository.save(note)
        .map_err(|e| AppError::DatabaseError(e.to_string()))
}
```

### Logging

```rust
use tracing::{info, error, warn};

pub fn create_note(&self, title: String) -> Result<NoteId> {
    info!("Creating note: {}", title);
    
    match self.repository.save(&note) {
        Ok(id) => {
            info!("Note created with id: {}", id.0);
            Ok(id)
        }
        Err(e) => {
            error!("Failed to create note: {}", e);
            Err(e)
        }
    }
}
```

### Tests

```rust
#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_create_note() {
        let repo = MockNoteRepository::new();
        let handler = CreateNoteHandler::new(repo);
        
        let cmd = CreateNoteCommand {
            title: "Test".to_string(),
            content: "Content".to_string(),
        };
        
        let result = handler.handle(cmd);
        assert!(result.is_ok());
    }
}
```

## Résumé

- **Architecture complète** : Tous les patterns du livre appliqués
- **Code réel** : Implémentation fonctionnelle
- **Bonnes pratiques** : Erreurs, logging, tests
- **Maintenable** : Code clair et organisé
