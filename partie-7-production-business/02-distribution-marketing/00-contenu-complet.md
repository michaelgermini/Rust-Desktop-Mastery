# Chapitre 30 : Distribution et Marketing

## Introduction

Une excellente application ne vaut rien si personne ne la connaît. La distribution et le marketing sont essentiels pour le succès commercial.

---

## 30.1 Site Web Produit

### Structure Recommandée

```
site-web/
├── index.html          # Landing page
├── features/           # Pages de fonctionnalités
├── pricing/            # Tarification
├── docs/               # Documentation
├── blog/               # Articles SEO
├── download/           # Page de téléchargement
└── support/            # FAQ, Contact
```

### Landing Page Efficace

```html
<!DOCTYPE html>
<html lang="fr">
<head>
    <title>Mon App - Solution de gestion pour PME</title>
    <meta name="description" content="Gérez vos clients et factures 
        en toute souveraineté avec notre logiciel desktop.">
    
    <!-- Open Graph pour partage social -->
    <meta property="og:title" content="Mon App">
    <meta property="og:description" content="Solution de gestion...">
    <meta property="og:image" content="https://example.com/preview.png">
</head>
<body>
    <!-- Hero Section -->
    <section class="hero">
        <h1>Gérez votre entreprise sans dépendre du cloud</h1>
        <p>Solution desktop complète : clients, factures, devis</p>
        <a href="/download" class="cta-primary">Télécharger gratuitement</a>
        <a href="/demo" class="cta-secondary">Voir la démo</a>
    </section>
    
    <!-- Social Proof -->
    <section class="social-proof">
        <p>Utilisé par plus de 5 000 entreprises</p>
        <div class="logos"><!-- Logos clients --></div>
    </section>
    
    <!-- Features -->
    <section class="features">
        <h2>Pourquoi choisir Mon App ?</h2>
        <!-- Liste des avantages -->
    </section>
    
    <!-- Pricing -->
    <section class="pricing">
        <h2>Tarification simple</h2>
        <!-- Grille de prix -->
    </section>
    
    <!-- FAQ -->
    <section class="faq">
        <h2>Questions fréquentes</h2>
        <!-- Accordéon FAQ -->
    </section>
</body>
</html>
```

---

## 30.2 SEO pour Applications Desktop

### Mots-clés Stratégiques

```rust
pub const SEO_KEYWORDS: &[&str] = &[
    // Termes génériques
    "logiciel de gestion",
    "application desktop",
    "logiciel facturation",
    
    // Long tail (conversion élevée)
    "logiciel facturation auto-entrepreneur",
    "gestion clients offline",
    "logiciel devis facture gratuit",
    
    // Problèmes
    "remplacer excel gestion clients",
    "alternative cloud facturation",
    
    // Comparaisons
    "alternatif sage facturation",
    "logiciel facturation vs cloud",
];
```

### Structure d'Article Blog

```markdown
# Comment choisir un logiciel de facturation pour auto-entrepreneur

## Introduction (problème)
En tant qu'auto-entrepreneur, vous avez besoin d'un outil simple...

## Critères de choix
1. Facilité d'utilisation
2. Conformité légale
3. Prix

## Comparatif des solutions
| Solution | Prix | Hors-ligne | Note |
|----------|------|------------|------|
| Mon App  | 49€  | ✅         | ⭐⭐⭐⭐⭐ |
| Solution B | Abo | ❌       | ⭐⭐⭐ |

## Conclusion
Pour une solution souveraine et efficace, Mon App...

[Télécharger gratuitement](/download)
```

---

## 30.3 Page de Téléchargement

```rust
/// Détecter l'OS du visiteur
pub fn detect_os_from_user_agent(ua: &str) -> DetectedOS {
    if ua.contains("Windows") {
        DetectedOS::Windows
    } else if ua.contains("Mac OS") {
        DetectedOS::MacOS
    } else if ua.contains("Linux") {
        DetectedOS::Linux
    } else {
        DetectedOS::Unknown
    }
}

/// Page de téléchargement
pub struct DownloadPage {
    pub detected_os: DetectedOS,
    pub downloads: HashMap<DetectedOS, DownloadInfo>,
}

pub struct DownloadInfo {
    pub url: String,
    pub filename: String,
    pub size_mb: f32,
    pub version: String,
    pub checksum: String,
}
```

```html
<section class="download-hero">
    <h1>Télécharger Mon App</h1>
    <p>Version 1.2.0 - 15 MB</p>
    
    <!-- Bouton principal basé sur l'OS détecté -->
    <a href="/downloads/mon-app-1.2.0.msi" 
       class="download-button primary"
       data-os="windows">
        ⬇️ Télécharger pour Windows
    </a>
    
    <!-- Autres plateformes -->
    <details class="other-platforms">
        <summary>Autres plateformes</summary>
        <ul>
            <li><a href="/downloads/mon-app-1.2.0.dmg">macOS (.dmg)</a></li>
            <li><a href="/downloads/mon-app-1.2.0.AppImage">Linux (.AppImage)</a></li>
            <li><a href="/downloads/mon-app-1.2.0.deb">Debian/Ubuntu (.deb)</a></li>
        </ul>
    </details>
</section>

<section class="install-instructions">
    <h2>Installation</h2>
    <div class="tabs">
        <button data-tab="windows">Windows</button>
        <button data-tab="macos">macOS</button>
        <button data-tab="linux">Linux</button>
    </div>
    <!-- Instructions par OS -->
</section>
```

---

## 30.4 Canaux de Distribution

### Stores et Marketplaces

```yaml
# Distribution multi-canal
distribution:
  direct:
    - website: "https://example.com/download"
    - github_releases: true
  
  windows:
    - microsoft_store: true  # Visibilité, mais commission 15%
    - chocolatey: true       # Geeks/entreprises
  
  macos:
    - mac_app_store: false   # Sandboxing limitant
    - homebrew_cask: true    # Développeurs
  
  linux:
    - flathub: true          # Universal
    - snapcraft: true        # Ubuntu
    - aur: true              # Arch Linux
```

---

## Résumé

- **Landing page** : Hero + Social proof + Features + CTA clair
- **SEO** : Articles blog ciblés, mots-clés long tail
- **Téléchargement** : Détection OS, instructions claires
- **Multi-canal** : Site + Stores + Package managers
