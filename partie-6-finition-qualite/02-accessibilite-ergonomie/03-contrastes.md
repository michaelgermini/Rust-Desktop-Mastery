# 26.3 Contrastes

## Vérification WCAG

```rust
pub struct AccessibilityChecker;

impl AccessibilityChecker {
    /// Vérifier le ratio de contraste (WCAG 2.1)
    pub fn contrast_ratio(fg: Color32, bg: Color32) -> f32 {
        let l1 = Self::relative_luminance(fg);
        let l2 = Self::relative_luminance(bg);
        
        let lighter = l1.max(l2);
        let darker = l1.min(l2);
        
        (lighter + 0.05) / (darker + 0.05)
    }
    
    fn relative_luminance(color: Color32) -> f32 {
        let r = Self::linearize(color.r() as f32 / 255.0);
        let g = Self::linearize(color.g() as f32 / 255.0);
        let b = Self::linearize(color.b() as f32 / 255.0);
        
        0.2126 * r + 0.7152 * g + 0.0722 * b
    }
    
    fn linearize(value: f32) -> f32 {
        if value <= 0.03928 {
            value / 12.92
        } else {
            ((value + 0.055) / 1.055).powf(2.4)
        }
    }
    
    /// Vérifier si le contraste est suffisant (WCAG AA)
    pub fn meets_wcag_aa(fg: Color32, bg: Color32, large_text: bool) -> bool {
        let ratio = Self::contrast_ratio(fg, bg);
        if large_text {
            ratio >= 3.0  // AA pour grand texte (18pt+ ou 14pt+ bold)
        } else {
            ratio >= 4.5  // AA pour texte normal
        }
    }
    
    /// Vérifier si le contraste est suffisant (WCAG AAA)
    pub fn meets_wcag_aaa(fg: Color32, bg: Color32, large_text: bool) -> bool {
        let ratio = Self::contrast_ratio(fg, bg);
        if large_text {
            ratio >= 4.5  // AAA pour grand texte
        } else {
            ratio >= 7.0  // AAA pour texte normal
        }
    }
}
```

## Ratios de contraste

```rust
impl DesignTokens {
    pub fn validate_contrasts(&self) -> Vec<ContrastIssue> {
        let mut issues = Vec::new();
        
        // Vérifier texte primaire sur fond primaire
        if !AccessibilityChecker::meets_wcag_aa(
            self.colors.text_primary,
            self.colors.bg_primary,
            false,
        ) {
            issues.push(ContrastIssue {
                element: "text_primary on bg_primary".to_string(),
                ratio: AccessibilityChecker::contrast_ratio(
                    self.colors.text_primary,
                    self.colors.bg_primary,
                ),
                required: 4.5,
            });
        }
        
        // Vérifier autres combinaisons...
        
        issues
    }
}

pub struct ContrastIssue {
    pub element: String,
    pub ratio: f32,
    pub required: f32,
}
```

## Outils de vérification

```rust
pub fn check_all_contrasts(tokens: &DesignTokens) -> AccessibilityReport {
    let mut report = AccessibilityReport::new();
    
    // Vérifier toutes les combinaisons texte/fond
    let combinations = vec![
        ("text_primary", tokens.colors.text_primary, "bg_primary", tokens.colors.bg_primary),
        ("text_secondary", tokens.colors.text_secondary, "bg_primary", tokens.colors.bg_primary),
        ("primary", tokens.colors.primary, "bg_primary", tokens.colors.bg_primary),
        // ...
    ];
    
    for (text_name, text_color, bg_name, bg_color) in combinations {
        let ratio = AccessibilityChecker::contrast_ratio(*text_color, *bg_color);
        let meets_aa = AccessibilityChecker::meets_wcag_aa(*text_color, *bg_color, false);
        let meets_aaa = AccessibilityChecker::meets_wcag_aaa(*text_color, *bg_color, false);
        
        report.add_check(ContrastCheck {
            text: text_name.to_string(),
            background: bg_name.to_string(),
            ratio,
            meets_aa,
            meets_aaa,
        });
    }
    
    report
}
```

## Résumé

- **WCAG 2.1** : Standards d'accessibilité web (applicables au desktop)
- **AA** : Ratio minimum 4.5:1 (texte normal), 3:1 (grand texte)
- **AAA** : Ratio minimum 7:1 (texte normal), 4.5:1 (grand texte)
- **Vérification** : Outils pour valider tous les contrastes
