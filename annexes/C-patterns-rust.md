# Annexe C : Patterns Rust Essentiels

Résumé des patterns Rust utilisés dans ce livre.

---

## 1. Builder Pattern

Constructeur fluide pour objets complexes.

```rust
pub struct Config {
    host: String,
    port: u16,
    timeout: Duration,
}

pub struct ConfigBuilder {
    host: String,
    port: u16,
    timeout: Duration,
}

impl ConfigBuilder {
    pub fn new() -> Self {
        Self {
            host: "localhost".to_string(),
            port: 8080,
            timeout: Duration::from_secs(30),
        }
    }
    
    pub fn host(mut self, host: impl Into<String>) -> Self {
        self.host = host.into();
        self
    }
    
    pub fn port(mut self, port: u16) -> Self {
        self.port = port;
        self
    }
    
    pub fn timeout(mut self, timeout: Duration) -> Self {
        self.timeout = timeout;
        self
    }
    
    pub fn build(self) -> Config {
        Config {
            host: self.host,
            port: self.port,
            timeout: self.timeout,
        }
    }
}

// Usage
let config = ConfigBuilder::new()
    .host("example.com")
    .port(443)
    .build();
```

---

## 2. Newtype Pattern

Wrapper typé pour plus de sécurité.

```rust
// Évite de confondre différents IDs
pub struct CustomerId(pub i64);
pub struct InvoiceId(pub i64);

// Implémentation des traits nécessaires
impl CustomerId {
    pub fn new(id: i64) -> Self {
        Self(id)
    }
    
    pub fn value(&self) -> i64 {
        self.0
    }
}

// Usage
fn get_customer(id: CustomerId) -> Customer { /* ... */ }
fn get_invoice(id: InvoiceId) -> Invoice { /* ... */ }

// Erreur de compilation si on passe le mauvais type !
// get_customer(InvoiceId(1)); // Erreur!
```

---

## 3. Type State Pattern

Valider les états au compile-time.

```rust
// États (types vides)
pub struct Draft;
pub struct Sent;
pub struct Paid;

pub struct Invoice<State> {
    id: InvoiceId,
    items: Vec<InvoiceItem>,
    _state: std::marker::PhantomData<State>,
}

impl Invoice<Draft> {
    pub fn new() -> Self {
        Self {
            id: InvoiceId::new(),
            items: vec![],
            _state: std::marker::PhantomData,
        }
    }
    
    pub fn add_item(&mut self, item: InvoiceItem) {
        self.items.push(item);
    }
    
    // Transition d'état
    pub fn send(self) -> Invoice<Sent> {
        Invoice {
            id: self.id,
            items: self.items,
            _state: std::marker::PhantomData,
        }
    }
}

impl Invoice<Sent> {
    pub fn mark_paid(self) -> Invoice<Paid> {
        Invoice {
            id: self.id,
            items: self.items,
            _state: std::marker::PhantomData,
        }
    }
}

// Usage
let mut invoice = Invoice::<Draft>::new();
invoice.add_item(item);
let sent = invoice.send();
// sent.add_item(item); // Erreur! Pas possible sur Sent
let paid = sent.mark_paid();
```

---

## 4. Error Handling avec thiserror

```rust
use thiserror::Error;

#[derive(Error, Debug)]
pub enum AppError {
    #[error("Base de données: {0}")]
    Database(#[from] rusqlite::Error),
    
    #[error("Fichier non trouvé: {path}")]
    FileNotFound { path: String },
    
    #[error("Validation: {0}")]
    Validation(String),
    
    #[error("Non autorisé")]
    Unauthorized,
}

// Usage
fn load_data() -> Result<Data, AppError> {
    let conn = Connection::open("app.db")?;  // Auto-convert
    
    if !path.exists() {
        return Err(AppError::FileNotFound { 
            path: path.to_string() 
        });
    }
    
    Ok(data)
}
```

---

## 5. Extension Trait

Ajouter des méthodes à des types existants.

```rust
pub trait StringExt {
    fn truncate_with_ellipsis(&self, max_len: usize) -> String;
}

impl StringExt for str {
    fn truncate_with_ellipsis(&self, max_len: usize) -> String {
        if self.len() <= max_len {
            self.to_string()
        } else {
            format!("{}...", &self[..max_len - 3])
        }
    }
}

// Usage
let text = "Très long texte qui doit être tronqué";
println!("{}", text.truncate_with_ellipsis(20));  // "Très long texte q..."
```

---

## 6. Repository Pattern

Abstraction de la persistance.

```rust
pub trait Repository<T, Id> {
    fn find_by_id(&self, id: Id) -> Result<Option<T>>;
    fn find_all(&self) -> Result<Vec<T>>;
    fn save(&self, entity: &T) -> Result<Id>;
    fn delete(&self, id: Id) -> Result<bool>;
}

// Implémentation SQLite
pub struct SqliteCustomerRepository {
    conn: Connection,
}

impl Repository<Customer, CustomerId> for SqliteCustomerRepository {
    fn find_by_id(&self, id: CustomerId) -> Result<Option<Customer>> {
        // Implémentation SQL
    }
    
    fn save(&self, customer: &Customer) -> Result<CustomerId> {
        // Implémentation SQL
    }
    
    // ...
}

// Implémentation In-Memory (pour tests)
pub struct InMemoryCustomerRepository {
    customers: RefCell<HashMap<CustomerId, Customer>>,
}
```

---

## 7. Command Pattern

Encapsuler les actions.

```rust
pub enum Command {
    CreateCustomer { name: String, email: String },
    UpdateCustomer { id: CustomerId, name: String },
    DeleteCustomer { id: CustomerId },
    CreateInvoice { customer_id: CustomerId },
}

pub struct CommandHandler {
    customer_repo: Arc<dyn Repository<Customer, CustomerId>>,
}

impl CommandHandler {
    pub fn handle(&self, command: Command) -> Result<()> {
        match command {
            Command::CreateCustomer { name, email } => {
                let customer = Customer::new(name, email);
                self.customer_repo.save(&customer)?;
            }
            Command::DeleteCustomer { id } => {
                self.customer_repo.delete(id)?;
            }
            // ...
        }
        Ok(())
    }
}
```

---

## 8. Observer Pattern avec Channels

```rust
use crossbeam_channel::{unbounded, Sender, Receiver};

pub enum Event {
    CustomerCreated(CustomerId),
    InvoicePaid(InvoiceId),
}

pub struct EventBus {
    sender: Sender<Event>,
    receiver: Receiver<Event>,
}

impl EventBus {
    pub fn new() -> Self {
        let (sender, receiver) = unbounded();
        Self { sender, receiver }
    }
    
    pub fn publish(&self, event: Event) {
        self.sender.send(event).ok();
    }
    
    pub fn subscribe(&self) -> Receiver<Event> {
        self.receiver.clone()
    }
}
```

---

## 9. RAII Pattern

Nettoyage automatique.

```rust
pub struct TempFile {
    path: PathBuf,
}

impl TempFile {
    pub fn new(name: &str) -> std::io::Result<Self> {
        let path = std::env::temp_dir().join(name);
        std::fs::File::create(&path)?;
        Ok(Self { path })
    }
    
    pub fn path(&self) -> &Path {
        &self.path
    }
}

impl Drop for TempFile {
    fn drop(&mut self) {
        // Nettoyage automatique
        std::fs::remove_file(&self.path).ok();
    }
}

// Usage
{
    let temp = TempFile::new("data.tmp")?;
    // Utiliser le fichier...
}  // Fichier automatiquement supprimé ici
```

---

## 10. Interior Mutability

Mutation derrière une référence partagée.

```rust
use std::cell::RefCell;
use std::rc::Rc;

// Single-threaded
pub struct Cache {
    data: RefCell<HashMap<String, String>>,
}

impl Cache {
    pub fn get(&self, key: &str) -> Option<String> {
        self.data.borrow().get(key).cloned()
    }
    
    pub fn set(&self, key: String, value: String) {
        self.data.borrow_mut().insert(key, value);
    }
}

// Multi-threaded avec Arc + RwLock
use std::sync::{Arc, RwLock};

pub struct ThreadSafeCache {
    data: Arc<RwLock<HashMap<String, String>>>,
}

impl ThreadSafeCache {
    pub fn get(&self, key: &str) -> Option<String> {
        self.data.read().unwrap().get(key).cloned()
    }
    
    pub fn set(&self, key: String, value: String) {
        self.data.write().unwrap().insert(key, value);
    }
}
```

---

## Résumé

| Pattern | Utilisation |
|---------|-------------|
| Builder | Objets complexes avec config fluide |
| Newtype | Typage fort, éviter les confusions |
| Type State | Validations compile-time |
| Extension Trait | Étendre des types existants |
| Repository | Abstraction de la persistance |
| Command | Encapsuler les actions |
| Observer | Communication découplée |
| RAII | Nettoyage automatique |
| Interior Mutability | Mutation thread-safe |
