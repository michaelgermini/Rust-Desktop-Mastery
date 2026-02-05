# 3.3 Dette UX vs Dette Technique

## Définitions

```rust
/// Types de dette dans un projet logiciel
pub enum ProjectDebt {
    /// Code mal structuré, difficile à maintenir
    Technical {
        examples: Vec<&'static str>,
        impact: &'static str,
    },
    
    /// Interface confuse, workflows inefficaces
    UX {
        examples: Vec<&'static str>,
        impact: &'static str,
    },
}

impl ProjectDebt {
    pub fn technical_examples() -> Self {
        Self::Technical {
            examples: vec![
                "Code dupliqué partout",
                "Pas de tests",
                "Architecture spaghetti",
                "Dépendances obsolètes",
                "Pas de documentation",
            ],
            impact: "Ralentit le développement futur",
        }
    }
    
    pub fn ux_examples() -> Self {
        Self::UX {
            examples: vec![
                "Workflow en 10 clics au lieu de 2",
                "Messages d'erreur incompréhensibles",
                "Pas de raccourcis clavier",
                "Incohérences visuelles",
                "Fonctionnalités cachées",
            ],
            impact: "Frustre les utilisateurs, réduit l'adoption",
        }
    }
}
```

## Pourquoi la dette UX est souvent ignorée

```rust
/// Raisons pour lesquelles la dette UX s'accumule
pub struct UXDebtAccumulation {
    pub reasons: Vec<Reason>,
}

impl Default for UXDebtAccumulation {
    fn default() -> Self {
        Self {
            reasons: vec![
                Reason {
                    name: "Invisible dans le code",
                    description: "Le code 'marche', donc on passe à autre chose",
                },
                Reason {
                    name: "Développeurs ≠ Utilisateurs",
                    description: "On connaît l'app par cœur, on ne voit plus les problèmes",
                },
                Reason {
                    name: "Pas de métrique claire",
                    description: "Comment mesurer 'l'UX est mauvaise' ?",
                },
                Reason {
                    name: "Feature > Polish",
                    description: "Pression pour ajouter des features, pas pour polir",
                },
                Reason {
                    name: "Subjectif",
                    description: "'L'UX est une question de goût' (faux)",
                },
            ],
        }
    }
}
```

## Impact comparé

```rust
/// Comparaison des impacts
pub struct DebtImpactComparison {
    pub metric: &'static str,
    pub technical_debt_impact: &'static str,
    pub ux_debt_impact: &'static str,
}

const IMPACT_COMPARISON: &[DebtImpactComparison] = &[
    DebtImpactComparison {
        metric: "Qui souffre ?",
        technical_debt_impact: "Les développeurs",
        ux_debt_impact: "Les utilisateurs (et donc le business)",
    },
    DebtImpactComparison {
        metric: "Quand ?",
        technical_debt_impact: "À chaque modification du code",
        ux_debt_impact: "À chaque utilisation de l'app",
    },
    DebtImpactComparison {
        metric: "Visible ?",
        technical_debt_impact: "Seulement par les devs",
        ux_debt_impact: "Par tout le monde",
    },
    DebtImpactComparison {
        metric: "Impact business",
        technical_debt_impact: "Indirect (vélocité réduite)",
        ux_debt_impact: "Direct (churn, mauvaises reviews)",
    },
    DebtImpactComparison {
        metric: "Temps pour corriger",
        technical_debt_impact: "Variable, souvent long",
        ux_debt_impact: "Souvent rapide (quick wins)",
    },
];
```

## Identifier la dette UX

```rust
/// Signaux de dette UX
pub struct UXDebtSignals {
    pub signals: Vec<Signal>,
}

impl Default for UXDebtSignals {
    fn default() -> Self {
        Self {
            signals: vec![
                Signal {
                    observation: "Les utilisateurs posent souvent la même question",
                    diagnosis: "L'interface n'est pas explicite",
                    quick_fix: "Ajouter un tooltip ou un label",
                },
                Signal {
                    observation: "Beaucoup de clics pour une action courante",
                    diagnosis: "Workflow inefficace",
                    quick_fix: "Ajouter un raccourci ou bouton direct",
                },
                Signal {
                    observation: "Les utilisateurs n'utilisent pas une feature",
                    diagnosis: "Feature cachée ou incomprise",
                    quick_fix: "Améliorer la découvrabilité",
                },
                Signal {
                    observation: "Erreurs fréquentes des utilisateurs",
                    diagnosis: "Design induisant en erreur",
                    quick_fix: "Revoir le flow, ajouter des confirmations",
                },
                Signal {
                    observation: "Demandes de formation récurrentes",
                    diagnosis: "App pas intuitive",
                    quick_fix: "Onboarding, aide contextuelle",
                },
            ],
        }
    }
}

/// Audit UX rapide
pub fn quick_ux_audit(app: &App) -> Vec<UXIssue> {
    let mut issues = vec![];
    
    // 1. Vérifier les workflows principaux
    for workflow in app.main_workflows() {
        if workflow.click_count() > 3 {
            issues.push(UXIssue {
                severity: Severity::Medium,
                location: workflow.name.clone(),
                problem: format!(
                    "Workflow '{}' nécessite {} clics",
                    workflow.name, workflow.click_count()
                ),
                suggestion: "Réduire à 2-3 clics max".to_string(),
            });
        }
    }
    
    // 2. Vérifier les messages d'erreur
    for error_msg in app.error_messages() {
        if error_msg.is_technical() {
            issues.push(UXIssue {
                severity: Severity::High,
                location: "Messages d'erreur".to_string(),
                problem: format!("Message technique: '{}'", error_msg.text),
                suggestion: "Reformuler en langage utilisateur".to_string(),
            });
        }
    }
    
    // 3. Vérifier les raccourcis
    if app.keyboard_shortcuts().len() < 10 {
        issues.push(UXIssue {
            severity: Severity::Low,
            location: "Raccourcis clavier".to_string(),
            problem: "Peu de raccourcis disponibles".to_string(),
            suggestion: "Ajouter des raccourcis pour les actions courantes".to_string(),
        });
    }
    
    issues
}
```

## Prioriser la correction

```rust
/// Matrice de priorisation UX
pub struct UXPrioritizationMatrix;

impl UXPrioritizationMatrix {
    /// Score = Fréquence × Friction × Facilité de correction
    pub fn calculate_priority(issue: &UXIssue) -> u32 {
        let frequency = match issue.usage_frequency {
            UsageFrequency::EverySession => 10,
            UsageFrequency::Daily => 7,
            UsageFrequency::Weekly => 4,
            UsageFrequency::Rarely => 1,
        };
        
        let friction = match issue.friction_level {
            FrictionLevel::Blocker => 10,
            FrictionLevel::Annoying => 7,
            FrictionLevel::Minor => 3,
            FrictionLevel::Cosmetic => 1,
        };
        
        let fix_ease = match issue.estimated_fix_time {
            Duration::Minutes(m) if m < 30 => 10,
            Duration::Hours(h) if h < 2 => 7,
            Duration::Hours(h) if h < 8 => 4,
            _ => 1,
        };
        
        frequency * friction * fix_ease
    }
    
    pub fn prioritize(issues: Vec<UXIssue>) -> Vec<UXIssue> {
        let mut sorted = issues;
        sorted.sort_by(|a, b| {
            Self::calculate_priority(b).cmp(&Self::calculate_priority(a))
        });
        sorted
    }
}
```

## Quick wins UX

```rust
/// Améliorations UX rapides à fort impact
pub const UX_QUICK_WINS: &[QuickWin] = &[
    QuickWin {
        effort: "30 min",
        impact: "Élevé",
        change: "Ajouter Ctrl+S pour sauvegarder",
    },
    QuickWin {
        effort: "1h",
        impact: "Élevé",
        change: "Améliorer les messages d'erreur",
    },
    QuickWin {
        effort: "2h",
        impact: "Moyen",
        change: "Ajouter des tooltips sur les icônes",
    },
    QuickWin {
        effort: "30 min",
        impact: "Élevé",
        change: "Confirmation avant suppression",
    },
    QuickWin {
        effort: "1h",
        impact: "Moyen",
        change: "Indicateur de chargement",
    },
    QuickWin {
        effort: "2h",
        impact: "Élevé",
        change: "Undo pour les actions destructrices",
    },
    QuickWin {
        effort: "30 min",
        impact: "Moyen",
        change: "Focus automatique sur le premier champ",
    },
    QuickWin {
        effort: "1h",
        impact: "Élevé",
        change: "Navigation au clavier dans les listes",
    },
];
```

## Équilibrer les deux dettes

```rust
/// Stratégie d'équilibrage
pub struct DebtBalancingStrategy {
    /// Pourcentage du temps sur la dette technique
    pub technical_debt_budget: f32,
    
    /// Pourcentage du temps sur la dette UX
    pub ux_debt_budget: f32,
    
    /// Pourcentage sur les nouvelles features
    pub feature_budget: f32,
}

impl Default for DebtBalancingStrategy {
    fn default() -> Self {
        Self {
            technical_debt_budget: 0.20,  // 20%
            ux_debt_budget: 0.15,          // 15%
            feature_budget: 0.65,          // 65%
        }
    }
}

impl DebtBalancingStrategy {
    /// Allocation recommandée selon la phase
    pub fn for_phase(phase: ProjectPhase) -> Self {
        match phase {
            ProjectPhase::PreLaunch => Self {
                technical_debt_budget: 0.10,
                ux_debt_budget: 0.30,  // Focus UX avant lancement
                feature_budget: 0.60,
            },
            ProjectPhase::Growth => Self {
                technical_debt_budget: 0.15,
                ux_debt_budget: 0.20,
                feature_budget: 0.65,
            },
            ProjectPhase::Mature => Self {
                technical_debt_budget: 0.25,  // Plus de refactoring
                ux_debt_budget: 0.15,
                feature_budget: 0.60,
            },
        }
    }
}
```

## Conclusion

La dette UX est aussi importante que la dette technique :

- **Impact direct** : Affecte les utilisateurs, pas les devs
- **Souvent invisible** : On s'habitue aux problèmes
- **Quick wins** : Souvent rapide à corriger
- **ROI élevé** : Améliore satisfaction et rétention
- **À budgéter** : Allouer 15-20% du temps au polish UX
