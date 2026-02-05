# 29.3 Activation Offline

## Activation liée à la machine

```rust
/// Activation liée à la machine (pas de serveur requis)
pub struct OfflineActivation {
    pub license_key: String,
    pub machine_id: String,
    pub activation_token: String,
    pub activated_at: DateTime<Utc>,
}

pub struct ActivationRequest {
    pub license_key: String,
    pub machine_id: String,
}

impl OfflineActivation {
    /// Générer une demande d'activation
    pub fn request_activation(license_key: &str) -> ActivationRequest {
        ActivationRequest {
            license_key: license_key.to_string(),
            machine_id: get_machine_id(),
        }
    }
    
    /// Vérifier que l'activation est valide pour cette machine
    pub fn verify(&self) -> bool {
        let expected_machine_id = get_machine_id();
        self.machine_id == expected_machine_id
    }
    
    /// Sauvegarder l'activation localement
    pub fn save(&self, path: &Path) -> Result<()> {
        let content = toml::to_string_pretty(self)?;
        std::fs::write(path, content)?;
        Ok(())
    }
    
    /// Charger l'activation depuis le disque
    pub fn load(path: &Path) -> Result<Self> {
        let content = std::fs::read_to_string(path)?;
        let activation: OfflineActivation = toml::from_str(&content)?;
        
        // Vérifier qu'elle est toujours valide
        if !activation.verify() {
            return Err(anyhow!("Activation invalide pour cette machine"));
        }
        
        Ok(activation)
    }
}
```

## Pas de serveur requis

```rust
/// Gestionnaire de licence fonctionnant entièrement offline
pub struct OfflineLicenseManager {
    activation: Option<OfflineActivation>,
    validator: LicenseValidator,
}

impl OfflineLicenseManager {
    pub fn new() -> Self {
        Self {
            activation: Self::load_activation(),
            validator: LicenseValidator::new(),
        }
    }
    
    pub fn activate(&mut self, license_key: &str) -> Result<()> {
        // Valider la clé de licence
        let license_data = self.validator.validate(license_key)?;
        
        // Créer l'activation
        let activation = OfflineActivation {
            license_key: license_key.to_string(),
            machine_id: get_machine_id(),
            activation_token: Self::generate_activation_token(&license_data),
            activated_at: Utc::now(),
        };
        
        // Sauvegarder
        let activation_path = Self::activation_path();
        activation.save(&activation_path)?;
        
        self.activation = Some(activation);
        Ok(())
    }
    
    pub fn is_activated(&self) -> bool {
        self.activation.as_ref()
            .map(|a| a.verify())
            .unwrap_or(false)
    }
    
    fn generate_activation_token(license_data: &LicenseData) -> String {
        // Générer un token basé sur la clé de licence et la machine ID
        use hmac::{Hmac, Mac};
        use sha2::Sha256;
        
        type HmacSha256 = Hmac<Sha256>;
        
        let mut mac = HmacSha256::new_from_slice(
            license_data.user_email.as_bytes()
        ).unwrap();
        mac.update(get_machine_id().as_bytes());
        let result = mac.finalize();
        
        hex::encode(&result.into_bytes()[..16])
    }
    
    fn load_activation() -> Option<OfflineActivation> {
        OfflineActivation::load(&Self::activation_path()).ok()
    }
    
    fn activation_path() -> PathBuf {
        dirs::data_dir()
            .unwrap_or_else(|| PathBuf::from("."))
            .join("mon-app")
            .join("activation.toml")
    }
}
```

## Sécurité

```rust
impl OfflineActivation {
    /// Vérifier l'intégrité de l'activation
    pub fn verify_integrity(&self) -> bool {
        // Vérifier que le token correspond bien à la machine et à la clé
        let expected_token = Self::generate_activation_token_from_key(
            &self.license_key,
            &self.machine_id
        );
        
        expected_token == self.activation_token
    }
    
    fn generate_activation_token_from_key(license_key: &str, machine_id: &str) -> String {
        use hmac::{Hmac, Mac};
        use sha2::Sha256;
        
        type HmacSha256 = Hmac<Sha256>;
        
        let mut mac = HmacSha256::new_from_slice(license_key.as_bytes()).unwrap();
        mac.update(machine_id.as_bytes());
        let result = mac.finalize();
        
        hex::encode(&result.into_bytes()[..16])
    }
}
```

## Résumé

- **Machine ID** : Activation liée au hardware unique
- **Offline** : Aucun serveur requis pour la validation
- **Sécurité** : Token HMAC basé sur la clé et la machine
- **Persistance** : Sauvegarde locale de l'activation
