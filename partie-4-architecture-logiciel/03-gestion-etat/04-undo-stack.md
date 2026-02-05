# 16.4 Undo Stack

## Implémentation d'un undo stack

```rust
/// Undo Stack pour l'historique des modifications
pub struct UndoStack<T: Clone> {
    history: Vec<T>,
    position: usize,
    max_size: usize,
}

impl<T: Clone> UndoStack<T> {
    pub fn new(initial_state: T, max_size: usize) -> Self {
        Self {
            history: vec![initial_state],
            position: 0,
            max_size,
        }
    }
    
    /// Enregistrer un nouvel état
    pub fn push(&mut self, state: T) {
        // Supprimer l'historique futur si on n'est pas à la fin
        self.history.truncate(self.position + 1);
        
        // Ajouter le nouvel état
        self.history.push(state);
        self.position = self.history.len() - 1;
        
        // Limiter la taille
        if self.history.len() > self.max_size {
            self.history.remove(0);
            self.position -= 1;
        }
    }
    
    /// Annuler (undo)
    pub fn undo(&mut self) -> Option<&T> {
        if self.position > 0 {
            self.position -= 1;
            Some(&self.history[self.position])
        } else {
            None
        }
    }
    
    /// Refaire (redo)
    pub fn redo(&mut self) -> Option<&T> {
        if self.position < self.history.len() - 1 {
            self.position += 1;
            Some(&self.history[self.position])
        } else {
            None
        }
    }
    
    pub fn can_undo(&self) -> bool {
        self.position > 0
    }
    
    pub fn can_redo(&self) -> bool {
        self.position < self.history.len() - 1
    }
    
    pub fn current(&self) -> &T {
        &self.history[self.position]
    }
}
```

## Mementos et commandes

### Pattern Command

```rust
pub trait Command {
    fn execute(&mut self) -> Result<()>;
    fn undo(&mut self) -> Result<()>;
}

pub struct UndoManager {
    undo_stack: Vec<Box<dyn Command>>,
    redo_stack: Vec<Box<dyn Command>>,
}

impl UndoManager {
    pub fn execute(&mut self, mut cmd: Box<dyn Command>) -> Result<()> {
        cmd.execute()?;
        self.undo_stack.push(cmd);
        self.redo_stack.clear();
        Ok(())
    }
    
    pub fn undo(&mut self) -> Result<()> {
        if let Some(mut cmd) = self.undo_stack.pop() {
            cmd.undo()?;
            self.redo_stack.push(cmd);
        }
        Ok(())
    }
    
    pub fn redo(&mut self) -> Result<()> {
        if let Some(mut cmd) = self.redo_stack.pop() {
            cmd.execute()?;
            self.undo_stack.push(cmd);
        }
        Ok(())
    }
}
```

## Limites et optimisations

### Limite de mémoire

```rust
impl<T: Clone> UndoStack<T> {
    pub fn set_max_size(&mut self, max_size: usize) {
        self.max_size = max_size;
        
        // Nettoyer si nécessaire
        if self.history.len() > max_size {
            let to_remove = self.history.len() - max_size;
            self.history.drain(0..to_remove);
            self.position = self.position.saturating_sub(to_remove);
        }
    }
}
```

### Compression

Pour les états volumineux, stocker seulement les différences :

```rust
pub struct DiffUndoStack {
    base_state: AppState,
    diffs: Vec<StateDiff>,
    position: usize,
}

pub enum StateDiff {
    InvoiceAdded { id: InvoiceId },
    InvoiceModified { id: InvoiceId, changes: InvoiceChanges },
    InvoiceDeleted { id: InvoiceId, invoice: Invoice },
}
```

## Résumé

- **Undo Stack** : Historique des états précédents
- **Command Pattern** : Commandes exécutables/annulables
- **Limite mémoire** : Taille maximale configurable
- **Compression** : Stocker seulement les différences pour gros états
