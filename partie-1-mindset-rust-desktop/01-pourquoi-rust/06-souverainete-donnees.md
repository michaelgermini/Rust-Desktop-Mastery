# 1.6 Souveraineté des Données

## Définition

La souveraineté des données signifie que l'utilisateur possède et contrôle entièrement ses données, sans dépendance à un tiers.

```rust
/// Principes de souveraineté des données
pub struct DataSovereignty {
    /// Les données sont stockées localement
    pub local_storage: bool,
    
    /// L'utilisateur peut exporter toutes ses données
    pub full_export: bool,
    
    /// Format de données ouvert et documenté
    pub open_format: bool,
    
    /// Pas de dépendance à un service externe
    pub no_vendor_lock: bool,
    
    /// Chiffrement sous contrôle de l'utilisateur
    pub user_controlled_encryption: bool,
}

impl Default for DataSovereignty {
    fn default() -> Self {
        Self {
            local_storage: true,
            full_export: true,
            open_format: true,
            no_vendor_lock: true,
            user_controlled_encryption: true,
        }
    }
}
```

## Problèmes du modèle SaaS

### Vous n'êtes pas propriétaire de vos données

```rust
/// Risques avec les données dans le cloud
pub enum CloudDataRisk {
    /// Le service ferme
    ServiceShutdown {
        notice_period_days: u32,
        data_export_available: bool,
    },
    
    /// Changement de conditions
    TermsChange {
        new_price: Decimal,
        data_usage_rights: DataUsageRights,
    },
    
    /// Violation de données
    DataBreach {
        records_exposed: u64,
    },
    
    /// Blocage de compte
    AccountLocked {
        reason: &'static str,
        appeal_possible: bool,
    },
    
    /// Accès gouvernemental
    GovernmentAccess {
        jurisdiction: &'static str,
        warrant_required: bool,
    },
}
```

### Exemples réels

```rust
/// Cas historiques de perte de données cloud
const CLOUD_INCIDENTS: &[CloudIncident] = &[
    CloudIncident {
        service: "Google Reader",
        year: 2013,
        impact: "Fermeture, des millions d'utilisateurs perdent leurs flux RSS",
    },
    CloudIncident {
        service: "Sunrise Calendar",
        year: 2016,
        impact: "Racheté par Microsoft, fermé",
    },
    CloudIncident {
        service: "Wunderlist",
        year: 2020,
        impact: "Fermé après rachat Microsoft",
    },
    CloudIncident {
        service: "Google+",
        year: 2019,
        impact: "Fermé, données utilisateurs perdues",
    },
];
```

## L'alternative locale

### Stockage contrôlé par l'utilisateur

```rust
use std::path::PathBuf;

pub struct UserDataStorage {
    /// Dossier principal des données
    pub data_dir: PathBuf,
    
    /// Base de données SQLite
    pub database: PathBuf,
    
    /// Fichiers attachés
    pub attachments: PathBuf,
    
    /// Sauvegardes
    pub backups: PathBuf,
}

impl UserDataStorage {
    pub fn new() -> Self {
        let data_dir = dirs::data_local_dir()
            .unwrap()
            .join("MonApplication");
        
        Self {
            data_dir: data_dir.clone(),
            database: data_dir.join("data.db"),
            attachments: data_dir.join("files"),
            backups: data_dir.join("backups"),
        }
    }
    
    /// L'utilisateur peut voir et copier ses données
    pub fn show_data_location(&self) -> String {
        format!(
            "Vos données sont stockées dans :\n{}",
            self.data_dir.display()
        )
    }
}
```

### Export complet

```rust
pub struct DataExporter {
    storage: UserDataStorage,
}

impl DataExporter {
    /// Exporter toutes les données dans un format ouvert
    pub fn export_all(&self, output_path: &Path) -> Result<ExportResult> {
        let mut archive = zip::ZipWriter::new(File::create(output_path)?);
        
        // Exporter la base de données en SQL lisible
        self.export_database_as_sql(&mut archive)?;
        
        // Exporter en JSON pour lisibilité
        self.export_database_as_json(&mut archive)?;
        
        // Copier les fichiers attachés
        self.export_attachments(&mut archive)?;
        
        // Métadonnées de l'export
        self.write_export_metadata(&mut archive)?;
        
        archive.finish()?;
        
        Ok(ExportResult {
            path: output_path.to_path_buf(),
            total_records: self.count_records()?,
            total_files: self.count_files()?,
            format: "ZIP contenant SQL, JSON et fichiers".to_string(),
        })
    }
}
```

### Format ouvert et documenté

```rust
/// Documentation du format de données
pub const DATA_FORMAT_DOCUMENTATION: &str = r#"
# Format de données MonApplication

## Base de données

SQLite 3, UTF-8, tables principales :

### Table `customers`
- id: INTEGER PRIMARY KEY
- name: TEXT NOT NULL
- email: TEXT
- created_at: TEXT (ISO 8601)
- updated_at: TEXT (ISO 8601)

### Table `invoices`
- id: INTEGER PRIMARY KEY
- customer_id: INTEGER REFERENCES customers(id)
- number: TEXT UNIQUE
- status: TEXT ('draft', 'sent', 'paid')
- total: TEXT (décimal, ex: "1234.56")
...

## Fichiers

Les fichiers attachés sont stockés avec leur nom original
dans le dossier `files/`, organisés par année/mois.

## Import

Pour importer dans un autre système :
1. Ouvrir data.db avec n'importe quel client SQLite
2. Ou utiliser les fichiers JSON pour un import programmatique
"#;
```

## Chiffrement contrôlé par l'utilisateur

### Clé de chiffrement locale

```rust
use aes_gcm::{Aes256Gcm, Key, Nonce};
use aes_gcm::aead::{Aead, NewAead};
use argon2::Argon2;

pub struct UserEncryption {
    /// Clé dérivée du mot de passe utilisateur
    key: Key<Aes256Gcm>,
}

impl UserEncryption {
    /// Dériver une clé depuis le mot de passe utilisateur
    /// La clé n'est JAMAIS envoyée sur internet
    pub fn from_password(password: &str, salt: &[u8]) -> Result<Self> {
        let mut key = [0u8; 32];
        
        Argon2::default()
            .hash_password_into(password.as_bytes(), salt, &mut key)
            .map_err(|_| Error::KeyDerivation)?;
        
        Ok(Self {
            key: Key::from_slice(&key).clone(),
        })
    }
    
    pub fn encrypt(&self, plaintext: &[u8]) -> Result<Vec<u8>> {
        let cipher = Aes256Gcm::new(&self.key);
        let nonce = Nonce::from_slice(&generate_nonce());
        
        cipher.encrypt(nonce, plaintext)
            .map_err(|_| Error::Encryption)
    }
    
    pub fn decrypt(&self, ciphertext: &[u8]) -> Result<Vec<u8>> {
        // L'utilisateur est le seul à pouvoir déchiffrer
        // Pas de "récupération de mot de passe" possible
        // = Vraie souveraineté
    }
}
```

## Conformité RGPD simplifiée

```rust
/// Droits RGPD facilement implémentables avec données locales
pub struct GdprCompliance {
    /// Droit d'accès : trivial, données sur le PC de l'utilisateur
    pub right_to_access: ComplianceStatus,
    
    /// Droit de rectification : l'utilisateur modifie lui-même
    pub right_to_rectification: ComplianceStatus,
    
    /// Droit à l'effacement : supprimer le fichier suffit
    pub right_to_erasure: ComplianceStatus,
    
    /// Droit à la portabilité : export intégré
    pub right_to_portability: ComplianceStatus,
}

impl Default for GdprCompliance {
    fn default() -> Self {
        Self {
            right_to_access: ComplianceStatus::ByDesign,
            right_to_rectification: ComplianceStatus::ByDesign,
            right_to_erasure: ComplianceStatus::ByDesign,
            right_to_portability: ComplianceStatus::ByDesign,
        }
    }
}
```

## Avantages business

```rust
pub struct SovereigntyBenefits {
    /// Argument commercial : "Vos données restent chez vous"
    pub marketing_angle: &'static str,
    
    /// Conformité facilitée
    pub compliance_ease: &'static str,
    
    /// Pas de coûts d'hébergement
    pub no_hosting_costs: bool,
    
    /// Pas de responsabilité de stockage
    pub no_data_liability: bool,
}

impl Default for SovereigntyBenefits {
    fn default() -> Self {
        Self {
            marketing_angle: "Vos données ne quittent jamais votre ordinateur",
            compliance_ease: "RGPD compliant by design",
            no_hosting_costs: true,
            no_data_liability: true,
        }
    }
}
```

## Conclusion

La souveraineté des données avec une app locale signifie :

- **Propriété** : L'utilisateur possède vraiment ses données
- **Contrôle** : Accès, modification, suppression instantanés
- **Pérennité** : Pas de risque de fermeture de service
- **Confidentialité** : Données jamais exposées sur internet
- **Conformité** : RGPD trivial à respecter
