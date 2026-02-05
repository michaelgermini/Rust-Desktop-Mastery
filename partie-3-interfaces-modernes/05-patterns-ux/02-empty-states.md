# 12.2 Empty States

## Design d'états vides engageants

```rust
pub struct EmptyState<'a> {
    icon: &'a str,
    title: &'a str,
    description: &'a str,
    action_label: Option<&'a str>,
}

impl<'a> EmptyState<'a> {
    pub fn new(icon: &'a str, title: &'a str, description: &'a str) -> Self {
        Self {
            icon,
            title,
            description,
            action_label: None,
        }
    }
    
    pub fn with_action(mut self, label: &'a str) -> Self {
        self.action_label = Some(label);
        self
    }
    
    pub fn show(self, ui: &mut Ui, tokens: &DesignTokens) -> bool {
        let mut action_clicked = false;
        
        ui.vertical_centered(|ui| {
            ui.add_space(tokens.spacing.xxxl);
            
            // Icône grande
            ui.label(
                egui::RichText::new(self.icon)
                    .size(64.0)
            );
            
            ui.add_space(tokens.spacing.lg);
            
            // Titre
            ui.label(
                egui::RichText::new(self.title)
                    .size(tokens.typography.size_xl)
                    .color(tokens.colors.text_primary)
                    .strong()
            );
            
            ui.add_space(tokens.spacing.sm);
            
            // Description
            ui.label(
                egui::RichText::new(self.description)
                    .size(tokens.typography.size_md)
                    .color(tokens.colors.text_muted)
            );
            
            // Action
            if let Some(label) = self.action_label {
                ui.add_space(tokens.spacing.lg);
                
                if Button::new(label)
                    .variant(ButtonVariant::Primary)
                    .show(ui, tokens)
                    .clicked()
                {
                    action_clicked = true;
                }
            }
            
            ui.add_space(tokens.spacing.xxxl);
        });
        
        action_clicked
    }
}
```

## Usage

```rust
if documents.is_empty() {
    if EmptyState::new(
        "📄",
        "Aucun document",
        "Créez votre premier document pour commencer",
    )
    .with_action("Nouveau document")
    .show(ui, &tokens)
    {
        create_new_document();
    }
}
```

## Empty states contextuels

```rust
pub fn render_search_empty_state(
    ui: &mut Ui,
    query: &str,
    tokens: &DesignTokens,
) {
    EmptyState::new(
        "🔍",
        "Aucun résultat",
        &format!("Aucun résultat pour \"{}\"", query),
    )
    .show(ui, tokens);
    
    // Suggestions
    ui.vertical_centered(|ui| {
        ui.label(
            egui::RichText::new("Suggestions:")
                .size(tokens.typography.size_sm)
                .color(tokens.colors.text_muted)
        );
        
        ui.label("• Vérifiez l'orthographe");
        ui.label("• Essayez des termes plus généraux");
        ui.label("• Utilisez moins de mots-clés");
    });
}
```

## Résumé

- **Engageant** : Icône, titre, description claire
- **Actionnable** : Bouton CTA pour guider l'utilisateur
- **Contextuel** : Message adapté à la situation
- **Suggestions** : Aide l'utilisateur à progresser
