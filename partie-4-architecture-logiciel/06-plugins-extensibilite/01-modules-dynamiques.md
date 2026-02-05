# 19.1 Modules Dynamiques

## Chargement dynamique de crates

Rust ne supporte pas le chargement dynamique de crates compilées directement. Alternatives :

### Option 1 : FFI et C ABI

```rust
use std::ffi::CStr;
use std::os::raw::c_char;

#[repr(C)]
pub struct PluginApi {
    pub name: *const c_char,
    pub version: *const c_char,
    pub init: extern "C" fn(),
    pub execute: extern "C" fn(*const c_char) -> *const c_char,
}

// Charger une bibliothèque C
#[cfg(unix)]
pub fn load_plugin(path: &Path) -> Result<PluginApi> {
    unsafe {
        let lib = libloading::Library::new(path)?;
        let api: libloading::Symbol<PluginApi> = lib.get(b"plugin_api")?;
        Ok(*api)
    }
}
```

### Option 2 : Scripts interprétés

```rust
use rhai::Engine;

pub struct ScriptPlugin {
    engine: Engine,
    script: String,
}

impl ScriptPlugin {
    pub fn load(path: &Path) -> Result<Self> {
        let script = std::fs::read_to_string(path)?;
        let mut engine = Engine::new();
        
        // Enregistrer les fonctions disponibles
        engine.register_fn("log", |msg: &str| println!("{}", msg));
        
        Ok(Self { engine, script })
    }
    
    pub fn execute(&mut self) -> Result<()> {
        self.engine.eval::<()>(&self.script)?;
        Ok(())
    }
}
```

## Limitations et alternatives

### Limitations

- **Pas de chargement dynamique Rust natif** : Les crates doivent être compilées statiquement
- **ABI instable** : L'ABI Rust n'est pas stable entre versions
- **Complexité** : FFI ajoute de la complexité

### Alternatives recommandées

1. **Feature flags** : Activer/désactiver des features à la compilation
2. **Scripts** : Utiliser un langage de script (Lua, Rhai, etc.)
3. **Configuration** : Modules activables via configuration
4. **WebAssembly** : Charger des plugins en WASM

## Résumé

- **FFI** : Interface C pour plugins externes
- **Scripts** : Langages interprétés pour extensibilité
- **Limitations** : Pas de chargement dynamique Rust natif
- **Alternatives** : Feature flags, scripts, WASM
