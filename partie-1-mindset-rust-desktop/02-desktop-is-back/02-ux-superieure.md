# 2.2 UX Supérieure au Navigateur

## Limitations inhérentes du navigateur

### Le navigateur comme couche d'indirection

```rust
/// Couches entre l'utilisateur et l'application web
pub struct WebAppStack {
    pub layers: Vec<Layer>,
}

impl Default for WebAppStack {
    fn default() -> Self {
        Self {
            layers: vec![
                Layer { name: "Application JavaScript", overhead_ms: 0 },
                Layer { name: "Framework UI (React/Vue)", overhead_ms: 5 },
                Layer { name: "Virtual DOM", overhead_ms: 3 },
                Layer { name: "DOM réel", overhead_ms: 5 },
                Layer { name: "Moteur de rendu (Blink/WebKit)", overhead_ms: 5 },
                Layer { name: "Compositor", overhead_ms: 2 },
                Layer { name: "GPU abstraction", overhead_ms: 1 },
                // Total: ~20ms de latence structurelle
            ],
        }
    }
}

/// Application native Rust
pub struct NativeAppStack {
    pub layers: Vec<Layer>,
}

impl Default for NativeAppStack {
    fn default() -> Self {
        Self {
            layers: vec![
                Layer { name: "Application Rust", overhead_ms: 0 },
                Layer { name: "egui layout", overhead_ms: 1 },
                Layer { name: "GPU direct (wgpu)", overhead_ms: 1 },
                // Total: ~2ms
            ],
        }
    }
}
```

### Sandbox et restrictions

```rust
/// Ce que le navigateur empêche de faire
pub struct BrowserRestrictions {
    pub restrictions: Vec<Restriction>,
}

impl Default for BrowserRestrictions {
    fn default() -> Self {
        Self {
            restrictions: vec![
                Restriction {
                    capability: "Accès au système de fichiers",
                    browser: "Limité à des dossiers autorisés un par un",
                    native: "Accès complet",
                },
                Restriction {
                    capability: "Raccourcis clavier système",
                    browser: "Conflits avec raccourcis navigateur",
                    native: "Contrôle total",
                },
                Restriction {
                    capability: "Notifications système",
                    browser: "Demande de permission, souvent bloqué",
                    native: "Intégration native",
                },
                Restriction {
                    capability: "Tray icon / Menu bar",
                    browser: "Impossible",
                    native: "Natif",
                },
                Restriction {
                    capability: "Intégration OS (drag & drop, etc.)",
                    browser: "Limité",
                    native: "Complet",
                },
                Restriction {
                    capability: "Accès hardware (USB, Bluetooth)",
                    browser: "APIs expérimentales limitées",
                    native: "Complet",
                },
            ],
        }
    }
}
```

## Avantages UX du natif

### Raccourcis clavier sans conflit

```rust
use egui::Key;

pub struct KeyboardShortcuts {
    shortcuts: Vec<Shortcut>,
}

impl KeyboardShortcuts {
    pub fn native_advantages() -> Vec<&'static str> {
        vec![
            // Ces raccourcis sont impossibles ou conflictuels dans le navigateur
            "Ctrl+W : Fermer le document (vs fermer l'onglet)",
            "Ctrl+N : Nouveau document (vs nouvelle fenêtre)",
            "Ctrl+T : Nouvelle tâche (vs nouvel onglet)",
            "Ctrl+Tab : Changer de vue (vs changer d'onglet)",
            "F1-F12 : Actions personnalisées (vs fonctions navigateur)",
            "Ctrl+Shift+P : Command palette (vs mode privé)",
        ]
    }
}
```

### Intégration système native

```rust
/// Fonctionnalités d'intégration système
pub struct SystemIntegration {
    pub features: Vec<SystemFeature>,
}

impl SystemIntegration {
    pub fn desktop_features() -> Self {
        Self {
            features: vec![
                SystemFeature {
                    name: "System Tray",
                    description: "Icône dans la barre système, toujours accessible",
                    code_example: r#"
                        let tray = TrayIcon::new()
                            .with_icon(icon)
                            .with_menu(menu)
                            .with_tooltip("Mon App");
                    "#,
                },
                SystemFeature {
                    name: "Notifications natives",
                    description: "Notifications système intégrées",
                    code_example: r#"
                        Notification::new()
                            .title("Rappel")
                            .body("Votre tâche est due")
                            .show()?;
                    "#,
                },
                SystemFeature {
                    name: "Drag & Drop depuis l'OS",
                    description: "Glisser des fichiers depuis l'explorateur",
                    code_example: r#"
                        if let Some(dropped) = ui.input(|i| i.raw.dropped_files.clone()) {
                            for file in dropped {
                                process_file(&file.path);
                            }
                        }
                    "#,
                },
                SystemFeature {
                    name: "Association de fichiers",
                    description: "Double-clic sur .myapp ouvre l'application",
                    code_example: r#"
                        // Enregistrement Windows
                        register_file_association(".invoice", "MonApp.Invoice");
                    "#,
                },
            ],
        }
    }
}
```

### Menus contextuels riches

```rust
pub fn show_context_menu(ui: &mut Ui, item: &Item) {
    egui::menu::context_menu(ui, |ui| {
        if ui.button("📋 Copier").clicked() {
            // Copier dans le presse-papier système
            clipboard::set_text(&item.to_string());
            ui.close_menu();
        }
        
        if ui.button("📁 Ouvrir l'emplacement").clicked() {
            // Ouvrir l'explorateur de fichiers
            open::that(item.path.parent().unwrap()).ok();
            ui.close_menu();
        }
        
        ui.separator();
        
        ui.menu_button("📤 Exporter", |ui| {
            if ui.button("PDF").clicked() {
                export_pdf(item);
            }
            if ui.button("CSV").clicked() {
                export_csv(item);
            }
        });
        
        ui.separator();
        
        if ui.button("🗑️ Supprimer").clicked() {
            // Confirmation native
            if native_dialog::confirm("Supprimer ?", "Cette action est irréversible") {
                delete_item(item);
            }
        }
    });
}
```

## Performance perçue

### Comparatif de réactivité

```rust
/// Temps de réponse typiques (mesurés)
pub struct ResponseTimes {
    pub action: &'static str,
    pub web_app_ms: u32,
    pub native_app_ms: u32,
}

const RESPONSE_COMPARISONS: &[ResponseTimes] = &[
    ResponseTimes {
        action: "Ouverture de l'application",
        web_app_ms: 3000,  // Chargement page + JS
        native_app_ms: 150,
    },
    ResponseTimes {
        action: "Affichage d'une liste (1000 items)",
        web_app_ms: 500,   // Virtual scroll, hydration
        native_app_ms: 5,
    },
    ResponseTimes {
        action: "Réponse au clic",
        web_app_ms: 50,    // Event loop JS
        native_app_ms: 1,
    },
    ResponseTimes {
        action: "Ouverture d'un modal",
        web_app_ms: 100,   // Animation CSS, render
        native_app_ms: 16, // Une frame
    },
    ResponseTimes {
        action: "Recherche instantanée",
        web_app_ms: 200,   // Debounce + API
        native_app_ms: 5,  // Index local
    },
];
```

### Perception utilisateur

```rust
/// Seuils de perception (recherche UX)
pub mod perception_thresholds {
    /// En dessous : perçu comme instantané
    pub const INSTANT_MS: u32 = 100;
    
    /// En dessous : flux de travail maintenu
    pub const FLOW_STATE_MS: u32 = 300;
    
    /// Au-dessus : attente consciente
    pub const CONSCIOUS_WAIT_MS: u32 = 1000;
    
    /// Au-dessus : frustration
    pub const FRUSTRATION_MS: u32 = 3000;
}

// Les apps web sont souvent entre FLOW_STATE et FRUSTRATION
// Les apps natives peuvent rester sous INSTANT
```

## Cohérence avec l'OS

```rust
/// Intégration avec les conventions de l'OS
pub struct OsConsistency {
    pub aspect: &'static str,
    pub native_behavior: &'static str,
    pub web_behavior: &'static str,
}

const OS_CONSISTENCY_EXAMPLES: &[OsConsistency] = &[
    OsConsistency {
        aspect: "Scrolling",
        native_behavior: "Inertie et élasticité système",
        web_behavior: "Scroll JavaScript, souvent différent",
    },
    OsConsistency {
        aspect: "Sélection de texte",
        native_behavior: "Triple-clic = paragraphe, comportement natif",
        web_behavior: "Parfois cassé par le CSS/JS",
    },
    OsConsistency {
        aspect: "Menus",
        native_behavior: "Menu bar système (macOS)",
        web_behavior: "Menus custom dans la page",
    },
    OsConsistency {
        aspect: "Dialogues fichiers",
        native_behavior: "Dialogue système avec favoris, récents",
        web_behavior: "Dialogue limité, pas d'accès aux favoris",
    },
    OsConsistency {
        aspect: "Accessibilité",
        native_behavior: "APIs système (VoiceOver, NVDA)",
        web_behavior: "ARIA, souvent incomplet",
    },
];
```

## Conclusion

L'UX native est supérieure car :

- **Pas de latence structurelle** : Réponse instantanée
- **Raccourcis clavier complets** : Pas de conflits avec le navigateur
- **Intégration système** : Tray, notifications, fichiers
- **Cohérence OS** : Comportements familiers
- **Accès complet** : Pas de sandbox restrictive
