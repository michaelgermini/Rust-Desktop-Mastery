# 22.3 Temps Réel

## Mise à jour automatique

```rust
pub struct DashboardScreen {
    data: DashboardData,
    refresh_interval: Duration,
    last_refresh: Instant,
    auto_refresh: bool,
}

impl DashboardScreen {
    pub fn render(&mut self, ui: &mut Ui, tokens: &DesignTokens) {
        // Auto-refresh
        if self.auto_refresh && self.last_refresh.elapsed() > self.refresh_interval {
            self.refresh_data();
            self.last_refresh = Instant::now();
        }
        
        // Indicateur de dernière mise à jour
        ui.horizontal(|ui| {
            ui.label(format!("Dernière mise à jour: {}", 
                self.last_refresh.elapsed().as_secs()));
            
            ui.checkbox(&mut self.auto_refresh, "Auto-refresh");
        });
        
        // Contenu du dashboard
        self.render_content(ui, tokens);
    }
    
    fn refresh_data(&mut self) {
        // Charger les données depuis la base
        // ...
    }
}
```

## Streaming de données

```rust
use crossbeam_channel::Receiver;

pub struct StreamingDashboard {
    data_receiver: Receiver<DashboardUpdate>,
    current_data: DashboardData,
}

pub enum DashboardUpdate {
    KpiUpdated { kpi: String, value: f64 },
    ChartDataUpdated { series: String, points: Vec<(f64, f64)> },
}

impl StreamingDashboard {
    pub fn render(&mut self, ui: &mut Ui, tokens: &DesignTokens) {
        // Traiter les mises à jour en streaming
        while let Ok(update) = self.data_receiver.try_recv() {
            match update {
                DashboardUpdate::KpiUpdated { kpi, value } => {
                    self.current_data.update_kpi(&kpi, value);
                }
                DashboardUpdate::ChartDataUpdated { series, points } => {
                    self.current_data.update_series(&series, points);
                }
            }
        }
        
        // Rendre avec les données à jour
        self.render_content(ui, tokens);
        
        // Demander un repaint pour les animations
        ui.ctx().request_repaint();
    }
}
```

## Performance

```rust
impl DashboardScreen {
    pub fn render_optimized(&mut self, ui: &mut Ui, tokens: &DesignTokens) {
        // Ne recalculer que si nécessaire
        if self.needs_recalculation() {
            self.recalculate_metrics();
        }
        
        // Utiliser des widgets optimisés
        egui::ScrollArea::vertical()
            .auto_shrink([false, false])
            .show(ui, |ui| {
                // KPIs (calculs légers)
                self.render_kpis(ui, tokens);
                
                // Charts (calculs plus lourds, mais lazy)
                if ui.visible() {
                    self.render_charts(ui, tokens);
                }
            });
    }
    
    fn needs_recalculation(&self) -> bool {
        // Vérifier si les données ont changé
        // ...
    }
}
```

## Résumé

- **Auto-refresh** : Mise à jour périodique automatique
- **Streaming** : Mises à jour en temps réel via channels
- **Performance** : Recalcul seulement si nécessaire
- **Repaint** : Demander le repaint pour les animations
