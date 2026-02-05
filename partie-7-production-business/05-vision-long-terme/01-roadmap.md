# 33.1 Roadmap Produit

## Planification des versions

```rust
/// Gestion de la roadmap
pub struct ProductRoadmap {
    pub milestones: Vec<Milestone>,
    pub current_version: Version,
}

pub struct Milestone {
    pub version: Version,
    pub name: String,
    pub features: Vec<PlannedFeature>,
    pub status: MilestoneStatus,
    pub target_date: Option<DateTime<Utc>>,
}

pub struct PlannedFeature {
    pub id: String,
    pub title: String,
    pub description: String,
    pub priority: Priority,
    pub votes: u32,  // Feedback communautaire
}

#[derive(Clone, PartialEq)]
pub enum MilestoneStatus {
    Planning,
    InDevelopment,
    Beta,
    Released,
}

#[derive(Clone, PartialEq)]
pub enum Priority {
    Critical,
    High,
    Medium,
    Low,
}
```

## Features prioritaires

```rust
impl ProductRoadmap {
    pub fn prioritize_features(&mut self) {
        // Trier par priorité et votes
        for milestone in &mut self.milestones {
            milestone.features.sort_by(|a, b| {
                match a.priority.cmp(&b.priority) {
                    std::cmp::Ordering::Equal => b.votes.cmp(&a.votes),
                    other => other,
                }
            });
        }
    }
    
    pub fn add_feature_request(&mut self, feature: PlannedFeature) {
        // Ajouter à la roadmap selon la priorité
        let target_milestone = self.find_target_milestone(&feature);
        target_milestone.features.push(feature);
    }
    
    fn find_target_milestone(&mut self, feature: &PlannedFeature) -> &mut Milestone {
        // Trouver le milestone approprié selon la priorité
        match feature.priority {
            Priority::Critical => {
                // Ajouter au prochain milestone
                self.milestones.iter_mut()
                    .find(|m| m.status == MilestoneStatus::Planning)
                    .unwrap()
            }
            _ => {
                // Ajouter à un milestone futur
                self.milestones.last_mut().unwrap()
            }
        }
    }
}
```

## Communication publique

```rust
impl ProductRoadmap {
    pub fn show_public_roadmap(&self, ui: &mut Ui, tokens: &DesignTokens) {
        ui.heading("Roadmap");
        ui.label("Voici ce qui arrive prochainement :");
        
        for milestone in &self.milestones {
            let color = match milestone.status {
                MilestoneStatus::Released => tokens.colors.success,
                MilestoneStatus::Beta => tokens.colors.warning,
                MilestoneStatus::InDevelopment => tokens.colors.primary,
                MilestoneStatus::Planning => tokens.colors.text_secondary,
            };
            
            ui.add_space(tokens.spacing.md);
            ui.colored_label(color, format!(
                "v{} - {}",
                milestone.version,
                milestone.name
            ));
            
            if let Some(date) = milestone.target_date {
                ui.label(format!("Date cible : {}", date.format("%Y-%m-%d")));
            }
            
            for feature in &milestone.features {
                ui.horizontal(|ui| {
                    ui.label("  •");
                    ui.label(&feature.title);
                    ui.colored_label(
                        tokens.colors.text_secondary,
                        format!("({} votes)", feature.votes)
                    );
                });
            }
        }
    }
}
```

## Résumé

- **Planification** : Versions planifiées avec dates cibles
- **Priorités** : Features triées par importance
- **Communication** : Roadmap publique transparente
- **Feedback** : Votes communautaires pour guider les priorités
