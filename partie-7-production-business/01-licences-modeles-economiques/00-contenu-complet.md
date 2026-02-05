# Chapitre 29 : Licences et Modèles Économiques

## Introduction

Choisir le bon modèle de licence détermine comment vous monétisez votre application tout en offrant une bonne expérience utilisateur.

---

## 29.1 Types de Licences

### Licence Perpétuelle

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
        let licensed_major: u32 = self.version.split('.').next()
            .and_then(|s| s.parse().ok())
            .unwrap_or(0);
        
        let current_major: u32 = current_version.split('.').next()
            .and_then(|s| s.parse().ok())
            .unwrap_or(0);
        
        current_major <= licensed_major
    }
}
```

### Abonnement

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
}
```

---

## 29.2 Génération et Validation de Clés

```rust
use ed25519_dalek::{Signer, SigningKey, Verifier, VerifyingKey};
use base64::{engine::general_purpose::URL_SAFE_NO_PAD, Engine};

pub struct LicenseGenerator {
    signing_key: SigningKey,
}

impl LicenseGenerator {
    pub fn generate(&self, data: &LicenseData) -> String {
        let payload = serde_json::to_string(data).unwrap();
        let signature = self.signing_key.sign(payload.as_bytes());
        
        // Format: BASE64(payload).BASE64(signature)
        format!(
            "{}.{}",
            URL_SAFE_NO_PAD.encode(&payload),
            URL_SAFE_NO_PAD.encode(signature.to_bytes())
        )
    }
}

pub struct LicenseValidator {
    verifying_key: VerifyingKey,
}

impl LicenseValidator {
    pub fn validate(&self, license_key: &str) -> Result<LicenseData, LicenseError> {
        let parts: Vec<&str> = license_key.split('.').collect();
        if parts.len() != 2 {
            return Err(LicenseError::InvalidFormat);
        }
        
        let payload = URL_SAFE_NO_PAD.decode(parts[0])
            .map_err(|_| LicenseError::InvalidFormat)?;
        
        let signature_bytes = URL_SAFE_NO_PAD.decode(parts[1])
            .map_err(|_| LicenseError::InvalidFormat)?;
        
        let signature = ed25519_dalek::Signature::from_slice(&signature_bytes)
            .map_err(|_| LicenseError::InvalidSignature)?;
        
        self.verifying_key.verify(&payload, &signature)
            .map_err(|_| LicenseError::InvalidSignature)?;
        
        serde_json::from_slice(&payload)
            .map_err(|_| LicenseError::InvalidData)
    }
}
```

---

## 29.3 Activation Offline

```rust
/// Machine ID basé sur le hardware
pub fn get_machine_id() -> String {
    use sha2::{Sha256, Digest};
    
    let mut hasher = Sha256::new();
    
    // Combiner plusieurs identifiants hardware
    #[cfg(target_os = "windows")]
    {
        if let Ok(output) = std::process::Command::new("wmic")
            .args(["baseboard", "get", "serialnumber"])
            .output()
        {
            hasher.update(&output.stdout);
        }
    }
    
    #[cfg(target_os = "macos")]
    {
        if let Ok(output) = std::process::Command::new("ioreg")
            .args(["-rd1", "-c", "IOPlatformExpertDevice"])
            .output()
        {
            hasher.update(&output.stdout);
        }
    }
    
    #[cfg(target_os = "linux")]
    {
        if let Ok(content) = std::fs::read_to_string("/etc/machine-id") {
            hasher.update(content.as_bytes());
        }
    }
    
    let result = hasher.finalize();
    hex::encode(&result[..16])  // 32 caractères hex
}

/// Activation liée à la machine
pub struct OfflineActivation {
    pub license_key: String,
    pub machine_id: String,
    pub activation_token: String,
}

impl OfflineActivation {
    pub fn request_activation(license_key: &str) -> ActivationRequest {
        ActivationRequest {
            license_key: license_key.to_string(),
            machine_id: get_machine_id(),
        }
    }
    
    pub fn verify(&self) -> bool {
        // Vérifier que le token correspond à la machine
        let expected_machine_id = get_machine_id();
        self.machine_id == expected_machine_id
    }
}
```

---

## 29.4 Modèle Freemium

```rust
pub struct FreemiumManager {
    current_license: Option<License>,
    free_tier_limits: FreeTierLimits,
}

pub struct FreeTierLimits {
    pub max_documents: usize,
    pub max_customers: usize,
    pub features_enabled: Vec<Feature>,
}

impl FreemiumManager {
    pub fn can_create_document(&self, current_count: usize) -> bool {
        if self.is_premium() {
            true
        } else {
            current_count < self.free_tier_limits.max_documents
        }
    }
    
    pub fn is_feature_available(&self, feature: Feature) -> bool {
        if self.is_premium() {
            true
        } else {
            self.free_tier_limits.features_enabled.contains(&feature)
        }
    }
    
    pub fn is_premium(&self) -> bool {
        self.current_license
            .as_ref()
            .map(|l| l.is_active())
            .unwrap_or(false)
    }
    
    pub fn show_upgrade_prompt(&self, ui: &mut Ui, tokens: &DesignTokens) {
        if !self.is_premium() {
            ui.horizontal(|ui| {
                ui.colored_label(tokens.colors.warning, "⭐");
                ui.label("Passez à la version Pro pour débloquer toutes les fonctionnalités");
                if ui.button("Mettre à niveau").clicked() {
                    open::that("https://example.com/upgrade").ok();
                }
            });
        }
    }
}
```

---

## Résumé

| Modèle | Avantages | Inconvénients |
|--------|-----------|---------------|
| Perpétuel | Revenu immédiat, Simple | Pas de revenus récurrents |
| Abonnement | Revenus prévisibles | Friction plus élevée |
| Freemium | Large adoption | Conversion faible |

- **Clés signées** : Impossible à falsifier
- **Machine ID** : Activation liée au hardware
- **Validation offline** : Fonctionne sans connexion
