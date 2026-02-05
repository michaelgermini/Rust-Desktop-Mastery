# 25.2 Lazy Loading

## Chargement différé

```rust
pub struct LazyApp {
    // Chargé immédiatement (essentiel pour le premier écran)
    ui_state: UiState,
    theme: Theme,
    
    // Chargé en arrière-plan
    data: LoadState<AppData>,
    search_index: LoadState<SearchIndex>,
}

impl LazyApp {
    pub fn new() -> Self {
        Self {
            // L'UI est prête immédiatement
            ui_state: UiState::default(),
            theme: Theme::default(),
            
            // Les données seront chargées async
            data: LoadState::Loading { started_at: Instant::now() },
            search_index: LoadState::Idle,
        }
    }
    
    pub fn start_background_loading(&self, runtime: &Runtime) {
        let data_tx = self.data_channel.clone();
        
        runtime.spawn(async move {
            // Charger les données essentielles d'abord
            let recent_docs = load_recent_documents().await;
            data_tx.send(DataEvent::RecentLoaded(recent_docs)).ok();
            
            // Puis les données secondaires
            let all_docs = load_all_documents().await;
            data_tx.send(DataEvent::AllLoaded(all_docs)).ok();
        });
    }
}
```

## Pagination

```rust
pub struct PaginatedList<T> {
    items: Vec<T>,
    page_size: usize,
    current_page: usize,
}

impl<T> PaginatedList<T> {
    pub fn new(items: Vec<T>, page_size: usize) -> Self {
        Self {
            items,
            page_size,
            current_page: 0,
        }
    }
    
    pub fn current_page_items(&self) -> &[T] {
        let start = self.current_page * self.page_size;
        let end = (start + self.page_size).min(self.items.len());
        &self.items[start..end]
    }
    
    pub fn total_pages(&self) -> usize {
        (self.items.len() + self.page_size - 1) / self.page_size
    }
    
    pub fn render(&mut self, ui: &mut Ui, tokens: &DesignTokens, render_item: impl Fn(&mut Ui, &T)) {
        // Items de la page courante
        for item in self.current_page_items() {
            render_item(ui, item);
        }
        
        // Pagination
        ui.horizontal(|ui| {
            if ui.button("◀ Précédent").clicked() && self.current_page > 0 {
                self.current_page -= 1;
            }
            
            ui.label(format!("Page {} / {}", self.current_page + 1, self.total_pages()));
            
            if ui.button("Suivant ▶").clicked() && self.current_page < self.total_pages() - 1 {
                self.current_page += 1;
            }
        });
    }
}
```

## Virtual scrolling

```rust
use egui::ScrollArea;

pub fn render_virtual_list<T>(
    ui: &mut Ui,
    items: &[T],
    item_height: f32,
    render_item: impl Fn(&mut Ui, &T),
) {
    let available_height = ui.available_height();
    let visible_count = (available_height / item_height).ceil() as usize + 2; // +2 pour buffer
    
    ScrollArea::vertical()
        .auto_shrink([false, false])
        .show(ui, |ui| {
            // Calculer la plage visible
            let scroll_offset = ui.clip_rect().min.y;
            let start_index = (scroll_offset / item_height).floor() as usize;
            let end_index = (start_index + visible_count).min(items.len());
            
            // Espacement avant les items visibles
            ui.add_space(start_index as f32 * item_height);
            
            // Rendre seulement les items visibles
            for item in &items[start_index..end_index] {
                let item_rect = ui.allocate_exact_size(
                    egui::vec2(ui.available_width(), item_height),
                    egui::Sense::hover(),
                ).0;
                
                if ui.is_rect_visible(item_rect) {
                    render_item(ui, item);
                }
            }
            
            // Espacement après les items visibles
            let remaining = items.len() - end_index;
            ui.add_space(remaining as f32 * item_height);
        });
}
```

## Résumé

- **Chargement différé** : UI immédiate, données en arrière-plan
- **Pagination** : Charger seulement une page à la fois
- **Virtual scrolling** : Rendre seulement les items visibles
- **Performance** : Réduire la charge initiale
