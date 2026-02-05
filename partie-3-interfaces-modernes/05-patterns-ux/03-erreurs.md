# 12.3 Gestion des Erreurs UX

## Messages d'erreur utilisables

```rust
pub struct UserError {
    pub title: String,
    pub message: String,
    pub suggestion: Option<String>,
    pub technical_details: Option<String>,
    pub actions: Vec<ErrorAction>,
}

pub struct ErrorAction {
    pub label: String,
    pub action_type: ErrorActionType,
}

pub enum ErrorActionType {
    Retry,
    Dismiss,
    OpenSettings,
    ContactSupport,
    Custom(Box<dyn FnOnce()>),
}

impl UserError {
    pub fn from_io_error(error: std::io::Error, context: &str) -> Self {
        match error.kind() {
            std::io::ErrorKind::NotFound => Self {
                title: "Fichier introuvable".to_string(),
                message: format!("Le fichier {} n'existe plus.", context),
                suggestion: Some("Il a peut-être été déplacé ou supprimé.".to_string()),
                technical_details: Some(error.to_string()),
                actions: vec![
                    ErrorAction { label: "Parcourir".to_string(), action_type: ErrorActionType::Custom(Box::new(|| {})) },
                    ErrorAction { label: "Fermer".to_string(), action_type: ErrorActionType::Dismiss },
                ],
            },
            std::io::ErrorKind::PermissionDenied => Self {
                title: "Accès refusé".to_string(),
                message: format!("Impossible d'accéder à {}.", context),
                suggestion: Some("Vérifiez les permissions du fichier.".to_string()),
                technical_details: Some(error.to_string()),
                actions: vec![
                    ErrorAction { label: "Fermer".to_string(), action_type: ErrorActionType::Dismiss },
                ],
            },
            _ => Self {
                title: "Erreur".to_string(),
                message: format!("Une erreur est survenue avec {}.", context),
                suggestion: None,
                technical_details: Some(error.to_string()),
                actions: vec![
                    ErrorAction { label: "Réessayer".to_string(), action_type: ErrorActionType::Retry },
                    ErrorAction { label: "Fermer".to_string(), action_type: ErrorActionType::Dismiss },
                ],
            },
        }
    }
    
    pub fn render(&self, ui: &mut Ui, tokens: &DesignTokens) -> Option<ErrorActionType> {
        // Rendu avec titre, message, suggestion, détails techniques collapsibles, actions
        // Voir 00-contenu-complet.md pour l'implémentation complète
    }
}
```

## Principes

1. **Expliquer** : Pas juste "Erreur", mais pourquoi
2. **Suggérer** : Que faire ensuite
3. **Permettre la récupération** : Retry, parcourir, etc.
4. **Détails techniques** : Disponibles mais cachés par défaut

## Résumé

- **Compréhensible** : Message clair pour l'utilisateur
- **Actionnable** : Boutons pour récupérer
- **Détails techniques** : Disponibles pour le debug
- **Contextuel** : Adapté au type d'erreur
