# 27.4 Installers

## Expérience d'installation

### Windows MSI

```xml
<!-- Ajouter des pages personnalisées -->
<UI>
    <UIRef Id="WixUI_InstallDir" />
    <Publish Dialog="WelcomeDlg" Control="Next" Event="NewDialog" Value="InstallDirDlg">1</Publish>
    <Publish Dialog="InstallDirDlg" Control="Back" Event="NewDialog" Value="WelcomeDlg">1</Publish>
</UI>
```

### macOS DMG

```bash
# DMG avec instructions visuelles
create-dmg \
    --volname "Mon Application" \
    --background "assets/dmg-background.png" \
    --window-pos 200 120 \
    --window-size 600 400 \
    --icon-size 100 \
    --icon "Mon Application.app" 175 120 \
    --app-drop-link 425 120 \
    "Mon Application.dmg" \
    "Mon Application.app"
```

## Désinstallation propre

### Windows

```xml
<!-- WiX gère automatiquement la désinstallation -->
<!-- Tous les fichiers et clés de registre sont supprimés -->
```

### macOS

```bash
# Le .app bundle peut être simplement supprimé
# Pour une désinstallation complète, supprimer aussi :
# ~/Library/Application Support/Mon Application
# ~/Library/Preferences/com.example.monapp.plist
```

### Linux

```bash
# AppImage : Supprimer le fichier
# Debian : sudo apt remove mon-app
# Flatpak : flatpak uninstall com.example.monapp
```

## Mises à jour

```rust
// Dans l'application
pub fn check_for_updates() -> Result<Option<UpdateInfo>> {
    // Vérifier la version disponible
    let latest_version = fetch_latest_version()?;
    let current_version = env!("CARGO_PKG_VERSION");
    
    if latest_version != current_version {
        Ok(Some(UpdateInfo {
            current: current_version.to_string(),
            latest: latest_version,
            download_url: format!("https://releases.example.com/v{}/", latest_version),
        }))
    } else {
        Ok(None)
    }
}
```

## Résumé

- **Expérience** : Installation simple et claire
- **Désinstallation** : Nettoyage complet des fichiers
- **Mises à jour** : Vérification et téléchargement automatiques
- **Multi-plateforme** : Support Windows, macOS, Linux
