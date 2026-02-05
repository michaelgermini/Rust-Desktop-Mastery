# 6.2 Channels (crossbeam)

## Pourquoi crossbeam ?

Les channels de la std sont limités. Crossbeam offre :
- **Channels multi-producteur multi-consommateur (MPMC)** : Plusieurs threads peuvent envoyer et recevoir
- **Meilleure performance** : Optimisé pour les cas d'usage courants
- **Select sur plusieurs channels** : Attendre sur plusieurs sources
- **Channels bornés et non-bornés** : Contrôle de la backpressure

```toml
# Cargo.toml
[dependencies]
crossbeam-channel = "0.5"
```

## Types de channels

```rust
use crossbeam_channel::{bounded, unbounded, Receiver, Sender};

// Channel borné : bloque si plein (backpressure)
let (tx, rx): (Sender<Message>, Receiver<Message>) = bounded(100);

// Channel non-borné : ne bloque jamais (attention à la mémoire)
let (tx, rx) = unbounded();

// Envoyer
tx.send(Message::Data(vec![1, 2, 3]))?;

// Recevoir (bloquant)
let msg = rx.recv()?;

// Recevoir (non-bloquant) - pour le thread UI
match rx.try_recv() {
    Ok(msg) => handle_message(msg),
    Err(crossbeam_channel::TryRecvError::Empty) => { /* rien à faire */ }
    Err(crossbeam_channel::TryRecvError::Disconnected) => { /* channel fermé */ }
}
```

## Select : attendre sur plusieurs channels

```rust
use crossbeam_channel::{select, bounded, unbounded};

let (tx_ui, rx_ui) = bounded(10);      // Commandes UI
let (tx_net, rx_net) = unbounded();    // Réponses réseau
let (tx_timer, rx_timer) = bounded(1); // Timer

// Dans le worker
loop {
    select! {
        recv(rx_ui) -> msg => {
            match msg {
                Ok(UiCommand::Quit) => break,
                Ok(UiCommand::Refresh) => refresh_data(),
                Err(_) => break,  // Channel fermé
            }
        }
        recv(rx_net) -> msg => {
            if let Ok(response) = msg {
                process_network_response(response);
            }
        }
        recv(rx_timer) -> _ => {
            // Tick périodique
            check_for_updates();
        }
        default(Duration::from_millis(100)) => {
            // Timeout : rien reçu
            idle_work();
        }
    }
}
```

## Pattern : Worker avec plusieurs sources

```rust
pub struct Worker {
    ui_rx: Receiver<UiCommand>,
    db_rx: Receiver<DbResult>,
    timer_rx: Receiver<()>,
}

impl Worker {
    pub fn run(&self) {
        loop {
            select! {
                recv(self.ui_rx) -> cmd => {
                    if let Ok(cmd) = cmd {
                        self.handle_ui_command(cmd);
                    } else {
                        break;  // Channel fermé
                    }
                }
                recv(self.db_rx) -> result => {
                    if let Ok(result) = result {
                        self.handle_db_result(result);
                    }
                }
                recv(self.timer_rx) -> _ => {
                    self.periodic_task();
                }
            }
        }
    }
}
```

## Résumé

- **bounded** : Contrôle la backpressure, limite la mémoire
- **unbounded** : Plus rapide mais peut consommer beaucoup de mémoire
- **select!** : Attendre sur plusieurs channels simultanément
- **try_recv** : Non-bloquant, idéal pour le thread UI
