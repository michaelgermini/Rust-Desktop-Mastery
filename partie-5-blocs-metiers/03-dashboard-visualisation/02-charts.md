# 22.2 Charts

## Graphiques avec egui_plot

```rust
use egui_plot::{Plot, Line, PlotPoints, Legend};

pub fn render_revenue_chart(ui: &mut Ui, data: &[MonthlyRevenue], tokens: &DesignTokens) {
    let points: PlotPoints = data.iter()
        .enumerate()
        .map(|(i, m)| [i as f64, m.amount.to_f64().unwrap_or(0.0)])
        .collect();
    
    let line = Line::new(points)
        .color(tokens.colors.primary)
        .name("Chiffre d'affaires")
        .width(2.0);
    
    Plot::new("revenue_chart")
        .legend(Legend::default().position(egui_plot::Corner::RightTop))
        .height(200.0)
        .show_axes([true, true])
        .show(ui, |plot_ui| {
            plot_ui.line(line);
        });
}
```

## Types de graphiques

### Graphique en ligne

```rust
pub fn render_line_chart(ui: &mut Ui, data: &[(f64, f64)], tokens: &DesignTokens) {
    let points: PlotPoints = data.iter().map(|(x, y)| [*x, *y]).collect();
    
    let line = Line::new(points)
        .color(tokens.colors.primary)
        .width(2.0);
    
    Plot::new("line_chart")
        .height(300.0)
        .show(ui, |plot_ui| {
            plot_ui.line(line);
        });
}
```

### Graphique en barres

```rust
use egui_plot::{Bar, BarChart};

pub fn render_bar_chart(ui: &mut Ui, data: &[(String, f64)], tokens: &DesignTokens) {
    let bars: Vec<Bar> = data.iter()
        .enumerate()
        .map(|(i, (label, value))| {
            Bar::new(i as f64, *value)
                .name(label.clone())
                .fill_color(tokens.colors.primary)
        })
        .collect();
    
    let chart = BarChart::new(bars);
    
    Plot::new("bar_chart")
        .height(300.0)
        .show(ui, |plot_ui| {
            plot_ui.bar_chart(chart);
        });
}
```

### Graphique en secteurs (pie)

```rust
pub fn render_pie_chart(ui: &mut Ui, data: &[CategoryData], tokens: &DesignTokens) {
    let total: f64 = data.iter().map(|d| d.value).sum();
    let rect = ui.available_rect_before_wrap();
    let center = rect.center();
    let radius = rect.width().min(rect.height()) / 2.0 - 10.0;
    
    let colors = [
        tokens.colors.primary,
        tokens.colors.success,
        tokens.colors.warning,
        tokens.colors.error,
        tokens.colors.info,
    ];
    
    let mut start_angle = 0.0;
    
    for (i, item) in data.iter().enumerate() {
        let sweep = (item.value / total) * std::f64::consts::TAU;
        let color = colors[i % colors.len()];
        
        // Dessiner le segment
        let points: Vec<_> = (0..=32).map(|j| {
            let angle = start_angle + (j as f64 / 32.0) * sweep;
            egui::pos2(
                center.x + (angle.cos() as f32) * radius,
                center.y + (angle.sin() as f32) * radius,
            )
        }).collect();
        
        let mut polygon = vec![center];
        polygon.extend(points);
        
        ui.painter().add(egui::Shape::convex_polygon(
            polygon,
            color,
            egui::Stroke::NONE,
        ));
        
        start_angle += sweep;
    }
}
```

## Interactivité

```rust
use egui_plot::{PlotPoint, HLine, VLine};

pub fn render_interactive_chart(ui: &mut Ui, data: &[DataPoint]) {
    Plot::new("interactive_chart")
        .height(300.0)
        .allow_zoom(true)
        .allow_drag(true)
        .allow_scroll(true)
        .show(ui, |plot_ui| {
            // Ligne principale
            let points: PlotPoints = data.iter().map(|d| [d.x, d.y]).collect();
            plot_ui.line(Line::new(points));
            
            // Ligne de référence
            plot_ui.hline(HLine::new(100.0).color(egui::Color32::GRAY));
            
            // Point de hover
            if let Some(hover_pos) = plot_ui.pointer_coordinate() {
                plot_ui.vline(VLine::new(hover_pos.x).color(egui::Color32::GRAY));
                
                // Afficher la valeur au hover
                ui.label(format!("Valeur: {:.2}", hover_pos.y));
            }
        });
}
```

## Résumé

- **egui_plot** : Bibliothèque de graphiques pour egui
- **Types** : Ligne, barres, secteurs
- **Interactivité** : Zoom, drag, hover
- **Personnalisation** : Couleurs, légendes, axes
