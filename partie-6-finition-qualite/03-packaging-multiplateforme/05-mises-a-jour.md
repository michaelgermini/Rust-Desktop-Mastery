# 27.5 Mises à Jour Automatiques

## Système d'auto-update

```rust
use self_update::backends::github::Update;

pub struct AutoUpdater {
    repo_owner: String,
    repo_name: String,
    current_version: String,
}

impl AutoUpdater {
    pub fn new(repo_owner: &str, repo_name: &str) -> Self {
        Self {
            repo_owner: repo_owner.to_string(),
            repo_name: repo_name.to_string(),
            current_version: env!("CARGO_PKG_VERSION").to_string(),
        }
    }
    
    pub fn check_for_updates(&self) -> Result<Option<UpdateInfo>> {
        let status = Update::configure()
            .repo_owner(&self.repo_owner)
            .repo_name(&self.repo_name)
            .bin_name("mon-app")
            .show_download_progress(true)
            .current_version(&self.current_version)
            .build()?
            .update()?;
        
        if status.updated() {
            Ok(Some(UpdateInfo {
                version: status.version().to_string(),
                release_notes: status.body().to_string(),
            }))
        } else {
            Ok(None)
        }
    }
    
    pub fn update_now(&self) -> Result<()> {
        Update::configure()
            .repo_owner(&self.repo_owner)
            .repo_name(&self.repo_name)
            .bin_name("mon-app")
            .show_download_progress(true)
            .current_version(&self.current_version)
            .build()?
            .update()?;
        
        Ok(())
    }
}

pub struct UpdateInfo {
    pub version: String,
    pub release_notes: String,
}
```

## Backend GitHub

```rust
// Configuration pour GitHub Releases
pub fn setup_github_updates(repo_owner: &str, repo_name: &str) -> Update {
    Update::configure()
        .repo_owner(repo_owner)
        .repo_name(repo_name)
        .bin_name("mon-app")
        .target("x86_64-pc-windows-msvc")  // Ou autre target
        .show_download_progress(true)
        .current_version(env!("CARGO_PKG_VERSION"))
        .build()
        .expect("Failed to configure updater")
}
```

## Notifications utilisateur

```rust
pub struct UpdateNotifier {
    updater: AutoUpdater,
    last_check: Option<Instant>,
    check_interval: Duration,
}

impl UpdateNotifier {
    pub fn check_periodically(&mut self) -> Option<UpdateInfo> {
        if let Some(last) = self.last_check {
            if last.elapsed() < self.check_interval {
                return None;
            }
        }
        
        self.last_check = Some(Instant::now());
        
        match self.updater.check_for_updates() {
            Ok(Some(info)) => Some(info),
            _ => None,
        }
    }
    
    pub fn show_update_dialog(&self, ctx: &egui::Context, info: &UpdateInfo) {
        let mut show = true;
        
        egui::Window::new("Mise à jour disponible")
            .open(&mut show)
            .show(ctx, |ui| {
                ui.heading(format!("Version {} disponible", info.version));
                ui.separator();
                ui.label(&info.release_notes);
                ui.add_space(10.0);
                
                ui.horizontal(|ui| {
                    if ui.button("Mettre à jour maintenant").clicked() {
                        self.updater.update_now().ok();
                        std::process::exit(0);  // Redémarrer
                    }
                    if ui.button("Plus tard").clicked() {
                        show = false;
                    }
                });
            });
    }
}
```

## Résumé

- **self_update** : Crate pour auto-update multi-plateforme
- **GitHub Releases** : Backend simple et gratuit
- **Notifications** : Informer l'utilisateur des mises à jour
- **Installation** : Mise à jour automatique en arrière-plan
