# 31.3 Télémétrie Respectueuse

## Opt-in uniquement

```rust
/// Télémétrie opt-in avec données anonymisées
pub struct TelemetryManager {
    enabled: bool,
    session_id: String,  // Pas d'identifiant permanent
    events: Vec<TelemetryEvent>,
}

#[derive(Serialize)]
pub struct TelemetryEvent {
    pub event_type: String,
    pub timestamp: DateTime<Utc>,
    pub properties: HashMap<String, String>,
    // PAS de données personnelles
}

impl TelemetryManager {
    pub fn new() -> Self {
        Self {
            enabled: false,  // Opt-in par défaut
            session_id: Uuid::new_v4().to_string(),
            events: Vec::new(),
        }
    }
    
    /// Demander le consentement au premier lancement
    pub fn show_consent_dialog(ui: &mut Ui) -> Option<bool> {
        let mut result = None;
        
        egui::Window::new("Améliorer Mon App")
            .collapsible(false)
            .resizable(false)
            .show(ui.ctx(), |ui| {
                ui.label("Nous collectons des données anonymes pour améliorer l'application.");
                ui.label("");
                ui.label("Données collectées :");
                ui.label("• Fonctionnalités utilisées (sans contenu)");
                ui.label("• Erreurs rencontrées");
                ui.label("• Performances");
                ui.label("");
                ui.colored_label(egui::Color32::GRAY, 
                    "Aucune donnée personnelle n'est collectée.");
                
                ui.add_space(16.0);
                
                ui.horizontal(|ui| {
                    if ui.button("Refuser").clicked() {
                        result = Some(false);
                    }
                    if ui.button("Accepter").clicked() {
                        result = Some(true);
                    }
                });
            });
        
        result
    }
}
```

## Données anonymisées

```rust
impl TelemetryManager {
    /// Tracker un événement (uniquement si consentement)
    pub fn track(&mut self, event_type: &str, properties: HashMap<String, String>) {
        if !self.enabled {
            return;
        }
        
        // Anonymiser les propriétés
        let anonymized_props = self.anonymize_properties(properties);
        
        self.events.push(TelemetryEvent {
            event_type: event_type.to_string(),
            timestamp: Utc::now(),
            properties: anonymized_props,
        });
        
        // Envoyer par batch de 10
        if self.events.len() >= 10 {
            self.flush();
        }
    }
    
    fn anonymize_properties(&self, props: HashMap<String, String>) -> HashMap<String, String> {
        let mut anonymized = HashMap::new();
        
        for (key, value) in props {
            // Ne jamais envoyer de données personnelles
            match key.as_str() {
                "email" | "name" | "customer_name" => {
                    // Ignorer
                }
                "document_count" | "feature_used" => {
                    // OK - données anonymes
                    anonymized.insert(key, value);
                }
                _ => {
                    // Vérifier avant d'inclure
                    if !self.contains_pii(&value) {
                        anonymized.insert(key, value);
                    }
                }
            }
        }
        
        anonymized
    }
    
    fn contains_pii(&self, text: &str) -> bool {
        // Détecter les emails, noms, etc.
        text.contains('@') || text.len() > 50
    }
}
```

## Transparence

```rust
impl TelemetryManager {
    pub fn show_telemetry_info(&self, ui: &mut Ui) {
        egui::Window::new("Télémétrie")
            .show(ui.ctx(), |ui| {
                ui.label("État :");
                ui.label(if self.enabled { "✅ Activée" } else { "❌ Désactivée" });
                
                ui.separator();
                
                ui.label("Données collectées :");
                ui.label("• Types d'événements (ex: 'button_clicked')");
                ui.label("• Fonctionnalités utilisées (sans contenu)");
                ui.label("• Erreurs techniques");
                ui.label("• Performances (temps de chargement)");
                
                ui.separator();
                
                ui.label("Données NON collectées :");
                ui.colored_label(egui::Color32::RED, "• Contenu des documents");
                ui.colored_label(egui::Color32::RED, "• Noms de clients");
                ui.colored_label(egui::Color32::RED, "• Informations personnelles");
                
                ui.separator();
                
                if ui.button("Désactiver").clicked() {
                    self.enabled = false;
                }
            });
    }
}
```

## Résumé

- **Opt-in** : Consentement explicite requis
- **Anonymisé** : Aucune donnée personnelle
- **Transparent** : Utilisateur peut voir ce qui est collecté
- **Respectueux** : Désactivation facile à tout moment
