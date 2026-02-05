# 1.4 Binaire Unique

## La simplicité du déploiement

Un binaire unique signifie : un seul fichier exécutable contenant tout le nécessaire pour faire fonctionner votre application.

```bash
# Déploiement d'une application Rust
$ ls -lh target/release/
-rwxr-xr-x  1 user  staff   8.5M mon-app

# C'est tout. Un fichier. Copier-coller et ça marche.
```

## Comparaison avec d'autres approches

### Application Electron

```
mon-app-electron/
├── mon-app.exe                    # 1 MB
├── resources/
│   ├── app.asar                   # 50 MB (votre code)
│   └── ...
├── locales/                       # 5 MB
├── chrome_100_percent.pak         # 1 MB
├── chrome_200_percent.pak         # 2 MB
├── d3dcompiler_47.dll            # 4 MB
├── ffmpeg.dll                     # 2 MB
├── icudtl.dat                     # 10 MB
├── libEGL.dll                     # 0.5 MB
├── libGLESv2.dll                  # 7 MB
├── node.dll                       # 30 MB
├── v8_context_snapshot.bin        # 1 MB
└── ... (50+ autres fichiers)

Total : ~150-200 MB, 100+ fichiers
```

### Application Python

```
mon-app-python/
├── mon-app.exe                    # PyInstaller
├── python39.dll                   # 4 MB
├── _internal/
│   ├── base_library.zip          # 2 MB
│   ├── certifi/                  # Certificats
│   ├── numpy/                    # 15 MB si utilisé
│   ├── pandas/                   # 50 MB si utilisé
│   └── ... (centaines de fichiers)

Total : 50-500 MB selon dépendances
```

### Application Rust

```
mon-app-rust/
└── mon-app.exe                    # 8-15 MB

Total : 8-15 MB, 1 fichier
```

## Compilation statique

### Tout est inclus

```rust
// Lors de la compilation, Rust lie statiquement :
// - La bibliothèque standard (si musl)
// - Toutes vos dépendances
// - Le code généré

// Cargo.toml
[profile.release]
lto = true           // Link-Time Optimization
strip = true         // Supprimer symboles debug
panic = "abort"      // Pas de déroulage de pile
codegen-units = 1    // Meilleure optimisation
```

### Compilation musl pour Linux

```bash
# Linux avec libc statique (vraiment portable)
rustup target add x86_64-unknown-linux-musl
cargo build --release --target x86_64-unknown-linux-musl

# Le binaire fonctionne sur n'importe quelle distribution Linux
# Même Alpine, même un container vide
```

## Avantages pratiques

### Distribution simplifiée

```rust
/// Processus de distribution
pub enum DistributionProcess {
    /// Rust : copier le binaire
    Rust {
        steps: Vec<&'static str>,
    },
    
    /// Electron : installer, configurer, dépendances
    Electron {
        steps: Vec<&'static str>,
    },
}

impl DistributionProcess {
    pub fn rust() -> Self {
        Self::Rust {
            steps: vec![
                "1. Télécharger le fichier",
                "2. Double-cliquer pour lancer",
            ],
        }
    }
    
    pub fn electron() -> Self {
        Self::Electron {
            steps: vec![
                "1. Télécharger l'installeur (150MB)",
                "2. Lancer l'installeur",
                "3. Accepter les permissions",
                "4. Attendre l'extraction",
                "5. Configurer",
                "6. Redémarrer si nécessaire",
            ],
        }
    }
}
```

### Pas de "DLL Hell"

```rust
// Problèmes évités avec un binaire unique :

// ❌ "msvcr140.dll is missing"
// ❌ "This app requires .NET Framework 4.8"
// ❌ "Please install Visual C++ Redistributable"
// ❌ Conflit de versions de DLL
// ❌ Dépendances système manquantes

// ✅ Un fichier, ça marche
```

### Portabilité USB

```rust
/// Application portable sur clé USB
pub struct PortableApp {
    /// Le binaire
    pub executable: PathBuf,
    
    /// Configuration locale (optionnel)
    pub config_dir: Option<PathBuf>,
    
    /// Données locales (optionnel)
    pub data_dir: Option<PathBuf>,
}

impl PortableApp {
    pub fn setup_portable_mode() -> Self {
        let exe_path = std::env::current_exe().unwrap();
        let app_dir = exe_path.parent().unwrap();
        
        Self {
            executable: exe_path,
            config_dir: Some(app_dir.join("config")),
            data_dir: Some(app_dir.join("data")),
        }
    }
}
```

## Optimisation de la taille

### Techniques de réduction

```toml
# Cargo.toml optimisé pour la taille
[profile.release]
opt-level = "z"      # Optimiser pour la taille (pas la vitesse)
lto = true           # Link-Time Optimization
strip = true         # Supprimer les symboles
panic = "abort"      # Pas de panic unwinding
codegen-units = 1    # Un seul codegen unit
```

```bash
# Outils supplémentaires
cargo install cargo-bloat   # Analyser ce qui prend de la place
cargo bloat --release       # Voir les fonctions les plus grosses

# UPX pour compression (optionnel)
upx --best target/release/mon-app
# Réduit souvent de 50-70%
```

### Analyse de la taille

```bash
$ cargo bloat --release --crates

 File  .text     Size Crate
 5.2%  14.8%  1.2MiB egui
 3.1%   8.9%  720KiB std
 2.8%   8.0%  648KiB eframe
 1.9%   5.4%  432KiB regex
 ...

Total: 8.1 MiB
```

## Cas pratiques

### Script d'installation zéro

```bash
#!/bin/bash
# Installation en une ligne

# Télécharger et exécuter directement
curl -L https://example.com/mon-app -o mon-app
chmod +x mon-app
./mon-app
```

### Container Docker minimal

```dockerfile
# Image Docker ultra-légère
FROM scratch
COPY mon-app /
ENTRYPOINT ["/mon-app"]

# Taille de l'image : ~10 MB (juste le binaire)
# Vs Alpine + Python : ~150 MB minimum
```

### Distribution air-gapped

```rust
/// Pour environnements sans internet
pub fn deploy_airgapped(usb_path: &Path) -> std::io::Result<()> {
    let binary = include_bytes!("../target/release/mon-app");
    
    std::fs::write(usb_path.join("mon-app.exe"), binary)?;
    
    // C'est tout. Pas de dépendances à copier.
    Ok(())
}
```

## Conclusion

Le binaire unique Rust offre :

- **Simplicité** : Un fichier = une application
- **Portabilité** : Fonctionne sans installation
- **Fiabilité** : Pas de dépendances externes cassées
- **Légèreté** : 8-15 MB vs 150+ MB
- **Sécurité** : Moins de surface d'attaque
