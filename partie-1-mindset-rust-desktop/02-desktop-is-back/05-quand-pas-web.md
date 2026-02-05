# 2.5 Quand Ne Pas Utiliser le Web

## Matrice de décision Desktop vs Web

```rust
/// Framework de décision pour choisir entre desktop et web
pub struct TechDecisionMatrix {
    pub criteria: Vec<Criterion>,
}

pub struct Criterion {
    pub name: &'static str,
    pub desktop_score: i8,  // -2 à +2
    pub web_score: i8,
}

impl TechDecisionMatrix {
    pub fn evaluate(requirements: &Requirements) -> Recommendation {
        let mut desktop_points = 0;
        let mut web_points = 0;
        
        // Critères favorisant le desktop
        if requirements.needs_offline { desktop_points += 3; }
        if requirements.sensitive_data { desktop_points += 3; }
        if requirements.heavy_computation { desktop_points += 2; }
        if requirements.local_file_access { desktop_points += 3; }
        if requirements.system_integration { desktop_points += 2; }
        if requirements.low_latency_required { desktop_points += 2; }
        
        // Critères favorisant le web
        if requirements.needs_collaboration { web_points += 3; }
        if requirements.frequent_updates { web_points += 2; }
        if requirements.cross_device_sync { web_points += 2; }
        if requirements.no_install_friction { web_points += 2; }
        if requirements.seo_important { web_points += 3; }
        
        if desktop_points > web_points + 3 {
            Recommendation::Desktop
        } else if web_points > desktop_points + 3 {
            Recommendation::Web
        } else {
            Recommendation::Hybrid  // Tauri, Electron
        }
    }
}
```

## Cas où le Web est préférable

### Collaboration temps réel

```rust
/// Scénarios de collaboration où le web excelle
pub enum CollaborationScenario {
    /// Édition simultanée (Google Docs style)
    RealtimeEditing {
        participants: u32,
        sync_latency_ms: u32,
    },
    
    /// Partage facile par lien
    LinkSharing {
        no_install_required: bool,
    },
    
    /// Commentaires et discussions
    ThreadedDiscussions {
        notifications: bool,
    },
}

// Pour ces cas, une app web est généralement meilleure
// car la collaboration nécessite un serveur central de toute façon
```

### Mises à jour fréquentes

```rust
/// Quand les mises à jour sont critiques
pub struct UpdateRequirements {
    /// Fréquence des mises à jour
    pub update_frequency: UpdateFrequency,
    
    /// Tous les utilisateurs doivent être sur la même version
    pub version_consistency_required: bool,
    
    /// Rollback rapide nécessaire
    pub instant_rollback_needed: bool,
}

impl UpdateRequirements {
    pub fn should_use_web(&self) -> bool {
        // Le web permet des mises à jour instantanées pour tous
        self.update_frequency == UpdateFrequency::Daily
            || self.version_consistency_required
            || self.instant_rollback_needed
    }
}
```

### Accessibilité multi-device sans friction

```rust
/// Cas d'usage mobile-first ou multi-device
pub struct MultiDeviceScenario {
    /// L'utilisateur change fréquemment d'appareil
    pub frequent_device_switching: bool,
    
    /// Accès depuis des appareils non-maîtrisés (cybercafé, etc.)
    pub untrusted_devices: bool,
    
    /// Mobile est le device principal
    pub mobile_first: bool,
}

impl MultiDeviceScenario {
    pub fn recommendation(&self) -> &str {
        if self.mobile_first {
            "Web responsive ou app mobile native"
        } else if self.untrusted_devices {
            "Web obligatoire (pas d'installation)"
        } else if self.frequent_device_switching {
            "Web avec PWA ou desktop avec sync"
        } else {
            "Desktop peut convenir"
        }
    }
}
```

## Cas où le Desktop est préférable

### Données sensibles et réglementées

```rust
/// Industries avec contraintes de données
pub enum RegulatedIndustry {
    /// Santé (HIPAA, données médicales)
    Healthcare {
        data_residency_required: bool,
        audit_trail_required: bool,
    },
    
    /// Finance (données bancaires, trading)
    Finance {
        offline_trading_needed: bool,
        latency_critical: bool,
    },
    
    /// Juridique (confidentialité client)
    Legal {
        client_confidentiality: bool,
    },
    
    /// Défense / Gouvernement
    Government {
        airgapped_environment: bool,
        clearance_required: bool,
    },
}

impl RegulatedIndustry {
    pub fn requires_local_data(&self) -> bool {
        // Ces industries bénéficient fortement du local
        match self {
            Self::Healthcare { data_residency_required, .. } => *data_residency_required,
            Self::Finance { offline_trading_needed, .. } => *offline_trading_needed,
            Self::Legal { client_confidentiality } => *client_confidentiality,
            Self::Government { airgapped_environment, .. } => *airgapped_environment,
        }
    }
}
```

### Performance critique

```rust
/// Applications nécessitant des performances élevées
pub struct PerformanceCriticalApp {
    pub category: &'static str,
    pub latency_requirement_ms: u32,
    pub data_volume: DataVolume,
}

const PERFORMANCE_CRITICAL_APPS: &[PerformanceCriticalApp] = &[
    PerformanceCriticalApp {
        category: "IDE / Éditeur de code",
        latency_requirement_ms: 16,  // 60 FPS
        data_volume: DataVolume::Large,  // Gros fichiers
    },
    PerformanceCriticalApp {
        category: "DAW (audio)",
        latency_requirement_ms: 5,   // Buffer audio
        data_volume: DataVolume::VeryLarge,
    },
    PerformanceCriticalApp {
        category: "3D / CAD",
        latency_requirement_ms: 16,
        data_volume: DataVolume::VeryLarge,
    },
    PerformanceCriticalApp {
        category: "Analyse de données",
        latency_requirement_ms: 100,
        data_volume: DataVolume::Massive,  // GB de données
    },
    PerformanceCriticalApp {
        category: "Jeux vidéo",
        latency_requirement_ms: 8,   // 120 FPS
        data_volume: DataVolume::Large,
    },
];
```

### Intégration système profonde

```rust
/// Fonctionnalités nécessitant un accès système
pub struct SystemIntegrationNeeds {
    pub needs: Vec<SystemNeed>,
}

pub enum SystemNeed {
    /// Accès au système de fichiers complet
    FullFilesystemAccess,
    
    /// Périphériques USB (scanners, imprimantes spéciales)
    UsbDevices,
    
    /// Ports série (équipements industriels)
    SerialPorts,
    
    /// Capture d'écran / enregistrement
    ScreenCapture,
    
    /// Raccourcis clavier globaux
    GlobalHotkeys,
    
    /// Tray icon / menu bar
    SystemTray,
    
    /// Démarrage automatique avec l'OS
    Autostart,
    
    /// VPN / réseau bas niveau
    NetworkStack,
}

impl SystemIntegrationNeeds {
    pub fn requires_desktop(&self) -> bool {
        // Ces besoins sont impossibles ou très limités dans le navigateur
        self.needs.iter().any(|n| matches!(n,
            SystemNeed::FullFilesystemAccess |
            SystemNeed::UsbDevices |
            SystemNeed::SerialPorts |
            SystemNeed::GlobalHotkeys |
            SystemNeed::Autostart |
            SystemNeed::NetworkStack
        ))
    }
}
```

## Arbre de décision simplifié

```rust
pub fn quick_decision(app: &AppRequirements) -> Platform {
    // Questions clés dans l'ordre
    
    // 1. Collaboration temps réel est-elle centrale ?
    if app.realtime_collaboration_is_core {
        return Platform::Web;
    }
    
    // 2. Doit fonctionner hors ligne de manière fiable ?
    if app.must_work_offline_reliably {
        return Platform::Desktop;
    }
    
    // 3. Données sensibles/réglementées ?
    if app.handles_sensitive_data {
        return Platform::Desktop;
    }
    
    // 4. Performance critique (< 100ms latency) ?
    if app.latency_critical {
        return Platform::Desktop;
    }
    
    // 5. Intégration système (USB, fichiers, etc.) ?
    if app.needs_system_integration {
        return Platform::Desktop;
    }
    
    // 6. Besoin d'être indexé par Google ?
    if app.seo_important {
        return Platform::Web;
    }
    
    // 7. Mises à jour quotidiennes ?
    if app.daily_updates {
        return Platform::Web;
    }
    
    // Par défaut, si aucun critère fort
    Platform::WebWithPWA
}
```

## Tableau récapitulatif

| Critère | Desktop | Web |
|---------|---------|-----|
| Offline | ✅ Natif | ⚠️ PWA limité |
| Données sensibles | ✅ Local | ❌ Serveur tiers |
| Performance | ✅ Native | ⚠️ JS/WASM |
| Collaboration | ⚠️ Complexe | ✅ Natif |
| Mises à jour | ⚠️ Distribution | ✅ Instantané |
| SEO | ❌ N/A | ✅ Indexable |
| Intégration OS | ✅ Complète | ❌ Sandbox |
| Installation | ⚠️ Friction | ✅ Aucune |

## Conclusion

Choisissez le **Desktop** quand :
- Offline est critique
- Données sensibles
- Performance exigeante
- Intégration système nécessaire
- Usage professionnel intensif (8h/jour)

Choisissez le **Web** quand :
- Collaboration temps réel centrale
- Mises à jour très fréquentes
- SEO important
- Accès depuis n'importe quel device
- Friction d'installation inacceptable
