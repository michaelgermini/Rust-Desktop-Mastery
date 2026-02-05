# 31.1 Documentation Utilisateur

## Documentation intégrée

```rust
/// Structure de documentation intégrée dans l'application
pub struct InAppDocumentation {
    articles: HashMap<String, HelpArticle>,
    search_index: SearchIndex,
}

pub struct HelpArticle {
    pub id: String,
    pub title: String,
    pub content: String,  // Markdown
    pub category: HelpCategory,
    pub keywords: Vec<String>,
}

#[derive(Clone)]
pub enum HelpCategory {
    GettingStarted,
    Features,
    Advanced,
    Troubleshooting,
}

impl InAppDocumentation {
    pub fn show_help_panel(&self, ui: &mut Ui, context: &str) {
        // Afficher l'aide contextuelle
        if let Some(article) = self.get_contextual_help(context) {
            egui::ScrollArea::vertical().show(ui, |ui| {
                ui.heading(&article.title);
                ui.separator();
                
                // Render markdown
                render_markdown(ui, &article.content);
            });
        }
    }
    
    pub fn show_search(&mut self, ui: &mut Ui, query: &mut String) {
        ui.horizontal(|ui| {
            ui.label("🔍");
            ui.text_edit_singleline(query);
        });
        
        if !query.is_empty() {
            let results = self.search_index.search(query);
            for article in results.iter().take(5) {
                if ui.link(&article.title).clicked() {
                    // Ouvrir l'article
                }
            }
        }
    }
    
    fn get_contextual_help(&self, context: &str) -> Option<&HelpArticle> {
        // Retourner l'article le plus pertinent pour le contexte
        self.articles.values()
            .find(|a| a.keywords.iter().any(|k| k == context))
    }
}
```

## Aide contextuelle

```rust
impl App {
    fn render_with_contextual_help(&mut self, ui: &mut Ui) {
        // Afficher un bouton d'aide contextuel
        if ui.button("❓").clicked() {
            self.show_help = true;
        }
        
        // Afficher l'aide selon le contexte actuel
        if self.show_help {
            let context = self.get_current_context();  // Ex: "invoice_creation"
            self.documentation.show_help_panel(ui, &context);
        }
    }
    
    fn get_current_context(&self) -> String {
        // Déterminer le contexte selon l'écran actuel
        match self.current_screen {
            Screen::InvoiceEditor => "invoice_creation".to_string(),
            Screen::CustomerList => "customer_management".to_string(),
            _ => "general".to_string(),
        }
    }
}
```

## FAQ

```rust
pub struct FAQ {
    pub questions: Vec<FAQItem>,
}

pub struct FAQItem {
    pub question: String,
    pub answer: String,
    pub category: String,
    pub keywords: Vec<String>,
}

impl FAQ {
    pub fn render(&self, ui: &mut Ui, tokens: &DesignTokens) {
        ui.heading("Questions fréquentes");
        
        for item in &self.questions {
            egui::CollapsingHeader::new(&item.question)
                .default_open(false)
                .show(ui, |ui| {
                    ui.label(&item.answer);
                });
        }
    }
}
```

## Structure de la documentation

```
docs/
├── getting-started/
│   ├── installation.md
│   ├── first-steps.md
│   └── basic-workflow.md
├── features/
│   ├── customers.md
│   ├── invoices.md
│   ├── reports.md
│   └── export.md
├── advanced/
│   ├── keyboard-shortcuts.md
│   ├── backup-restore.md
│   └── customization.md
├── troubleshooting/
│   ├── common-issues.md
│   └── error-messages.md
└── changelog.md
```

## Résumé

- **Intégrée** : Documentation accessible depuis l'application
- **Contextuelle** : Aide selon l'écran actuel
- **Recherche** : Index de recherche pour trouver rapidement
- **FAQ** : Questions fréquentes facilement accessibles
