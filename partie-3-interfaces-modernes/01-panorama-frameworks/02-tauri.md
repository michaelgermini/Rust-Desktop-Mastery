# 8.2 Tauri : Le Meilleur des Deux Mondes

## Présentation

Tauri permet de construire des applications desktop avec un frontend web (HTML/CSS/JS) et un backend Rust.

```toml
# Cargo.toml
[dependencies]
tauri = { version = "1.5", features = ["shell-open"] }
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
```

## Architecture

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

## Exemple de commande Tauri

### Backend Rust

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

### Frontend JavaScript

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

## Forces et faiblesses

| Forces | Faiblesses |
|--------|------------|
| UI web moderne (CSS, etc.) | Deux écosystèmes à maintenir |
| Binaire léger (~10 MB) | WebView dépendant du système |
| Sécurité renforcée | Complexité de la communication |
| Écosystème web mature | |
| Auto-update intégré | |

## Cas d'usage idéaux

- **Applications grand public** : UI riche et animée
- **Interfaces complexes** : Utilisation de frameworks web modernes
- **Équipes avec expertise web** : Réutilisation des compétences
- **Migration d'apps Electron** : Alternative plus légère

## Quand choisir Tauri

Choisissez Tauri si :
- Vous avez une équipe web forte
- Vous avez besoin d'une UI très riche/animée
- Vous migrez depuis Electron
- Le style visuel natif n'est pas une priorité

## Résumé

Tauri combine la puissance de Rust avec la flexibilité du web, idéal pour les applications nécessitant une UI web moderne avec un backend performant.
