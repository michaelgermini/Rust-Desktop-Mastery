# 30.3 Page de Téléchargement

## Détection automatique de l'OS

```rust
/// Détecter l'OS du visiteur depuis le User-Agent
pub fn detect_os_from_user_agent(ua: &str) -> DetectedOS {
    if ua.contains("Windows") {
        DetectedOS::Windows
    } else if ua.contains("Mac OS") || ua.contains("macOS") {
        DetectedOS::MacOS
    } else if ua.contains("Linux") {
        DetectedOS::Linux
    } else {
        DetectedOS::Unknown
    }
}

#[derive(Clone, PartialEq)]
pub enum DetectedOS {
    Windows,
    MacOS,
    Linux,
    Unknown,
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
    pub release_date: DateTime<Utc>,
}
```

## Instructions d'installation

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
        <button data-tab="windows" class="active">Windows</button>
        <button data-tab="macos">macOS</button>
        <button data-tab="linux">Linux</button>
    </div>
    
    <div id="windows-instructions" class="tab-content active">
        <ol>
            <li>Téléchargez le fichier .msi</li>
            <li>Double-cliquez sur le fichier téléchargé</li>
            <li>Suivez les instructions de l'installateur</li>
            <li>Lancez Mon App depuis le menu Démarrer</li>
        </ol>
    </div>
    
    <div id="macos-instructions" class="tab-content">
        <ol>
            <li>Téléchargez le fichier .dmg</li>
            <li>Ouvrez le fichier .dmg</li>
            <li>Glissez Mon App dans le dossier Applications</li>
            <li>Lancez Mon App depuis Applications</li>
        </ol>
        <p class="warning">
            ⚠️ Si macOS bloque l'ouverture, faites un clic droit > Ouvrir
        </p>
    </div>
    
    <div id="linux-instructions" class="tab-content">
        <h3>AppImage</h3>
        <pre><code>chmod +x mon-app-1.2.0.AppImage
./mon-app-1.2.0.AppImage</code></pre>
        
        <h3>Debian/Ubuntu</h3>
        <pre><code>sudo dpkg -i mon-app-1.2.0.deb
sudo apt-get install -f</code></pre>
    </div>
</section>
```

## Checksums

```html
<section class="checksums">
    <h2>Vérification d'intégrité</h2>
    <p>Vérifiez que votre téléchargement n'a pas été corrompu :</p>
    
    <details>
        <summary>Windows (SHA256)</summary>
        <code>a1b2c3d4e5f6...</code>
    </details>
    
    <details>
        <summary>macOS (SHA256)</summary>
        <code>f6e5d4c3b2a1...</code>
    </details>
    
    <details>
        <summary>Linux (SHA256)</summary>
        <code>1234567890ab...</code>
    </details>
</section>
```

## Résumé

- **Détection OS** : Afficher le bon téléchargement automatiquement
- **Instructions** : Guide clair pour chaque plateforme
- **Checksums** : Vérification d'intégrité des fichiers
- **Multi-plateforme** : Support Windows, macOS, Linux
