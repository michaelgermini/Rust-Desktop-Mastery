# Chapitre 24 : Cas d'Étude Complet

## Introduction

Ce chapitre assemble tous les concepts précédents dans une application complète : un outil de gestion de notes avec recherche intelligente.

---

## 24.1 Spécifications

```
Application : NoteVault
- Gestion de notes en Markdown
- Stockage local SQLite
- Recherche full-text et sémantique
- Export PDF
- Thèmes light/dark
- Raccourcis clavier
```

---

## 24.2 Structure du Projet

```
notevault/
├── Cargo.toml
├── crates/
│   ├── core/           # Entités, logique métier
│   ├── application/    # Use cases
│   ├── infrastructure/ # SQLite, fichiers
│   └── ui/            # Interface egui
└── src/
    └── main.rs
```

---

## 24.3 Code Principal

```rust
// src/main.rs
use notevault_ui::App;
use notevault_infrastructure::Database;
use std::sync::Arc;

fn main() -> eframe::Result<()> {
    // Configuration
    let data_dir = dirs::data_dir().unwrap().join("notevault");
    std::fs::create_dir_all(&data_dir).ok();
    
    // Base de données
    let db = Arc::new(
        Database::open(&data_dir.join("notes.db"))
            .expect("Failed to open database")
    );
    db.run_migrations().expect("Failed to run migrations");
    
    // Moteur de recherche
    let search = Arc::new(
        SearchEngine::new(&data_dir.join("search_index"))
            .expect("Failed to create search index")
    );
    
    // Application
    let options = eframe::NativeOptions {
        viewport: egui::ViewportBuilder::default()
            .with_inner_size([1200.0, 800.0])
            .with_min_inner_size([800.0, 600.0])
            .with_title("NoteVault"),
        ..Default::default()
    };
    
    eframe::run_native(
        "NoteVault",
        options,
        Box::new(|cc| {
            // Charger les polices custom
            setup_fonts(&cc.egui_ctx);
            
            // Créer l'app
            Ok(Box::new(App::new(db, search)))
        }),
    )
}
```

---

## 24.4 Application UI

```rust
// crates/ui/src/app.rs
pub struct App {
    // Dépendances
    db: Arc<Database>,
    search: Arc<SearchEngine>,
    
    // État
    notes: Vec<Note>,
    selected_note: Option<NoteId>,
    editor_content: String,
    
    // UI State
    search_query: String,
    show_settings: bool,
    theme: ThemeManager,
    toasts: ToastManager,
}

impl eframe::App for App {
    fn update(&mut self, ctx: &egui::Context, _frame: &mut eframe::Frame) {
        // Raccourcis globaux
        self.handle_shortcuts(ctx);
        
        // Appliquer le thème
        apply_theme_to_egui(ctx, self.theme.tokens());
        
        // Sidebar
        egui::SidePanel::left("sidebar")
            .default_width(250.0)
            .show(ctx, |ui| {
                self.render_sidebar(ui);
            });
        
        // Contenu principal
        egui::CentralPanel::default().show(ctx, |ui| {
            if let Some(note_id) = self.selected_note {
                self.render_editor(ui, note_id);
            } else {
                self.render_welcome(ui);
            }
        });
        
        // Modales
        if self.show_settings {
            self.render_settings_modal(ctx);
        }
        
        // Toasts
        self.toasts.render(ctx, self.theme.tokens());
    }
}

impl App {
    fn handle_shortcuts(&mut self, ctx: &egui::Context) {
        ctx.input(|i| {
            // Cmd+N : Nouvelle note
            if i.modifiers.command && i.key_pressed(egui::Key::N) {
                self.create_new_note();
            }
            
            // Cmd+S : Sauvegarder
            if i.modifiers.command && i.key_pressed(egui::Key::S) {
                self.save_current_note();
            }
            
            // Cmd+F : Rechercher
            if i.modifiers.command && i.key_pressed(egui::Key::F) {
                self.focus_search();
            }
            
            // Cmd+, : Paramètres
            if i.modifiers.command && i.key_pressed(egui::Key::Comma) {
                self.show_settings = true;
            }
        });
    }
    
    fn render_sidebar(&mut self, ui: &mut Ui) {
        let tokens = self.theme.tokens();
        
        // Recherche
        ui.horizontal(|ui| {
            let response = ui.add(
                egui::TextEdit::singleline(&mut self.search_query)
                    .hint_text("🔍 Rechercher...")
            );
            
            if response.changed() {
                self.filter_notes();
            }
        });
        
        ui.add_space(tokens.spacing.md);
        
        // Bouton nouvelle note
        if Button::new("+ Nouvelle note")
            .variant(ButtonVariant::Primary)
            .full_width()
            .show(ui, tokens)
            .clicked()
        {
            self.create_new_note();
        }
        
        ui.add_space(tokens.spacing.md);
        ui.separator();
        ui.add_space(tokens.spacing.sm);
        
        // Liste des notes
        egui::ScrollArea::vertical().show(ui, |ui| {
            for note in &self.notes {
                let is_selected = self.selected_note == Some(note.id);
                
                let response = ui.selectable_label(is_selected, &note.title);
                
                if response.clicked() {
                    self.select_note(note.id);
                }
                
                // Menu contextuel
                response.context_menu(|ui| {
                    if ui.button("Supprimer").clicked() {
                        self.delete_note(note.id);
                        ui.close_menu();
                    }
                    if ui.button("Exporter PDF").clicked() {
                        self.export_note_pdf(note.id);
                        ui.close_menu();
                    }
                });
            }
        });
    }
    
    fn render_editor(&mut self, ui: &mut Ui, note_id: NoteId) {
        let tokens = self.theme.tokens();
        
        // Toolbar éditeur
        ui.horizontal(|ui| {
            if ui.button("💾 Sauvegarder").clicked() {
                self.save_current_note();
            }
            
            ui.separator();
            
            if ui.button("📄 Export PDF").clicked() {
                self.export_note_pdf(note_id);
            }
        });
        
        ui.add_space(tokens.spacing.md);
        
        // Éditeur Markdown
        egui::ScrollArea::vertical().show(ui, |ui| {
            ui.add(
                egui::TextEdit::multiline(&mut self.editor_content)
                    .font(egui::FontId::monospace(tokens.typography.size_md))
                    .desired_width(f32::INFINITY)
                    .desired_rows(30)
            );
        });
    }
}
```

---

## 24.5 Fonctionnalités Complètes

```rust
impl App {
    fn create_new_note(&mut self) {
        let note = Note::new("Nouvelle note");
        let id = self.db.save_note(&note).expect("Failed to save note");
        
        self.notes.insert(0, note);
        self.selected_note = Some(id);
        self.editor_content = String::new();
        
        // Indexer pour la recherche
        self.search.index_document(&id.to_string(), "Nouvelle note", "");
    }
    
    fn save_current_note(&mut self) {
        if let Some(note_id) = self.selected_note {
            if let Some(note) = self.notes.iter_mut().find(|n| n.id == note_id) {
                note.content = self.editor_content.clone();
                note.updated_at = Utc::now();
                
                self.db.save_note(note).expect("Failed to save");
                
                // Mettre à jour l'index de recherche
                self.search.index_document(
                    &note_id.to_string(),
                    &note.title,
                    &note.content,
                );
                
                self.toasts.success("Note sauvegardée");
            }
        }
    }
    
    fn export_note_pdf(&self, note_id: NoteId) {
        if let Some(note) = self.notes.iter().find(|n| n.id == note_id) {
            let pdf = PdfGenerator::new().generate_note(note);
            
            if let Some(path) = save_file_dialog("Exporter PDF", &note.title, "pdf") {
                std::fs::write(&path, pdf).expect("Failed to write PDF");
                self.toasts.success(format!("Exporté vers {}", path.display()));
                
                // Ouvrir le PDF
                open_with_default_app(&path).ok();
            }
        }
    }
}
```

---

## Résumé

Cette application démontre :
- **Clean Architecture** : Séparation claire des responsabilités
- **UI professionnelle** : Design system, thèmes, raccourcis
- **Persistence robuste** : SQLite avec migrations
- **Recherche avancée** : Full-text et sémantique
- **Export** : Génération PDF
- **UX soignée** : Toasts, autosave, feedback

C'est une base solide pour tout outil de productivité desktop.
