# 29.1 Types de Licences

## Licence perpétuelle

```rust
pub struct PerpetualLicense {
    pub key: String,
    pub user_email: String,
    pub purchased_at: DateTime<Utc>,
    pub version: String,  // Version majeure couverte
    pub machine_id: Option<String>,
}

impl PerpetualLicense {
    pub fn is_valid_for_version(&self, current_version: &str) -> bool {
        // Licence valide pour les versions mineures
        let licensed_major: u32 = self.version.split('.')
            .next()
            .and_then(|s| s.parse().ok())
            .unwrap_or(0);
        
        let current_major: u32 = current_version.split('.')
            .next()
            .and_then(|s| s.parse().ok())
            .unwrap_or(0);
        
        current_major <= licensed_major
    }
}
```

## Abonnement

```rust
pub struct SubscriptionLicense {
    pub key: String,
    pub user_email: String,
    pub started_at: DateTime<Utc>,
    pub expires_at: DateTime<Utc>,
    pub plan: SubscriptionPlan,
    pub auto_renew: bool,
}

#[derive(Clone, PartialEq)]
pub enum SubscriptionPlan {
    Monthly,
    Yearly,
    Lifetime,
}

impl SubscriptionLicense {
    pub fn is_active(&self) -> bool {
        Utc::now() < self.expires_at
    }
    
    pub fn days_remaining(&self) -> i64 {
        (self.expires_at - Utc::now()).num_days().max(0)
    }
    
    pub fn should_renew(&self) -> bool {
        self.auto_renew && self.days_remaining() < 7
    }
}
```

## Freemium

```rust
pub struct FreemiumLicense {
    pub tier: FreemiumTier,
    pub upgrade_prompts_shown: u32,
}

#[derive(Clone, PartialEq)]
pub enum FreemiumTier {
    Free,
    Pro,
    Enterprise,
}

impl FreemiumLicense {
    pub fn is_free(&self) -> bool {
        matches!(self.tier, FreemiumTier::Free)
    }
}
```

## Comparaison des modèles

| Modèle | Avantages | Inconvénients | Revenus |
|--------|-----------|---------------|---------|
| Perpétuel | Revenu immédiat, Simple | Pas de revenus récurrents | Ponctuel |
| Abonnement | Revenus prévisibles | Friction plus élevée | Récurrent |
| Freemium | Large adoption | Conversion faible | Variable |

## Résumé

- **Perpétuel** : Achat unique, valide pour une version majeure
- **Abonnement** : Renouvellement périodique, accès continu
- **Freemium** : Version gratuite avec limitations, upgrade payant
