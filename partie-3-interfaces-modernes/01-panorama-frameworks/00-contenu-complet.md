# Chapitre 8 : Panorama des Frameworks UI

## Introduction

L'écosystème Rust offre plusieurs approches pour construire des interfaces graphiques. Ce chapitre présente les options principales et vous aide à choisir celle qui convient à votre projet.

---

## 8.1 egui : Immediate Mode Simple et Puissant

### Présentation

egui (prononcé "e-gooey") est une bibliothèque UI en mode immédiat, portable et facile à utiliser.

```toml
# Cargo.toml
[dependencies]
eframe = "0.27"  # egui + framework d'application
egui = "0.27"
```

### Exemple minimal

```rust
use eframe::egui;

fn main() -> eframe::Result<()> {
    eframe::run_native(
        "Mon Application",
        eframe::NativeOptions::default(),
        Box::new(|_cc| Ok(Box::new(MyApp::default()))),
    )
}

#[derive(Default)]
struct MyApp {
    name: String,
    counter: i32,
}

impl eframe::App for MyApp {
    fn update(&mut self, ctx: &egui::Context, _frame: &mut eframe::Frame) {
        egui::CentralPanel::default().show(ctx, |ui| {
            ui.heading("Hello egui!");
            
            ui.horizontal(|ui| {
                ui.label("Nom:");
                ui.text_edit_singleline(&mut self.name);
            });
            
            if ui.button("Incrémenter").clicked() {
                self.counter += 1;
            }
            
            ui.label(format!("Compteur: {}", self.counter));
        });
    }
}
```

### Forces et faiblesses

| Forces | Faiblesses |
|--------|------------|
| Très simple à apprendre | Style visuel distinctif |
| Portable (Web, Desktop, Mobile) | Moins de widgets natifs |
| Rendu GPU efficace | Accessibilité limitée |
| Excellente documentation | Mode immédiat parfois déroutant |
| Idéal pour outils dev | |

### Cas d'usage idéaux

- Outils de développement
- Éditeurs de jeux
- Applications internes
- Prototypage rapide
- Dashboards

---

## 8.2 Tauri : Le Meilleur des Deux Mondes

### Présentation

Tauri permet de construire des applications desktop avec un frontend web (HTML/CSS/JS) et un backend Rust.

```toml
# Cargo.toml
[dependencies]
tauri = { version = "1.5", features = ["shell-open"] }
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
```

### Architecture

```
┌────────────────────────────────────────┐
│           Frontend (Web)               │
│  - React / Vue / Svelte / Vanilla      │
│  - HTML / CSS / JavaScript             │
│  - Tailwind, etc.                      │
├────────────────────────────────────────┤
│           Tauri Bridge                  │
│  - Commands (Rust ← JS)                │
│  - Events (Rust → JS)                  │
├────────────────────────────────────────┤
│           Backend (Rust)               │
│  - Logique métier                      │
│  - Accès fichiers                      │
│  - Base de données                     │
│  - Performance critique                │
└────────────────────────────────────────┘
```

### Exemple de commande Tauri

```rust
// src-tauri/src/main.rs
use tauri::Manager;

#[tauri::command]
fn process_file(path: String) -> Result<ProcessedData, String> {
    let content = std::fs::read_to_string(&path)
        .map_err(|e| e.to_string())?;
    
    // Traitement lourd en Rust
    let processed = heavy_computation(&content);
    
    Ok(processed)
}

fn main() {
    tauri::Builder::default()
        .invoke_handler(tauri::generate_handler![process_file])
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

```javascript
// Frontend JavaScript
import { invoke } from '@tauri-apps/api/tauri';

async function handleFile(path) {
    try {
        const result = await invoke('process_file', { path });
        console.log('Processed:', result);
    } catch (error) {
        console.error('Error:', error);
    }
}
```

### Forces et faiblesses

| Forces | Faiblesses |
|--------|------------|
| UI web moderne (CSS, etc.) | Deux écosystèmes à maintenir |
| Binaire léger (~10 MB) | WebView dépendant du système |
| Sécurité renforcée | Complexité de la communication |
| Écosystème web mature | |
| Auto-update intégré | |

### Cas d'usage idéaux

- Applications grand public
- Interfaces riches et animées
- Équipes avec expertise web
- Migration d'apps Electron

---

## 8.3 wgpu : Rendu GPU Direct

### Présentation

wgpu est une abstraction bas niveau pour le rendu GPU, utilisable pour des UI custom ou des visualisations avancées.

```toml
# Cargo.toml
[dependencies]
wgpu = "0.19"
winit = "0.29"
```

### Quand utiliser wgpu directement

- Visualisations 3D ou 2D complexes
- Rendu de millions de points
- Éditeurs graphiques (comme Figma)
- Jeux ou simulations

### Architecture typique

```rust
// wgpu est souvent utilisé AVEC egui
use egui_wgpu::Renderer;

// egui utilise wgpu en backend pour le rendu GPU
// Vous pouvez aussi combiner du rendu custom wgpu avec egui
```

### Forces et faiblesses

| Forces | Faiblesses |
|--------|------------|
| Performance maximale | Courbe d'apprentissage raide |
| Contrôle total | Beaucoup de code pour des UI simples |
| Cross-platform (Vulkan, Metal, DX12, WebGPU) | Pas de widgets prêts à l'emploi |

---

## 8.4 iced : UI Déclarative Style Elm

### Présentation

iced propose une approche déclarative inspirée d'Elm, avec des widgets natifs et un système de messages.

```toml
# Cargo.toml
[dependencies]
iced = "0.12"
```

### Exemple

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

### Forces et faiblesses

| Forces | Faiblesses |
|--------|------------|
| Architecture claire (Elm) | Moins mature qu'egui |
| Widgets natifs sur certaines plateformes | Documentation en développement |
| Async intégré | Moins de widgets disponibles |
| Type-safe | |

---

## 8.5 Quand Choisir Quoi

### Matrice de décision

| Critère | egui | Tauri | iced | wgpu |
|---------|------|-------|------|------|
| Courbe d'apprentissage | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐ |
| UI complexe/belle | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐ |
| Performance | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| Taille binaire | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| Widgets natifs | ⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐ |
| Accessibilité | ⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐ |

### Guide par type de projet

**Outil de développement / Debug viewer**
→ **egui** : Simple, rapide à développer, parfait pour les outils internes

**Application grand public**
→ **Tauri** : UI moderne, familière aux utilisateurs, auto-update

**Dashboard / Visualisation de données**
→ **egui** ou **iced** : Rendu efficace, facile à customiser

**Éditeur graphique / CAD**
→ **wgpu** + **egui** : Rendu custom + panels UI

**Application métier PME**
→ **egui** ou **Tauri** : Selon l'importance de l'esthétique

### Mon choix pour ce livre : egui

Pour le reste de ce livre, nous utiliserons principalement **egui** car :

1. **Productivité** : Code UI très concis
2. **Portabilité** : Fonctionne partout identiquement
3. **Performance** : Rendu GPU efficace
4. **Apprentissage** : Concepts transférables aux autres frameworks
5. **Écosystème** : Beaucoup de crates complémentaires

```rust
// Ce que nous allons construire : une application complète avec egui
fn main() -> eframe::Result<()> {
    let options = eframe::NativeOptions {
        viewport: egui::ViewportBuilder::default()
            .with_inner_size([1200.0, 800.0])
            .with_min_inner_size([800.0, 600.0]),
        ..Default::default()
    };
    
    eframe::run_native(
        "Application Professionnelle",
        options,
        Box::new(|cc| {
            // Configuration du style
            setup_custom_style(&cc.egui_ctx);
            Ok(Box::new(ProfessionalApp::new(cc)))
        }),
    )
}
```

---

## Résumé du Chapitre

| Framework | Quand l'utiliser |
|-----------|-----------------|
| **egui** | Outils, dashboards, prototypes, apps métier |
| **Tauri** | Apps grand public, UI web, équipes web |
| **iced** | Apps déclaratives, architecture Elm |
| **wgpu** | Rendu custom, visualisations, jeux |

---

**Dans le prochain chapitre, nous plongerons dans egui pour construire des interfaces utilisateur réactives et professionnelles.**
