# 3.1 Ship Plutôt que Démo

## La différence fondamentale

```rust
/// Qu'est-ce qui distingue une démo d'un produit ?
pub struct DemoVsProduct {
    pub aspect: &'static str,
    pub demo: &'static str,
    pub product: &'static str,
}

const DIFFERENCES: &[DemoVsProduct] = &[
    DemoVsProduct {
        aspect: "Cas d'erreur",
        demo: "Ignorés ou println!()",
        product: "Gérés gracieusement avec feedback utilisateur",
    },
    DemoVsProduct {
        aspect: "Edge cases",
        demo: "\"Ça marche dans mon cas\"",
        product: "Testés et documentés",
    },
    DemoVsProduct {
        aspect: "Performance",
        demo: "\"Suffisant pour la démo\"",
        product: "Mesurée et optimisée",
    },
    DemoVsProduct {
        aspect: "Données",
        demo: "Hardcodées ou reset à chaque lancement",
        product: "Persistées, migrées, sauvegardées",
    },
    DemoVsProduct {
        aspect: "Installation",
        demo: "cargo run",
        product: "Installer en 2 clics",
    },
    DemoVsProduct {
        aspect: "Documentation",
        demo: "README minimal",
        product: "Aide intégrée, tooltips, onboarding",
    },
];
```

## Le piège du perfectionnisme

### Syndrome du "encore une feature"

```rust
/// Le cycle infini du perfectionnisme
pub enum PerfectionismTrap {
    /// "Il me faut encore cette feature avant de lancer"
    OneMoreFeature,
    
    /// "Le code n'est pas assez propre"
    CodeNotCleanEnough,
    
    /// "L'UI n'est pas assez belle"
    UiNotPrettyEnough,
    
    /// "Je dois d'abord écrire des tests"
    NeedMoreTests,
    
    /// "Je vais d'abord refactorer"
    NeedToRefactorFirst,
}

impl PerfectionismTrap {
    pub fn reality_check(&self) -> &'static str {
        match self {
            Self::OneMoreFeature => 
                "Les utilisateurs vous diront quelles features ils veulent vraiment",
            Self::CodeNotCleanEnough => 
                "Le code parfait n'existe pas. Ship, puis améliorez.",
            Self::UiNotPrettyEnough => 
                "Mieux vaut une UI moche qui marche qu'une belle qui n'existe pas",
            Self::NeedMoreTests => 
                "Testez les chemins critiques. Le reste viendra.",
            Self::NeedToRefactorFirst => 
                "Refactorez après avoir des utilisateurs réels",
        }
    }
}
```

### Le coût de l'attente

```rust
/// Impact du délai de lancement
pub struct LaunchDelayCost {
    /// Mois de retard
    pub months_delayed: u32,
    
    /// Revenus perdus (estimation)
    pub lost_revenue: Decimal,
    
    /// Feedback utilisateur non collecté
    pub missed_feedback_cycles: u32,
    
    /// Motivation de l'équipe
    pub team_morale_impact: MoraleLevel,
}

impl LaunchDelayCost {
    pub fn calculate(months: u32, potential_mrr: Decimal) -> Self {
        Self {
            months_delayed: months,
            lost_revenue: potential_mrr * Decimal::from(months),
            missed_feedback_cycles: months,  // ~1 cycle par mois
            team_morale_impact: if months > 6 {
                MoraleLevel::Critical
            } else if months > 3 {
                MoraleLevel::Declining
            } else {
                MoraleLevel::Stable
            },
        }
    }
}
```

## Définir le MVP réel

### Méthode de priorisation

```rust
/// Framework MoSCoW pour prioriser
pub enum Priority {
    /// Doit être présent pour le lancement
    MustHave,
    
    /// Devrait être présent si possible
    ShouldHave,
    
    /// Serait bien d'avoir
    CouldHave,
    
    /// Pas pour cette version
    WontHave,
}

pub struct FeaturePrioritization {
    pub features: Vec<(String, Priority)>,
}

impl FeaturePrioritization {
    pub fn example_invoice_app() -> Self {
        Self {
            features: vec![
                // Must Have - Sans ça, pas de produit
                ("Créer une facture".into(), Priority::MustHave),
                ("Sauvegarder les factures".into(), Priority::MustHave),
                ("Exporter en PDF".into(), Priority::MustHave),
                ("Gérer les clients".into(), Priority::MustHave),
                
                // Should Have - Important mais pas bloquant
                ("Modèles de facture".into(), Priority::ShouldHave),
                ("Calcul TVA automatique".into(), Priority::ShouldHave),
                ("Recherche".into(), Priority::ShouldHave),
                
                // Could Have - Nice to have
                ("Thème sombre".into(), Priority::CouldHave),
                ("Export CSV".into(), Priority::CouldHave),
                ("Statistiques".into(), Priority::CouldHave),
                
                // Won't Have - v2+
                ("Multi-devises".into(), Priority::WontHave),
                ("Sync cloud".into(), Priority::WontHave),
                ("App mobile".into(), Priority::WontHave),
            ],
        }
    }
    
    pub fn mvp_scope(&self) -> Vec<&String> {
        self.features.iter()
            .filter(|(_, p)| matches!(p, Priority::MustHave))
            .map(|(f, _)| f)
            .collect()
    }
}
```

### Définition du "Done"

```rust
/// Quand une feature est-elle vraiment finie ?
pub struct FeatureDefinitionOfDone {
    pub checks: Vec<DoneCheck>,
}

impl Default for FeatureDefinitionOfDone {
    fn default() -> Self {
        Self {
            checks: vec![
                DoneCheck {
                    category: "Fonctionnel",
                    items: vec![
                        "Fonctionne pour le cas nominal",
                        "Gère les cas d'erreur courants",
                        "Feedback visuel pour les actions longues",
                    ],
                },
                DoneCheck {
                    category: "UX",
                    items: vec![
                        "L'utilisateur comprend quoi faire",
                        "Les erreurs sont compréhensibles",
                        "Undo disponible si destructif",
                    ],
                },
                DoneCheck {
                    category: "Technique",
                    items: vec![
                        "Pas de panic en production",
                        "Données persistées correctement",
                        "Performance acceptable",
                    ],
                },
            ],
        }
    }
}
```

## Stratégie de lancement

### Lancement progressif

```rust
/// Phases de lancement recommandées
pub struct LaunchPhases {
    pub phases: Vec<LaunchPhase>,
}

impl Default for LaunchPhases {
    fn default() -> Self {
        Self {
            phases: vec![
                LaunchPhase {
                    name: "Alpha privée",
                    audience: "5-10 utilisateurs de confiance",
                    duration_weeks: 2,
                    goal: "Trouver les bugs majeurs",
                },
                LaunchPhase {
                    name: "Beta fermée",
                    audience: "50-100 early adopters",
                    duration_weeks: 4,
                    goal: "Valider le product-market fit",
                },
                LaunchPhase {
                    name: "Beta ouverte",
                    audience: "Accès libre avec label 'beta'",
                    duration_weeks: 4,
                    goal: "Scaling et polish",
                },
                LaunchPhase {
                    name: "Lancement officiel",
                    audience: "Public",
                    duration_weeks: 0,
                    goal: "Marketing et croissance",
                },
            ],
        }
    }
}
```

### Checklist de lancement minimal

```rust
/// Le strict minimum pour lancer
pub fn minimum_launch_checklist() -> Vec<ChecklistItem> {
    vec![
        // Fonctionnel
        ChecklistItem::new("Core feature fonctionne", true),
        ChecklistItem::new("Données sauvegardées", true),
        ChecklistItem::new("Pas de crash évident", true),
        
        // Distribution
        ChecklistItem::new("Installeur pour l'OS principal", true),
        ChecklistItem::new("Page de téléchargement", true),
        
        // Support
        ChecklistItem::new("Email de contact", true),
        ChecklistItem::new("FAQ basique", false),  // Nice to have
        
        // Légal (si nécessaire)
        ChecklistItem::new("CGU/CGV si vente", true),
        ChecklistItem::new("Politique de confidentialité", true),
    ]
}
```

## Mentalité "Ship Early"

```rust
/// Principes du shipping rapide
pub const SHIP_EARLY_PRINCIPLES: &[&str] = &[
    "Si vous n'êtes pas embarrassé par la v1, vous avez lancé trop tard",
    "Le feedback réel > vos suppositions",
    "Chaque jour sans utilisateurs = apprentissage perdu",
    "Le code parfait sans utilisateurs ne vaut rien",
    "Mieux vaut 80% maintenant que 100% jamais",
    "Les vraies priorités émergent après le lancement",
];

/// Questions à se poser avant d'ajouter une feature
pub fn should_add_feature(feature: &str) -> FeatureDecision {
    // Questions clés
    let questions = vec![
        "Un utilisateur a-t-il demandé cette feature ?",
        "Est-ce bloquant pour l'usage principal ?",
        "Peut-on lancer sans ?",
        "Combien de temps pour l'implémenter ?",
    ];
    
    // Si aucun utilisateur ne l'a demandée et qu'on peut lancer sans : skip
    FeatureDecision::DeferToV2
}
```

## Conclusion

Ship early signifie :

- **Définir un MVP réaliste** : Seulement les Must Have
- **Accepter l'imperfection** : La v1 sera imparfaite, c'est normal
- **Valoriser le feedback** : Les utilisateurs > vos suppositions
- **Itérer rapidement** : Petit, souvent, basé sur les retours
- **Éviter le perfectionnisme** : Le parfait est l'ennemi du bien
