# 22.4 UX Orientée Data

## Présentation des données

```rust
pub struct DataPresentation {
    format: DataFormat,
    precision: usize,
}

pub enum DataFormat {
    Number,
    Currency,
    Percentage,
    Duration,
}

impl DataPresentation {
    pub fn format_value(&self, value: f64) -> String {
        match self.format {
            DataFormat::Number => format!("{:.precision$}", value, precision = self.precision),
            DataFormat::Currency => format!("{:.2} €", value),
            DataFormat::Percentage => format!("{:.1}%", value * 100.0),
            DataFormat::Duration => format_duration(value),
        }
    }
}

fn format_duration(seconds: f64) -> String {
    if seconds < 60.0 {
        format!("{:.0}s", seconds)
    } else if seconds < 3600.0 {
        format!("{:.0}m", seconds / 60.0)
    } else {
        format!("{:.1}h", seconds / 3600.0)
    }
}
```

## Filtres et drill-down

```rust
pub struct FilterableDashboard {
    filters: DashboardFilters,
    data: DashboardData,
}

pub struct DashboardFilters {
    date_range: Option<(DateTime<Utc>, DateTime<Utc>)>,
    category: Option<String>,
    min_value: Option<f64>,
}

impl FilterableDashboard {
    pub fn render(&mut self, ui: &mut Ui, tokens: &DesignTokens) {
        // Barre de filtres
        ui.horizontal(|ui| {
            ui.label("Filtres:");
            
            // Filtre de date
            if ui.button("📅 Période").clicked() {
                self.show_date_picker = true;
            }
            
            // Filtre de catégorie
            egui::ComboBox::from_id_source("category_filter")
                .selected_text(self.filters.category.as_deref().unwrap_or("Toutes"))
                .show_ui(ui, |ui| {
                    ui.selectable_value(&mut self.filters.category, None, "Toutes");
                    for cat in &self.categories {
                        ui.selectable_value(&mut self.filters.category, Some(cat.clone()), cat);
                    }
                });
        });
        
        // Appliquer les filtres
        let filtered_data = self.apply_filters();
        
        // Rendre avec les données filtrées
        self.render_content(ui, &filtered_data, tokens);
    }
    
    fn apply_filters(&self) -> DashboardData {
        let mut data = self.data.clone();
        
        if let Some((start, end)) = self.filters.date_range {
            data.filter_by_date_range(start, end);
        }
        
        if let Some(ref category) = self.filters.category {
            data.filter_by_category(category);
        }
        
        if let Some(min) = self.filters.min_value {
            data.filter_by_min_value(min);
        }
        
        data
    }
}
```

## Export des visualisations

```rust
impl DashboardScreen {
    pub fn export_chart(&self, chart_id: &str) -> Result<Vec<u8>> {
        // Récupérer les données du graphique
        let chart_data = self.get_chart_data(chart_id)?;
        
        // Générer une image PNG du graphique
        let image = self.render_chart_to_image(&chart_data)?;
        
        // Encoder en PNG
        let mut png_data = Vec::new();
        let encoder = image::codecs::png::PngEncoder::new(&mut png_data);
        encoder.encode(
            &image.as_raw(),
            image.width(),
            image.height(),
            image::ColorType::Rgba8,
        )?;
        
        Ok(png_data)
    }
    
    pub fn export_dashboard_pdf(&self) -> Result<Vec<u8>> {
        // Générer un PDF avec tous les graphiques
        let pdf_generator = PdfGenerator::new();
        
        // Ajouter chaque graphique comme image
        for chart_id in &self.chart_ids {
            let chart_image = self.export_chart(chart_id)?;
            pdf_generator.add_image(chart_id, chart_image)?;
        }
        
        pdf_generator.generate()
    }
}
```

## Résumé

- **Formatage** : Présentation adaptée (devise, pourcentage, durée)
- **Filtres** : Restreindre les données affichées
- **Drill-down** : Navigation dans les détails
- **Export** : Exporter les visualisations en PNG/PDF
