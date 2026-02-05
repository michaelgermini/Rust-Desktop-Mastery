# 15.2 Pub/Sub Pattern

## Pattern publish/subscribe

Le pattern pub/sub permet un découplage total : les émetteurs ne connaissent pas les récepteurs.

```rust
use std::collections::HashMap;
use std::any::{Any, TypeId};
use std::sync::{Arc, Mutex};

pub trait Message: Clone + Send + 'static {}

pub struct PubSub {
    subscribers: Arc<Mutex<HashMap<TypeId, Vec<Box<dyn Fn(&dyn Any) + Send + Sync>>>>>,
}

impl PubSub {
    pub fn new() -> Self {
        Self { 
            subscribers: Arc::new(Mutex::new(HashMap::new()))
        }
    }
    
    pub fn subscribe<M: Message>(&self, handler: impl Fn(&M) + Send + Sync + 'static) {
        let type_id = TypeId::of::<M>();
        let boxed_handler = Box::new(move |msg: &dyn Any| {
            if let Some(typed_msg) = msg.downcast_ref::<M>() {
                handler(typed_msg);
            }
        });
        
        let mut subs = self.subscribers.lock().unwrap();
        subs.entry(type_id).or_default().push(boxed_handler);
    }
    
    pub fn publish<M: Message>(&self, message: M) {
        let subs = self.subscribers.lock().unwrap();
        if let Some(handlers) = subs.get(&TypeId::of::<M>()) {
            for handler in handlers {
                handler(&message);
            }
        }
    }
}
```

## Types de messages

```rust
#[derive(Clone)]
pub struct InvoiceCreated {
    pub invoice_id: InvoiceId,
    pub customer_id: CustomerId,
}

#[derive(Clone)]
pub struct InvoicePaid {
    pub invoice_id: InvoiceId,
    pub amount: Decimal,
}

#[derive(Clone)]
pub struct CustomerUpdated {
    pub customer_id: CustomerId,
}

// Implémenter le trait Message
impl Message for InvoiceCreated {}
impl Message for InvoicePaid {}
impl Message for CustomerUpdated {}
```

## Subscriptions multiples

```rust
let pubsub = PubSub::new();

// Plusieurs handlers pour le même type de message
pubsub.subscribe(|event: &InvoiceCreated| {
    println!("Invoice {} created", event.invoice_id.0);
});

pubsub.subscribe(|event: &InvoiceCreated| {
    // Envoyer une notification
    send_notification(&format!("New invoice: {}", event.invoice_id.0));
});

pubsub.subscribe(|event: &InvoiceCreated| {
    // Mettre à jour les statistiques
    update_stats();
});

// Publier un événement
pubsub.publish(InvoiceCreated {
    invoice_id: InvoiceId(123),
    customer_id: CustomerId(456),
});
// Les 3 handlers sont appelés
```

## Filtrage des événements

```rust
pub struct FilteredPubSub {
    pubsub: PubSub,
    filters: Arc<Mutex<HashMap<TypeId, Vec<Box<dyn Fn(&dyn Any) -> bool + Send + Sync>>>>>,
}

impl FilteredPubSub {
    pub fn subscribe_with_filter<M: Message>(
        &self,
        filter: impl Fn(&M) -> bool + Send + Sync + 'static,
        handler: impl Fn(&M) + Send + Sync + 'static,
    ) {
        let type_id = TypeId::of::<M>();
        
        // Enregistrer le filtre
        let mut filters = self.filters.lock().unwrap();
        filters.entry(type_id).or_default().push(Box::new(move |msg: &dyn Any| {
            if let Some(typed_msg) = msg.downcast_ref::<M>() {
                filter(typed_msg)
            } else {
                false
            }
        }));
        
        // Enregistrer le handler avec vérification du filtre
        self.pubsub.subscribe(move |msg: &M| {
            let filters = self.filters.lock().unwrap();
            if let Some(filter_list) = filters.get(&type_id) {
                for f in filter_list {
                    if f(msg) {
                        handler(msg);
                        break;
                    }
                }
            }
        });
    }
}

// Usage
pubsub.subscribe_with_filter(
    |event: &InvoiceCreated| event.customer_id == my_customer_id,
    |event: &InvoiceCreated| {
        // Seulement pour mon customer
        println!("My customer's invoice created!");
    }
);
```

## Avantages

- **Découplage** : Émetteurs et récepteurs ne se connaissent pas
- **Scalabilité** : Facile d'ajouter de nouveaux subscribers
- **Flexibilité** : Handlers multiples pour le même événement
- **Testabilité** : Facile de mocker les événements

## Résumé

- **Pub/Sub** : Pattern de communication découplée
- **Subscriptions multiples** : Plusieurs handlers par type
- **Filtrage** : Handlers conditionnels
- **Type-safe** : Utilisation de TypeId pour la sécurité des types
