# Chapitre 22 : Dashboard et Visualisation de Données

## Introduction

Les dashboards transforment les données brutes en insights visuels. Ce chapitre montre comment créer des visualisations performantes avec egui.

---

## 22.1 KPI Cards

```rust
pub struct KpiCard {
    title: String,
    value: String,
    change: Option<f32>,
    icon: String,
}

impl KpiCard {
    pub fn new(title: &str, value: impl ToString) -> Self {
        Self {
            title: title.to_string(),
            value: value.to_string(),
            change: None,
            icon: "📊".to_string(),
        }
    }
    
    pub fn with_change(mut self, change: f32) -> Self {
        self.change = Some(change);
        self
    }
    
    pub fn with_icon(mut self, icon: &str) -> Self {
        self.icon = icon.to_string();
        self
    }
    
    pub fn render(&self, ui: &mut Ui, tokens: &DesignTokens) {
        egui::Frame::none()
            .fill(tokens.colors.bg_elevated)
            .rounding(tokens.radius.md)
            .inner_margin(tokens.spacing.md)
            .stroke(egui::Stroke::new(1.0, tokens.colors.border_default))
            .show(ui, |ui| {
                ui.horizontal(|ui| {
                    // Icône
                    ui.label(egui::RichText::new(&self.icon).size(32.0));
                    
                    ui.vertical(|ui| {
                        // Titre
                        ui.label(
                            egui::RichText::new(&self.title)
                                .size(tokens.typography.size_sm)
                                .color(tokens.colors.text_muted)
                        );
                        
                        // Valeur
                        ui.label(
                            egui::RichText::new(&self.value)
                                .size(tokens.typography.size_2xl)
                                .color(tokens.colors.text_primary)
                                .strong()
                        );
                        
                        // Changement
                        if let Some(change) = self.change {
                            let (icon, color) = if change >= 0.0 {
                                ("↑", tokens.colors.success)
                            } else {
                                ("↓", tokens.colors.error)
                            };
                            
                            ui.label(
                                egui::RichText::new(format!("{} {:.1}%", icon, change.abs()))
                                    .size(tokens.typography.size_sm)
                                    .color(color)
                            );
                        }
                    });
                });
            });
    }
}

// Usage
fn render_kpi_grid(ui: &mut Ui, data: &DashboardData, tokens: &DesignTokens) {
    ui.columns(4, |columns| {
        KpiCard::new("Chiffre d'affaires", format!("{:.0} €", data.revenue))
            .with_icon("💰")
            .with_change(data.revenue_change)
            .render(&mut columns[0], tokens);
        
        KpiCard::new("Factures", data.invoice_count.to_string())
            .with_icon("📄")
            .render(&mut columns[1], tokens);
        
        KpiCard::new("Clients", data.customer_count.to_string())
            .with_icon("👥")
            .with_change(data.customer_change)
            .render(&mut columns[2], tokens);
        
        KpiCard::new("Impayés", format!("{:.0} €", data.unpaid_amount))
            .with_icon("⚠️")
            .render(&mut columns[3], tokens);
    });
}
```

---

## 22.2 Graphiques avec egui_plot

```rust
use egui_plot::{Plot, Line, PlotPoints, Legend};

pub fn render_revenue_chart(ui: &mut Ui, data: &[MonthlyRevenue], tokens: &DesignTokens) {
    let points: PlotPoints = data.iter()
        .enumerate()
        .map(|(i, m)| [i as f64, m.amount.to_f64().unwrap_or(0.0)])
        .collect();
    
    let line = Line::new(points)
        .color(egui::Color32::from_rgb(59, 130, 246))
        .name("Chiffre d'affaires");
    
    Plot::new("revenue_chart")
        .legend(Legend::default())
        .height(200.0)
        .show_axes([true, true])
        .show(ui, |plot_ui| {
            plot_ui.line(line);
        });
}

pub fn render_pie_chart(ui: &mut Ui, data: &[CategoryData], tokens: &DesignTokens) {
    let total: f64 = data.iter().map(|d| d.value).sum();
    let rect = ui.available_rect_before_wrap();
    let center = rect.center();
    let radius = rect.width().min(rect.height()) / 2.0 - 10.0;
    
    let colors = [
        egui::Color32::from_rgb(59, 130, 246),
        egui::Color32::from_rgb(34, 197, 94),
        egui::Color32::from_rgb(234, 179, 8),
        egui::Color32::from_rgb(239, 68, 68),
        egui::Color32::from_rgb(139, 92, 246),
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

---

## 22.3 Tableau de Bord Complet

```rust
pub struct DashboardScreen {
    data: DashboardData,
    refresh_interval: Duration,
    last_refresh: Instant,
}

impl DashboardScreen {
    pub fn render(&mut self, ui: &mut Ui, tokens: &DesignTokens) {
        // Auto-refresh
        if self.last_refresh.elapsed() > self.refresh_interval {
            self.refresh_data();
            self.last_refresh = Instant::now();
        }
        
        egui::ScrollArea::vertical().show(ui, |ui| {
            // KPIs
            ui.heading("Vue d'ensemble");
            ui.add_space(tokens.spacing.md);
            self.render_kpis(ui, tokens);
            
            ui.add_space(tokens.spacing.xl);
            
            // Graphiques
            ui.columns(2, |columns| {
                columns[0].vertical(|ui| {
                    ui.heading("Évolution du CA");
                    render_revenue_chart(ui, &self.data.monthly_revenue, tokens);
                });
                
                columns[1].vertical(|ui| {
                    ui.heading("Répartition par catégorie");
                    render_pie_chart(ui, &self.data.categories, tokens);
                });
            });
            
            ui.add_space(tokens.spacing.xl);
            
            // Dernières factures
            ui.heading("Dernières factures");
            self.render_recent_invoices(ui, tokens);
        });
    }
}
```

---

## Résumé

- **KPI Cards** : Métriques clés avec tendances
- **Charts** : Visualisations avec egui_plot
- **Auto-refresh** : Données temps réel
- **Layout responsive** : Colonnes adaptatives
