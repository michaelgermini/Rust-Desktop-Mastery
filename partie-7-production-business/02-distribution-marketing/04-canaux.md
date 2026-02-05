# 30.4 Canaux de Distribution

## Site web direct

```yaml
# Distribution principale
distribution:
  direct:
    website: "https://example.com/download"
    github_releases: true
    benefits:
      - "Pas de commission"
      - "Contrôle total"
      - "Mises à jour directes"
```

## Stores (Microsoft Store, Mac App Store)

```yaml
windows:
  microsoft_store:
    enabled: true
    commission: "15%"
    benefits:
      - "Visibilité accrue"
      - "Mises à jour automatiques"
      - "Confiance utilisateur"
    drawbacks:
      - "Commission"
      - "Process de validation"

macos:
  mac_app_store:
    enabled: false  # Sandboxing limitant
    alternative: "Direct download + notarization"
```

## Package managers (Homebrew, Chocolatey)

```bash
# Homebrew Cask (macOS)
brew install --cask mon-app

# Chocolatey (Windows)
choco install mon-app

# Flatpak (Linux)
flatpak install flathub com.example.monapp
```

```yaml
# Configuration pour package managers
package_managers:
  homebrew:
    cask_file: "mon-app.rb"
    formula: |
      cask 'mon-app' do
        version '1.2.0'
        sha256 '...'
        url "https://example.com/releases/mon-app-#{version}.dmg"
        name 'Mon App'
        homepage 'https://example.com'
      end
  
  chocolatey:
    nuspec_file: "mon-app.nuspec"
    install_script: "chocolateyinstall.ps1"
  
  flatpak:
    manifest: "com.example.monapp.yml"
    repository: "flathub"
```

## Résumé

- **Site direct** : Contrôle total, pas de commission
- **Stores** : Visibilité mais commission et restrictions
- **Package managers** : Pour utilisateurs techniques
- **Multi-canal** : Diversifier les canaux de distribution
