# 10.6 DPI Scaling

## Gérer les écrans haute résolution

```rust
pub struct DpiManager {
    scale_factor: f32,
    base_scale: f32,
}

impl DpiManager {
    pub fn new(native_scale: f32) -> Self {
        Self {
            scale_factor: native_scale,
            base_scale: 1.0,
        }
    }
    
    /// Facteur de scale combiné (système + préférence utilisateur)
    pub fn effective_scale(&self) -> f32 {
        self.scale_factor * self.base_scale
    }
    
    /// Définir une préférence utilisateur de zoom
    pub fn set_user_scale(&mut self, scale: f32) {
        self.base_scale = scale.clamp(0.75, 2.0);
    }
    
    /// Convertir une taille en pixels logiques vers pixels physiques
    pub fn to_physical(&self, logical: f32) -> f32 {
        logical * self.effective_scale()
    }
    
    /// Convertir une taille en pixels physiques vers pixels logiques
    pub fn to_logical(&self, physical: f32) -> f32 {
        physical / self.effective_scale()
    }
}
```

## Tokens avec DPI awareness

```rust
/// Tokens de design avec DPI awareness
impl DesignTokens {
    pub fn scaled(&self, dpi: &DpiManager) -> ScaledTokens {
        let scale = dpi.effective_scale();
        
        ScaledTokens {
            spacing: ScaledSpacing {
                xs: self.spacing.xs * scale,
                sm: self.spacing.sm * scale,
                md: self.spacing.md * scale,
                lg: self.spacing.lg * scale,
                xl: self.spacing.xl * scale,
            },
            typography: ScaledTypography {
                size_sm: self.typography.size_sm * scale,
                size_md: self.typography.size_md * scale,
                size_lg: self.typography.size_lg * scale,
                size_xl: self.typography.size_xl * scale,
            },
            radius: ScaledRadius {
                sm: self.radius.sm * scale,
                md: self.radius.md * scale,
                lg: self.radius.lg * scale,
            },
        }
    }
}
```

## Configuration egui pour HiDPI

```rust
fn configure_egui_for_hidpi(ctx: &egui::Context, scale: f32) {
    // egui gère le DPI automatiquement, mais on peut ajuster
    ctx.set_pixels_per_point(scale);
    
    // Ou laisser egui détecter automatiquement
    // ctx.set_pixels_per_point(ctx.native_pixels_per_point().unwrap_or(1.0));
}
```

## Tests multi-DPI

```rust
#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_scaling_consistency() {
        let tokens = DesignTokens::light();
        let dpi_100 = DpiManager::new(1.0);
        let dpi_150 = DpiManager::new(1.5);
        let dpi_200 = DpiManager::new(2.0);
        
        let scaled_100 = tokens.scaled(&dpi_100);
        let scaled_150 = tokens.scaled(&dpi_150);
        let scaled_200 = tokens.scaled(&dpi_200);
        
        // Vérifier que les proportions sont maintenues
        assert_eq!(scaled_150.spacing.md / scaled_150.spacing.sm, 
                   scaled_100.spacing.md / scaled_100.spacing.sm);
    }
}
```

## Résumé

- **Détection automatique** : egui détecte le DPI système
- **Scaling manuel** : Contrôle via DpiManager
- **Tokens scalés** : Tous les tokens respectent le DPI
- **Tests** : Vérifier la cohérence à différentes résolutions
