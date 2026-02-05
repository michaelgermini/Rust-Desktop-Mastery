# 4.3 Erreurs Idiomatiques (Result, anyhow, thiserror)

## Philosophie des erreurs en Rust

Rust distingue deux types d'erreurs :

```rust
/// Erreurs récupérables : Result<T, E>
/// - Fichier non trouvé → demander un autre fichier
/// - Connexion échouée → réessayer
/// - Validation échouée → afficher un message

/// Erreurs irrécupérables : panic!
/// - Index hors limites sur un bug
/// - État impossible atteint
/// - Invariant violé
```

## Result<T, E> : le fondement

```rust
/// Result a deux variantes
enum Result<T, E> {
    Ok(T),   // Succès avec une valeur
    Err(E),  // Échec avec une erreur
}

fn divide(a: f64, b: f64) -> Result<f64, String> {
    if b == 0.0 {
        Err("Division par zéro".to_string())
    } else {
        Ok(a / b)
    }
}

fn use_result() {
    // Pattern matching complet
    match divide(10.0, 2.0) {
        Ok(result) => println!("Résultat : {}", result),
        Err(e) => println!("Erreur : {}", e),
    }
    
    // Avec if let
    if let Ok(result) = divide(10.0, 0.0) {
        println!("Résultat : {}", result);
    } else {
        println!("Échec de la division");
    }
}
```

## L'opérateur ? : propagation élégante

```rust
use std::fs;
use std::io;

fn read_username_from_file() -> Result<String, io::Error> {
    // Sans ? : verbose
    // let file_result = fs::read_to_string("username.txt");
    // let username = match file_result {
    //     Ok(content) => content,
    //     Err(e) => return Err(e),
    // };
    
    // Avec ? : élégant
    let username = fs::read_to_string("username.txt")?;
    Ok(username.trim().to_string())
}

/// ? fonctionne aussi dans main
fn main() -> Result<(), Box<dyn std::error::Error>> {
    let content = fs::read_to_string("config.toml")?;
    println!("Config: {}", content);
    Ok(())
}
```

## Option<T> : absence de valeur

```rust
/// Option représente une valeur optionnelle
enum Option<T> {
    Some(T),  // Une valeur existe
    None,     // Pas de valeur
}

fn find_user(id: u32) -> Option<String> {
    if id == 1 {
        Some("Alice".to_string())
    } else {
        None
    }
}

fn use_option() {
    // Pattern matching
    match find_user(1) {
        Some(name) => println!("Trouvé : {}", name),
        None => println!("Utilisateur non trouvé"),
    }
    
    // Méthodes utiles
    let name = find_user(1).unwrap_or("Inconnu".to_string());
    let name = find_user(1).unwrap_or_default();
    
    // ? avec Option (retourne None si None)
    fn get_first_char(s: Option<String>) -> Option<char> {
        s?.chars().next()
    }
}
```

## thiserror : erreurs typées pour bibliothèques

```rust
use thiserror::Error;

/// Définir des erreurs typées avec thiserror
#[derive(Error, Debug)]
pub enum DatabaseError {
    #[error("Connexion échouée : {0}")]
    ConnectionFailed(String),
    
    #[error("Requête invalide : {0}")]
    InvalidQuery(String),
    
    #[error("Enregistrement non trouvé : id={id}")]
    NotFound { id: u64 },
    
    #[error("Erreur IO")]
    Io(#[from] std::io::Error),  // Conversion automatique
    
    #[error("Erreur SQL")]
    Sql(#[from] rusqlite::Error),
}

/// Utilisation
fn get_user(id: u64) -> Result<User, DatabaseError> {
    if id == 0 {
        return Err(DatabaseError::NotFound { id });
    }
    
    // ? convertit automatiquement rusqlite::Error en DatabaseError
    let conn = Connection::open("users.db")?;
    // ...
    Ok(User { id, name: "Alice".to_string() })
}
```

## anyhow : erreurs simples pour applications

```rust
use anyhow::{anyhow, bail, Context, Result};

/// anyhow::Result<T> = Result<T, anyhow::Error>
fn load_config() -> Result<Config> {
    // context() ajoute du contexte à l'erreur
    let content = std::fs::read_to_string("config.toml")
        .context("Impossible de lire le fichier de configuration")?;
    
    let config: Config = toml::from_str(&content)
        .context("Format de configuration invalide")?;
    
    // bail! pour erreur rapide
    if config.port == 0 {
        bail!("Le port ne peut pas être 0");
    }
    
    // anyhow! pour créer une erreur ad-hoc
    if config.name.is_empty() {
        return Err(anyhow!("Le nom ne peut pas être vide"));
    }
    
    Ok(config)
}

fn main() {
    match load_config() {
        Ok(config) => println!("Config chargée : {:?}", config),
        Err(e) => {
            // Affiche l'erreur avec tout le contexte
            eprintln!("Erreur : {:#}", e);
            // Erreur : Impossible de lire le fichier de configuration
            // Caused by: No such file or directory
        }
    }
}
```

## Quand utiliser quoi ?

```rust
/// thiserror vs anyhow
pub mod error_strategy {
    // THISERROR : pour les bibliothèques
    // - Les utilisateurs doivent pouvoir matcher sur vos erreurs
    // - Erreurs structurées et documentées
    // - Conversions From automatiques
    
    // ANYHOW : pour les applications
    // - Vous voulez juste propager et afficher les erreurs
    // - Pas besoin de matcher sur les types d'erreur
    // - Contexte facile à ajouter
    
    // COMBINAISON : thiserror dans vos modules core,
    // anyhow dans main() et les handlers
}
```

## Patterns courants

### Conversion d'erreurs

```rust
use thiserror::Error;

#[derive(Error, Debug)]
pub enum AppError {
    #[error("Base de données : {0}")]
    Database(#[from] DatabaseError),
    
    #[error("Configuration : {0}")]
    Config(#[from] ConfigError),
    
    #[error("Réseau : {0}")]
    Network(#[from] reqwest::Error),
}

// Maintenant ? convertit automatiquement ces erreurs en AppError
fn init_app() -> Result<App, AppError> {
    let config = load_config()?;  // ConfigError → AppError
    let db = connect_db()?;       // DatabaseError → AppError
    Ok(App { config, db })
}
```

### Erreurs optionnelles

```rust
fn find_and_process(id: u64) -> Result<Option<String>> {
    let user = match find_user(id)? {  // Erreur si DB échoue
        Some(u) => u,
        None => return Ok(None),  // Pas d'erreur, juste pas trouvé
    };
    
    Ok(Some(process(user)?))
}
```

### Custom error avec données

```rust
#[derive(Error, Debug)]
#[error("Validation échouée")]
pub struct ValidationError {
    pub field: String,
    pub message: String,
    pub value: String,
}

fn validate_email(email: &str) -> Result<(), ValidationError> {
    if !email.contains('@') {
        return Err(ValidationError {
            field: "email".to_string(),
            message: "Format d'email invalide".to_string(),
            value: email.to_string(),
        });
    }
    Ok(())
}
```

## Résumé

```rust
/// Aide-mémoire gestion d'erreurs
pub mod error_cheatsheet {
    // Result<T, E> : opération qui peut échouer
    // Option<T> : valeur optionnelle
    // ? : propager l'erreur vers l'appelant
    // thiserror : erreurs structurées pour libs
    // anyhow : erreurs simples pour apps
    // .context() : ajouter du contexte
    // bail!() : retourner une erreur rapidement
    
    // NE JAMAIS :
    // - .unwrap() en production (sauf si vraiment impossible)
    // - panic! pour des erreurs récupérables
    // - Ignorer les Result avec let _ = ...
}
```
