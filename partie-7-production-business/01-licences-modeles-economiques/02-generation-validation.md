# 29.2 Génération et Validation

## Clés de licence signées

```rust
use ed25519_dalek::{Signer, SigningKey, Verifier, VerifyingKey};
use base64::{engine::general_purpose::URL_SAFE_NO_PAD, Engine};

pub struct LicenseGenerator {
    signing_key: SigningKey,
}

#[derive(Serialize, Deserialize)]
pub struct LicenseData {
    pub user_email: String,
    pub license_type: LicenseType,
    pub expires_at: Option<DateTime<Utc>>,
    pub version: String,
    pub machine_id: Option<String>,
}

#[derive(Serialize, Deserialize)]
pub enum LicenseType {
    Perpetual,
    Subscription { plan: SubscriptionPlan },
    Freemium { tier: FreemiumTier },
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
```

## Validation

```rust
pub struct LicenseValidator {
    verifying_key: VerifyingKey,
}

#[derive(Debug)]
pub enum LicenseError {
    InvalidFormat,
    InvalidSignature,
    InvalidData,
    Expired,
    WrongMachine,
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
        
        // Vérifier la signature
        self.verifying_key.verify(&payload, &signature)
            .map_err(|_| LicenseError::InvalidSignature)?;
        
        // Décoder les données
        let data: LicenseData = serde_json::from_slice(&payload)
            .map_err(|_| LicenseError::InvalidData)?;
        
        // Vérifier l'expiration
        if let Some(expires_at) = data.expires_at {
            if Utc::now() > expires_at {
                return Err(LicenseError::Expired);
            }
        }
        
        // Vérifier la machine ID si présente
        if let Some(licensed_machine_id) = &data.machine_id {
            let current_machine_id = get_machine_id();
            if licensed_machine_id != &current_machine_id {
                return Err(LicenseError::WrongMachine);
            }
        }
        
        Ok(data)
    }
}
```

## Machine ID

```rust
use sha2::{Sha256, Digest};

/// Générer un identifiant unique basé sur le hardware
pub fn get_machine_id() -> String {
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
```

## Résumé

- **Signature** : Clés signées avec Ed25519 pour sécurité
- **Validation** : Vérification de signature, expiration, machine ID
- **Machine ID** : Identifiant unique basé sur le hardware
- **Sécurité** : Impossible à falsifier sans la clé privée
