# 31.4 Gestion des Bugs

## Crash reporting

```rust
/// Capture automatique des erreurs
pub fn setup_panic_handler() {
    std::panic::set_hook(Box::new(|panic_info| {
        let location = panic_info.location().map(|l| {
            format!("{}:{}:{}", l.file(), l.line(), l.column())
        }).unwrap_or_default();
        
        let message = panic_info.payload()
            .downcast_ref::<&str>()
            .map(|s| s.to_string())
            .or_else(|| panic_info.payload()
                .downcast_ref::<String>()
                .cloned())
            .unwrap_or_else(|| "Unknown panic".to_string());
        
        let report = CrashReport {
            message,
            location,
            backtrace: std::backtrace::Backtrace::capture().to_string(),
            app_version: env!("CARGO_PKG_VERSION").to_string(),
            os: std::env::consts::OS.to_string(),
            timestamp: Utc::now(),
        };
        
        // Sauvegarder localement
        save_crash_report(&report).ok();
        
        // Afficher un message à l'utilisateur
        eprintln!("Une erreur s'est produite. Un rapport a été créé.");
    }));
}

pub struct CrashReport {
    pub message: String,
    pub location: String,
    pub backtrace: String,
    pub app_version: String,
    pub os: String,
    pub timestamp: DateTime<Utc>,
}

fn save_crash_report(report: &CrashReport) -> Result<()> {
    let crash_dir = dirs::data_dir()
        .unwrap_or_else(|| PathBuf::from("."))
        .join("mon-app")
        .join("crashes");
    
    std::fs::create_dir_all(&crash_dir)?;
    
    let filename = format!("crash-{}.json", report.timestamp.timestamp());
    let path = crash_dir.join(filename);
    
    let content = serde_json::to_string_pretty(report)?;
    std::fs::write(path, content)?;
    
    Ok(())
}
```

## Reproduction de bugs

```rust
pub struct BugReporter {
    pub steps_to_reproduce: Vec<String>,
    pub expected_behavior: String,
    pub actual_behavior: String,
    pub screenshots: Vec<Vec<u8>>,
}

impl BugReporter {
    pub fn capture_bug_context(&self) -> BugContext {
        BugContext {
            app_state: self.serialize_app_state(),
            recent_actions: self.get_recent_actions(),
            system_info: self.get_system_info(),
        }
    }
    
    fn serialize_app_state(&self) -> String {
        // Sérialiser l'état de l'application (sans données sensibles)
        // ...
        "{}".to_string()
    }
    
    fn get_recent_actions(&self) -> Vec<String> {
        // Retourner les dernières actions utilisateur
        vec![]
    }
    
    fn get_system_info(&self) -> SystemInfo {
        SystemInfo {
            os: std::env::consts::OS.to_string(),
            os_version: get_os_version(),
            app_version: env!("CARGO_PKG_VERSION").to_string(),
        }
    }
}
```

## Suivi des issues

```rust
pub struct IssueTracker {
    pub api_url: String,
    pub api_key: String,
}

impl IssueTracker {
    pub fn create_issue(&self, bug: &BugReport) -> Result<String> {
        let client = reqwest::Client::new();
        
        let response = client
            .post(&format!("{}/api/issues", self.api_url))
            .header("Authorization", &format!("Bearer {}", self.api_key))
            .json(bug)
            .send()?;
        
        let issue_id: IssueResponse = response.json()?;
        Ok(issue_id.id)
    }
}

pub struct BugReport {
    pub title: String,
    pub description: String,
    pub steps_to_reproduce: Vec<String>,
    pub crash_report: Option<CrashReport>,
    pub system_info: SystemInfo,
}
```

## Résumé

- **Crash reporting** : Capture automatique des panics
- **Reproduction** : Contexte pour reproduire le bug
- **Suivi** : Intégration avec système de tickets
- **Amélioration** : Données pour corriger les bugs
