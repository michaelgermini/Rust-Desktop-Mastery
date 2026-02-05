# 11.2 Input

## TextInput avec validation

```rust
pub struct TextInput<'a> {
    value: &'a mut String,
    placeholder: &'a str,
    label: Option<&'a str>,
    error: Option<&'a str>,
    disabled: bool,
    password: bool,
}

impl<'a> TextInput<'a> {
    pub fn new(value: &'a mut String) -> Self {
        Self {
            value,
            placeholder: "",
            label: None,
            error: None,
            disabled: false,
            password: false,
        }
    }
    
    pub fn placeholder(mut self, placeholder: &'a str) -> Self {
        self.placeholder = placeholder;
        self
    }
    
    pub fn label(mut self, label: &'a str) -> Self {
        self.label = Some(label);
        self
    }
    
    pub fn error(mut self, error: &'a str) -> Self {
        self.error = Some(error);
        self
    }
    
    pub fn password(mut self) -> Self {
        self.password = true;
        self
    }
    
    pub fn show(self, ui: &mut Ui, tokens: &DesignTokens) -> Response {
        // Implémentation avec label, erreur, etc.
        // Voir 00-contenu-complet.md pour le code complet
    }
}
```

## Usage

```rust
// Input simple
TextInput::new(&mut name)
    .label("Nom")
    .placeholder("Entrez votre nom")
    .show(ui, &tokens);

// Input avec erreur
TextInput::new(&mut email)
    .label("Email")
    .error("Email invalide")
    .show(ui, &tokens);

// Password input
TextInput::new(&mut password)
    .label("Mot de passe")
    .password()
    .show(ui, &tokens);
```

## Features

- **Label** : Texte au-dessus du champ
- **Placeholder** : Texte d'indication
- **Error** : Message d'erreur en rouge
- **Password** : Masquage du texte
- **Disabled** : État désactivé
