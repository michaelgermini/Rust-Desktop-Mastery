# 15.3 Messages

## Types de messages

### Command (Commande)

Une commande représente une intention de modifier l'état.

```rust
#[derive(Clone)]
pub enum Command {
    CreateInvoice { customer_id: CustomerId, items: Vec<InvoiceItem> },
    UpdateInvoice { id: InvoiceId, changes: InvoiceChanges },
    DeleteInvoice { id: InvoiceId },
    SendInvoice { id: InvoiceId },
}
```

**Caractéristiques** :
- Modifie l'état
- Retourne un résultat (succès/erreur)
- Ne doit être exécutée qu'une fois

### Event (Événement)

Un événement représente quelque chose qui s'est passé.

```rust
#[derive(Clone)]
pub enum Event {
    InvoiceCreated { invoice_id: InvoiceId },
    InvoiceUpdated { invoice_id: InvoiceId },
    InvoiceDeleted { invoice_id: InvoiceId },
    InvoiceSent { invoice_id: InvoiceId },
}
```

**Caractéristiques** :
- Décrit un fait passé
- Peut avoir plusieurs handlers
- Idempotent (peut être rejoué)

### Query (Requête)

Une query représente une demande de lecture.

```rust
#[derive(Clone)]
pub enum Query {
    GetInvoice { id: InvoiceId },
    ListInvoices { filter: InvoiceFilter },
    GetInvoiceStats { period: Period },
}
```

**Caractéristiques** :
- Ne modifie pas l'état
- Retourne des données
- Peut être mise en cache

## Sérialisation

```rust
use serde::{Deserialize, Serialize};

#[derive(Clone, Serialize, Deserialize)]
pub enum Command {
    CreateInvoice {
        customer_id: CustomerId,
        items: Vec<InvoiceItem>,
    },
    // ...
}

// Sérialiser pour la persistance ou le réseau
let json = serde_json::to_string(&command)?;
let command: Command = serde_json::from_str(&json)?;
```

## Versioning

```rust
#[derive(Clone, Serialize, Deserialize)]
#[serde(tag = "version")]
pub enum Command {
    #[serde(rename = "v1")]
    V1(CommandV1),
    #[serde(rename = "v2")]
    V2(CommandV2),
}

#[derive(Serialize, Deserialize)]
pub struct CommandV1 {
    pub customer_id: i64,
    // ...
}

#[derive(Serialize, Deserialize)]
pub struct CommandV2 {
    pub customer_id: CustomerId,  // Type plus fort
    pub metadata: Option<HashMap<String, String>>,  // Nouveau champ
    // ...
}

// Migration automatique
impl From<CommandV1> for CommandV2 {
    fn from(v1: CommandV1) -> Self {
        Self {
            customer_id: CustomerId(v1.customer_id),
            metadata: None,
        }
    }
}
```

## Résumé

- **Command** : Intention de modifier (une fois)
- **Event** : Fait passé (plusieurs handlers)
- **Query** : Demande de lecture (cacheable)
- **Sérialisation** : Pour persistance/réseau
- **Versioning** : Évolutivité des messages
