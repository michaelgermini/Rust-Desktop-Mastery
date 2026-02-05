# 24.4 Packaging

## Build pour toutes les plateformes

### Windows

```toml
# Cargo.toml
[package]
name = "notevault"
version = "1.0.0"

[target.'cfg(windows)'.dependencies]
winapi = { version = "0.3", features = ["winuser"] }
```

```bash
# Build pour Windows
cargo build --release --target x86_64-pc-windows-msvc

# Créer un installer MSI
cargo wix --nocapture
```

### macOS

```bash
# Build pour macOS (Intel)
cargo build --release --target x86_64-apple-darwin

# Build pour macOS (Apple Silicon)
cargo build --release --target aarch64-apple-darwin

# Universal binary
lipo -create \
    target/x86_64-apple-darwin/release/notevault \
    target/aarch64-apple-darwin/release/notevault \
    -output notevault-universal

# Créer un DMG
create-dmg --volname "NoteVault" \
    --window-pos 200 120 \
    --window-size 800 400 \
    --icon-size 100 \
    --app-drop-link 600 185 \
    "NoteVault.dmg" \
    "target/release/"
```

### Linux

```bash
# Build pour Linux
cargo build --release --target x86_64-unknown-linux-gnu

# Créer un AppImage
appimagetool target/release/notevault.AppDir notevault.AppImage

# Créer un .deb
cargo deb
```

## Installers

### Windows MSI avec cargo-wix

```toml
# wix/main.wxs
<?xml version="1.0" encoding="UTF-8"?>
<Wix xmlns="http://schemas.microsoft.com/wix/2006/wi">
    <Product Id="*" Name="NoteVault" Language="1033" Version="1.0.0"
             Manufacturer="My Company" UpgradeCode="...">
        <Package InstallerVersion="200" Compressed="yes" />
        <MediaTemplate />
        
        <Feature Id="ProductFeature" Title="NoteVault" Level="1">
            <ComponentRef Id="ApplicationFiles" />
        </Feature>
    </Product>
    
    <Fragment>
        <Directory Id="TARGETDIR" Name="SourceDir">
            <Directory Id="ProgramFilesFolder">
                <Directory Id="INSTALLFOLDER" Name="NoteVault" />
            </Directory>
        </Directory>
    </Fragment>
    
    <Fragment>
        <ComponentGroup Id="ProductComponents" Directory="INSTALLFOLDER">
            <Component Id="ApplicationFiles">
                <File Id="NoteVault.exe" Source="target/release/notevault.exe" />
            </Component>
        </ComponentGroup>
    </Fragment>
</Wix>
```

### macOS DMG

```bash
# Script de création DMG
#!/bin/bash
APP_NAME="NoteVault"
DMG_NAME="${APP_NAME}.dmg"
APP_PATH="target/release/${APP_NAME}.app"

# Créer le DMG
hdiutil create -volname "${APP_NAME}" \
    -srcfolder "${APP_PATH}" \
    -ov -format UDZO \
    "${DMG_NAME}"

# Codesign (optionnel mais recommandé)
codesign --deep --force --verify --verbose \
    --sign "Developer ID Application: Your Name" \
    "${DMG_NAME}"

# Notarize (pour distribution hors App Store)
xcrun notarytool submit "${DMG_NAME}" \
    --apple-id "your@email.com" \
    --team-id "YOUR_TEAM_ID" \
    --password "app-specific-password"
```

## Distribution

### GitHub Releases

```yaml
# .github/workflows/release.yml
name: Release

on:
  push:
    tags:
      - 'v*'

jobs:
  build:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
    
    steps:
      - uses: actions/checkout@v3
      - uses: actions-rs/toolchain@v1
      
      - name: Build
        run: cargo build --release
      
      - name: Package
        run: |
          # Créer l'installer selon l'OS
          # ...
      
      - name: Upload
        uses: softprops/action-gh-release@v1
        with:
          files: |
            notevault-*.msi
            notevault-*.dmg
            notevault-*.AppImage
```

## Résumé

- **Multi-plateforme** : Windows, macOS, Linux
- **Installers** : MSI, DMG, AppImage, deb
- **Distribution** : GitHub Releases, site web
- **Codesign/Notarize** : Pour macOS (recommandé)
