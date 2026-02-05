# Annexe D : Checklist de Lancement

Todo-list complète avant de lancer une application desktop.

---

## 1. Code et Qualité

### Tests

- [ ] Tests unitaires passent (`cargo test`)
- [ ] Tests d'intégration passent
- [ ] Couverture de code acceptable (> 60%)
- [ ] Tests manuels sur chaque plateforme cible

### Qualité de Code

- [ ] `cargo fmt` appliqué
- [ ] `cargo clippy` sans warnings
- [ ] Pas de `TODO` ou `FIXME` bloquants
- [ ] Documentation Rust (`cargo doc`)
- [ ] Pas de `unwrap()` en production
- [ ] Gestion d'erreurs complète

### Sécurité

- [ ] Pas de secrets dans le code
- [ ] Dépendances auditées (`cargo audit`)
- [ ] Données sensibles chiffrées
- [ ] Validation des entrées utilisateur

---

## 2. Fonctionnalités Essentielles

### UX Minimum

- [ ] Premier lancement guidé (onboarding)
- [ ] États de chargement (skeleton, spinner)
- [ ] Messages d'erreur compréhensibles
- [ ] Confirmation avant actions destructrices
- [ ] Undo/Redo pour actions critiques

### Persistance

- [ ] Sauvegarde automatique
- [ ] Backup des données utilisateur
- [ ] Migration de données (versions précédentes)
- [ ] Export des données (portabilité)

### Performance

- [ ] Démarrage < 2 secondes
- [ ] Pas de freeze UI
- [ ] Mémoire raisonnable (< 200MB idle)

---

## 3. Packaging

### Windows

- [ ] Binaire compilé (x86_64)
- [ ] Icône d'application
- [ ] Installer MSI ou NSIS
- [ ] Signature du code (optionnel mais recommandé)
- [ ] Test sur Windows 10 et 11

### macOS

- [ ] Universal binary (Intel + Apple Silicon)
- [ ] App bundle (.app)
- [ ] DMG pour distribution
- [ ] Signature Developer ID
- [ ] Notarization Apple
- [ ] Test sur macOS récent

### Linux

- [ ] AppImage
- [ ] Paquet .deb (Debian/Ubuntu)
- [ ] Test sur Ubuntu LTS

---

## 4. Distribution

### Site Web

- [ ] Landing page
- [ ] Page de téléchargement
- [ ] Documentation utilisateur
- [ ] FAQ
- [ ] Politique de confidentialité
- [ ] Conditions d'utilisation

### Versioning

- [ ] Version définie (semver)
- [ ] CHANGELOG.md à jour
- [ ] Tag Git pour la release

### Téléchargement

- [ ] URLs de téléchargement fonctionnelles
- [ ] Checksums publiés (SHA256)
- [ ] Détection automatique de l'OS

---

## 5. Post-Lancement

### Monitoring

- [ ] Télémétrie (opt-in) configurée
- [ ] Crash reporting actif
- [ ] Alertes configurées

### Support

- [ ] Canal de support (email, Discord, forum)
- [ ] Feedback widget fonctionnel
- [ ] Process de traitement des bugs

### Mises à Jour

- [ ] Auto-update configuré et testé
- [ ] Notification de nouvelle version
- [ ] Rollback possible

---

## 6. Marketing

### Présence en Ligne

- [ ] Description claire du produit
- [ ] Screenshots de qualité
- [ ] Vidéo de démo (optionnel)
- [ ] Comparaison avec concurrents

### SEO

- [ ] Meta tags corrects
- [ ] Open Graph pour réseaux sociaux
- [ ] Schema.org pour rich snippets

### Réseaux Sociaux

- [ ] Annonce préparée
- [ ] Thread Twitter/X
- [ ] Post LinkedIn
- [ ] Post sur Reddit/HackerNews (si approprié)

---

## 7. Légal

### Documentation Légale

- [ ] Licence logicielle définie
- [ ] EULA si nécessaire
- [ ] Politique de confidentialité
- [ ] Conformité RGPD (si EU)

### Propriété Intellectuelle

- [ ] Nom de marque vérifié
- [ ] Logo original
- [ ] Assets tiers sous licence correcte

---

## Template de Release Notes

```markdown
# v1.0.0 - Lancement Initial

Date: YYYY-MM-DD

## Nouveautés

- Gestion des clients
- Création de factures
- Export PDF
- Thème sombre/clair

## Corrections

- N/A (première version)

## Notes

- Configuration système requise: Windows 10+, macOS 12+, Ubuntu 22.04+
- Cette version requiert une nouvelle installation

## Téléchargement

- [Windows (MSI)](https://...)
- [macOS (DMG)](https://...)
- [Linux (AppImage)](https://...)

Checksums SHA256:
- mon-app-1.0.0.msi: abc123...
- mon-app-1.0.0.dmg: def456...
- mon-app-1.0.0.AppImage: ghi789...
```

---

## Script de Release

```bash
#!/bin/bash
# scripts/release.sh

VERSION=$1

if [ -z "$VERSION" ]; then
    echo "Usage: ./release.sh 1.0.0"
    exit 1
fi

echo "🚀 Releasing version $VERSION"

# Vérifications
echo "📋 Running checks..."
cargo test || exit 1
cargo clippy -- -D warnings || exit 1
cargo fmt --check || exit 1

# Build
echo "🔨 Building..."
cargo build --release

# Tag
echo "🏷️ Creating tag..."
git tag -a "v$VERSION" -m "Release v$VERSION"

# Package (à adapter selon vos scripts)
echo "📦 Packaging..."
./scripts/build-windows.ps1
./scripts/build-macos.sh
./scripts/build-linux.sh

echo "✅ Release $VERSION ready!"
echo "Next steps:"
echo "  1. Push tag: git push origin v$VERSION"
echo "  2. Upload packages to GitHub Releases"
echo "  3. Update website download page"
echo "  4. Announce on social media"
```
