# 27.3 Linux

## AppImage

```bash
# Installer cargo-appimage
cargo install cargo-appimage

# Créer l'AppImage
cargo appimage

# Le résultat est dans target/appimage/mon-app-x86_64.AppImage
```

```toml
# Cargo.toml
[package.metadata.appimage]
name = "mon-app"
description = "Mon application desktop"
icon = "assets/icon.png"
```

## Debian package (.deb)

```bash
# Installer cargo-deb
cargo install cargo-deb

# Créer le package .deb
cargo deb

# Le résultat est dans target/debian/mon-app_1.0.0_amd64.deb
```

```toml
# Cargo.toml
[package.metadata.deb]
maintainer = "Votre Nom <email@example.com>"
copyright = "Copyright © 2024"
license-file = ["LICENSE", "0"]
extended-description = """\
Description longue de l'application.
"""
depends = "$auto"
section = "utils"
priority = "optional"
```

## Flatpak

```yaml
# com.example.monapp.yml - Flatpak manifest
app-id: com.example.monapp
runtime: org.freedesktop.Platform
runtime-version: '23.08'
sdk: org.freedesktop.Sdk
command: mon-app
finish-args:
  - --share=ipc
  - --socket=x11
  - --socket=wayland
  - --filesystem=home
modules:
  - name: mon-app
    buildsystem: simple
    build-commands:
      - install -D mon-app /app/bin/mon-app
    sources:
      - type: file
        path: target/release/mon-app
```

```bash
# Construire le Flatpak
flatpak-builder --force-clean build-dir com.example.monapp.yml

# Exporter vers un repo
flatpak build-export repo build-dir

# Créer le bundle
flatpak build-bundle repo mon-app.flatpak com.example.monapp
```

## Résumé

- **AppImage** : Format portable, pas d'installation requise
- **Debian package** : Pour distributions basées sur Debian
- **Flatpak** : Format universel avec sandboxing
- **Choix** : Selon la cible de distribution
