# 9.1 Immediate Mode Mental Model

## Retained Mode vs Immediate Mode

### Retained Mode (Qt, GTK, React...)

```rust
// Créer les widgets une fois
let button = Button::new("Click me");
button.set_on_click(|_| {
    println!("Clicked!");
});
container.add(button);

// Les widgets existent comme objets persistants
// Problème : synchronisation état ↔ UI
```

**Caractéristiques** :
- Widgets sont des objets persistants
- État séparé de l'affichage
- Nécessite de synchroniser manuellement
- Plus de contrôle mais plus complexe

### Immediate Mode (egui)

```rust
// Reconstruire à chaque frame (60 FPS)
fn update(&mut self, ctx: &egui::Context, _frame: &mut eframe::Frame) {
    egui::CentralPanel::default().show(ctx, |ui| {
        // Le bouton est "créé" à chaque frame
        // Mais egui gère l'identité en interne
        if ui.button("Click me").clicked() {
            println!("Clicked!");
        }
    });
}

// Avantage : l'UI reflète TOUJOURS l'état actuel
// Pas de désynchronisation possible
```

**Caractéristiques** :
- UI reconstruite chaque frame
- État et affichage toujours synchronisés
- Plus simple mentalement
- Moins de contrôle mais moins d'erreurs

## Le flux egui

```
┌─────────────────────────────────────────────────────────┐
│                    Boucle principale                     │
│                                                         │
│  1. Début de frame                                      │
│     └─ egui prépare le contexte                        │
│                                                         │
│  2. Construction UI (votre code)                        │
│     └─ ui.button(), ui.label(), etc.                   │
│     └─ egui collecte les "shapes" à dessiner          │
│     └─ egui détecte les interactions                   │
│                                                         │
│  3. Rendu                                               │
│     └─ egui envoie les shapes au GPU                   │
│     └─ Rendu à l'écran                                 │
│                                                         │
│  4. Fin de frame → retour à 1                           │
└─────────────────────────────────────────────────────────┘
```

## L'astuce de l'identité

### Comment egui reconnaît les widgets

```rust
// Comment egui sait que c'est le "même" bouton ?
// → Par son texte et sa position dans l'arbre UI

fn update(&mut self, ctx: &egui::Context, _frame: &mut eframe::Frame) {
    egui::CentralPanel::default().show(ctx, |ui| {
        // Ces deux boutons ont des identités différentes
        if ui.button("Bouton A").clicked() { /* ... */ }
        if ui.button("Bouton B").clicked() { /* ... */ }
        
        // Pour des boutons avec le même texte, utiliser push_id
        for i in 0..3 {
            ui.push_id(i, |ui| {
                if ui.button("Action").clicked() {
                    println!("Bouton {} cliqué", i);
                }
            });
        }
    });
}
```

### Identité basée sur le contexte

```rust
// L'identité d'un widget dépend de :
// 1. Son type (Button, Label, etc.)
// 2. Son texte/contenu
// 3. Sa position dans l'arbre UI
// 4. Les IDs parents (push_id)

ui.push_id("section-1", |ui| {
    ui.button("Action");  // ID: section-1 > Action
});

ui.push_id("section-2", |ui| {
    ui.button("Action");  // ID: section-2 > Action (différent!)
});
```

## Avantages du Immediate Mode

### 1. Pas de désynchronisation

```rust
// ❌ Retained mode : risque de désynchronisation
struct RetainedUI {
    button: Button,
    state: AppState,
}

// Si state change mais button pas mis à jour → bug

// ✅ Immediate mode : toujours synchronisé
fn update(&mut self, ctx: &egui::Context, _frame: &mut eframe::Frame) {
    // L'UI reflète toujours self.state
    if self.state.enabled {
        ui.button("Actif");
    } else {
        ui.button("Inactif");
    }
}
```

### 2. Code plus simple

```rust
// Pas besoin de gérer le cycle de vie des widgets
// Pas besoin de callbacks complexes
// Juste : "Si cliqué, faire X"

if ui.button("Sauvegarder").clicked() {
    self.save();
}
```

### 3. Debugging facilité

```rust
// L'UI est juste du code Rust normal
// Facile à déboguer, à tester, à comprendre

fn render_debug_info(ui: &mut egui::Ui, state: &AppState) {
    ui.label(format!("État: {:?}", state));  // Toujours à jour!
}
```

## Inconvénients à connaître

### 1. Performance

```rust
// Chaque frame reconstruit tout
// Mais egui est optimisé pour ça

// Si vous avez 10,000 widgets :
// - egui optimise automatiquement
// - Utilisez virtual scrolling pour les listes longues
```

### 2. Style visuel

```rust
// egui a son propre style
// Moins "natif" que les widgets système
// Mais moderne et cohérent
```

## Résumé

**Immediate Mode** signifie :
- UI reconstruite chaque frame
- État et affichage toujours synchronisés
- Code plus simple et prévisible
- Performance gérée automatiquement par egui

**Avantages principaux** :
- Pas de bugs de synchronisation
- Code plus simple
- Debugging facilité
- Moins de concepts à apprendre
