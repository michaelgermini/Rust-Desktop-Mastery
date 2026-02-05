# 5.3 Architecture de Dossiers Claire

## Structure complète recommandée

```
mon_application/
├── Cargo.toml
├── Cargo.lock
├── README.md
├── LICENSE
├── .gitignore
│
├── crates/
│   ├── app/
│   │   ├── Cargo.toml
│   │   └── src/
│   │       ├── main.rs          # Point d'entrée
│   │       └── cli.rs           # Arguments ligne de commande
│   │
│   ├── core/
│   │   ├── Cargo.toml
│   │   └── src/
│   │       ├── lib.rs
│   │       ├── domain/          # Entités métier
│   │       │   ├── mod.rs
│   │       │   ├── invoice.rs
│   │       │   ├── customer.rs
│   │       │   └── product.rs
│   │       ├── services/        # Logique métier
│   │       │   ├── mod.rs
│   │       │   ├── billing.rs
│   │       │   └── reporting.rs
│   │       └── error.rs
│   │
│   ├── ui/
│   │   ├── Cargo.toml
│   │   └── src/
│   │       ├── lib.rs
│   │       ├── app.rs           # Application egui
│   │       ├── theme/           # Design system
│   │       │   ├── mod.rs
│   │       │   ├── colors.rs
│   │       │   ├── spacing.rs
│   │       │   └── typography.rs
│   │       ├── components/     # Composants réutilisables
│   │       │   ├── mod.rs
│   │       │   ├── button.rs
│   │       │   ├── input.rs
│   │       │   ├── table.rs
│   │       │   └── modal.rs
│   │       └── screens/         # Écrans de l'app
│   │           ├── mod.rs
│   │           ├── dashboard.rs
│   │           ├── invoices.rs
│   │           └── settings.rs
│   │
│   └── storage/
│       ├── Cargo.toml
│       └── src/
│           ├── lib.rs
│           ├── database.rs      # Connexion SQLite
│           ├── migrations/     # Migrations SQL
│           │   ├── mod.rs
│           │   ├── v001_initial.sql
│           │   └── v002_add_index.sql
│           └── repository/     # Repositories
│               ├── mod.rs
│               ├── invoice_repo.rs
│               └── customer_repo.rs
│
├── tests/                      # Tests d'intégration
│   ├── common/
│   │   └── mod.rs
│   ├── invoice_flow_test.rs
│   └── export_test.rs
│
├── benches/                    # Benchmarks
│   └── search_benchmark.rs
│
├── assets/                     # Ressources statiques
│   ├── icons/
│   ├── fonts/
│   └── images/
│
├── config/                     # Fichiers de config exemple
│   └── default.toml
│
└── scripts/                    # Scripts de build/deploy
    ├── build-release.sh
    └── package-windows.ps1
```

## Conventions de nommage

```rust
// Fichiers et modules : snake_case
// crates/core/src/domain/invoice_item.rs
mod invoice_item;

// Structs et Enums : PascalCase
pub struct InvoiceItem { ... }
pub enum InvoiceStatus { ... }

// Fonctions et méthodes : snake_case
fn calculate_total() { ... }
impl Invoice {
    fn add_item(&mut self) { ... }
}

// Constantes : SCREAMING_SNAKE_CASE
const MAX_ITEMS_PER_INVOICE: usize = 100;
const DEFAULT_TAX_RATE: Decimal = dec!(0.20);

// Traits : PascalCase, souvent avec suffixe -able ou -er
trait Billable { ... }
trait InvoiceRepository { ... }
```

## Organisation des modules

### Principe : un fichier = un concept

```rust
// ✅ BON : Un fichier par entité
// domain/invoice.rs
// domain/customer.rs
// domain/product.rs

// ❌ MAUVAIS : Tout dans un fichier
// domain/entities.rs  // Trop gros, difficile à naviguer
```

### Modules imbriqués

```rust
// domain/mod.rs
mod invoice;
mod customer;
mod product;

// Expose ce qui est nécessaire
pub use invoice::{Invoice, InvoiceId, InvoiceStatus};
pub use customer::{Customer, CustomerId};
pub use product::{Product, ProductId};
```

## Résumé

- **Structure claire** : Un dossier par responsabilité
- **Nommage cohérent** : Suivre les conventions Rust
- **Modules organisés** : Un fichier = un concept
- **Exposition contrôlée** : Exposer seulement ce qui est nécessaire
