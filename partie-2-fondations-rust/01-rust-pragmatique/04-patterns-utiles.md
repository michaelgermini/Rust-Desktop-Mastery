# 4.4 Patterns Utiles

## Builder Pattern

Le Builder permet de construire des objets complexes de manière fluide et sûre.

```rust
/// Structure finale
pub struct HttpRequest {
    url: String,
    method: Method,
    headers: HashMap<String, String>,
    body: Option<String>,
    timeout: Duration,
}

/// Builder pour HttpRequest
pub struct HttpRequestBuilder {
    url: String,
    method: Method,
    headers: HashMap<String, String>,
    body: Option<String>,
    timeout: Duration,
}

impl HttpRequestBuilder {
    pub fn new(url: impl Into<String>) -> Self {
        Self {
            url: url.into(),
            method: Method::Get,
            headers: HashMap::new(),
            body: None,
            timeout: Duration::from_secs(30),
        }
    }
    
    pub fn method(mut self, method: Method) -> Self {
        self.method = method;
        self
    }
    
    pub fn header(mut self, key: impl Into<String>, value: impl Into<String>) -> Self {
        self.headers.insert(key.into(), value.into());
        self
    }
    
    pub fn body(mut self, body: impl Into<String>) -> Self {
        self.body = Some(body.into());
        self
    }
    
    pub fn timeout(mut self, timeout: Duration) -> Self {
        self.timeout = timeout;
        self
    }
    
    pub fn build(self) -> HttpRequest {
        HttpRequest {
            url: self.url,
            method: self.method,
            headers: self.headers,
            body: self.body,
            timeout: self.timeout,
        }
    }
}

// Utilisation
fn use_builder() {
    let request = HttpRequestBuilder::new("https://api.example.com")
        .method(Method::Post)
        .header("Content-Type", "application/json")
        .body(r#"{"name": "test"}"#)
        .timeout(Duration::from_secs(10))
        .build();
}
```

## Newtype Pattern

Créer des types distincts pour plus de sécurité.

```rust
/// Sans newtype : confusion possible
fn bad_create_user(name: String, email: String, id: u64) {
    // name et email sont interchangeables par erreur
}

/// Avec newtype : impossible de confondre
pub struct UserId(pub u64);
pub struct UserName(pub String);
pub struct Email(pub String);

impl Email {
    pub fn new(value: impl Into<String>) -> Result<Self, ValidationError> {
        let value = value.into();
        if !value.contains('@') {
            return Err(ValidationError::InvalidEmail);
        }
        Ok(Self(value))
    }
    
    pub fn as_str(&self) -> &str {
        &self.0
    }
}

fn good_create_user(id: UserId, name: UserName, email: Email) {
    // Impossible de passer les arguments dans le mauvais ordre
}

// Utilisation
fn newtype_example() {
    let id = UserId(123);
    let name = UserName("Alice".to_string());
    let email = Email::new("alice@example.com").unwrap();
    
    good_create_user(id, name, email);
    
    // good_create_user(name, email, id);  // ERREUR de compilation !
}
```

## State Machine avec Enums

Modéliser des états et transitions de manière type-safe.

```rust
/// États d'une commande
pub enum OrderState {
    Draft,
    Submitted { submitted_at: DateTime<Utc> },
    Paid { paid_at: DateTime<Utc>, amount: Decimal },
    Shipped { shipped_at: DateTime<Utc>, tracking: String },
    Delivered { delivered_at: DateTime<Utc> },
    Cancelled { reason: String },
}

pub struct Order {
    id: OrderId,
    state: OrderState,
    items: Vec<OrderItem>,
}

impl Order {
    /// Transitions valides uniquement
    pub fn submit(&mut self) -> Result<(), OrderError> {
        match &self.state {
            OrderState::Draft => {
                self.state = OrderState::Submitted {
                    submitted_at: Utc::now(),
                };
                Ok(())
            }
            _ => Err(OrderError::InvalidTransition {
                from: self.state_name(),
                to: "Submitted",
            }),
        }
    }
    
    pub fn pay(&mut self, amount: Decimal) -> Result<(), OrderError> {
        match &self.state {
            OrderState::Submitted { .. } => {
                self.state = OrderState::Paid {
                    paid_at: Utc::now(),
                    amount,
                };
                Ok(())
            }
            _ => Err(OrderError::InvalidTransition {
                from: self.state_name(),
                to: "Paid",
            }),
        }
    }
    
    pub fn ship(&mut self, tracking: String) -> Result<(), OrderError> {
        match &self.state {
            OrderState::Paid { .. } => {
                self.state = OrderState::Shipped {
                    shipped_at: Utc::now(),
                    tracking,
                };
                Ok(())
            }
            _ => Err(OrderError::InvalidTransition {
                from: self.state_name(),
                to: "Shipped",
            }),
        }
    }
    
    fn state_name(&self) -> &'static str {
        match &self.state {
            OrderState::Draft => "Draft",
            OrderState::Submitted { .. } => "Submitted",
            OrderState::Paid { .. } => "Paid",
            OrderState::Shipped { .. } => "Shipped",
            OrderState::Delivered { .. } => "Delivered",
            OrderState::Cancelled { .. } => "Cancelled",
        }
    }
}
```

## Extension Trait

Ajouter des méthodes à des types existants.

```rust
/// Trait d'extension pour String
pub trait StringExt {
    fn truncate_ellipsis(&self, max_len: usize) -> String;
    fn to_slug(&self) -> String;
}

impl StringExt for str {
    fn truncate_ellipsis(&self, max_len: usize) -> String {
        if self.len() <= max_len {
            self.to_string()
        } else if max_len <= 3 {
            "...".to_string()
        } else {
            format!("{}...", &self[..max_len - 3])
        }
    }
    
    fn to_slug(&self) -> String {
        self.to_lowercase()
            .chars()
            .map(|c| if c.is_alphanumeric() { c } else { '-' })
            .collect::<String>()
            .split('-')
            .filter(|s| !s.is_empty())
            .collect::<Vec<_>>()
            .join("-")
    }
}

// Utilisation
fn extension_trait_example() {
    let title = "Hello World! This is a very long title";
    println!("{}", title.truncate_ellipsis(20));  // "Hello World! Thi..."
    println!("{}", title.to_slug());  // "hello-world-this-is-a-very-long-title"
}
```

## Default et derive

```rust
/// Utiliser Default pour les valeurs par défaut
#[derive(Debug, Clone, Default)]
pub struct Settings {
    pub theme: Theme,
    pub font_size: f32,
    pub autosave: bool,
    pub autosave_interval: Duration,
}

#[derive(Debug, Clone, Default)]
pub enum Theme {
    #[default]
    System,
    Light,
    Dark,
}

impl Default for Settings {
    fn default() -> Self {
        Self {
            theme: Theme::default(),
            font_size: 14.0,
            autosave: true,
            autosave_interval: Duration::from_secs(60),
        }
    }
}

// Utilisation avec struct update syntax
fn default_example() {
    let settings = Settings {
        font_size: 16.0,
        ..Default::default()  // Reste des champs par défaut
    };
}
```

## From et Into

```rust
/// Conversions idiomatiques
pub struct UserId(u64);

impl From<u64> for UserId {
    fn from(id: u64) -> Self {
        Self(id)
    }
}

impl From<UserId> for u64 {
    fn from(id: UserId) -> Self {
        id.0
    }
}

// Into est automatiquement implémenté quand From est implémenté
fn from_into_example() {
    let id: UserId = 123u64.into();
    let raw: u64 = id.into();
    
    // Fonctionne aussi avec From directement
    let id = UserId::from(456);
}

/// impl Into<String> pour accepter &str et String
fn accepts_string(s: impl Into<String>) {
    let s: String = s.into();
    println!("{}", s);
}

fn into_string_example() {
    accepts_string("literal");           // &str
    accepts_string(String::from("owned")); // String
}
```

## Iterator et collect

```rust
fn iterator_patterns() {
    let numbers = vec![1, 2, 3, 4, 5];
    
    // Map + collect
    let doubled: Vec<i32> = numbers.iter().map(|n| n * 2).collect();
    
    // Filter + collect
    let evens: Vec<&i32> = numbers.iter().filter(|n| *n % 2 == 0).collect();
    
    // Filter_map (combine filter et map)
    let parsed: Vec<i32> = vec!["1", "two", "3"]
        .iter()
        .filter_map(|s| s.parse().ok())
        .collect();
    
    // Collect into different types
    let set: HashSet<i32> = numbers.iter().cloned().collect();
    let map: HashMap<i32, i32> = numbers.iter().map(|n| (*n, n * 2)).collect();
    
    // fold / reduce
    let sum: i32 = numbers.iter().sum();
    let product: i32 = numbers.iter().product();
    let custom: i32 = numbers.iter().fold(0, |acc, n| acc + n * n);
}
```

## Résumé

| Pattern | Usage |
|---------|-------|
| Builder | Objets complexes avec config fluide |
| Newtype | Types distincts pour éviter les confusions |
| State Machine | États et transitions type-safe |
| Extension Trait | Ajouter des méthodes à des types existants |
| Default | Valeurs par défaut sensées |
| From/Into | Conversions idiomatiques |
| Iterator | Transformations de collections |
