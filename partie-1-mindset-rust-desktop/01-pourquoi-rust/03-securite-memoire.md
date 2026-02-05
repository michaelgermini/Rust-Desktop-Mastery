# 1.3 Sécurité Mémoire

## Le problème des bugs mémoire

Les bugs liés à la mémoire représentent historiquement 70% des vulnérabilités de sécurité dans les logiciels systèmes (Microsoft, Google Chrome, etc.).

```rust
/// Types de bugs mémoire courants en C/C++
pub enum MemoryBug {
    /// Accès après libération
    UseAfterFree,
    
    /// Double libération
    DoubleFree,
    
    /// Dépassement de buffer
    BufferOverflow,
    
    /// Pointeur nul déréférencé
    NullPointerDereference,
    
    /// Fuite mémoire
    MemoryLeak,
    
    /// Data race (accès concurrent non synchronisé)
    DataRace,
}
```

## Comment Rust élimine ces bugs

### Ownership : un seul propriétaire

```rust
fn main() {
    let data = vec![1, 2, 3];  // `data` possède le Vec
    
    let moved = data;          // Ownership transféré à `moved`
    
    // println!("{:?}", data); // ERREUR DE COMPILATION !
    // "value borrowed here after move"
    
    println!("{:?}", moved);   // OK, `moved` est le propriétaire
}
```

### Borrowing : emprunts contrôlés

```rust
fn main() {
    let mut data = vec![1, 2, 3];
    
    // Emprunt immutable (lecture seule)
    let ref1 = &data;
    let ref2 = &data;  // Plusieurs emprunts immutables OK
    println!("{:?} {:?}", ref1, ref2);
    
    // Emprunt mutable (écriture)
    let ref_mut = &mut data;
    ref_mut.push(4);
    
    // let ref3 = &data;  // ERREUR ! Pas d'emprunt pendant un mut
}
```

### Lifetimes : durées de vie vérifiées

```rust
// Cette fonction retourne une référence
// Le compilateur vérifie qu'elle vit assez longtemps
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() { x } else { y }
}

fn main() {
    let string1 = String::from("hello");
    let result;
    {
        let string2 = String::from("world!");
        result = longest(&string1, &string2);
        println!("{}", result);  // OK ici
    }
    // println!("{}", result);  // ERREUR ! string2 est détruit
}
```

## Exemples de bugs évités

### Use-After-Free (impossible en Rust)

```rust
// En C, ce code compile et crash au runtime :
// char* ptr = malloc(10);
// free(ptr);
// printf("%s", ptr);  // Use-after-free !

// En Rust, impossible :
fn main() {
    let data = Box::new([0u8; 10]);
    drop(data);  // Libération explicite
    
    // println!("{:?}", data);  // ERREUR COMPILATION
    // "value used here after move"
}
```

### Buffer Overflow (impossible en Rust)

```rust
fn main() {
    let arr = [1, 2, 3, 4, 5];
    
    // Accès par index vérifié
    // println!("{}", arr[10]);  // PANIC au runtime (pas UB)
    
    // Accès sécurisé recommandé
    if let Some(value) = arr.get(10) {
        println!("{}", value);
    } else {
        println!("Index hors limites");
    }
}
```

### Data Race (impossible en Rust safe)

```rust
use std::thread;
use std::sync::Arc;
use std::sync::Mutex;

fn main() {
    // Sans synchronisation, ça ne compile pas :
    // let mut counter = 0;
    // thread::spawn(|| counter += 1);  // ERREUR
    
    // Avec Arc + Mutex, thread-safe garanti :
    let counter = Arc::new(Mutex::new(0));
    
    let handles: Vec<_> = (0..10).map(|_| {
        let counter = Arc::clone(&counter);
        thread::spawn(move || {
            let mut num = counter.lock().unwrap();
            *num += 1;
        })
    }).collect();
    
    for handle in handles {
        handle.join().unwrap();
    }
    
    println!("Final: {}", *counter.lock().unwrap());
}
```

## Null Safety

### Option au lieu de null

```rust
// En Rust, pas de null. On utilise Option<T>

pub struct User {
    pub name: String,
    pub email: Option<String>,  // Explicitement optionnel
}

fn get_email_domain(user: &User) -> Option<String> {
    // Le compilateur force à gérer le cas None
    user.email.as_ref().map(|email| {
        email.split('@').nth(1).unwrap_or("").to_string()
    })
}

fn main() {
    let user = User {
        name: "Alice".to_string(),
        email: Some("alice@example.com".to_string()),
    };
    
    // Pattern matching obligatoire
    match get_email_domain(&user) {
        Some(domain) => println!("Domain: {}", domain),
        None => println!("Pas d'email"),
    }
    
    // Ou avec if let
    if let Some(domain) = get_email_domain(&user) {
        println!("Domain: {}", domain);
    }
}
```

## Sécurité sans garbage collector

```rust
/// RAII : Resource Acquisition Is Initialization
/// Les ressources sont libérées automatiquement à la fin du scope

pub struct DatabaseConnection {
    // ... connection details
}

impl Drop for DatabaseConnection {
    fn drop(&mut self) {
        // Fermeture automatique de la connexion
        println!("Connection fermée proprement");
    }
}

fn process_data() {
    let conn = DatabaseConnection::new();
    
    // Utiliser la connexion...
    
}  // `conn` est automatiquement drop ici, connexion fermée

// Pas de fuite de ressource possible !
```

## Audit et certification

```rust
/// La sécurité mémoire de Rust facilite les audits
pub struct SecurityBenefits {
    /// Classes entières de bugs éliminées
    pub eliminated_bug_classes: Vec<&'static str>,
    
    /// Réduction de la surface d'attaque
    pub attack_surface_reduction: f32,
    
    /// Facilité d'audit
    pub audit_complexity: AuditComplexity,
}

impl Default for SecurityBenefits {
    fn default() -> Self {
        Self {
            eliminated_bug_classes: vec![
                "Use-after-free",
                "Double-free",
                "Buffer overflow",
                "Null pointer dereference",
                "Data races",
            ],
            attack_surface_reduction: 0.70,  // 70% des vulnérabilités
            audit_complexity: AuditComplexity::Lower,
        }
    }
}
```

## Conclusion

La sécurité mémoire de Rust signifie :

- **Pas de CVE** liés à la mémoire dans votre code
- **Pas de crashes mystérieux** en production
- **Pas de failles de sécurité** dues aux bugs classiques
- **Audit simplifié** : classes entières de bugs éliminées
- **Confiance** : si ça compile, ces bugs n'existent pas
