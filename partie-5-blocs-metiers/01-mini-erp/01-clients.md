# 20.1 Clients

## Modèle de données Client

```rust
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct Customer {
    pub id: CustomerId,
    pub name: String,
    pub email: String,
    pub address: Address,
    pub vat_number: Option<String>,
    pub phone: Option<String>,
    pub created_at: DateTime<Utc>,
    pub updated_at: DateTime<Utc>,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct Address {
    pub street: String,
    pub city: String,
    pub postal_code: String,
    pub country: String,
}

#[derive(Debug, Clone, Copy, PartialEq, Eq, Hash, Serialize, Deserialize)]
pub struct CustomerId(pub i64);

impl Customer {
    pub fn new(name: String, email: String, address: Address) -> Self {
        let now = Utc::now();
        Self {
            id: CustomerId(0),  // Sera assigné par le repository
            name,
            email,
            address,
            vat_number: None,
            phone: None,
            created_at: now,
            updated_at: now,
        }
    }
    
    pub fn validate(&self) -> Result<(), Vec<ValidationError>> {
        let mut errors = Vec::new();
        
        if self.name.trim().is_empty() {
            errors.push(ValidationError::EmptyName);
        }
        
        if !self.email.contains('@') {
            errors.push(ValidationError::InvalidEmail);
        }
        
        if self.vat_number.is_some() && !self.validate_vat_number() {
            errors.push(ValidationError::InvalidVatNumber);
        }
        
        if errors.is_empty() {
            Ok(())
        } else {
            Err(errors)
        }
    }
    
    fn validate_vat_number(&self) -> bool {
        // Validation basique du numéro de TVA
        // En production, utiliser une bibliothèque dédiée
        self.vat_number.as_ref()
            .map(|v| v.len() >= 8 && v.chars().all(|c| c.is_alphanumeric()))
            .unwrap_or(true)
    }
}
```

## CRUD complet

### Create

```rust
pub struct CreateCustomerCommand {
    pub name: String,
    pub email: String,
    pub address: Address,
    pub vat_number: Option<String>,
    pub phone: Option<String>,
}

pub struct CreateCustomerHandler<R: CustomerRepository> {
    repository: R,
}

impl<R: CustomerRepository> CreateCustomerHandler<R> {
    pub fn handle(&self, cmd: CreateCustomerCommand) -> Result<Customer, AppError> {
        let customer = Customer::new(cmd.name, cmd.email, cmd.address);
        customer.vat_number = cmd.vat_number;
        customer.phone = cmd.phone;
        
        customer.validate()?;
        
        let id = self.repository.save(&customer)?;
        let mut customer = customer;
        customer.id = id;
        
        Ok(customer)
    }
}
```

### Read

```rust
pub struct GetCustomerQuery<R: CustomerRepository> {
    repository: R,
}

impl<R: CustomerRepository> GetCustomerQuery<R> {
    pub fn by_id(&self, id: CustomerId) -> Result<Option<Customer>, AppError> {
        self.repository.find_by_id(id)
    }
    
    pub fn all(&self) -> Result<Vec<Customer>, AppError> {
        self.repository.find_all()
    }
    
    pub fn search(&self, query: &str) -> Result<Vec<Customer>, AppError> {
        self.repository.search(query)
    }
}
```

### Update

```rust
pub struct UpdateCustomerCommand {
    pub id: CustomerId,
    pub changes: CustomerChanges,
}

pub struct CustomerChanges {
    pub name: Option<String>,
    pub email: Option<String>,
    pub address: Option<Address>,
    pub vat_number: Option<Option<String>>,
    pub phone: Option<Option<String>>,
}

pub struct UpdateCustomerHandler<R: CustomerRepository> {
    repository: R,
}

impl<R: CustomerRepository> UpdateCustomerHandler<R> {
    pub fn handle(&self, cmd: UpdateCustomerCommand) -> Result<Customer, AppError> {
        let mut customer = self.repository.find_by_id(cmd.id)?
            .ok_or(AppError::CustomerNotFound)?;
        
        if let Some(name) = cmd.changes.name {
            customer.name = name;
        }
        if let Some(email) = cmd.changes.email {
            customer.email = email;
        }
        if let Some(address) = cmd.changes.address {
            customer.address = address;
        }
        if let Some(vat_number) = cmd.changes.vat_number {
            customer.vat_number = vat_number;
        }
        if let Some(phone) = cmd.changes.phone {
            customer.phone = phone;
        }
        
        customer.updated_at = Utc::now();
        customer.validate()?;
        
        self.repository.save(&customer)?;
        Ok(customer)
    }
}
```

### Delete

```rust
pub struct DeleteCustomerHandler<R: CustomerRepository> {
    repository: R,
}

impl<R: CustomerRepository> DeleteCustomerHandler<R> {
    pub fn handle(&self, id: CustomerId) -> Result<bool, AppError> {
        // Vérifier qu'il n'y a pas de factures associées
        if self.repository.has_invoices(id)? {
            return Err(AppError::CannotDeleteCustomerWithInvoices);
        }
        
        self.repository.delete(id)
    }
}
```

## Validation et règles métier

```rust
#[derive(Debug, Clone)]
pub enum ValidationError {
    EmptyName,
    InvalidEmail,
    InvalidVatNumber,
    DuplicateEmail,
}

impl Customer {
    pub fn validate_unique_email(&self, repository: &impl CustomerRepository) -> Result<(), ValidationError> {
        if let Ok(existing) = repository.find_by_email(&self.email) {
            if existing.id != self.id {
                return Err(ValidationError::DuplicateEmail);
            }
        }
        Ok(())
    }
}
```

## Résumé

- **Modèle complet** : Toutes les informations nécessaires
- **CRUD** : Create, Read, Update, Delete
- **Validation** : Vérification des données
- **Règles métier** : Contraintes et validations métier
