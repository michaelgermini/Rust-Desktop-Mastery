# 33.3 Contributions Open Source

## Stratégie open source

```rust
/// Stratégie open source
pub struct OpenSourceStrategy {
    pub core_license: License,
    pub open_components: Vec<OpenSourceComponent>,
}

pub struct OpenSourceComponent {
    pub name: String,
    pub repository: String,
    pub description: String,
    pub license: License,
}

#[derive(Clone)]
pub enum License {
    MIT,
    Apache2,
    Proprietary,
}

impl OpenSourceStrategy {
    pub fn example() -> Self {
        Self {
            // Le produit principal reste propriétaire
            core_license: License::Proprietary,
            
            // Mais certains composants sont open source
            open_components: vec![
                OpenSourceComponent {
                    name: "mon-app-ui-kit".to_string(),
                    repository: "https://github.com/org/mon-app-ui-kit".to_string(),
                    description: "Composants UI réutilisables pour egui".to_string(),
                    license: License::MIT,
                },
                OpenSourceComponent {
                    name: "mon-app-plugin-sdk".to_string(),
                    repository: "https://github.com/org/mon-app-plugin-sdk".to_string(),
                    description: "SDK pour développer des plugins".to_string(),
                    license: License::Apache2,
                },
            ],
        }
    }
}
```

## Composants réutilisables

```rust
impl OpenSourceStrategy {
    pub fn identify_reusable_components(&self) -> Vec<String> {
        vec![
            "Design system".to_string(),
            "UI components library".to_string(),
            "Plugin SDK".to_string(),
            "Database migration tools".to_string(),
            "License validation library".to_string(),
        ]
    }
    
    pub fn open_source_benefits() -> Vec<&'static str> {
        vec![
            "Visibilité et crédibilité",
            "Contributions communautaires",
            "Feedback et améliorations",
            "Recrutement de talents",
            "Écosystème autour du produit",
        ]
    }
}
```

## Licence et gouvernance

```rust
pub struct OpenSourceGovernance {
    pub maintainers: Vec<Maintainer>,
    pub contribution_guidelines: String,
    pub code_of_conduct: String,
    pub license: License,
}

pub struct Maintainer {
    pub name: String,
    pub role: MaintainerRole,
}

#[derive(Clone)]
pub enum MaintainerRole {
    Lead,
    Core,
    Community,
}

impl OpenSourceGovernance {
    pub fn setup_governance() -> Self {
        Self {
            maintainers: vec![
                Maintainer {
                    name: "Core Team".to_string(),
                    role: MaintainerRole::Lead,
                },
            ],
            contribution_guidelines: include_str!("../CONTRIBUTING.md").to_string(),
            code_of_conduct: include_str!("../CODE_OF_CONDUCT.md").to_string(),
            license: License::MIT,
        }
    }
}
```

## Résumé

- **Stratégie** : Composants open source, produit principal propriétaire
- **Réutilisables** : Bibliothèques et outils partagés
- **Gouvernance** : Règles de contribution et code de conduite
- **Bénéfices** : Visibilité, contributions, écosystème
