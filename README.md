# Rust Desktop Mastery

<p align="center">
  <strong>Architecture, UI, Design System et Mise en Production d'Applications Rust Réelles</strong>
</p>

<p align="center">
  <em>Le guide complet pour créer des applications desktop professionnelles en Rust</em>
</p>

---

## Vue d'ensemble

Ce livre est un guide complet pour créer des applications desktop professionnelles en Rust. Il couvre l'ensemble du cycle de développement : de la conception architecturale au déploiement, en passant par la création d'interfaces modernes et de systèmes de design réutilisables.

| Statistiques du livre |  |
|----------------------|---|
| **Parties** | 7 |
| **Chapitres** | 33 |
| **Sous-sections** | 100+ |
| **Annexes** | 6 |

## Pourquoi ce livre ?

Le développement desktop connaît une renaissance. Après des années de domination du web et des applications SaaS, les développeurs et les entreprises redécouvrent les avantages des applications natives :

| Approche Web/SaaS | Approche Desktop Rust |
|-------------------|----------------------|
| Latence réseau | Réponse instantanée |
| Abonnement mensuel | Achat unique possible |
| Données sur serveur tiers | Données 100% locales |
| Dépendance internet | Fonctionne offline |
| Complexité infrastructure | Un seul fichier binaire |

**Rust** est le langage idéal pour cette nouvelle ère : il combine la performance du C++ avec la sécurité mémoire et une ergonomie moderne.

---

## Structure du livre

### [Partie I — Le Mindset Rust Desktop](./partie-1-mindset-rust-desktop/)

> **Objectif** : Comprendre pourquoi Rust est le choix idéal pour le desktop et adopter la bonne philosophie produit.

Cette partie pose les fondations philosophiques et stratégiques. Avant d'écrire une seule ligne de code, il est crucial de comprendre *pourquoi* nous faisons ces choix.

#### [Chapitre 1 : Pourquoi Rust pour le logiciel moderne](./partie-1-mindset-rust-desktop/01-pourquoi-rust/)

> Explore les raisons fondamentales qui font de Rust le choix idéal : fatigue de l'écosystème JS/Electron, performance native, sécurité mémoire, et souveraineté des données.

- [La fatigue JS / Electron / SaaS](./partie-1-mindset-rust-desktop/01-pourquoi-rust/01-fatigue-js-electron.md) — Problèmes de mémoire, taille, instabilité npm
- [Performance native](./partie-1-mindset-rust-desktop/01-pourquoi-rust/02-performance-native.md) — Zero-cost abstractions, comparaison avec JS/JVM
- [Sécurité mémoire](./partie-1-mindset-rust-desktop/01-pourquoi-rust/03-securite-memoire.md) — Ownership, borrowing, null safety
- [Binaire unique](./partie-1-mindset-rust-desktop/01-pourquoi-rust/04-binaire-unique.md) — Déploiement simplifié, pas de DLL hell
- [Offline-first](./partie-1-mindset-rust-desktop/01-pourquoi-rust/05-offline-first.md) — Architecture locale par défaut
- [Souveraineté des données](./partie-1-mindset-rust-desktop/01-pourquoi-rust/06-souverainete-donnees.md) — Contrôle utilisateur, RGPD

#### [Chapitre 2 : Desktop is back](./partie-1-mindset-rust-desktop/02-desktop-is-back/)

> Le retour en force des applications locales : pourquoi et quand le desktop surpasse le web.

- [Le retour des apps locales](./partie-1-mindset-rust-desktop/02-desktop-is-back/01-retour-apps-locales.md) — Tendances et motivations
- [UX supérieure au navigateur](./partie-1-mindset-rust-desktop/02-desktop-is-back/02-ux-superieure.md) — Raccourcis, intégration système
- [IA locale, SQLite, fichiers](./partie-1-mindset-rust-desktop/02-desktop-is-back/03-ia-locale-sqlite.md) — Technologies clés
- [Cas concrets](./partie-1-mindset-rust-desktop/02-desktop-is-back/04-cas-concrets.md) — ERP, notes, outils, automation
- [Quand ne pas utiliser le web](./partie-1-mindset-rust-desktop/02-desktop-is-back/05-quand-pas-web.md) — Critères de décision

#### [Chapitre 3 : Penser produit, pas repo GitHub](./partie-1-mindset-rust-desktop/03-penser-produit/)

> Adopter une mentalité produit orientée utilisateur plutôt qu'une approche technique.

- [Ship plutôt que démo](./partie-1-mindset-rust-desktop/03-penser-produit/01-ship-plutot-que-demo.md) — Livrer de la valeur
- [Logiciel utilisable 8h/jour](./partie-1-mindset-rust-desktop/03-penser-produit/02-logiciel-8h-jour.md) — Ergonomie professionnelle
- [Dette UX vs dette technique](./partie-1-mindset-rust-desktop/03-penser-produit/03-dette-ux-vs-technique.md) — Prioriser l'expérience
- [Le design comme avantage](./partie-1-mindset-rust-desktop/03-penser-produit/04-design-avantage-concurrentiel.md) — Différenciation

---

### [Partie II — Fondations Rust Solides](./partie-2-fondations-rust/)

> **Objectif** : Maîtriser les concepts Rust essentiels pour construire des applications robustes.

Cette partie couvre les concepts Rust nécessaires au développement desktop. L'objectif n'est pas d'enseigner Rust depuis zéro, mais de maîtriser les patterns spécifiques aux applications réelles.

#### [Chapitre 4 : Rust pour développeurs pragmatiques](./partie-2-fondations-rust/01-rust-pragmatique/)

> Les concepts Rust essentiels expliqués de manière pratique, avec focus sur le développement desktop.

- [Ownership expliqué simplement](./partie-2-fondations-rust/01-rust-pragmatique/01-ownership-simplement.md) — Les 3 règles, Move vs Copy
- [Borrow checker sans douleur](./partie-2-fondations-rust/01-rust-pragmatique/02-borrow-checker.md) — Références, règle d'or
- [Erreurs idiomatiques](./partie-2-fondations-rust/01-rust-pragmatique/03-erreurs-idiomatiques.md) — Result, Option, thiserror, anyhow
- [Patterns utiles](./partie-2-fondations-rust/01-rust-pragmatique/04-patterns-utiles.md) — Builder, Newtype, State Machine

#### [Chapitre 5 : Organiser un vrai projet](./partie-2-fondations-rust/02-organiser-projet/)

> Structurer une application Rust professionnelle, maintenable et évolutive.

- [Cargo Workspace](./partie-2-fondations-rust/02-organiser-projet/01-cargo-workspace.md) — Multi-crates, configuration
- [Crates Modulaires](./partie-2-fondations-rust/02-organiser-projet/02-crates-modulaires.md) — Core, storage, UI séparés
- [Architecture de Dossiers](./partie-2-fondations-rust/02-organiser-projet/03-architecture-dossiers.md) — Conventions, organisation
- [Tests et Benchmarks](./partie-2-fondations-rust/02-organiser-projet/04-tests-benchmarks.md) — Unitaires, intégration, Criterion

#### [Chapitre 6 : Async et concurrence desktop](./partie-2-fondations-rust/03-async-concurrence/)

> Gérer la concurrence sans bloquer l'interface utilisateur.

- [Tokio](./partie-2-fondations-rust/03-async-concurrence/01-tokio.md) — Runtime async pour desktop
- [Channels (crossbeam)](./partie-2-fondations-rust/03-async-concurrence/02-channels-crossbeam.md) — Communication inter-threads
- [Arc et ArcSwap](./partie-2-fondations-rust/03-async-concurrence/03-arc-arcswap.md) — Partage de données thread-safe
- [Thread UI vs workers](./partie-2-fondations-rust/03-async-concurrence/04-thread-ui-workers.md) — Séparation des responsabilités
- [Éviter les freezes](./partie-2-fondations-rust/03-async-concurrence/05-eviter-freezes.md) — UI toujours réactive

#### [Chapitre 7 : Logs, debug et profiling](./partie-2-fondations-rust/04-logs-debug-profiling/)

> Diagnostiquer et optimiser les performances de manière rigoureuse.

- [Tracing](./partie-2-fondations-rust/04-logs-debug-profiling/01-tracing.md) — Logs structurés, spans, niveaux
- [Logs visuels](./partie-2-fondations-rust/04-logs-debug-profiling/02-logs-visuels.md) — Panel de logs intégré
- [Flamegraphs](./partie-2-fondations-rust/04-logs-debug-profiling/03-flamegraphs.md) — Profiling visuel
- [Optimiser sans se mentir](./partie-2-fondations-rust/04-logs-debug-profiling/04-optimiser-sans-mentir.md) — Mesurer avant d'optimiser

---

### [Partie III — Interfaces Modernes en Rust](./partie-3-interfaces-modernes/)

> **Objectif** : Construire des interfaces utilisateur professionnelles et réactives.

Cette partie est consacrée à la création d'UI en Rust. Nous couvrons les frameworks disponibles, les patterns de design system, et les techniques pour construire des interfaces modernes.

#### [Chapitre 8 : Panorama des frameworks UI](./partie-3-interfaces-modernes/01-panorama-frameworks/)

> Comparaison détaillée des frameworks UI Rust pour choisir le bon outil.

- [egui](./partie-3-interfaces-modernes/01-panorama-frameworks/01-egui.md) — Immediate mode, simple et rapide
- [Tauri](./partie-3-interfaces-modernes/01-panorama-frameworks/02-tauri.md) — Web frontend + Rust backend
- [wgpu](./partie-3-interfaces-modernes/01-panorama-frameworks/03-wgpu.md) — GPU rendering bas niveau
- [iced](./partie-3-interfaces-modernes/01-panorama-frameworks/04-iced.md) — Déclaratif, style Elm
- [Quand choisir quoi](./partie-3-interfaces-modernes/01-panorama-frameworks/05-quand-choisir-quoi.md) — Matrice de décision

#### [Chapitre 9 : Construire une UI avec egui](./partie-3-interfaces-modernes/02-construire-ui-egui/)

> Guide pratique complet pour maîtriser egui, le framework choisi pour ce livre.

- [Immediate mode](./partie-3-interfaces-modernes/02-construire-ui-egui/01-immediate-mode.md) — Concept et mental model
- [Layouts](./partie-3-interfaces-modernes/02-construire-ui-egui/02-layouts.md) — Vertical, horizontal, grid, panels
- [Widgets](./partie-3-interfaces-modernes/02-construire-ui-egui/03-widgets.md) — De base et avancés
- [Gestion de l'état](./partie-3-interfaces-modernes/02-construire-ui-egui/04-gestion-etat.md) — Local vs global, persistence

#### [Chapitre 10 : Créer son Design System](./partie-3-interfaces-modernes/03-design-system/)

> Construire un système de design cohérent et maintenable.

- [Tokens](./partie-3-interfaces-modernes/03-design-system/01-tokens.md) — Variables de design centralisées
- [Couleurs](./partie-3-interfaces-modernes/03-design-system/02-couleurs.md) — Palettes, sémantique
- [Spacing](./partie-3-interfaces-modernes/03-design-system/03-spacing.md) — Grille, marges, paddings
- [Typographie](./partie-3-interfaces-modernes/03-design-system/04-typographie.md) — Hiérarchie, lisibilité
- [Thèmes](./partie-3-interfaces-modernes/03-design-system/05-themes.md) — Dark/light mode
- [DPI scaling](./partie-3-interfaces-modernes/03-design-system/06-dpi-scaling.md) — Support multi-résolutions

#### [Chapitre 11 : Composants réutilisables](./partie-3-interfaces-modernes/04-composants-reutilisables/)

> Bibliothèque de composants UI professionnels prêts à l'emploi.

- [Button](./partie-3-interfaces-modernes/04-composants-reutilisables/01-button.md) — Variantes, états, icônes
- [Input](./partie-3-interfaces-modernes/04-composants-reutilisables/02-input.md) — Texte, validation, masques
- [Table](./partie-3-interfaces-modernes/04-composants-reutilisables/03-table.md) — Tri, pagination, sélection
- [Sidebar](./partie-3-interfaces-modernes/04-composants-reutilisables/04-sidebar.md) — Navigation, collapse
- [Modals](./partie-3-interfaces-modernes/04-composants-reutilisables/05-modals.md) — Dialogues, confirmations
- [Toasts](./partie-3-interfaces-modernes/04-composants-reutilisables/06-toasts.md) — Notifications temporaires
- [Inspector panels](./partie-3-interfaces-modernes/04-composants-reutilisables/07-inspector-panels.md) — Propriétés, détails
- [Command palette](./partie-3-interfaces-modernes/04-composants-reutilisables/08-command-palette.md) — Recherche d'actions (Ctrl+K)

#### [Chapitre 12 : Patterns UX professionnels](./partie-3-interfaces-modernes/05-patterns-ux/)

> Les patterns UX qui font la différence entre une app amateur et professionnelle.

- [Loading](./partie-3-interfaces-modernes/05-patterns-ux/01-loading.md) — États de chargement
- [Empty states](./partie-3-interfaces-modernes/05-patterns-ux/02-empty-states.md) — Écrans vides informatifs
- [Erreurs](./partie-3-interfaces-modernes/05-patterns-ux/03-erreurs.md) — Messages clairs et actionnables
- [Undo et redo](./partie-3-interfaces-modernes/05-patterns-ux/04-undo-redo.md) — Historique d'actions
- [Autosave](./partie-3-interfaces-modernes/05-patterns-ux/05-autosave.md) — Sauvegarde transparente
- [Feedback instantané](./partie-3-interfaces-modernes/05-patterns-ux/06-feedback-instantane.md) — Réponse < 100ms

#### [Chapitre 13 : Du wireframe SVG au code Rust](./partie-3-interfaces-modernes/06-wireframe-vers-code/)

> Workflow efficace pour transformer des maquettes en code.

- [Maquettes SVG](./partie-3-interfaces-modernes/06-wireframe-vers-code/01-maquettes-svg.md) — Création de wireframes
- [Mapping IDs](./partie-3-interfaces-modernes/06-wireframe-vers-code/02-mapping-ids.md) — Lier design et composants
- [Design vers egui](./partie-3-interfaces-modernes/06-wireframe-vers-code/03-design-vers-egui.md) — Traduction pratique
- [Workflow rapide](./partie-3-interfaces-modernes/06-wireframe-vers-code/04-workflow.md) — Itérations design/code

---

### [Partie IV — Architecture d'un Vrai Logiciel](./partie-4-architecture-logiciel/)

> **Objectif** : Structurer une application maintenable, testable et évolutive.

Cette partie couvre l'architecture logicielle. Nous appliquons les principes de clean architecture adaptés au contexte desktop.

#### [Chapitre 14 : Clean Architecture pour apps desktop](./partie-4-architecture-logiciel/01-clean-architecture/)

> Organiser le code en couches indépendantes et testables.

- [Couches](./partie-4-architecture-logiciel/01-clean-architecture/01-couches.md) — Core / App / UI / Infra
- [Séparation logique](./partie-4-architecture-logiciel/01-clean-architecture/02-separation-logique.md) — Responsabilités claires
- [Découplage](./partie-4-architecture-logiciel/01-clean-architecture/03-decouplage.md) — Interfaces et abstractions
- [Dépendances](./partie-4-architecture-logiciel/01-clean-architecture/04-dependances.md) — Vers l'intérieur uniquement

#### [Chapitre 15 : Event Bus et communication interne](./partie-4-architecture-logiciel/02-event-bus/)

> Patterns de communication découplée entre composants.

- [Channels](./partie-4-architecture-logiciel/02-event-bus/01-channels.md) — Communication via messages
- [Pub/sub](./partie-4-architecture-logiciel/02-event-bus/02-pub-sub.md) — Abonnements multiples
- [Messages](./partie-4-architecture-logiciel/02-event-bus/03-messages.md) — Command, Event, Query
- [Flux de données](./partie-4-architecture-logiciel/02-event-bus/04-flux-donnees.md) — Unidirectionnel et prévisible

#### [Chapitre 16 : Gestion d'état robuste](./partie-4-architecture-logiciel/03-gestion-etat/)

> Maintenir un état applicatif cohérent et prévisible.

- [Store global](./partie-4-architecture-logiciel/03-gestion-etat/01-store-global.md) — Source unique de vérité
- [État local](./partie-4-architecture-logiciel/03-gestion-etat/02-etat-local.md) — UI state vs app state
- [Cache](./partie-4-architecture-logiciel/03-gestion-etat/03-cache.md) — Performance et invalidation
- [Undo stack](./partie-4-architecture-logiciel/03-gestion-etat/04-undo-stack.md) — Historique des changements

#### [Chapitre 17 : Stockage local souverain](./partie-4-architecture-logiciel/04-stockage-local/)

> Persister les données localement de façon fiable et sécurisée.

- [SQLite](./partie-4-architecture-logiciel/04-stockage-local/01-sqlite.md) — Base de données embarquée
- [Migrations](./partie-4-architecture-logiciel/04-stockage-local/02-migrations.md) — Évolution du schéma
- [Offline-first](./partie-4-architecture-logiciel/04-stockage-local/03-offline-first.md) — Sync queue, conflits
- [Chiffrement](./partie-4-architecture-logiciel/04-stockage-local/04-chiffrement.md) — AES-GCM, Argon2

#### [Chapitre 18 : Fichiers, import, export et PDF](./partie-4-architecture-logiciel/05-fichiers-import-export/)

> Générer des documents et échanger des données.

- [Rapports](./partie-4-architecture-logiciel/05-fichiers-import-export/01-rapports.md) — Génération PDF
- [Factures](./partie-4-architecture-logiciel/05-fichiers-import-export/02-factures.md) — Documents commerciaux
- [CSV et JSON](./partie-4-architecture-logiciel/05-fichiers-import-export/03-csv-json.md) — Import/export de données
- [Intégration système](./partie-4-architecture-logiciel/05-fichiers-import-export/04-integration-systeme.md) — Dialogues fichiers, associations

#### [Chapitre 19 : Plugins et extensibilité](./partie-4-architecture-logiciel/06-plugins-extensibilite/)

> Concevoir une architecture extensible par des tiers.

- [Modules dynamiques](./partie-4-architecture-logiciel/06-plugins-extensibilite/01-modules-dynamiques.md) — Chargement à runtime
- [Feature flags](./partie-4-architecture-logiciel/06-plugins-extensibilite/02-feature-flags.md) — Activation conditionnelle
- [Architecture plugin](./partie-4-architecture-logiciel/06-plugins-extensibilite/03-architecture-plugin.md) — API stable, sandboxing

---

### [Partie V — Blocs Métiers Réels](./partie-5-blocs-metiers/)

> **Objectif** : Implémenter des fonctionnalités business concrètes et réutilisables.

Cette partie présente l'implémentation de fonctionnalités business. Chaque chapitre est un module réutilisable pour vos applications.

#### [Chapitre 20 : Construire un mini-ERP Rust](./partie-5-blocs-metiers/01-mini-erp/)

> Système complet de gestion commerciale : clients, factures, TVA, exports.

- [Clients](./partie-5-blocs-metiers/01-mini-erp/01-clients.md) — CRUD, validation, règles métier
- [Factures](./partie-5-blocs-metiers/01-mini-erp/02-factures.md) — Lignes, numérotation auto
- [TVA](./partie-5-blocs-metiers/01-mini-erp/03-tva.md) — Calcul, taux, conformité
- [PDF](./partie-5-blocs-metiers/01-mini-erp/04-pdf.md) — Génération professionnelle
- [Exports](./partie-5-blocs-metiers/01-mini-erp/05-exports.md) — CSV, Excel, comptabilité

#### [Chapitre 21 : Moteur de recherche local](./partie-5-blocs-metiers/02-moteur-recherche/)

> Recherche full-text performante avec Tantivy.

- [Tantivy](./partie-5-blocs-metiers/02-moteur-recherche/01-tantivy.md) — Moteur de recherche Rust
- [Indexation](./partie-5-blocs-metiers/02-moteur-recherche/02-indexation.md) — Création d'index
- [Full-text search](./partie-5-blocs-metiers/02-moteur-recherche/03-full-text-search.md) — Requêtes avancées
- [Performance](./partie-5-blocs-metiers/02-moteur-recherche/04-performance.md) — Optimisation, cache

#### [Chapitre 22 : Dashboard et visualisation](./partie-5-blocs-metiers/03-dashboard-visualisation/)

> Tableaux de bord avec KPIs et graphiques en temps réel.

- [KPI cards](./partie-5-blocs-metiers/03-dashboard-visualisation/01-kpi-cards.md) — Métriques clés
- [Charts](./partie-5-blocs-metiers/03-dashboard-visualisation/02-charts.md) — Graphiques avec egui_plot
- [Temps réel](./partie-5-blocs-metiers/03-dashboard-visualisation/03-temps-reel.md) — Mise à jour live
- [UX data](./partie-5-blocs-metiers/03-dashboard-visualisation/04-ux-data.md) — Présentation des données

#### [Chapitre 23 : IA locale dans une app Rust](./partie-5-blocs-metiers/04-ia-locale/)

> Intégrer l'intelligence artificielle sans cloud avec Candle.

- [Embeddings](./partie-5-blocs-metiers/04-ia-locale/01-embeddings.md) — Vecteurs de texte
- [Recherche sémantique](./partie-5-blocs-metiers/04-ia-locale/02-recherche-semantique.md) — Similarité cosinus
- [Inférence CPU](./partie-5-blocs-metiers/04-ia-locale/03-inference-cpu.md) — Modèles quantifiés
- [Souveraineté IA](./partie-5-blocs-metiers/04-ia-locale/04-souverainete-ia.md) — Données privées

#### [Chapitre 24 : Cas d'étude complet](./partie-5-blocs-metiers/05-cas-etude-complet/)

> Application NoteVault : de zéro à finie, code réel inclus.

- [De zéro à finie](./partie-5-blocs-metiers/05-cas-etude-complet/01-zero-a-finie.md) — Progression complète
- [Code réel](./partie-5-blocs-metiers/05-cas-etude-complet/02-code-reel.md) — Implémentation détaillée
- [Design system](./partie-5-blocs-metiers/05-cas-etude-complet/03-design-system.md) — Thème appliqué
- [Packaging](./partie-5-blocs-metiers/05-cas-etude-complet/04-packaging.md) — Distribution finale

---

### [Partie VI — Finition et Qualité Produit](./partie-6-finition-qualite/)

> **Objectif** : Polir l'application pour une qualité professionnelle.

Cette partie couvre les aspects qui transforment une application fonctionnelle en produit professionnel : performance perçue, accessibilité, packaging et tests.

#### [Chapitre 25 : Performance perçue](./partie-6-finition-qualite/01-performance-percue/)

> Techniques pour améliorer la perception de performance par l'utilisateur.

- [Skeleton](./partie-6-finition-qualite/01-performance-percue/01-skeleton.md) — Placeholders animés
- [Lazy loading](./partie-6-finition-qualite/01-performance-percue/02-lazy-loading.md) — Chargement différé
- [Démarrage instantané](./partie-6-finition-qualite/01-performance-percue/03-demarrage-instantane.md) — First paint rapide
- [Caches](./partie-6-finition-qualite/01-performance-percue/04-caches.md) — LRU, invalidation

#### [Chapitre 26 : Accessibilité et ergonomie](./partie-6-finition-qualite/02-accessibilite-ergonomie/)

> Créer une application utilisable par tous.

- [Raccourcis clavier](./partie-6-finition-qualite/02-accessibilite-ergonomie/01-raccourcis-clavier.md) — Système complet
- [Navigation clavier](./partie-6-finition-qualite/02-accessibilite-ergonomie/02-navigation-souris.md) — Focus, tab order
- [Contrastes](./partie-6-finition-qualite/02-accessibilite-ergonomie/03-contrastes.md) — WCAG AA/AAA
- [Tailles et lisibilité](./partie-6-finition-qualite/02-accessibilite-ergonomie/04-tailles-lisibilite.md) — Zones cliquables

#### [Chapitre 27 : Packaging multiplateforme](./partie-6-finition-qualite/03-packaging-multiplateforme/)

> Distribuer sur Windows, macOS et Linux.

- [Windows](./partie-6-finition-qualite/03-packaging-multiplateforme/01-windows.md) — MSI, cargo-wix, signing
- [macOS](./partie-6-finition-qualite/03-packaging-multiplateforme/02-macos.md) — DMG, notarization, universal
- [Linux](./partie-6-finition-qualite/03-packaging-multiplateforme/03-linux.md) — AppImage, deb, Flatpak
- [Installers](./partie-6-finition-qualite/03-packaging-multiplateforme/04-installers.md) — UX d'installation
- [Mises à jour](./partie-6-finition-qualite/03-packaging-multiplateforme/05-mises-a-jour.md) — Auto-update

#### [Chapitre 28 : Tests UI et assurance qualité](./partie-6-finition-qualite/04-tests-ui-qualite/)

> Stratégies de test pour une qualité constante.

- [Tests unitaires](./partie-6-finition-qualite/04-tests-ui-qualite/01-tests-unitaires.md) — Logique métier, mocks
- [Snapshots UI](./partie-6-finition-qualite/04-tests-ui-qualite/02-snapshots-ui.md) — Régression visuelle
- [Tests manuels](./partie-6-finition-qualite/04-tests-ui-qualite/03-tests-manuels.md) — Checklists, scénarios

---

### [Partie VII — Production et Business](./partie-7-production-business/)

> **Objectif** : Transformer le code en produit viable et rentable.

Cette dernière partie couvre les aspects business et de mise en production d'une application desktop professionnelle.

#### [Chapitre 29 : Licences et modèles économiques](./partie-7-production-business/01-licences-modeles-economiques/)

> Choisir et implémenter un modèle de monétisation.

- [Types de licences](./partie-7-production-business/01-licences-modeles-economiques/01-types-licences.md) — Perpétuelle, abo, freemium
- [Génération et validation](./partie-7-production-business/01-licences-modeles-economiques/02-generation-validation.md) — Clés signées, machine ID
- [Activation offline](./partie-7-production-business/01-licences-modeles-economiques/03-activation-offline.md) — Sans serveur
- [Modèle freemium](./partie-7-production-business/01-licences-modeles-economiques/04-modele-freemium.md) — Limites, upgrade

#### [Chapitre 30 : Distribution et marketing](./partie-7-production-business/02-distribution-marketing/)

> Faire connaître et distribuer votre application.

- [Site web produit](./partie-7-production-business/02-distribution-marketing/01-site-web.md) — Landing page efficace
- [SEO](./partie-7-production-business/02-distribution-marketing/02-seo.md) — Visibilité Google
- [Page de téléchargement](./partie-7-production-business/02-distribution-marketing/03-page-telechargement.md) — Détection OS, checksums
- [Canaux](./partie-7-production-business/02-distribution-marketing/04-canaux.md) — Stores, package managers

#### [Chapitre 31 : Support et maintenance](./partie-7-production-business/03-support-maintenance/)

> Accompagner les utilisateurs et maintenir le produit.

- [Documentation](./partie-7-production-business/03-support-maintenance/01-documentation.md) — In-app, contextuelle
- [Feedback in-app](./partie-7-production-business/03-support-maintenance/02-feedback.md) — Widget, logs
- [Télémétrie](./partie-7-production-business/03-support-maintenance/03-telemetrie.md) — Opt-in, anonymisée
- [Gestion des bugs](./partie-7-production-business/03-support-maintenance/04-gestion-bugs.md) — Crash reports, triage

#### [Chapitre 32 : Monétisation avancée](./partie-7-production-business/04-monetisation-avancee/)

> Stratégies de revenus complémentaires.

- [Modules complémentaires](./partie-7-production-business/04-monetisation-avancee/01-modules-complementaires.md) — Add-ons payants
- [Personnalisation](./partie-7-production-business/04-monetisation-avancee/02-personnalisation.md) — White-label
- [Services associés](./partie-7-production-business/04-monetisation-avancee/03-services.md) — Support premium, formation
- [Affiliation](./partie-7-production-business/04-monetisation-avancee/04-affiliation.md) — Programme partenaires

#### [Chapitre 33 : Vision long terme](./partie-7-production-business/05-vision-long-terme/)

> Faire évoluer le produit sur le long terme.

- [Roadmap](./partie-7-production-business/05-vision-long-terme/01-roadmap.md) — Planification, priorisation
- [Communauté](./partie-7-production-business/05-vision-long-terme/02-communaute.md) — Feature voting, Discord
- [Open source](./partie-7-production-business/05-vision-long-terme/03-open-source.md) — Stratégie hybride
- [Évolution technique](./partie-7-production-business/05-vision-long-terme/04-evolution-technique.md) — Migrations, compatibilité
- [Métriques](./partie-7-production-business/05-vision-long-terme/05-metriques.md) — KPIs, NPS, churn

---

### [Annexes](./annexes/)

> Ressources de référence rapide.

| Annexe | Description |
|--------|-------------|
| [A. Cargo.toml complet](./annexes/A-cargo-toml-complet.md) | Configuration avec toutes les dépendances commentées |
| [B. Cheatsheet egui](./annexes/B-cheatsheet-egui.md) | Référence rapide des widgets et layouts |
| [C. Patterns Rust](./annexes/C-patterns-rust.md) | Résumé des patterns utilisés |
| [D. Checklist de lancement](./annexes/D-checklist-lancement.md) | Todo-list avant release |
| [E. Ressources et liens](./annexes/E-ressources-liens.md) | Documentation, tutoriels, communautés |
| [F. Glossaire](./annexes/F-glossaire.md) | Définitions des termes techniques |

---

## Pour commencer

### Prérequis

- Connaissance basique de Rust (variables, fonctions, structs)
- Familiarité avec la ligne de commande
- Envie de créer des logiciels de qualité

### Comment utiliser ce livre

| Mode | Description |
|------|-------------|
| **Lecture linéaire** | Suivez les parties dans l'ordre pour une progression cohérente |
| **Référence** | Consultez les chapitres spécifiques selon vos besoins |
| **Pratique** | Chaque chapitre contient des exemples de code exécutables |
| **Projet fil rouge** | Construisez progressivement une application complète |

### Code source

Tous les exemples sont testés et prêts à l'emploi :

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

## Licence

Ce livre est publié sous licence **MIT**. Vous êtes libre de l'utiliser, le modifier et le distribuer.

---

<p align="center">
  <strong>Commençons le voyage vers la maîtrise du développement desktop en Rust.</strong>
</p>
