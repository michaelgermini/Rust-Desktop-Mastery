# 6.4 Thread UI vs Workers

## Le problème du thread UI

### Pourquoi l'interface freeze

```rust
// ❌ MAUVAIS : Tout sur le thread UI
fn handle_export_button(ui: &mut Ui, data: &AppData) {
    if ui.button("Exporter").clicked() {
        // Cette opération bloque le thread UI pendant 5 secondes
        let report = generate_large_report(data);  // 3 secondes
        save_to_file(&report);                      // 2 secondes
        
        // Pendant ce temps : l'interface est gelée
        // L'utilisateur ne peut pas annuler
        // L'OS peut afficher "Application ne répond pas"
    }
}
```

### La règle d'or

```
Règle : Le thread UI ne doit JAMAIS bloquer plus de 16ms
        (1 frame à 60 FPS)

Opérations > 16ms :
- Requêtes base de données
- Lecture/écriture fichiers
- Calculs lourds
- Requêtes réseau

Ces opérations doivent être déplacées sur des threads workers.
```

## Architecture thread UI / Workers

```
┌─────────────────────────────────────────────────────┐
│                    Thread UI                         │
│  - Rendu de l'interface (60 FPS)                    │
│  - Gestion des événements (clics, clavier)          │
│  - Mise à jour de l'affichage                       │
│  - Envoi de commandes aux workers                   │
│  - Réception des résultats                          │
└─────────────────┬───────────────────────────────────┘
                  │ Channels (non-bloquants)
┌─────────────────▼───────────────────────────────────┐
│                 Workers (Pool de threads)            │
│  - Calculs lourds                                   │
│  - I/O fichiers                                     │
│  - Requêtes base de données                         │
│  - Requêtes réseau                                  │
└─────────────────────────────────────────────────────┘
```

## Pattern : Worker pool

```rust
use std::sync::mpsc;
use std::thread;

pub struct WorkerPool {
    sender: mpsc::Sender<Job>,
}

type Job = Box<dyn FnOnce() + Send + 'static>;

impl WorkerPool {
    pub fn new(num_workers: usize) -> Self {
        let (sender, receiver) = mpsc::channel();
        let receiver = Arc::new(Mutex::new(receiver));
        
        for _ in 0..num_workers {
            let receiver = Arc::clone(&receiver);
            thread::spawn(move || {
                loop {
                    let job = receiver.lock().unwrap().recv();
                    match job {
                        Ok(job) => job(),
                        Err(_) => break,
                    }
                }
            });
        }
        
        Self { sender }
    }
    
    pub fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + 'static,
    {
        self.sender.send(Box::new(f)).unwrap();
    }
}
```

## Pattern : UI réactive avec workers

```rust
pub struct App {
    worker_pool: WorkerPool,
    pending_tasks: Vec<TaskId>,
    results: HashMap<TaskId, TaskResult>,
}

impl App {
    pub fn handle_heavy_operation(&mut self, data: &AppData) {
        let task_id = TaskId::new();
        self.pending_tasks.push(task_id);
        
        let data_clone = data.clone();
        let task_id_clone = task_id;
        
        // Déléguer au worker
        self.worker_pool.execute(move || {
            let result = heavy_computation(&data_clone);
            
            // Envoyer le résultat via channel
            send_result(task_id_clone, result);
        });
        
        // Le thread UI continue immédiatement
        // L'utilisateur peut toujours interagir
    }
    
    pub fn poll_results(&mut self) {
        // Vérifier les résultats disponibles (non-bloquant)
        while let Ok((task_id, result)) = try_receive_result() {
            self.pending_tasks.retain(|&id| id != task_id);
            self.results.insert(task_id, result);
        }
    }
}
```

## Résumé

- **Thread UI** : Rendu et événements uniquement, < 16ms par frame
- **Workers** : Toutes les opérations longues
- **Communication** : Channels non-bloquants
- **Feedback** : Progression et résultats via events
