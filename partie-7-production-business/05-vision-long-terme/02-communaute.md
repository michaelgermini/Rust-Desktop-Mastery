# 33.2 Gestion de la Communauté

## Canaux de communication

```rust
/// Canaux de communication
pub struct CommunityChannels {
    pub discord_url: String,
    pub forum_url: String,
    pub twitter_url: String,
    pub newsletter_subscribers: u32,
    pub github_discussions: String,
}

impl CommunityChannels {
    pub fn show_community_links(&self, ui: &mut Ui) {
        ui.heading("Rejoignez la communauté");
        
        ui.horizontal(|ui| {
            if ui.button("💬 Discord").clicked() {
                open::that(&self.discord_url).ok();
            }
            
            if ui.button("📝 Forum").clicked() {
                open::that(&self.forum_url).ok();
            }
            
            if ui.button("🐦 Twitter").clicked() {
                open::that(&self.twitter_url).ok();
            }
        });
    }
}
```

## Feature voting

```rust
/// Système de feedback communautaire
pub struct FeatureVoting {
    pub features: Vec<VotableFeature>,
}

pub struct VotableFeature {
    pub id: String,
    pub title: String,
    pub description: String,
    pub votes: u32,
    pub status: FeatureStatus,
    pub submitted_by: String,
}

#[derive(Clone, PartialEq)]
pub enum FeatureStatus {
    UnderReview,
    Planned,
    InProgress,
    Completed,
    Declined,
}

impl FeatureVoting {
    pub fn show_voting_board(&mut self, ui: &mut Ui, tokens: &DesignTokens, user_id: &str) {
        ui.heading("Votez pour les prochaines fonctionnalités");
        
        // Tri par votes
        let mut sorted = self.features.clone();
        sorted.sort_by(|a, b| b.votes.cmp(&a.votes));
        
        for feature in sorted.iter().take(10) {
            egui::Frame::none()
                .fill(tokens.colors.bg_secondary)
                .rounding(tokens.radius.md)
                .inner_margin(tokens.spacing.sm)
                .show(ui, |ui| {
                    ui.horizontal(|ui| {
                        // Bouton de vote
                        if ui.button(format!("▲ {}", feature.votes)).clicked() {
                            // Voter
                            self.vote_for_feature(&feature.id, user_id);
                        }
                        
                        ui.vertical(|ui| {
                            ui.strong(&feature.title);
                            ui.colored_label(
                                tokens.colors.text_secondary,
                                &feature.description
                            );
                        });
                        
                        // Status badge
                        let (text, color) = match feature.status {
                            FeatureStatus::Planned => ("Planifié", tokens.colors.primary),
                            FeatureStatus::InProgress => ("En cours", tokens.colors.warning),
                            FeatureStatus::Completed => ("Terminé", tokens.colors.success),
                            _ => ("À l'étude", tokens.colors.text_secondary),
                        };
                        ui.colored_label(color, text);
                    });
                });
            ui.add_space(tokens.spacing.xs);
        }
    }
    
    fn vote_for_feature(&mut self, feature_id: &str, user_id: &str) {
        if let Some(feature) = self.features.iter_mut().find(|f| f.id == feature_id) {
            feature.votes += 1;
        }
    }
}
```

## Contribution open source

```rust
pub struct OpenSourceContributions {
    pub components: Vec<OpenSourceComponent>,
    pub contributors: Vec<Contributor>,
}

pub struct Contributor {
    pub name: String,
    pub contributions: u32,
    pub github_username: String,
}

impl OpenSourceContributions {
    pub fn show_contributors(&self, ui: &mut Ui) {
        ui.heading("Contributeurs");
        
        for contributor in &self.contributors {
            ui.horizontal(|ui| {
                ui.label(&contributor.name);
                ui.label(format!("{} contributions", contributor.contributions));
            });
        }
    }
}
```

## Résumé

- **Canaux** : Discord, Forum, Twitter, GitHub Discussions
- **Voting** : Système de votes pour les features
- **Contribution** : Encouragement des contributions open source
- **Engagement** : Communauté active et impliquée
