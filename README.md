# Rust Desktop Mastery

## Architecture, UI, Design System et Mise en Production d'Applications Rust Réelles

---

Ce livre est un guide complet pour créer des applications desktop professionnelles en Rust. Il couvre l'ensemble du cycle de développement : de la conception architecturale au déploiement, en passant par la création d'interfaces modernes et de systèmes de design réutilisables.

## Pourquoi ce livre ?

Le développement desktop connaît une renaissance. Après des années de domination du web et des applications SaaS, les développeurs et les entreprises redécouvrent les avantages des applications natives :

- **Performance** : Démarrage instantané, réactivité parfaite
- **Souveraineté** : Données locales, pas de dépendance cloud
- **Fiabilité** : Fonctionne hors ligne, pas d'abonnement
- **Simplicité** : Un binaire, pas d'infrastructure

Rust est le langage idéal pour cette nouvelle ère : il combine la performance du C++ avec la sécurité mémoire et une ergonomie moderne.

## Structure du livre

### [Partie I — Le Mindset Rust Desktop](./partie-1-mindset-rust-desktop/)
Comprendre pourquoi Rust est le choix idéal pour le développement desktop moderne et adopter la bonne philosophie produit.

1. [Pourquoi Rust pour le logiciel moderne](./partie-1-mindset-rust-desktop/01-pourquoi-rust.md)
2. [Desktop is back](./partie-1-mindset-rust-desktop/02-desktop-is-back.md)
3. [Penser produit, pas repo GitHub](./partie-1-mindset-rust-desktop/03-penser-produit.md)

### [Partie II — Fondations Rust Solides](./partie-2-fondations-rust/)
Maîtriser les concepts Rust essentiels pour construire des applications robustes.

1. [Rust pour développeurs pragmatiques](./partie-2-fondations-rust/01-rust-pragmatique.md)
2. [Organiser un vrai projet](./partie-2-fondations-rust/02-organiser-projet.md)
3. [Async et concurrence desktop](./partie-2-fondations-rust/03-async-concurrence.md)
4. [Logs, debug et profiling](./partie-2-fondations-rust/04-logs-debug-profiling.md)

### [Partie III — Interfaces Modernes en Rust](./partie-3-interfaces-modernes/)
Construire des interfaces utilisateur professionnelles avec les frameworks Rust.

1. [Panorama des frameworks UI](./partie-3-interfaces-modernes/01-panorama-frameworks.md)
2. [Construire une UI avec egui](./partie-3-interfaces-modernes/02-construire-ui-egui.md)
3. [Créer son Design System](./partie-3-interfaces-modernes/03-design-system.md)
4. [Composants réutilisables](./partie-3-interfaces-modernes/04-composants-reutilisables.md)
5. [Patterns UX professionnels](./partie-3-interfaces-modernes/05-patterns-ux.md)
6. [Du wireframe SVG au code Rust](./partie-3-interfaces-modernes/06-wireframe-vers-code.md)

### [Partie IV — Architecture d'un Vrai Logiciel](./partie-4-architecture-logiciel/)
Structurer une application maintenable et évolutive.

1. [Clean architecture pour apps desktop](./partie-4-architecture-logiciel/01-clean-architecture.md)
2. [Event bus et communication interne](./partie-4-architecture-logiciel/02-event-bus.md)
3. [Gestion d'état robuste](./partie-4-architecture-logiciel/03-gestion-etat.md)
4. [Stockage local souverain](./partie-4-architecture-logiciel/04-stockage-local.md)
5. [Fichiers, import, export et PDF](./partie-4-architecture-logiciel/05-fichiers-import-export.md)
6. [Plugins et extensibilité](./partie-4-architecture-logiciel/06-plugins-extensibilite.md)

### [Partie V — Blocs Métiers Réels](./partie-5-blocs-metiers/)
Implémenter des fonctionnalités business concrètes.

1. [Construire un mini-ERP Rust](./partie-5-blocs-metiers/01-mini-erp.md)
2. [Moteur de recherche local](./partie-5-blocs-metiers/02-moteur-recherche.md)
3. [Dashboard et visualisation de données](./partie-5-blocs-metiers/03-dashboard-visualisation.md)
4. [IA locale dans une app Rust](./partie-5-blocs-metiers/04-ia-locale.md)
5. [Cas d'étude complet](./partie-5-blocs-metiers/05-cas-etude-complet.md)

### [Partie VI — Finition et Qualité Produit](./partie-6-finition-qualite/)
Polir l'application pour une qualité professionnelle.

1. [Performance perçue](./partie-6-finition-qualite/01-performance-percue.md)
2. [Accessibilité et ergonomie](./partie-6-finition-qualite/02-accessibilite-ergonomie.md)
3. [Packaging multiplateforme](./partie-6-finition-qualite/03-packaging-multiplateforme.md)
4. [Tests UI et assurance qualité](./partie-6-finition-qualite/04-tests-ui-qualite.md)

### [Partie VII — Production et Business](./partie-7-production-business/)
Transformer le code en produit viable.

1. [Open source vs produit commercial](./partie-7-production-business/01-opensource-vs-commercial.md)
2. [Vendre un logiciel desktop aujourd'hui](./partie-7-production-business/02-vendre-logiciel-desktop.md)
3. [Souveraineté numérique et données locales](./partie-7-production-business/03-souverainete-numerique.md)
4. [Créer son starter kit Rust réutilisable](./partie-7-production-business/04-starter-kit.md)
5. [Lancer et itérer](./partie-7-production-business/05-lancer-iterer.md)

### [Annexes](./annexes/)

- [A. Starter kit Rust desktop](./annexes/A-starter-kit.md)
- [B. Design tokens JSON](./annexes/B-design-tokens.md)
- [C. Structure d'architecture exemple](./annexes/C-structure-architecture.md)
- [D. Cheatsheet egui](./annexes/D-cheatsheet-egui.md)
- [E. Checklist ready to ship](./annexes/E-checklist-ship.md)
- [F. Ressources et crates utiles](./annexes/F-ressources-crates.md)

## Prérequis

- Connaissance basique de Rust (variables, fonctions, structs)
- Familiarité avec la ligne de commande
- Envie de créer des logiciels de qualité

## Comment utiliser ce livre

1. **Lecture linéaire** : Suivez les parties dans l'ordre pour une progression cohérente
2. **Référence** : Consultez les chapitres spécifiques selon vos besoins
3. **Pratique** : Chaque chapitre contient des exemples de code exécutables
4. **Projet fil rouge** : Construisez progressivement une application complète

## Code source

Tous les exemples de code sont disponibles et testés. Les snippets sont conçus pour être copiés-collés et adaptés à vos projets.

```rust
// Un avant-goût de ce que nous allons construire
use eframe::egui;

fn main() -> eframe::Result<()> {
    eframe::run_native(
        "Mon App Rust",
        eframe::NativeOptions::default(),
        Box::new(|_cc| Ok(Box::new(MonApp::default()))),
    )
}
```

---

**Commençons le voyage vers la maîtrise du développement desktop en Rust.**
