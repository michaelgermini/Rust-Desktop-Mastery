# 1.2 Performance Native

## Qu'est-ce que la performance native ?

La performance native signifie que votre code s'exécute directement sur le CPU, sans couche d'interprétation ou de machine virtuelle.

```rust
// Code Rust compilé en assembleur optimisé
pub fn sum_array(data: &[i64]) -> i64 {
    data.iter().sum()
}

// Devient (simplifié) :
// mov rax, 0
// loop:
//   add rax, [rdi]
//   add rdi, 8
//   dec rcx
//   jnz loop
//   ret
```

## Zero-cost abstractions

### Le principe

```rust
/// En Rust, les abstractions de haut niveau
/// n'ont pas de coût runtime

// Cette fonction utilisant des itérateurs...
pub fn process_optimized(data: &[u32]) -> u32 {
    data.iter()
        .filter(|&&x| x > 10)
        .map(|&x| x * 2)
        .sum()
}

// ...compile au même code que cette version manuelle :
pub fn process_manual(data: &[u32]) -> u32 {
    let mut sum = 0;
    for &x in data {
        if x > 10 {
            sum += x * 2;
        }
    }
    sum
}
```

### Comparaison avec d'autres langages

```rust
// JavaScript (interprété + JIT)
// - Parsing du code
// - Compilation JIT à chaud
// - Garbage collection imprévisible
// - Boxed objects partout

// Java/C# (VM + GC)
// - Bytecode → machine code
// - GC pauses (même optimisé)
// - Indirection des objets

// Rust (natif)
// - Compilation AOT optimisée
// - Pas de GC
// - Données inline en mémoire
// - Déroulage de boucles automatique
```

## Benchmarks concrets

### UI rendering (60 FPS target = 16.6ms par frame)

```rust
use std::time::Instant;

pub struct FrameStats {
    pub frame_time_ms: f64,
    pub fps: f64,
    pub dropped_frames: u32,
}

// Mesures typiques pour une UI complexe (1000 éléments)

// Electron/React :
//   Frame time: 25-40ms
//   FPS: 25-40
//   Dropped frames: fréquents

// Rust/egui :
//   Frame time: 2-5ms
//   FPS: 60 stable
//   Dropped frames: 0
```

### Démarrage de l'application

```rust
use std::time::Duration;

pub struct StartupMetrics {
    pub cold_start: Duration,
    pub warm_start: Duration,
    pub time_to_interactive: Duration,
}

// Comparatif application de gestion (~50 écrans)

const ELECTRON_STARTUP: StartupMetrics = StartupMetrics {
    cold_start: Duration::from_millis(3500),
    warm_start: Duration::from_millis(1500),
    time_to_interactive: Duration::from_millis(5000),
};

const RUST_STARTUP: StartupMetrics = StartupMetrics {
    cold_start: Duration::from_millis(150),
    warm_start: Duration::from_millis(80),
    time_to_interactive: Duration::from_millis(200),
};
```

### Consommation mémoire

```rust
pub struct MemoryFootprint {
    pub baseline_mb: u32,
    pub with_1000_items: u32,
    pub with_10000_items: u32,
}

// Application de gestion de tâches

const ELECTRON_MEMORY: MemoryFootprint = MemoryFootprint {
    baseline_mb: 280,
    with_1000_items: 450,
    with_10000_items: 1200,
};

const RUST_MEMORY: MemoryFootprint = MemoryFootprint {
    baseline_mb: 25,
    with_1000_items: 45,
    with_10000_items: 120,
};
```

## Optimisations automatiques du compilateur

### LLVM optimizations

```rust
// Le compilateur Rust (via LLVM) applique automatiquement :

// 1. Inlining des petites fonctions
#[inline]
pub fn add(a: i32, b: i32) -> i32 {
    a + b  // Pas d'appel de fonction, code inline
}

// 2. Élimination de code mort
pub fn example(condition: bool) -> i32 {
    if condition {
        compute_expensive()  // Gardé
    } else {
        unreachable_code()   // Supprimé si prouvé impossible
    }
}

// 3. Vectorisation automatique (SIMD)
pub fn multiply_arrays(a: &[f32], b: &[f32], out: &mut [f32]) {
    for i in 0..a.len() {
        out[i] = a[i] * b[i];  // Compilé en instructions AVX
    }
}

// 4. Déroulage de boucles
pub fn sum_small(arr: &[i32; 4]) -> i32 {
    arr.iter().sum()
    // Devient : arr[0] + arr[1] + arr[2] + arr[3]
}
```

### Profile-guided optimization (PGO)

```bash
# Compilation en deux passes pour optimiser les chemins chauds

# 1. Build instrumenté
RUSTFLAGS="-Cprofile-generate=/tmp/pgo" cargo build --release

# 2. Exécuter avec workload typique
./target/release/mon-app < typical_workload.txt

# 3. Build optimisé avec profil
RUSTFLAGS="-Cprofile-use=/tmp/pgo" cargo build --release
```

## Impact sur l'expérience utilisateur

### Réactivité instantanée

```rust
/// Temps de réponse perçu par l'utilisateur
pub enum ResponsePerception {
    /// < 100ms : Instantané
    Instant,
    /// 100-300ms : Rapide
    Fast,
    /// 300-1000ms : Perceptible
    Noticeable,
    /// > 1000ms : Lent
    Slow,
}

impl ResponsePerception {
    pub fn from_ms(ms: u64) -> Self {
        match ms {
            0..=100 => Self::Instant,
            101..=300 => Self::Fast,
            301..=1000 => Self::Noticeable,
            _ => Self::Slow,
        }
    }
}

// Avec Rust, la plupart des actions restent dans "Instant"
```

### Batterie sur laptop

```rust
pub struct PowerConsumption {
    pub idle_watts: f32,
    pub active_watts: f32,
    pub estimated_battery_hours: f32,
}

// Sur un laptop avec batterie 60Wh

const ELECTRON_POWER: PowerConsumption = PowerConsumption {
    idle_watts: 8.0,      // Chromium en veille
    active_watts: 25.0,   // Rendering constant
    estimated_battery_hours: 3.5,
};

const RUST_POWER: PowerConsumption = PowerConsumption {
    idle_watts: 0.5,      // Vraiment idle
    active_watts: 5.0,    // Rendering efficace
    estimated_battery_hours: 10.0,
};
```

## Conclusion

La performance native de Rust se traduit par :

- **Démarrage instantané** : < 200ms vs plusieurs secondes
- **UI fluide** : 60 FPS constants sans drops
- **Mémoire raisonnable** : 10x moins qu'Electron
- **Batterie préservée** : 3x plus d'autonomie
- **Scaling linéaire** : Performance prévisible avec les données
