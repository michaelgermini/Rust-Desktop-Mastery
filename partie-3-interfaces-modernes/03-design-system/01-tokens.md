# 10.1 Tokens de Design

## Qu'est-ce qu'un token ?

Les tokens sont les valeurs atomiques de votre design : couleurs, espacements, tailles de police. Ils permettent de modifier l'apparence globale depuis un seul endroit.

## Structure des tokens

```rust
/// Tokens de design - les valeurs atomiques du système
pub struct DesignTokens {
    pub colors: ColorTokens,
    pub spacing: SpacingTokens,
    pub typography: TypographyTokens,
    pub radius: RadiusTokens,
    pub shadows: ShadowTokens,
}

impl DesignTokens {
    /// Tokens par défaut (thème light)
    pub fn light() -> Self {
        Self {
            colors: ColorTokens::light(),
            spacing: SpacingTokens::default(),
            typography: TypographyTokens::default(),
            radius: RadiusTokens::default(),
            shadows: ShadowTokens::light(),
        }
    }
    
    /// Tokens pour le thème dark
    pub fn dark() -> Self {
        Self {
            colors: ColorTokens::dark(),
            spacing: SpacingTokens::default(),
            typography: TypographyTokens::default(),
            radius: RadiusTokens::default(),
            shadows: ShadowTokens::dark(),
        }
    }
}
```

## Avantages des tokens

### 1. Cohérence

```rust
// ❌ Sans tokens : valeurs dispersées
ui.add_space(8.0);  // Ici
ui.add_space(16.0); // Là
ui.add_space(8.0);  // Ailleurs

// ✅ Avec tokens : valeurs centralisées
ui.add_space(tokens.spacing.sm);  // Partout
ui.add_space(tokens.spacing.md);
ui.add_space(tokens.spacing.sm);
```

### 2. Maintenabilité

```rust
// Changer l'espacement global en un seul endroit
impl SpacingTokens {
    pub fn default() -> Self {
        Self {
            sm: 8.0,   // Avant : 6.0
            md: 16.0,  // Avant : 12.0
            // ...
        }
    }
}
```

### 3. Thèmes faciles

```rust
// Basculer entre light et dark
let tokens = if is_dark_mode {
    DesignTokens::dark()
} else {
    DesignTokens::light()
};
```

## Organisation recommandée

```
design_system/
├── tokens.rs          # Structure principale
├── colors.rs          # Tokens de couleur
├── spacing.rs         # Tokens d'espacement
├── typography.rs      # Tokens typographiques
├── radius.rs          # Tokens de rayon
└── shadows.rs         # Tokens d'ombre
```

## Résumé

Les tokens sont la fondation d'un design system :
- **Valeurs atomiques** : Couleurs, espacements, tailles
- **Centralisation** : Un seul endroit pour modifier
- **Cohérence** : Même apparence partout
- **Thèmes** : Changement global facile
