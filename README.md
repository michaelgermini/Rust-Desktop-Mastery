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

#### [Chapitre 1 : Pourquoi Rust pour le logiciel moderne](./partie-1-mindset-rust-desktop/01-pourquoi-rust/)
- [La fatigue JS / Electron / SaaS](./partie-1-mindset-rust-desktop/01-pourquoi-rust/01-fatigue-js-electron.md)
- [Performance native](./partie-1-mindset-rust-desktop/01-pourquoi-rust/02-performance-native.md)
- [Sécurité mémoire](./partie-1-mindset-rust-desktop/01-pourquoi-rust/03-securite-memoire.md)
- [Binaire unique](./partie-1-mindset-rust-desktop/01-pourquoi-rust/04-binaire-unique.md)
- [Offline-first](./partie-1-mindset-rust-desktop/01-pourquoi-rust/05-offline-first.md)
- [Souveraineté des données](./partie-1-mindset-rust-desktop/01-pourquoi-rust/06-souverainete-donnees.md)

#### [Chapitre 2 : Desktop is back](./partie-1-mindset-rust-desktop/02-desktop-is-back/)
- [Le retour des apps locales](./partie-1-mindset-rust-desktop/02-desktop-is-back/01-retour-apps-locales.md)
- [UX supérieure au navigateur](./partie-1-mindset-rust-desktop/02-desktop-is-back/02-ux-superieure.md)
- [IA locale, SQLite, fichiers](./partie-1-mindset-rust-desktop/02-desktop-is-back/03-ia-locale-sqlite.md)
- [Cas concrets : ERP, notes, outils, automation](./partie-1-mindset-rust-desktop/02-desktop-is-back/04-cas-concrets.md)
- [Quand ne pas utiliser le web](./partie-1-mindset-rust-desktop/02-desktop-is-back/05-quand-pas-web.md)

#### [Chapitre 3 : Penser produit, pas repo GitHub](./partie-1-mindset-rust-desktop/03-penser-produit/)
- [Ship plutôt que démo](./partie-1-mindset-rust-desktop/03-penser-produit/01-ship-plutot-que-demo.md)
- [Logiciel utilisable 8 heures par jour](./partie-1-mindset-rust-desktop/03-penser-produit/02-logiciel-8h-jour.md)
- [Dette UX vs dette technique](./partie-1-mindset-rust-desktop/03-penser-produit/03-dette-ux-vs-technique.md)
- [Le design comme avantage concurrentiel](./partie-1-mindset-rust-desktop/03-penser-produit/04-design-avantage-concurrentiel.md)

---

### [Partie II — Fondations Rust Solides](./partie-2-fondations-rust/)

Maîtriser les concepts Rust essentiels pour construire des applications robustes.

#### [Chapitre 4 : Rust pour développeurs pragmatiques](./partie-2-fondations-rust/01-rust-pragmatique/)
- [Ownership expliqué simplement](./partie-2-fondations-rust/01-rust-pragmatique/01-ownership-simplement.md)
- [Borrow checker sans douleur](./partie-2-fondations-rust/01-rust-pragmatique/02-borrow-checker.md)
- [Erreurs idiomatiques](./partie-2-fondations-rust/01-rust-pragmatique/03-erreurs-idiomatiques.md)
- [Patterns utiles](./partie-2-fondations-rust/01-rust-pragmatique/04-patterns-utiles.md)

#### [Chapitre 5 : Organiser un vrai projet](./partie-2-fondations-rust/02-organiser-projet/)
- [Cargo Workspace](./partie-2-fondations-rust/02-organiser-projet/01-cargo-workspace.md)
- [Crates Modulaires](./partie-2-fondations-rust/02-organiser-projet/02-crates-modulaires.md)
- [Architecture de Dossiers Claire](./partie-2-fondations-rust/02-organiser-projet/03-architecture-dossiers.md)
- [Tests et Benchmarks](./partie-2-fondations-rust/02-organiser-projet/04-tests-benchmarks.md)

#### [Chapitre 6 : Async et concurrence desktop](./partie-2-fondations-rust/03-async-concurrence/)
- [Tokio](./partie-2-fondations-rust/03-async-concurrence/01-tokio.md)
- [Channels (crossbeam)](./partie-2-fondations-rust/03-async-concurrence/02-channels-crossbeam.md)
- [Arc et ArcSwap](./partie-2-fondations-rust/03-async-concurrence/03-arc-arcswap.md)
- [Thread UI vs workers](./partie-2-fondations-rust/03-async-concurrence/04-thread-ui-workers.md)
- [Éviter les freezes](./partie-2-fondations-rust/03-async-concurrence/05-eviter-freezes.md)

#### [Chapitre 7 : Logs, debug et profiling](./partie-2-fondations-rust/04-logs-debug-profiling/)
- [Tracing](./partie-2-fondations-rust/04-logs-debug-profiling/01-tracing.md)
- [Logs visuels](./partie-2-fondations-rust/04-logs-debug-profiling/02-logs-visuels.md)
- [Flamegraphs](./partie-2-fondations-rust/04-logs-debug-profiling/03-flamegraphs.md)
- [Optimiser sans se mentir](./partie-2-fondations-rust/04-logs-debug-profiling/04-optimiser-sans-mentir.md)

---

### [Partie III — Interfaces Modernes en Rust](./partie-3-interfaces-modernes/)

Construire des interfaces utilisateur professionnelles avec les frameworks Rust.

#### [Chapitre 8 : Panorama des frameworks UI](./partie-3-interfaces-modernes/01-panorama-frameworks/)
- [egui](./partie-3-interfaces-modernes/01-panorama-frameworks/01-egui.md)
- [Tauri](./partie-3-interfaces-modernes/01-panorama-frameworks/02-tauri.md)
- [wgpu](./partie-3-interfaces-modernes/01-panorama-frameworks/03-wgpu.md)
- [iced](./partie-3-interfaces-modernes/01-panorama-frameworks/04-iced.md)
- [Quand choisir quoi](./partie-3-interfaces-modernes/01-panorama-frameworks/05-quand-choisir-quoi.md)

#### [Chapitre 9 : Construire une UI avec egui](./partie-3-interfaces-modernes/02-construire-ui-egui/)
- [Immediate mode mental model](./partie-3-interfaces-modernes/02-construire-ui-egui/01-immediate-mode.md)
- [Layouts](./partie-3-interfaces-modernes/02-construire-ui-egui/02-layouts.md)
- [Widgets](./partie-3-interfaces-modernes/02-construire-ui-egui/03-widgets.md)
- [Gestion propre de l'état](./partie-3-interfaces-modernes/02-construire-ui-egui/04-gestion-etat.md)

#### [Chapitre 10 : Créer son Design System](./partie-3-interfaces-modernes/03-design-system/)
- [Tokens](./partie-3-interfaces-modernes/03-design-system/01-tokens.md)
- [Couleurs](./partie-3-interfaces-modernes/03-design-system/02-couleurs.md)
- [Spacing](./partie-3-interfaces-modernes/03-design-system/03-spacing.md)
- [Typographie](./partie-3-interfaces-modernes/03-design-system/04-typographie.md)
- [Thèmes](./partie-3-interfaces-modernes/03-design-system/05-themes.md)
- [DPI scaling](./partie-3-interfaces-modernes/03-design-system/06-dpi-scaling.md)

#### [Chapitre 11 : Composants réutilisables](./partie-3-interfaces-modernes/04-composants-reutilisables/)
- [Button](./partie-3-interfaces-modernes/04-composants-reutilisables/01-button.md)
- [Input](./partie-3-interfaces-modernes/04-composants-reutilisables/02-input.md)
- [Table](./partie-3-interfaces-modernes/04-composants-reutilisables/03-table.md)
- [Sidebar](./partie-3-interfaces-modernes/04-composants-reutilisables/04-sidebar.md)
- [Modals](./partie-3-interfaces-modernes/04-composants-reutilisables/05-modals.md)
- [Toasts](./partie-3-interfaces-modernes/04-composants-reutilisables/06-toasts.md)
- [Inspector panels](./partie-3-interfaces-modernes/04-composants-reutilisables/07-inspector-panels.md)
- [Command palette](./partie-3-interfaces-modernes/04-composants-reutilisables/08-command-palette.md)

#### [Chapitre 12 : Patterns UX professionnels](./partie-3-interfaces-modernes/05-patterns-ux/)
- [Loading](./partie-3-interfaces-modernes/05-patterns-ux/01-loading.md)
- [Empty states](./partie-3-interfaces-modernes/05-patterns-ux/02-empty-states.md)
- [Erreurs](./partie-3-interfaces-modernes/05-patterns-ux/03-erreurs.md)
- [Undo et redo](./partie-3-interfaces-modernes/05-patterns-ux/04-undo-redo.md)
- [Autosave](./partie-3-interfaces-modernes/05-patterns-ux/05-autosave.md)
- [Feedback instantané](./partie-3-interfaces-modernes/05-patterns-ux/06-feedback-instantane.md)

#### [Chapitre 13 : Du wireframe SVG au code Rust](./partie-3-interfaces-modernes/06-wireframe-vers-code/)
- [Maquettes SVG](./partie-3-interfaces-modernes/06-wireframe-vers-code/01-maquettes-svg.md)
- [Mapping IDs vers composants](./partie-3-interfaces-modernes/06-wireframe-vers-code/02-mapping-ids.md)
- [Design vers egui](./partie-3-interfaces-modernes/06-wireframe-vers-code/03-design-vers-egui.md)
- [Workflow rapide design vers code](./partie-3-interfaces-modernes/06-wireframe-vers-code/04-workflow.md)

---

### [Partie IV — Architecture d'un Vrai Logiciel](./partie-4-architecture-logiciel/)

Structurer une application maintenable et évolutive.

#### [Chapitre 14 : Clean Architecture pour apps desktop](./partie-4-architecture-logiciel/01-clean-architecture/)
- [Couches (Core / App / UI / Infra)](./partie-4-architecture-logiciel/01-clean-architecture/01-couches.md)
- [Séparation logique](./partie-4-architecture-logiciel/01-clean-architecture/02-separation-logique.md)
- [Découplage](./partie-4-architecture-logiciel/01-clean-architecture/03-decouplage.md)
- [Dépendances](./partie-4-architecture-logiciel/01-clean-architecture/04-dependances.md)

#### [Chapitre 15 : Event Bus et communication interne](./partie-4-architecture-logiciel/02-event-bus/)
- [Channels](./partie-4-architecture-logiciel/02-event-bus/01-channels.md)
- [Pub/sub](./partie-4-architecture-logiciel/02-event-bus/02-pub-sub.md)
- [Messages](./partie-4-architecture-logiciel/02-event-bus/03-messages.md)
- [Flux de données clair](./partie-4-architecture-logiciel/02-event-bus/04-flux-donnees.md)

#### [Chapitre 16 : Gestion d'état robuste](./partie-4-architecture-logiciel/03-gestion-etat/)
- [Store global](./partie-4-architecture-logiciel/03-gestion-etat/01-store-global.md)
- [État local](./partie-4-architecture-logiciel/03-gestion-etat/02-etat-local.md)
- [Cache](./partie-4-architecture-logiciel/03-gestion-etat/03-cache.md)
- [Undo stack](./partie-4-architecture-logiciel/03-gestion-etat/04-undo-stack.md)

#### [Chapitre 17 : Stockage local souverain](./partie-4-architecture-logiciel/04-stockage-local/)
- [SQLite](./partie-4-architecture-logiciel/04-stockage-local/01-sqlite.md)
- [Migrations](./partie-4-architecture-logiciel/04-stockage-local/02-migrations.md)
- [Offline-first](./partie-4-architecture-logiciel/04-stockage-local/03-offline-first.md)
- [Chiffrement](./partie-4-architecture-logiciel/04-stockage-local/04-chiffrement.md)

#### [Chapitre 18 : Fichiers, import, export et PDF](./partie-4-architecture-logiciel/05-fichiers-import-export/)
- [Rapports](./partie-4-architecture-logiciel/05-fichiers-import-export/01-rapports.md)
- [Factures](./partie-4-architecture-logiciel/05-fichiers-import-export/02-factures.md)
- [CSV et JSON](./partie-4-architecture-logiciel/05-fichiers-import-export/03-csv-json.md)
- [Intégration système](./partie-4-architecture-logiciel/05-fichiers-import-export/04-integration-systeme.md)

#### [Chapitre 19 : Plugins et extensibilité](./partie-4-architecture-logiciel/06-plugins-extensibilite/)
- [Modules dynamiques](./partie-4-architecture-logiciel/06-plugins-extensibilite/01-modules-dynamiques.md)
- [Feature flags](./partie-4-architecture-logiciel/06-plugins-extensibilite/02-feature-flags.md)
- [Architecture plugin](./partie-4-architecture-logiciel/06-plugins-extensibilite/03-architecture-plugin.md)

---

### [Partie V — Blocs Métiers Réels](./partie-5-blocs-metiers/)

Implémenter des fonctionnalités business concrètes.

#### [Chapitre 20 : Construire un mini-ERP Rust](./partie-5-blocs-metiers/01-mini-erp/)
- [Clients](./partie-5-blocs-metiers/01-mini-erp/01-clients.md)
- [Factures](./partie-5-blocs-metiers/01-mini-erp/02-factures.md)
- [TVA](./partie-5-blocs-metiers/01-mini-erp/03-tva.md)
- [PDF](./partie-5-blocs-metiers/01-mini-erp/04-pdf.md)
- [Exports](./partie-5-blocs-metiers/01-mini-erp/05-exports.md)

#### [Chapitre 21 : Moteur de recherche local](./partie-5-blocs-metiers/02-moteur-recherche/)
- [Tantivy](./partie-5-blocs-metiers/02-moteur-recherche/01-tantivy.md)
- [Indexation](./partie-5-blocs-metiers/02-moteur-recherche/02-indexation.md)
- [Full-text search](./partie-5-blocs-metiers/02-moteur-recherche/03-full-text-search.md)
- [Performance](./partie-5-blocs-metiers/02-moteur-recherche/04-performance.md)

#### [Chapitre 22 : Dashboard et visualisation de données](./partie-5-blocs-metiers/03-dashboard-visualisation/)
- [KPI cards](./partie-5-blocs-metiers/03-dashboard-visualisation/01-kpi-cards.md)
- [Charts](./partie-5-blocs-metiers/03-dashboard-visualisation/02-charts.md)
- [Temps réel](./partie-5-blocs-metiers/03-dashboard-visualisation/03-temps-reel.md)
- [UX data](./partie-5-blocs-metiers/03-dashboard-visualisation/04-ux-data.md)

#### [Chapitre 23 : IA locale dans une app Rust](./partie-5-blocs-metiers/04-ia-locale/)
- [Embeddings](./partie-5-blocs-metiers/04-ia-locale/01-embeddings.md)
- [Recherche sémantique](./partie-5-blocs-metiers/04-ia-locale/02-recherche-semantique.md)
- [Inférence CPU](./partie-5-blocs-metiers/04-ia-locale/03-inference-cpu.md)
- [Souveraineté IA](./partie-5-blocs-metiers/04-ia-locale/04-souverainete-ia.md)

#### [Chapitre 24 : Cas d'étude complet](./partie-5-blocs-metiers/05-cas-etude-complet/)
- [De zéro à finie](./partie-5-blocs-metiers/05-cas-etude-complet/01-zero-a-finie.md)
- [Code réel](./partie-5-blocs-metiers/05-cas-etude-complet/02-code-reel.md)
- [Design system](./partie-5-blocs-metiers/05-cas-etude-complet/03-design-system.md)
- [Packaging](./partie-5-blocs-metiers/05-cas-etude-complet/04-packaging.md)

---

### [Partie VI — Finition et Qualité Produit](./partie-6-finition-qualite/)

Polir l'application pour une qualité professionnelle.

#### [Chapitre 25 : Performance perçue](./partie-6-finition-qualite/01-performance-percue/)
- [Skeleton](./partie-6-finition-qualite/01-performance-percue/01-skeleton.md)
- [Lazy loading](./partie-6-finition-qualite/01-performance-percue/02-lazy-loading.md)
- [Démarrage instantané](./partie-6-finition-qualite/01-performance-percue/03-demarrage-instantane.md)
- [Caches](./partie-6-finition-qualite/01-performance-percue/04-caches.md)

#### [Chapitre 26 : Accessibilité et ergonomie](./partie-6-finition-qualite/02-accessibilite-ergonomie/)
- [Raccourcis clavier](./partie-6-finition-qualite/02-accessibilite-ergonomie/01-raccourcis-clavier.md)
- [Navigation souris](./partie-6-finition-qualite/02-accessibilite-ergonomie/02-navigation-souris.md)
- [Contrastes](./partie-6-finition-qualite/02-accessibilite-ergonomie/03-contrastes.md)
- [Tailles et lisibilité](./partie-6-finition-qualite/02-accessibilite-ergonomie/04-tailles-lisibilite.md)

#### [Chapitre 27 : Packaging multiplateforme](./partie-6-finition-qualite/03-packaging-multiplateforme/)
- [Windows](./partie-6-finition-qualite/03-packaging-multiplateforme/01-windows.md)
- [macOS](./partie-6-finition-qualite/03-packaging-multiplateforme/02-macos.md)
- [Linux](./partie-6-finition-qualite/03-packaging-multiplateforme/03-linux.md)
- [Installers](./partie-6-finition-qualite/03-packaging-multiplateforme/04-installers.md)
- [Mises à jour](./partie-6-finition-qualite/03-packaging-multiplateforme/05-mises-a-jour.md)

#### [Chapitre 28 : Tests UI et assurance qualité](./partie-6-finition-qualite/04-tests-ui-qualite/)
- [Tests unitaires](./partie-6-finition-qualite/04-tests-ui-qualite/01-tests-unitaires.md)
- [Snapshots UI](./partie-6-finition-qualite/04-tests-ui-qualite/02-snapshots-ui.md)
- [Tests manuels](./partie-6-finition-qualite/04-tests-ui-qualite/03-tests-manuels.md)

---

### [Partie VII — Production et Business](./partie-7-production-business/)

Transformer le code en produit viable.

#### [Chapitre 29 : Licences et modèles économiques](./partie-7-production-business/01-licences-modeles-economiques/)
- [Types de licences](./partie-7-production-business/01-licences-modeles-economiques/01-types-licences.md)
- [Génération et validation](./partie-7-production-business/01-licences-modeles-economiques/02-generation-validation.md)
- [Activation offline](./partie-7-production-business/01-licences-modeles-economiques/03-activation-offline.md)
- [Modèle freemium](./partie-7-production-business/01-licences-modeles-economiques/04-modele-freemium.md)

#### [Chapitre 30 : Distribution et marketing](./partie-7-production-business/02-distribution-marketing/)
- [Site web produit](./partie-7-production-business/02-distribution-marketing/01-site-web.md)
- [SEO pour applications desktop](./partie-7-production-business/02-distribution-marketing/02-seo.md)
- [Page de téléchargement](./partie-7-production-business/02-distribution-marketing/03-page-telechargement.md)
- [Canaux de distribution](./partie-7-production-business/02-distribution-marketing/04-canaux.md)

#### [Chapitre 31 : Support et maintenance](./partie-7-production-business/03-support-maintenance/)
- [Documentation utilisateur](./partie-7-production-business/03-support-maintenance/01-documentation.md)
- [Feedback in-app](./partie-7-production-business/03-support-maintenance/02-feedback.md)
- [Télémétrie respectueuse](./partie-7-production-business/03-support-maintenance/03-telemetrie.md)
- [Gestion des bugs](./partie-7-production-business/03-support-maintenance/04-gestion-bugs.md)

#### [Chapitre 32 : Monétisation avancée](./partie-7-production-business/04-monetisation-avancee/)
- [Modules complémentaires](./partie-7-production-business/04-monetisation-avancee/01-modules-complementaires.md)
- [Personnalisation](./partie-7-production-business/04-monetisation-avancee/02-personnalisation.md)
- [Services associés](./partie-7-production-business/04-monetisation-avancee/03-services.md)
- [Affiliation](./partie-7-production-business/04-monetisation-avancee/04-affiliation.md)

#### [Chapitre 33 : Vision long terme](./partie-7-production-business/05-vision-long-terme/)
- [Roadmap produit](./partie-7-production-business/05-vision-long-terme/01-roadmap.md)
- [Communauté](./partie-7-production-business/05-vision-long-terme/02-communaute.md)
- [Open source](./partie-7-production-business/05-vision-long-terme/03-open-source.md)
- [Évolution technique](./partie-7-production-business/05-vision-long-terme/04-evolution-technique.md)
- [Métriques](./partie-7-production-business/05-vision-long-terme/05-metriques.md)

---

### [Annexes](./annexes/)

Ressources de référence complémentaires.

- [A. Cargo.toml complet commenté](./annexes/A-cargo-toml-complet.md)
- [B. Cheatsheet egui](./annexes/B-cheatsheet-egui.md)
- [C. Patterns Rust essentiels](./annexes/C-patterns-rust.md)
- [D. Checklist de lancement](./annexes/D-checklist-lancement.md)
- [E. Ressources et liens](./annexes/E-ressources-liens.md)
- [F. Glossaire](./annexes/F-glossaire.md)

---

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

## Licence

Ce livre est publié sous licence MIT. Vous êtes libre de l'utiliser, le modifier et le distribuer.

---

**Commençons le voyage vers la maîtrise du développement desktop en Rust.**
