# 22.1 KPI Cards

## Cartes de métriques

```rust
pub struct KpiCard {
    title: String,
    value: String,
    change: Option<f32>,
    icon: String,
    trend: Option<Trend>,
}

#[derive(Clone)]
pub enum Trend {
    Up,
    Down,
    Stable,
}

impl KpiCard {
    pub fn new(title: &str, value: impl ToString) -> Self {
        Self {
            title: title.to_string(),
            value: value.to_string(),
            change: None,
            icon: "📊".to_string(),
            trend: None,
        }
    }
    
    pub fn with_change(mut self, change: f32) -> Self {
        self.change = Some(change);
        self.trend = if change > 0.0 {
            Some(Trend::Up)
        } else if change < 0.0 {
            Some(Trend::Down)
        } else {
            Some(Trend::Stable)
        };
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
                            let (icon, color) = match self.trend {
                                Some(Trend::Up) => ("↑", tokens.colors.success),
                                Some(Trend::Down) => ("↓", tokens.colors.error),
                                Some(Trend::Stable) | None => ("→", tokens.colors.text_muted),
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
```

## Comparaisons périodiques

```rust
pub struct KpiWithComparison {
    current: f64,
    previous: f64,
    period: ComparisonPeriod,
}

pub enum ComparisonPeriod {
    Day,
    Week,
    Month,
    Year,
}

impl KpiWithComparison {
    pub fn change_percent(&self) -> f32 {
        if self.previous == 0.0 {
            return 0.0;
        }
        ((self.current - self.previous) / self.previous * 100.0) as f32
    }
    
    pub fn to_kpi_card(&self, title: &str, icon: &str) -> KpiCard {
        KpiCard::new(title, format!("{:.0}", self.current))
            .with_icon(icon)
            .with_change(self.change_percent())
    }
}
```

## Tendances et indicateurs

```rust
pub struct TrendIndicator {
    values: Vec<f64>,
    period: Duration,
}

impl TrendIndicator {
    pub fn calculate_trend(&self) -> Trend {
        if self.values.len() < 2 {
            return Trend::Stable;
        }
        
        let recent_avg: f64 = self.values.iter().rev().take(5).sum::<f64>() / 5.0;
        let older_avg: f64 = self.values.iter().rev().skip(5).take(5).sum::<f64>() / 5.0;
        
        if recent_avg > older_avg * 1.05 {
            Trend::Up
        } else if recent_avg < older_avg * 0.95 {
            Trend::Down
        } else {
            Trend::Stable
        }
    }
}
```

## Résumé

- **KPI Cards** : Affichage visuel des métriques clés
- **Comparaisons** : Évolution par rapport à la période précédente
- **Tendances** : Indicateurs visuels de direction
- **Icônes** : Identification visuelle rapide
