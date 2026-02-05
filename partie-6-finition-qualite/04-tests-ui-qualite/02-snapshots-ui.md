# 28.2 Snapshots UI

## Capture d'état visuel

```rust
use std::collections::HashMap;

pub struct UiSnapshot {
    pub name: String,
    pub shapes: Vec<egui::Shape>,
    pub text_output: HashMap<egui::TextShape, String>,
    pub layout: LayoutSnapshot,
}

pub struct LayoutSnapshot {
    pub panels: Vec<PanelSnapshot>,
    pub widgets: Vec<WidgetSnapshot>,
}

pub struct PanelSnapshot {
    pub id: String,
    pub rect: egui::Rect,
    pub visible: bool,
}

pub struct WidgetSnapshot {
    pub id: String,
    pub rect: egui::Rect,
    pub text: Option<String>,
    pub visible: bool,
}

pub fn capture_ui_snapshot(ctx: &egui::Context, name: &str) -> UiSnapshot {
    let output = ctx.run(Default::default(), |ctx| {
        // Render UI
        egui::CentralPanel::default().show(ctx, |ui| {
            ui.heading("Test UI");
            ui.button("Test Button");
        });
    });
    
    UiSnapshot {
        name: name.to_string(),
        shapes: output.shapes,
        text_output: extract_text(&output.shapes),
        layout: capture_layout(ctx),
    }
}
```

## Comparaison de snapshots

```rust
pub fn compare_snapshots(current: &UiSnapshot, reference: &UiSnapshot) -> SnapshotDiff {
    let mut diff = SnapshotDiff::new();
    
    // Comparer les shapes
    if current.shapes.len() != reference.shapes.len() {
        diff.add_shape_count_mismatch(current.shapes.len(), reference.shapes.len());
    }
    
    // Comparer le texte
    for (key, current_text) in &current.text_output {
        if let Some(ref_text) = reference.text_output.get(key) {
            if current_text != ref_text {
                diff.add_text_mismatch(key.clone(), current_text.clone(), ref_text.clone());
            }
        } else {
            diff.add_missing_text(key.clone());
        }
    }
    
    diff
}

pub struct SnapshotDiff {
    pub shape_count_mismatch: Option<(usize, usize)>,
    pub text_mismatches: Vec<(String, String, String)>,
    pub missing_text: Vec<String>,
}

impl SnapshotDiff {
    pub fn has_changes(&self) -> bool {
        self.shape_count_mismatch.is_some()
            || !self.text_mismatches.is_empty()
            || !self.missing_text.is_empty()
    }
}
```

## Détection de régressions

```rust
#[test]
fn test_invoice_list_snapshot() {
    let ctx = egui::Context::default();
    let invoices = vec![
        Invoice::sample(),
        Invoice::sample(),
    ];
    
    let snapshot = capture_invoice_list_snapshot(&ctx, &invoices);
    
    // Charger le snapshot de référence
    let reference_path = Path::new("tests/snapshots/invoice_list.json");
    let reference = if reference_path.exists() {
        load_snapshot(reference_path).unwrap()
    } else {
        // Créer le snapshot de référence
        save_snapshot(&snapshot, reference_path).unwrap();
        snapshot.clone()
    };
    
    // Comparer
    let diff = compare_snapshots(&snapshot, &reference);
    
    if diff.has_changes() {
        // Sauvegarder le diff pour inspection
        save_diff(&diff, "tests/snapshots/invoice_list.diff").unwrap();
        panic!("UI regression detected! See tests/snapshots/invoice_list.diff");
    }
}
```

## Résumé

- **Snapshots** : Capture de l'état visuel de l'UI
- **Comparaison** : Détecter les changements visuels
- **Régressions** : Alerter si l'UI change de manière inattendue
- **Référence** : Snapshots de référence versionnés
