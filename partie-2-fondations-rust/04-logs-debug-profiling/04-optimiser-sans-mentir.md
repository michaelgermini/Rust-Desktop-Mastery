# 7.4 Optimiser Sans Se Mentir

## Mesurer avant d'optimiser

### Principe fondamental

```rust
/// ❌ MAUVAIS : Optimiser sans mesurer
fn process_data(data: &[u32]) -> Vec<u32> {
    // "Je pense que ça va être plus rapide"
    data.iter().map(|x| x * 2).collect()
}

/// ✅ BON : Mesurer d'abord
fn process_data(data: &[u32]) -> Vec<u32> {
    let start = Instant::now();
    let result = data.iter().map(|x| x * 2).collect();
    let duration = start.elapsed();
    
    tracing::debug!(duration_ms = duration.as_millis(), "process_data");
    
    result
}
```

## Profiling méthodique

### Étape 1 : Identifier le problème

```rust
use std::time::Instant;

#[instrument]
fn slow_function() {
    let start = Instant::now();
    
    step1();
    let t1 = start.elapsed();
    debug!(step1_ms = t1.as_millis());
    
    step2();
    let t2 = start.elapsed();
    debug!(step2_ms = t2.as_millis(), step1_ms = t1.as_millis());
    
    step3();
    let t3 = start.elapsed();
    debug!(step3_ms = t3.as_millis(), total_ms = t3.as_millis());
}
```

### Étape 2 : Benchmarker les changements

```rust
use criterion::{black_box, criterion_group, criterion_main, Criterion};

fn benchmark_optimization(c: &mut Criterion) {
    let data = generate_test_data(10000);
    
    c.bench_function("avant optimisation", |b| {
        b.iter(|| black_box(process_data_old(&data)))
    });
    
    c.bench_function("après optimisation", |b| {
        b.iter(|| black_box(process_data_new(&data)))
    });
}

criterion_group!(benches, benchmark_optimization);
criterion_main!(benches);
```

### Étape 3 : Valider l'amélioration

```bash
# Avant
$ cargo bench
avant optimisation    time:   [125.34 ms 126.12 ms 127.01 ms]

# Après
$ cargo bench
après optimisation    time:   [45.23 ms 45.67 ms 46.12 ms]

# Amélioration : ~2.7x plus rapide ✅
```

## Optimisations courantes

### 1. Éviter les allocations inutiles

```rust
// ❌ Allocation à chaque appel
fn format_message(name: &str) -> String {
    format!("Hello, {}!", name)
}

// ✅ Réutiliser un buffer
fn format_message(name: &str, buf: &mut String) {
    buf.clear();
    buf.push_str("Hello, ");
    buf.push_str(name);
    buf.push('!');
}
```

### 2. Pré-allouer les collections

```rust
// ❌ Réallocations multiples
let mut vec = Vec::new();
for i in 0..1000 {
    vec.push(i);
}

// ✅ Pré-allocation
let mut vec = Vec::with_capacity(1000);
for i in 0..1000 {
    vec.push(i);
}
```

### 3. Utiliser des slices au lieu de Vec quand possible

```rust
// ❌ Clone inutile
fn process(items: Vec<Item>) { ... }

// ✅ Référence
fn process(items: &[Item]) { ... }
```

## Checklist d'optimisation

```rust
pub struct OptimizationChecklist {
    pub measured: bool,           // A-t-on mesuré avant ?
    pub identified_bottleneck: bool, // A-t-on identifié le problème ?
    pub benchmarked: bool,        // A-t-on benchmarké la solution ?
    pub validated: bool,          // L'amélioration est-elle réelle ?
    pub documented: bool,         // A-t-on documenté pourquoi ?
}

impl OptimizationChecklist {
    pub fn is_ready_to_optimize(&self) -> bool {
        self.measured && self.identified_bottleneck
    }
    
    pub fn is_optimization_complete(&self) -> bool {
        self.benchmarked && self.validated && self.documented
    }
}
```

## Résumé

- **Mesurer** : Toujours avant d'optimiser
- **Identifier** : Trouver le vrai bottleneck
- **Benchmarker** : Comparer avant/après
- **Valider** : S'assurer que ça marche vraiment
- **Documenter** : Expliquer pourquoi l'optimisation était nécessaire
