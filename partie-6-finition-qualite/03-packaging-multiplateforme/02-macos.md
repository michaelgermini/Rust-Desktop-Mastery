# 27.2 macOS

## App bundle

```bash
#!/bin/bash
# scripts/build-macos.sh

APP_NAME="Mon Application.app"
BINARY_NAME="mon-app"

# Créer la structure du .app bundle
mkdir -p "${APP_NAME}/Contents/MacOS"
mkdir -p "${APP_NAME}/Contents/Resources"

# Copier le binaire
cp "target/release/${BINARY_NAME}" "${APP_NAME}/Contents/MacOS/"

# Copier l'icône
cp "assets/icon.icns" "${APP_NAME}/Contents/Resources/"

# Créer Info.plist
cat > "${APP_NAME}/Contents/Info.plist" << EOF
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>CFBundleExecutable</key>
    <string>${BINARY_NAME}</string>
    <key>CFBundleIdentifier</key>
    <string>com.example.monapp</string>
    <key>CFBundleName</key>
    <string>Mon Application</string>
    <key>CFBundleVersion</key>
    <string>1.0.0</string>
    <key>CFBundleShortVersionString</key>
    <string>1.0.0</string>
    <key>CFBundleIconFile</key>
    <string>icon</string>
</dict>
</plist>
EOF
```

## DMG avec create-dmg

```bash
# Installer create-dmg
brew install create-dmg

# Créer le DMG
create-dmg \
    --volname "Mon Application" \
    --window-pos 200 120 \
    --window-size 600 400 \
    --icon-size 100 \
    --icon "Mon Application.app" 175 120 \
    --app-drop-link 425 120 \
    "Mon Application.dmg" \
    "Mon Application.app"
```

## Universal binary (Intel + Apple Silicon)

```bash
# Compiler pour les deux architectures
cargo build --release --target x86_64-apple-darwin
cargo build --release --target aarch64-apple-darwin

# Créer un universal binary
mkdir -p target/universal-apple-darwin/release
lipo -create \
    target/x86_64-apple-darwin/release/mon-app \
    target/aarch64-apple-darwin/release/mon-app \
    -output target/universal-apple-darwin/release/mon-app

# Utiliser le universal binary dans le .app bundle
cp target/universal-apple-darwin/release/mon-app "${APP_NAME}/Contents/MacOS/"
```

## Signature et notarization

```bash
# Signer le .app bundle
codesign --force --deep --sign "Developer ID Application: Votre Nom" \
    --options runtime \
    --entitlements entitlements.plist \
    "Mon Application.app"

# Vérifier la signature
codesign --verify --verbose "Mon Application.app"

# Notariser (nécessite Apple Developer account)
xcrun notarytool submit "Mon Application.dmg" \
    --apple-id "$APPLE_ID" \
    --password "$APP_SPECIFIC_PASSWORD" \
    --team-id "$TEAM_ID" \
    --wait

# Stapler (attacher le ticket de notarisation)
xcrun stapler staple "Mon Application.dmg"
```

## Résumé

- **App bundle** : Structure .app avec Info.plist
- **DMG** : Format de distribution macOS
- **Universal binary** : Support Intel et Apple Silicon
- **Signature/Notarization** : Obligatoire pour distribution hors App Store
