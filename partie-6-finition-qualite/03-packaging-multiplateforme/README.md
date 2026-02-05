# Chapitre 27 : Packaging Multiplateforme

Ce chapitre couvre le packaging et la distribution pour Windows, macOS et Linux.

## Sous-sections

1. **[Windows](./01-windows.md)**
   - MSI avec cargo-wix
   - NSIS
   - Signature de code

2. **[macOS](./02-macos.md)**
   - App bundle
   - DMG avec create-dmg
   - Universal binary (Intel + Apple Silicon)
   - Signature et notarization

3. **[Linux](./03-linux.md)**
   - AppImage
   - Debian package (.deb)
   - Flatpak

4. **[Installers](./04-installers.md)**
   - Expérience d'installation
   - Désinstallation propre
   - Mises à jour

5. **[Mises à jour automatiques](./05-mises-a-jour.md)**
   - Système d'auto-update
   - Backend GitHub
   - Notifications utilisateur

## Objectifs d'apprentissage

À la fin de ce chapitre, vous serez capable de :

- Créer des installers pour toutes les plateformes
- Signer et notariser vos applications
- Implémenter l'auto-update
