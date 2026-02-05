# 7.3 Flamegraphs

## Génération de flamegraphs

### Installation

```bash
cargo install cargo-flamegraph
```

### Utilisation basique

```bash
# Générer un flamegraph
cargo flamegraph

# Avec options
cargo flamegraph --bin mon_app -- --some-args

# Pour un benchmark spécifique
cargo flamegraph --bench search_benchmark
```

### Configuration

```toml
# Cargo.toml
[profile.release]
debug = true  # Nécessaire pour les symboles
```

## Analyse des performances

### Interpréter un flamegraph

```
┌─────────────────────────────────────┐
│  main()                             │
│  ├─ render_frame()                  │
│  │  ├─ render_ui()                   │
│  │  │  ├─ render_list()             │  ← 40% du temps ici
│  │  │  └─ render_buttons()          │
│  │  └─ process_events()             │
│  └─ update_state()                  │
└─────────────────────────────────────┘
```

### Identifier les bottlenecks

```rust
// Avant optimisation
fn render_list(items: &[Item]) {
    for item in items {
        render_item(item);  // Appelé 10,000 fois
    }
}

// Après optimisation
fn render_list(items: &[Item]) {
    // Virtual scrolling : ne rendre que les items visibles
    let visible_range = calculate_visible_range();
    for item in &items[visible_range] {
        render_item(item);  // Seulement ~20 items rendus
    }
}
```

## Tracing Chrome pour profiling

```toml
# Cargo.toml
[dependencies]
tracing-chrome = "0.7"
```

```rust
use tracing_chrome::ChromeLayerBuilder;
use tracing_subscriber::prelude::*;

fn setup_chrome_tracing() {
    let (chrome_layer, guard) = ChromeLayerBuilder::new()
        .file("trace.json")
        .build();
    
    tracing_subscriber::registry()
        .with(chrome_layer)
        .init();
    
    // Le guard doit rester en vie pendant l'exécution
    std::mem::forget(guard);
}
```

### Visualiser dans Chrome

1. Ouvrir `chrome://tracing`
2. Charger `trace.json`
3. Analyser la timeline

## Résumé

- **cargo-flamegraph** : Outil standard pour générer des flamegraphs
- **Interprétation** : Largeur = temps passé, hauteur = call stack
- **Optimisation** : Cibler les fonctions les plus larges
- **Chrome tracing** : Alternative avec timeline interactive
