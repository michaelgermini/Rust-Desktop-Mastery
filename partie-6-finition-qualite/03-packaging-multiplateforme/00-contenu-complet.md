# Chapitre 27 : Packaging Multiplateforme

## Introduction

Distribuer une application desktop nécessite de créer des installers natifs pour chaque plateforme.

---

## 27.1 Configuration Cargo

```toml
# Cargo.toml
[package]
name = "mon-app"
version = "1.0.0"
edition = "2021"
description = "Mon application desktop"
authors = ["Votre Nom <email@example.com>"]
license = "MIT"

[package.metadata.bundle]
name = "Mon Application"
identifier = "com.example.monapp"
icon = ["assets/icon.icns", "assets/icon.ico", "assets/icon.png"]
copyright = "Copyright © 2024"
category = "Productivity"
short_description = "Description courte"
long_description = "Description longue de l'application"

[profile.release]
lto = true
strip = true
panic = "abort"
codegen-units = 1
opt-level = "z"  # Optimiser pour la taille
```

---

## 27.2 Windows

```powershell
# scripts/build-windows.ps1

# Compiler
cargo build --release --target x86_64-pc-windows-msvc

# Créer l'installer avec NSIS ou WiX
# Option 1: cargo-wix
cargo install cargo-wix
cargo wix init
cargo wix

# Le résultat est dans target/wix/mon-app-1.0.0-x86_64.msi
```

```xml
<!-- wix/main.wxs - Configuration WiX -->
<?xml version="1.0" encoding="UTF-8"?>
<Wix xmlns="http://schemas.microsoft.com/wix/2006/wi">
    <Product Name="Mon Application" 
             Manufacturer="Votre Société"
             Version="1.0.0"
             UpgradeCode="GUID-HERE">
        
        <Package InstallerVersion="500" 
                 Compressed="yes" 
                 InstallScope="perUser" />
        
        <MajorUpgrade DowngradeErrorMessage="Une version plus récente est installée." />
        
        <Feature Id="ProductFeature" Title="Mon Application" Level="1">
            <ComponentGroupRef Id="ProductComponents" />
        </Feature>
    </Product>
</Wix>
```

---

## 27.3 macOS

```bash
#!/bin/bash
# scripts/build-macos.sh

# Compiler pour les deux architectures
cargo build --release --target x86_64-apple-darwin
cargo build --release --target aarch64-apple-darwin

# Créer un universal binary
mkdir -p target/universal-apple-darwin/release
lipo -create \
    target/x86_64-apple-darwin/release/mon-app \
    target/aarch64-apple-darwin/release/mon-app \
    -output target/universal-apple-darwin/release/mon-app

# Créer le .app bundle
APP_NAME="Mon Application.app"
mkdir -p "$APP_NAME/Contents/MacOS"
mkdir -p "$APP_NAME/Contents/Resources"

cp target/universal-apple-darwin/release/mon-app "$APP_NAME/Contents/MacOS/"
cp assets/icon.icns "$APP_NAME/Contents/Resources/"
cp macos/Info.plist "$APP_NAME/Contents/"

# Signer (nécessite un certificat Apple Developer)
codesign --force --deep --sign "Developer ID Application: Votre Nom" "$APP_NAME"

# Créer le DMG
create-dmg \
    --volname "Mon Application" \
    --window-pos 200 120 \
    --window-size 600 400 \
    --icon-size 100 \
    --icon "Mon Application.app" 175 120 \
    --app-drop-link 425 120 \
    "Mon Application.dmg" \
    "$APP_NAME"

# Notariser
xcrun notarytool submit "Mon Application.dmg" \
    --apple-id "$APPLE_ID" \
    --password "$APP_SPECIFIC_PASSWORD" \
    --team-id "$TEAM_ID" \
    --wait

xcrun stapler staple "Mon Application.dmg"
```

---

## 27.4 Linux

```bash
#!/bin/bash
# scripts/build-linux.sh

# Compiler
cargo build --release --target x86_64-unknown-linux-gnu

# AppImage
cargo install cargo-appimage
cargo appimage

# Debian package
cargo install cargo-deb
cargo deb

# Flatpak
flatpak-builder --force-clean build-dir com.example.monapp.yml
flatpak build-export repo build-dir
flatpak build-bundle repo mon-app.flatpak com.example.monapp
```

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

---

## 27.5 Auto-Update

```rust
use self_update::backends::github::Update;

pub fn check_for_updates() -> Result<Option<String>> {
    let status = Update::configure()
        .repo_owner("votre-org")
        .repo_name("mon-app")
        .bin_name("mon-app")
        .show_download_progress(true)
        .current_version(env!("CARGO_PKG_VERSION"))
        .build()?
        .update()?;
    
    if status.updated() {
        Ok(Some(status.version().to_string()))
    } else {
        Ok(None)
    }
}

// Vérification au démarrage
fn main() {
    // Vérifier les mises à jour en arrière-plan
    std::thread::spawn(|| {
        if let Ok(Some(version)) = check_for_updates() {
            println!("Mise à jour vers {} disponible", version);
        }
    });
    
    // Lancer l'application
    run_app();
}
```

---

## Résumé

| Plateforme | Format | Outil |
|------------|--------|-------|
| Windows | .msi | cargo-wix |
| macOS | .dmg | create-dmg |
| Linux | .AppImage, .deb, .flatpak | cargo-appimage, cargo-deb |

- **Signature** : Obligatoire pour distribution
- **Notarization** : Requis sur macOS
- **Auto-update** : self_update crate
