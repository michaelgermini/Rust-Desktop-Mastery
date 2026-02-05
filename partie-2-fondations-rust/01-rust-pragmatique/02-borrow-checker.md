# 4.2 Borrow Checker Sans Douleur

## Qu'est-ce que le borrowing ?

Le borrowing permet d'accéder à une valeur sans en prendre l'ownership. C'est comme prêter un livre : le propriétaire le récupère après.

```rust
fn main() {
    let livre = String::from("Le Guide du Routard Galactique");
    
    // Emprunt immutable : lire sans modifier
    lire_livre(&livre);
    
    // Le livre est toujours à nous
    println!("Mon livre : {}", livre);
}

fn lire_livre(livre: &String) {
    println!("Je lis : {}", livre);
}  // L'emprunt se termine ici
```

## Les deux types d'emprunts

```rust
/// Emprunt immutable (&T) : lecture seule
fn immutable_borrow() {
    let data = vec![1, 2, 3];
    
    // Plusieurs emprunts immutables OK
    let ref1 = &data;
    let ref2 = &data;
    let ref3 = &data;
    
    println!("{:?} {:?} {:?}", ref1, ref2, ref3);  // OK
}

/// Emprunt mutable (&mut T) : lecture et écriture
fn mutable_borrow() {
    let mut data = vec![1, 2, 3];
    
    // UN SEUL emprunt mutable à la fois
    let ref_mut = &mut data;
    ref_mut.push(4);
    
    println!("{:?}", ref_mut);  // [1, 2, 3, 4]
}
```

## La règle d'or

```rust
/// À tout moment, vous pouvez avoir :
/// - SOIT plusieurs références immutables (&T)
/// - SOIT une seule référence mutable (&mut T)
/// - JAMAIS les deux en même temps

fn the_golden_rule() {
    let mut data = vec![1, 2, 3];
    
    // ✅ OK : plusieurs & 
    let r1 = &data;
    let r2 = &data;
    println!("{:?} {:?}", r1, r2);
    
    // ✅ OK : un seul &mut (après que les & ne sont plus utilisés)
    let r_mut = &mut data;
    r_mut.push(4);
    
    // ❌ ERREUR : & et &mut en même temps
    // let r1 = &data;
    // let r_mut = &mut data;
    // println!("{:?}", r1);  // r1 utilisé pendant que r_mut existe
}
```

## Non-Lexical Lifetimes (NLL)

Le compilateur Rust moderne est intelligent : il sait quand un emprunt n'est plus utilisé.

```rust
fn nll_example() {
    let mut data = vec![1, 2, 3];
    
    let r1 = &data;
    println!("Immutable: {:?}", r1);
    // r1 n'est plus utilisé après cette ligne
    
    // ✅ OK grâce à NLL : r1 n'est plus "vivant"
    let r_mut = &mut data;
    r_mut.push(4);
    
    // Avant NLL (Rust < 1.31), c'était une erreur
}
```

## Patterns pratiques

### Emprunter dans une boucle

```rust
fn borrow_in_loop() {
    let items = vec!["a", "b", "c"];
    
    // ✅ Emprunt immutable dans la boucle
    for item in &items {  // &items emprunte le vec
        println!("{}", item);
    }
    // items toujours valide
    
    let mut numbers = vec![1, 2, 3];
    
    // ✅ Emprunt mutable dans la boucle
    for num in &mut numbers {
        *num *= 2;
    }
    println!("{:?}", numbers);  // [2, 4, 6]
}
```

### Emprunter des champs de struct

```rust
struct Person {
    name: String,
    age: u32,
}

fn borrow_struct_fields() {
    let mut person = Person {
        name: String::from("Alice"),
        age: 30,
    };
    
    // ✅ Emprunter différents champs simultanément
    let name_ref = &person.name;
    let age_ref = &mut person.age;
    
    *age_ref += 1;
    println!("{} a {} ans", name_ref, age_ref);
    
    // Rust comprend que name et age sont indépendants
}
```

### Retourner une référence

```rust
/// Retourner une référence nécessite un lifetime
fn first_word(s: &str) -> &str {
    match s.find(' ') {
        Some(i) => &s[..i],
        None => s,
    }
}

fn return_reference() {
    let sentence = String::from("Hello world");
    let word = first_word(&sentence);
    println!("Premier mot : {}", word);
}
```

## Erreurs courantes et solutions

### Cannot borrow as mutable

```rust
fn error_cannot_borrow_mut() {
    let data = vec![1, 2, 3];  // Pas `mut`
    
    // ❌ ERREUR : cannot borrow as mutable
    // data.push(4);
    
    // ✅ Solution : déclarer comme mut
    let mut data = vec![1, 2, 3];
    data.push(4);
}
```

### Cannot borrow while already borrowed

```rust
fn error_already_borrowed() {
    let mut data = vec![1, 2, 3];
    
    // ❌ ERREUR : cannot borrow as mutable because already borrowed
    // let r1 = &data;
    // data.push(4);  // Essaie de modifier pendant l'emprunt
    // println!("{:?}", r1);
    
    // ✅ Solution 1 : finir d'utiliser l'emprunt avant de modifier
    let r1 = &data;
    println!("{:?}", r1);  // Utilisation terminée
    data.push(4);  // OK maintenant
    
    // ✅ Solution 2 : scope explicite
    {
        let r1 = &data;
        println!("{:?}", r1);
    }  // r1 terminé
    data.push(4);  // OK
}
```

### Does not live long enough

```rust
fn error_does_not_live_long_enough() {
    // ❌ ERREUR : borrowed value does not live long enough
    // let r;
    // {
    //     let x = 5;
    //     r = &x;  // x sera détruit à la fin du bloc
    // }
    // println!("{}", r);  // r pointe vers une valeur détruite
    
    // ✅ Solution : s'assurer que la valeur vit assez longtemps
    let x = 5;
    let r = &x;
    println!("{}", r);  // OK, x est toujours vivant
}
```

## Astuces pour éviter les problèmes

### Préférer les slices aux références de collections

```rust
fn prefer_slices() {
    let data = vec![1, 2, 3, 4, 5];
    
    // &Vec<i32> → &[i32] (plus flexible)
    fn process(slice: &[i32]) {
        println!("{:?}", slice);
    }
    
    process(&data);           // Vec complet
    process(&data[1..3]);     // Sous-slice
}
```

### Utiliser des méthodes qui prennent &self

```rust
impl Person {
    // Préférer &self quand possible (lecture)
    fn name(&self) -> &str {
        &self.name
    }
    
    // &mut self seulement si modification nécessaire
    fn set_name(&mut self, name: String) {
        self.name = name;
    }
    
    // self seulement si consommation intentionnelle
    fn into_name(self) -> String {
        self.name
    }
}
```

### Clone en dernier recours

```rust
fn clone_escape_hatch() {
    let data = vec![1, 2, 3];
    
    // Si le borrow checker vous bloque et que clone est acceptable :
    let data_copy = data.clone();
    
    // Maintenant deux vecs indépendants
    // Coût : allocation mémoire
    // À éviter si possible, mais parfois c'est la solution pragmatique
}
```

## Résumé mental

```rust
/// Aide-mémoire borrow checker
pub mod borrow_cheatsheet {
    // 1. & = lecture seule, plusieurs OK
    // 2. &mut = lecture + écriture, un seul
    // 3. Jamais & et &mut en même temps
    // 4. L'emprunt dure jusqu'à la dernière utilisation (NLL)
    // 5. Les références ne peuvent pas outlive leur source
    // 6. En cas de doute : clone() puis optimiser
}
```
