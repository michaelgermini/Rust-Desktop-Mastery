# 11.3 Table

## Tableau avec tri et sélection

```rust
pub struct Table<'a, T> {
    items: &'a [T],
    columns: Vec<TableColumn<'a, T>>,
    selected: Option<&'a mut Option<usize>>,
    striped: bool,
}

impl<'a, T> Table<'a, T> {
    pub fn new(items: &'a [T]) -> Self {
        Self {
            items,
            columns: Vec::new(),
            selected: None,
            striped: true,
        }
    }
    
    pub fn column(
        mut self,
        header: &'a str,
        render: impl Fn(&T, &mut Ui, &DesignTokens) + 'a,
    ) -> Self {
        // Ajouter une colonne
    }
    
    pub fn selectable(mut self, selected: &'a mut Option<usize>) -> Self {
        self.selected = Some(selected);
        self
    }
    
    pub fn show(self, ui: &mut Ui, tokens: &DesignTokens) -> Option<usize> {
        // Rendu du tableau
    }
}
```

## Usage

```rust
Table::new(&invoices)
    .column("N°", |inv, ui, _| { ui.label(&inv.number); })
    .column("Client", |inv, ui, _| { ui.label(&inv.client_name); })
    .column_with_width("Montant", 100.0, |inv, ui, tokens| {
        ui.label(
            egui::RichText::new(format!("{:.2} €", inv.total))
                .color(tokens.colors.success)
        );
    })
    .selectable(&mut selected_invoice)
    .show(ui, &tokens);
```

## Features

- **Colonnes flexibles** : Largeur fixe ou automatique
- **Sélection** : Clic sur une ligne
- **Striped** : Alternance de couleurs
- **Scroll** : Défilement horizontal si nécessaire
