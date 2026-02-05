# 2.4 Cas Concrets : ERP, Notes, Outils, Automation

## Mini-ERP pour PME

### Le problème

```rust
/// Situation typique d'une PME
pub struct TypicalSMBSituation {
    pub tools_used: Vec<Tool>,
    pub monthly_cost: Decimal,
    pub pain_points: Vec<&'static str>,
}

impl Default for TypicalSMBSituation {
    fn default() -> Self {
        Self {
            tools_used: vec![
                Tool { name: "Sage/Ciel/EBP", cost: 50, purpose: "Comptabilité" },
                Tool { name: "Salesforce/Hubspot", cost: 80, purpose: "CRM" },
                Tool { name: "Notion/Asana", cost: 30, purpose: "Gestion projets" },
                Tool { name: "Stripe/PayPal", cost: 0, purpose: "Paiements" }, // + commissions
                Tool { name: "Mailchimp", cost: 20, purpose: "Emails" },
                Tool { name: "Calendly", cost: 15, purpose: "RDV" },
            ],
            monthly_cost: Decimal::new(19500, 2),  // 195€/mois minimum
            pain_points: vec![
                "Données dispersées dans 6+ outils",
                "Pas de vue unifiée du client",
                "Export manuel entre outils",
                "Coût cumulé qui explose",
                "Dépendance aux connexions internet",
            ],
        }
    }
}
```

### La solution : ERP local unifié

```rust
/// Architecture d'un mini-ERP local
pub struct LocalERP {
    /// Gestion des clients
    pub crm: CrmModule,
    
    /// Facturation
    pub invoicing: InvoicingModule,
    
    /// Gestion de stock (optionnel)
    pub inventory: Option<InventoryModule>,
    
    /// Projets et tâches
    pub projects: ProjectModule,
    
    /// Reporting
    pub reports: ReportModule,
    
    /// Base de données unifiée
    db: Connection,
}

impl LocalERP {
    /// Vue client 360°
    pub fn customer_360(&self, customer_id: CustomerId) -> Customer360View {
        Customer360View {
            customer: self.crm.get_customer(customer_id),
            invoices: self.invoicing.get_by_customer(customer_id),
            projects: self.projects.get_by_customer(customer_id),
            total_revenue: self.invoicing.total_revenue(customer_id),
            last_interaction: self.crm.last_interaction(customer_id),
        }
    }
}
```

## Application de Notes type Obsidian

### Architecture

```rust
/// Application de notes local-first
pub struct NotesApp {
    /// Dossier racine des notes (Markdown)
    vault_path: PathBuf,
    
    /// Index de recherche
    search_index: SearchIndex,
    
    /// Graph des liens
    link_graph: LinkGraph,
    
    /// Cache des notes parsées
    note_cache: NoteCache,
}

impl NotesApp {
    /// Créer une nouvelle note
    pub fn create_note(&mut self, title: &str) -> Result<Note> {
        let filename = slugify(title);
        let path = self.vault_path.join(format!("{}.md", filename));
        
        let content = format!("# {}\n\nCréé le {}\n", title, Utc::now().format("%Y-%m-%d"));
        std::fs::write(&path, &content)?;
        
        let note = Note {
            path,
            title: title.to_string(),
            content,
            links: vec![],
            backlinks: vec![],
        };
        
        self.index_note(&note)?;
        Ok(note)
    }
    
    /// Recherche instantanée
    pub fn search(&self, query: &str) -> Vec<SearchResult> {
        self.search_index.search(query)
    }
    
    /// Graph des connexions
    pub fn get_graph(&self) -> &LinkGraph {
        &self.link_graph
    }
    
    /// Synchronisation avec filesystem
    pub fn sync_from_disk(&mut self) -> Result<SyncReport> {
        let mut report = SyncReport::default();
        
        for entry in walkdir::WalkDir::new(&self.vault_path)
            .into_iter()
            .filter_map(|e| e.ok())
            .filter(|e| e.path().extension() == Some("md".as_ref()))
        {
            let path = entry.path();
            let content = std::fs::read_to_string(path)?;
            let note = Note::parse(path, &content);
            
            if self.note_cache.needs_update(&note) {
                self.index_note(&note)?;
                report.updated += 1;
            }
        }
        
        Ok(report)
    }
}
```

### Liens bidirectionnels

```rust
pub struct LinkGraph {
    /// Note -> Notes qu'elle référence
    outgoing: HashMap<NoteId, Vec<NoteId>>,
    
    /// Note -> Notes qui la référencent (backlinks)
    incoming: HashMap<NoteId, Vec<NoteId>>,
}

impl LinkGraph {
    pub fn update_links(&mut self, note: &Note) {
        // Parser les liens [[...]]
        let regex = regex::Regex::new(r"\[\[([^\]]+)\]\]").unwrap();
        
        let links: Vec<NoteId> = regex.captures_iter(&note.content)
            .filter_map(|cap| self.resolve_link(&cap[1]))
            .collect();
        
        // Mettre à jour les outgoing
        self.outgoing.insert(note.id, links.clone());
        
        // Mettre à jour les backlinks
        for target in links {
            self.incoming.entry(target)
                .or_default()
                .push(note.id);
        }
    }
    
    pub fn get_backlinks(&self, note_id: NoteId) -> Vec<NoteId> {
        self.incoming.get(&note_id).cloned().unwrap_or_default()
    }
}
```

## Outils de développeur

### Gestionnaire de snippets local

```rust
pub struct SnippetManager {
    snippets: Vec<Snippet>,
    db: Connection,
}

#[derive(Clone)]
pub struct Snippet {
    pub id: SnippetId,
    pub title: String,
    pub language: String,
    pub code: String,
    pub tags: Vec<String>,
    pub usage_count: u32,
}

impl SnippetManager {
    /// Recherche fuzzy rapide
    pub fn fuzzy_search(&self, query: &str) -> Vec<&Snippet> {
        let matcher = fuzzy_matcher::skim::SkimMatcherV2::default();
        
        let mut scored: Vec<_> = self.snippets.iter()
            .filter_map(|s| {
                let score = matcher.fuzzy_match(&s.title, query)
                    .or_else(|| matcher.fuzzy_match(&s.code, query));
                score.map(|sc| (s, sc))
            })
            .collect();
        
        scored.sort_by(|a, b| b.1.cmp(&a.1));
        scored.into_iter().map(|(s, _)| s).collect()
    }
    
    /// Copier dans le presse-papier
    pub fn copy_to_clipboard(&mut self, snippet: &Snippet) {
        clipboard::set_text(&snippet.code).ok();
        
        // Incrémenter le compteur d'utilisation
        self.db.execute(
            "UPDATE snippets SET usage_count = usage_count + 1 WHERE id = ?",
            [snippet.id.0],
        ).ok();
    }
}
```

### Analyseur de logs local

```rust
pub struct LogAnalyzer {
    logs: Vec<LogEntry>,
    filters: LogFilters,
}

impl LogAnalyzer {
    /// Charger des fichiers de log volumineux efficacement
    pub fn load_file(&mut self, path: &Path) -> Result<()> {
        let file = File::open(path)?;
        let reader = BufReader::new(file);
        
        for line in reader.lines() {
            if let Ok(entry) = LogEntry::parse(&line?) {
                self.logs.push(entry);
            }
        }
        
        Ok(())
    }
    
    /// Statistiques en temps réel
    pub fn compute_stats(&self) -> LogStats {
        let filtered = self.apply_filters();
        
        LogStats {
            total: filtered.len(),
            by_level: filtered.iter()
                .fold(HashMap::new(), |mut acc, e| {
                    *acc.entry(e.level).or_insert(0) += 1;
                    acc
                }),
            errors_per_minute: self.compute_error_rate(&filtered),
            top_errors: self.top_errors(&filtered, 10),
        }
    }
    
    /// Recherche avec regex
    pub fn search_regex(&self, pattern: &str) -> Vec<&LogEntry> {
        let regex = regex::Regex::new(pattern).ok();
        
        self.logs.iter()
            .filter(|e| {
                regex.as_ref()
                    .map(|r| r.is_match(&e.message))
                    .unwrap_or(false)
            })
            .collect()
    }
}
```

## Automation et scripts

### Automatisation de tâches récurrentes

```rust
pub struct AutomationEngine {
    workflows: Vec<Workflow>,
    scheduler: Scheduler,
}

pub struct Workflow {
    pub name: String,
    pub trigger: Trigger,
    pub actions: Vec<Action>,
}

pub enum Trigger {
    /// Déclenché par un schedule cron
    Cron(String),
    
    /// Déclenché par un changement de fichier
    FileChange { path: PathBuf, pattern: String },
    
    /// Déclenché manuellement
    Manual,
    
    /// Déclenché par un raccourci global
    Hotkey(KeyCombo),
}

pub enum Action {
    /// Exécuter une commande shell
    RunCommand { cmd: String, args: Vec<String> },
    
    /// Copier des fichiers
    CopyFiles { from: PathBuf, to: PathBuf },
    
    /// Envoyer une notification
    Notify { title: String, body: String },
    
    /// Exécuter un script Lua/Rhai
    RunScript { code: String },
    
    /// Appeler une API HTTP
    HttpRequest { method: String, url: String, body: Option<String> },
}

impl AutomationEngine {
    /// Exemple : backup automatique quotidien
    pub fn daily_backup_workflow() -> Workflow {
        Workflow {
            name: "Backup quotidien".to_string(),
            trigger: Trigger::Cron("0 0 2 * * *".to_string()), // 2h du matin
            actions: vec![
                Action::RunCommand {
                    cmd: "sqlite3".to_string(),
                    args: vec![
                        "data.db".to_string(),
                        ".backup backup.db".to_string(),
                    ],
                },
                Action::CopyFiles {
                    from: PathBuf::from("backup.db"),
                    to: PathBuf::from("/external/backups/"),
                },
                Action::Notify {
                    title: "Backup terminé".to_string(),
                    body: "Sauvegarde quotidienne effectuée".to_string(),
                },
            ],
        }
    }
}
```

## Tableau récapitulatif

| Type d'app | Avantage local | Exemple |
|------------|----------------|---------|
| ERP/CRM | Données unifiées, offline | Mini-ERP PME |
| Notes | Fichiers locaux, liens | Obsidian-like |
| Dev tools | Performance, pas de compte | Snippet manager |
| Logs | Gros volumes, temps réel | Log analyzer |
| Automation | Accès système, fiabilité | Task scheduler |

## Conclusion

Les cas concrets montrent que le desktop local excelle pour :

- **Données sensibles** : CRM, comptabilité, notes personnelles
- **Gros volumes** : Logs, analytics, fichiers
- **Performance critique** : Outils de dev, temps réel
- **Fiabilité** : Automation, scripts critiques
- **Offline** : Tout ce qui doit fonctionner sans internet
