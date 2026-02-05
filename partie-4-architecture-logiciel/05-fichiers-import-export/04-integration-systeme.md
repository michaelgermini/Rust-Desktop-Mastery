# 18.4 Intégration Système

## Dialogues de fichiers natifs

```rust
use native_dialog::FileDialog;

/// Ouvrir un dialogue de sélection de fichier
pub fn pick_file(title: &str, filters: &[(&str, &[&str])]) -> Option<PathBuf> {
    let mut dialog = FileDialog::new().set_title(title);
    
    for (name, extensions) in filters {
        dialog = dialog.add_filter(name, extensions);
    }
    
    dialog.show_open_single_file().ok().flatten()
}

/// Sauvegarder avec dialogue
pub fn save_file_dialog(title: &str, default_name: &str, extension: &str) -> Option<PathBuf> {
    FileDialog::new()
        .set_title(title)
        .set_filename(default_name)
        .add_filter("Fichier", &[extension])
        .show_save_single_file()
        .ok()
        .flatten()
}
```

## Association de fichiers

### Windows

```rust
#[cfg(target_os = "windows")]
pub fn register_file_association(extension: &str, app_name: &str) -> Result<()> {
    use winreg::RegKey;
    use winreg::enums::*;
    
    let hkcu = RegKey::predef(HKEY_CURRENT_USER);
    let key = hkcu.create_subkey(&format!("Software\\Classes\\.{}", extension))?;
    key.set_value("", &format!("{}.file", app_name))?;
    
    Ok(())
}
```

### macOS

```rust
#[cfg(target_os = "macos")]
pub fn register_file_association(extension: &str, app_name: &str) -> Result<()> {
    // Configurer dans Info.plist
    // CFBundleDocumentTypes avec CFBundleTypeExtensions
    Ok(())
}
```

## Ouvrir avec l'application par défaut

```rust
use open;

pub fn open_with_default_app(path: &Path) -> Result<()> {
    #[cfg(target_os = "windows")]
    std::process::Command::new("explorer").arg(path).spawn()?;
    
    #[cfg(target_os = "macos")]
    std::process::Command::new("open").arg(path).spawn()?;
    
    #[cfg(target_os = "linux")]
    std::process::Command::new("xdg-open").arg(path).spawn()?;
    
    // Ou utiliser la crate `open`
    open::that(path)?;
    
    Ok(())
}
```

## Résumé

- **Dialogues natifs** : Utiliser les dialogues système
- **Associations** : Enregistrer les types de fichiers
- **Ouvrir avec** : Utiliser l'application par défaut
- **Multi-plateforme** : Support Windows, macOS, Linux
