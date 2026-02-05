# 4.1 Ownership Expliqué Simplement

## L'idée centrale

L'ownership est le système de Rust pour gérer la mémoire sans garbage collector. Une analogie simple : chaque valeur a un **propriétaire unique**, comme une clé de voiture.

```rust
fn main() {
    // `data` est le propriétaire du Vec
    let data = vec![1, 2, 3];
    
    // On "donne la clé" à `autre`
    let autre = data;
    
    // `data` n'a plus la clé, on ne peut plus l'utiliser
    // println!("{:?}", data);  // ERREUR !
    
    // `autre` est maintenant le propriétaire
    println!("{:?}", autre);  // OK
}
```

## Les trois règles

```rust
/// Règle 1 : Chaque valeur a UN propriétaire
fn regle_1() {
    let s = String::from("hello");  // `s` possède la String
    // La String a exactement un propriétaire : `s`
}

/// Règle 2 : Une seule propriétaire à la fois
fn regle_2() {
    let s1 = String::from("hello");
    let s2 = s1;  // Ownership transféré à s2
    // s1 n'est plus valide
}

/// Règle 3 : Quand le propriétaire sort du scope, la valeur est détruite
fn regle_3() {
    {
        let s = String::from("hello");  // s est valide ici
    }  // s sort du scope, la mémoire est libérée automatiquement
}
```

## Move vs Copy

```rust
/// Types qui implémentent Copy (petits, sur la stack)
fn copy_types() {
    let x: i32 = 5;
    let y = x;  // Copie, pas move
    println!("x = {}, y = {}", x, y);  // Les deux sont valides
    
    // Types Copy : i32, f64, bool, char, tuples de Copy, arrays de Copy
}

/// Types qui sont moved (heap allocated)
fn move_types() {
    let s1 = String::from("hello");  // String est sur le heap
    let s2 = s1;  // Move, pas copy
    // println!("{}", s1);  // ERREUR : s1 a été moved
    println!("{}", s2);  // OK
    
    // Types Move : String, Vec, Box, structs custom sans Copy
}

/// Implémenter Copy pour vos types
#[derive(Clone, Copy)]  // Seulement si tous les champs sont Copy
struct Point {
    x: f64,
    y: f64,
}

fn copy_custom() {
    let p1 = Point { x: 1.0, y: 2.0 };
    let p2 = p1;  // Copie
    println!("p1 = ({}, {})", p1.x, p1.y);  // OK, p1 est toujours valide
}
```

## Ownership et fonctions

```rust
/// Passer une valeur à une fonction = move
fn take_ownership(s: String) {
    println!("J'ai pris : {}", s);
}  // s est détruit ici

fn ownership_with_functions() {
    let s = String::from("hello");
    take_ownership(s);
    // println!("{}", s);  // ERREUR : s a été moved
}

/// Pour garder l'ownership, utiliser des références (voir 4.2)
fn borrow_instead(s: &String) {
    println!("J'emprunte : {}", s);
}  // s n'est pas détruit, juste l'emprunt se termine

fn borrowing_example() {
    let s = String::from("hello");
    borrow_instead(&s);  // Passer une référence
    println!("Je l'ai toujours : {}", s);  // OK !
}
```

## Ownership et méthodes

```rust
pub struct Document {
    content: String,
}

impl Document {
    /// Méthode qui emprunte self (lecture)
    pub fn len(&self) -> usize {
        self.content.len()
    }
    
    /// Méthode qui emprunte self mutuellement (modification)
    pub fn append(&mut self, text: &str) {
        self.content.push_str(text);
    }
    
    /// Méthode qui prend ownership de self (consommation)
    pub fn into_string(self) -> String {
        self.content  // self est consommé
    }
}

fn method_ownership() {
    let mut doc = Document { content: String::new() };
    
    // &self : emprunte en lecture
    println!("Longueur : {}", doc.len());
    
    // &mut self : emprunte en écriture
    doc.append("Hello");
    
    // self : consomme le document
    let content = doc.into_string();
    // doc n'est plus utilisable ici
}
```

## Patterns courants

### Clone quand nécessaire

```rust
fn clone_when_needed() {
    let original = String::from("data importante");
    
    // Si on a vraiment besoin de deux copies indépendantes
    let copy = original.clone();
    
    // Maintenant les deux sont valides et indépendants
    println!("Original: {}", original);
    println!("Copie: {}", copy);
}
```

### Retourner l'ownership

```rust
/// Pattern pour transformer et retourner
fn transform_and_return(mut data: Vec<i32>) -> Vec<i32> {
    data.push(42);
    data  // Retourne l'ownership
}

fn return_ownership_example() {
    let data = vec![1, 2, 3];
    let data = transform_and_return(data);  // Re-bind avec le résultat
    println!("{:?}", data);  // [1, 2, 3, 42]
}
```

### Ownership dans les structs

```rust
pub struct User {
    // User possède ces Strings
    pub name: String,
    pub email: String,
}

impl User {
    pub fn new(name: String, email: String) -> Self {
        Self { name, email }  // Prend ownership des paramètres
    }
    
    // Alternative avec &str si on veut copier
    pub fn from_strs(name: &str, email: &str) -> Self {
        Self {
            name: name.to_string(),
            email: email.to_string(),
        }
    }
}
```

## Erreurs courantes et solutions

```rust
/// Erreur : use after move
fn error_use_after_move() {
    let data = vec![1, 2, 3];
    let _moved = data;
    // let len = data.len();  // ERREUR !
    
    // Solution 1 : utiliser une référence
    let data = vec![1, 2, 3];
    let borrowed = &data;
    let len = data.len();  // OK
    
    // Solution 2 : clone si vraiment nécessaire
    let data = vec![1, 2, 3];
    let cloned = data.clone();
    let len = data.len();  // OK
}

/// Erreur : partial move
fn error_partial_move() {
    struct Container {
        name: String,
        value: i32,
    }
    
    let c = Container { name: String::from("test"), value: 42 };
    let name = c.name;  // Move partiel
    // let full = c;  // ERREUR : c.name a été moved
    
    // Solution : clone le champ ou utiliser ref
    let c = Container { name: String::from("test"), value: 42 };
    let name = c.name.clone();  // Clone au lieu de move
    let full = c;  // OK maintenant
}
```

## Résumé mental

```rust
/// Aide-mémoire ownership
pub mod ownership_cheatsheet {
    // 1. Par défaut, assigner = MOVE
    // 2. Types simples (i32, bool...) = COPY automatique
    // 3. Types heap (String, Vec...) = MOVE (ou clone explicite)
    // 4. Passer à une fonction = MOVE (sauf si référence)
    // 5. Sortie de scope = destruction automatique
    // 6. En cas de doute : utiliser une référence &
}
```
