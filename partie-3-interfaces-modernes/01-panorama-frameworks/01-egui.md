# 8.1 egui : Immediate Mode Simple et Puissant

## Présentation

egui (prononcé "e-gooey") est une bibliothèque UI en mode immédiat, portable et facile à utiliser.

```toml
# Cargo.toml
[dependencies]
eframe = "0.27"  # egui + framework d'application
egui = "0.27"
```

## Exemple minimal

```rust
use eframe::egui;

fn main() -> eframe::Result<()> {
    eframe::run_native(
        "Mon Application",
        eframe::NativeOptions::default(),
        Box::new(|_cc| Ok(Box::new(MyApp::default()))),
    )
}

#[derive(Default)]
struct MyApp {
    name: String,
    counter: i32,
}

impl eframe::App for MyApp {
    fn update(&mut self, ctx: &egui::Context, _frame: &mut eframe::Frame) {
        egui::CentralPanel::default().show(ctx, |ui| {
            ui.heading("Hello egui!");
            
            ui.horizontal(|ui| {
                ui.label("Nom:");
                ui.text_edit_singleline(&mut self.name);
            });
            
            if ui.button("Incrémenter").clicked() {
                self.counter += 1;
            }
            
            ui.label(format!("Compteur: {}", self.counter));
        });
    }
}
```

## Forces et faiblesses

| Forces | Faiblesses |
|--------|------------|
| Très simple à apprendre | Style visuel distinctif |
| Portable (Web, Desktop, Mobile) | Moins de widgets natifs |
| Rendu GPU efficace | Accessibilité limitée |
| Excellente documentation | Mode immédiat parfois déroutant |
| Idéal pour outils dev | |

## Cas d'usage idéaux

- **Outils de développement** : Debuggers, profilers, éditeurs
- **Éditeurs de jeux** : Unity-like editors
- **Applications internes** : Dashboards, outils admin
- **Prototypage rapide** : UI fonctionnelle rapidement
- **Dashboards** : Visualisations de données

## Pourquoi egui pour ce livre ?

egui a été choisi comme framework principal pour ce livre car :

1. **Simplicité** : Courbe d'apprentissage douce
2. **Performance** : Rendu GPU efficace
3. **Portabilité** : Fonctionne partout
4. **Maturité** : Écosystème stable et documenté
5. **Fit desktop** : Parfait pour applications desktop natives

## Résumé

egui est le choix idéal pour les applications desktop Rust qui privilégient :
- La simplicité de développement
- La performance native
- La portabilité
- Un style visuel moderne mais fonctionnel
