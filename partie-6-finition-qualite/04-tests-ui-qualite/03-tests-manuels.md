# 28.3 Tests Manuels Intelligents

## Checklist de test

```rust
/// Checklist de qualité avant release
pub struct QualityChecklist {
    pub tests_pass: bool,
    pub no_warnings: bool,
    pub no_clippy_lints: bool,
    pub documentation_complete: bool,
    pub changelog_updated: bool,
    pub version_bumped: bool,
    pub manual_tests: ManualTestResults,
}

pub struct ManualTestResults {
    pub ui_tests: Vec<UITestResult>,
    pub workflow_tests: Vec<WorkflowTestResult>,
}

pub struct UITestResult {
    pub name: String,
    pub passed: bool,
    pub notes: String,
}

impl QualityChecklist {
    pub fn run() -> Self {
        Self {
            tests_pass: run_tests(),
            no_warnings: check_warnings(),
            no_clippy_lints: run_clippy(),
            documentation_complete: check_docs(),
            changelog_updated: check_changelog(),
            version_bumped: check_version(),
            manual_tests: ManualTestResults::default(),
        }
    }
    
    pub fn is_ready(&self) -> bool {
        self.tests_pass
            && self.no_warnings
            && self.no_clippy_lints
            && self.changelog_updated
            && self.version_bumped
            && self.manual_tests.all_passed()
    }
}
```

## Scénarios utilisateur

```markdown
# Scénarios de test manuel

## Scénario 1 : Créer une facture
1. Ouvrir l'application
2. Cliquer sur "Nouvelle facture"
3. Sélectionner un client
4. Ajouter des items
5. Vérifier le calcul automatique
6. Sauvegarder
7. Vérifier que la facture apparaît dans la liste

## Scénario 2 : Recherche
1. Ouvrir la recherche (Cmd+F)
2. Taper "test"
3. Vérifier que les résultats apparaissent
4. Cliquer sur un résultat
5. Vérifier la navigation

## Scénario 3 : Export PDF
1. Sélectionner une facture
2. Cliquer sur "Exporter PDF"
3. Choisir un emplacement
4. Vérifier que le PDF est généré
5. Ouvrir le PDF et vérifier le contenu
```

## Documentation des tests

```rust
pub struct TestScenario {
    pub name: String,
    pub description: String,
    pub steps: Vec<TestStep>,
    pub expected_result: String,
}

pub struct TestStep {
    pub action: String,
    pub expected_state: String,
}

impl TestScenario {
    pub fn to_markdown(&self) -> String {
        format!(
            "## {}\n\n{}\n\n### Étapes:\n{}\n\n### Résultat attendu:\n{}",
            self.name,
            self.description,
            self.steps.iter()
                .enumerate()
                .map(|(i, step)| format!("{}. {} → {}", i + 1, step.action, step.expected_state))
                .collect::<Vec<_>>()
                .join("\n"),
            self.expected_result
        )
    }
}
```

## Résumé

- **Checklist** : Liste de vérifications avant release
- **Scénarios** : Tests de workflows utilisateur complets
- **Documentation** : Scénarios documentés en Markdown
- **Répétabilité** : Tests manuels structurés et traçables
