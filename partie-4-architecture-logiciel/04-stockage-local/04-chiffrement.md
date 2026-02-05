# 17.4 Chiffrement

## Chiffrement des données sensibles

```rust
use aes_gcm::{Aes256Gcm, Key, Nonce, aead::{Aead, NewAead}};
use argon2::{Argon2, password_hash::{SaltString, rand_core::OsRng}};

pub struct EncryptedStorage {
    cipher: Aes256Gcm,
}

impl EncryptedStorage {
    pub fn new(password: &str) -> Result<Self> {
        // Générer un salt aléatoire
        let salt = SaltString::generate(&mut OsRng);
        
        // Dériver une clé du mot de passe avec Argon2
        let mut key_bytes = [0u8; 32];
        Argon2::default()
            .hash_password_into(password.as_bytes(), salt.as_salt().as_bytes(), &mut key_bytes)
            .map_err(|e| format!("Key derivation failed: {}", e))?;
        
        let key = Key::from_slice(&key_bytes);
        let cipher = Aes256Gcm::new(key);
        
        Ok(Self { cipher })
    }
    
    pub fn encrypt(&self, data: &[u8]) -> Result<Vec<u8>> {
        // Générer un nonce aléatoire
        let nonce = Aes256Gcm::generate_nonce(&mut OsRng);
        
        // Chiffrer
        let ciphertext = self.cipher.encrypt(&nonce, data)
            .map_err(|e| format!("Encryption failed: {}", e))?;
        
        // Préfixer le nonce au ciphertext
        let mut result = nonce.to_vec();
        result.extend_from_slice(&ciphertext);
        
        Ok(result)
    }
    
    pub fn decrypt(&self, encrypted: &[u8]) -> Result<Vec<u8>> {
        // Extraire le nonce (12 premiers bytes)
        let nonce = Nonce::from_slice(&encrypted[..12]);
        let ciphertext = &encrypted[12..];
        
        // Déchiffrer
        let plaintext = self.cipher.decrypt(nonce, ciphertext)
            .map_err(|e| format!("Decryption failed: {}", e))?;
        
        Ok(plaintext)
    }
}
```

## Gestion des clés

```rust
pub struct KeyManager {
    master_key: Vec<u8>,
}

impl KeyManager {
    pub fn derive_from_password(password: &str, salt: &[u8]) -> Self {
        let mut key = [0u8; 32];
        Argon2::default()
            .hash_password_into(password.as_bytes(), salt, &mut key)
            .expect("Key derivation failed");
        
        Self {
            master_key: key.to_vec(),
        }
    }
    
    pub fn derive_key(&self, context: &str) -> Vec<u8> {
        // Dériver une clé spécifique depuis la clé maître
        use hmac::{Hmac, Mac};
        use sha2::Sha256;
        
        type HmacSha256 = Hmac<Sha256>;
        let mut mac = HmacSha256::new_from_slice(&self.master_key)
            .expect("HMAC can take key of any size");
        mac.update(context.as_bytes());
        mac.finalize().into_bytes().to_vec()
    }
}
```

## Performance

Pour de gros volumes, chiffrer seulement les champs sensibles :

```rust
pub struct Invoice {
    pub id: InvoiceId,
    pub number: String,  // Non chiffré
    pub customer_id: CustomerId,  // Non chiffré
    pub encrypted_notes: Option<Vec<u8>>,  // Chiffré
}

impl Invoice {
    pub fn set_notes(&mut self, notes: &str, storage: &EncryptedStorage) -> Result<()> {
        self.encrypted_notes = Some(storage.encrypt(notes.as_bytes())?);
        Ok(())
    }
    
    pub fn get_notes(&self, storage: &EncryptedStorage) -> Result<String> {
        if let Some(encrypted) = &self.encrypted_notes {
            let decrypted = storage.decrypt(encrypted)?;
            Ok(String::from_utf8(decrypted)?)
        } else {
            Ok(String::new())
        }
    }
}
```

## Résumé

- **AES-GCM** : Chiffrement authentifié
- **Argon2** : Dérivation de clé sécurisée
- **Nonce unique** : Un nonce par chiffrement
- **Champs sélectifs** : Chiffrer seulement ce qui est sensible
