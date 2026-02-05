# 8.4 iced : UI Déclarative Style Elm

## Présentation

iced propose une approche déclarative inspirée d'Elm, avec des widgets natifs et un système de messages.

```toml
# Cargo.toml
[dependencies]
iced = "0.12"
```

## Exemple

```rust
use iced::widget::{button, column, text, text_input};
use iced::{Element, Sandbox, Settings};

fn main() -> iced::Result {
    Counter::run(Settings::default())
}

#[derive(Default)]
struct Counter {
    value: i32,
    input: String,
}

#[derive(Debug, Clone)]
enum Message {
    Increment,
    Decrement,
    InputChanged(String),
}

impl Sandbox for Counter {
    type Message = Message;

    fn new() -> Self {
        Self::default()
    }

    fn title(&self) -> String {
        String::from("Counter")
    }

    fn update(&mut self, message: Message) {
        match message {
            Message::Increment => self.value += 1,
            Message::Decrement => self.value -= 1,
            Message::InputChanged(s) => self.input = s,
        }
    }

    fn view(&self) -> Element<Message> {
        column![
            button("Increment").on_press(Message::Increment),
            text(self.value).size(50),
            button("Decrement").on_press(Message::Decrement),
            text_input("Enter text...", &self.input)
                .on_input(Message::InputChanged),
        ]
        .padding(20)
        .into()
    }
}
```

## Architecture Elm-like

### Pattern Model-View-Update

```rust
// 1. MODEL : L'état de l'application
struct AppState {
    counter: i32,
    name: String,
}

// 2. MESSAGE : Les actions possibles
enum Message {
    Increment,
    Decrement,
    NameChanged(String),
}

// 3. UPDATE : Comment l'état change
impl AppState {
    fn update(&mut self, message: Message) {
        match message {
            Message::Increment => self.counter += 1,
            Message::Decrement => self.counter -= 1,
            Message::NameChanged(s) => self.name = s,
        }
    }
}

// 4. VIEW : Comment afficher l'état
fn view(state: &AppState) -> Element<Message> {
    column![
        text(&state.name),
        button("+").on_press(Message::Increment),
        text(state.counter).size(50),
        button("-").on_press(Message::Decrement),
    ].into()
}
```

## Forces et faiblesses

| Forces | Faiblesses |
|--------|------------|
| Architecture claire (Elm) | Moins mature qu'egui |
| Widgets natifs sur certaines plateformes | Documentation en développement |
| Async intégré | Moins de widgets disponibles |
| Type-safe | |
| Prédictible | |

## Cas d'usage idéaux

- **Applications avec état complexe** : Le pattern Elm gère bien la complexité
- **Équipes habituées à Elm/Redux** : Architecture familière
- **Applications nécessitant widgets natifs** : Sur certaines plateformes

## Comparaison avec egui

| Aspect | egui | iced |
|--------|------|------|
| Paradigme | Immediate mode | Declarative (Elm) |
| Courbe d'apprentissage | Douce | Moyenne |
| Performance | Excellente | Bonne |
| Widgets | Nombreux | En développement |
| Maturité | Très mature | En développement actif |

## Résumé

iced est un excellent choix si :
- Vous préférez le style déclaratif
- Vous avez besoin de widgets natifs
- Vous appréciez l'architecture Elm
- Vous êtes prêt à contribuer à un projet en développement

Pour ce livre, nous choisissons egui pour sa simplicité et sa maturité.
