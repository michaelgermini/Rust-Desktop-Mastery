# 12.4 Undo et Redo

## Système d'historique

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
```

## Intégration avec les raccourcis clavier

```rust
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

## Usage

```rust
let mut undo_stack = UndoStack::new(initial_state, 100);

// Après chaque modification
undo_stack.push(new_state);

// Undo
if let Some(previous_state) = undo_stack.undo() {
    app_state = previous_state.clone();
}

// Redo
if let Some(next_state) = undo_stack.redo() {
    app_state = next_state.clone();
}
```

## Résumé

- **Historique** : Stack d'états précédents
- **Limite** : Taille maximale pour éviter la mémoire excessive
- **Raccourcis** : Cmd+Z (undo), Cmd+Shift+Z (redo)
- **État actuel** : Toujours accessible
