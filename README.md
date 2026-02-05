# Rust Desktop Mastery

<p align="center">
  <img src="https://www.rust-lang.org/logos/rust-logo-512x512.png" alt="Rust Logo" width="120"/>
</p>

<p align="center">
  <strong>Architecture, UI, Design System et Mise en Production d'Applications Rust Réelles</strong>
</p>

<p align="center">
  <em>Le guide complet pour créer des applications desktop professionnelles en Rust</em>
</p>

<p align="center">
  <a href="#partie-i--le-mindset-rust-desktop">Mindset</a> •
  <a href="#partie-ii--fondations-rust-solides">Fondations</a> •
  <a href="#partie-iii--interfaces-modernes-en-rust">UI</a> •
  <a href="#partie-iv--architecture-dun-vrai-logiciel">Architecture</a> •
  <a href="#partie-v--blocs-métiers-réels">Métiers</a> •
  <a href="#partie-vi--finition-et-qualité-produit">Qualité</a> •
  <a href="#partie-vii--production-et-business">Business</a>
</p>

---

## Vue d'ensemble

Ce livre est un **guide complet** pour créer des applications desktop professionnelles en Rust. Il couvre l'ensemble du cycle de développement : de la conception architecturale au déploiement, en passant par la création d'interfaces modernes et de systèmes de design réutilisables.

### Ce que vous allez apprendre

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         RUST DESKTOP MASTERY                                │
├─────────────────────────────────────────────────────────────────────────────┤
│  📚 7 Parties  │  📖 33 Chapitres  │  📝 100+ Sous-sections  │  📎 6 Annexes │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  PARTIE I      Mindset & Philosophie                    [Chapitres 1-3]    │
│  PARTIE II     Fondations Rust                          [Chapitres 4-7]    │
│  PARTIE III    Interfaces Utilisateur                   [Chapitres 8-13]   │
│  PARTIE IV     Architecture Logicielle                  [Chapitres 14-19]  │
│  PARTIE V      Fonctionnalités Métiers                  [Chapitres 20-24]  │
│  PARTIE VI     Finition & Qualité                       [Chapitres 25-28]  │
│  PARTIE VII    Production & Business                    [Chapitres 29-33]  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Technologies couvertes

| Catégorie | Technologies |
|-----------|--------------|
| **Langage** | Rust 2021 Edition |
| **UI Framework** | egui, eframe, Tauri, iced, wgpu |
| **Base de données** | SQLite (rusqlite), migrations |
| **Async/Concurrence** | Tokio, crossbeam, Arc, ArcSwap |
| **Recherche** | Tantivy (full-text search) |
| **IA locale** | Candle, embeddings, inférence CPU |
| **PDF** | printpdf, génération de documents |
| **Packaging** | cargo-wix (Windows), DMG (macOS), AppImage (Linux) |
| **Tests** | cargo test, criterion (benchmarks), snapshots UI |

---

## Pourquoi ce livre ?

### Le retour du desktop

Le développement desktop connaît une **renaissance**. Après des années de domination du web et des applications SaaS, les développeurs et les entreprises redécouvrent les avantages des applications natives.

| Problème Web/SaaS | Solution Desktop Rust |
|-------------------|----------------------|
| ⏱️ Latence réseau (100-500ms) | ⚡ Réponse instantanée (<10ms) |
| 💸 Abonnement mensuel obligatoire | 💰 Achat unique possible |
| ☁️ Données sur serveur tiers | 🏠 Données 100% locales |
| 🌐 Dépendance internet | 📴 Fonctionne offline |
| 🏗️ Infrastructure complexe | 📦 Un seul fichier binaire |
| 🔓 Risques de sécurité cloud | 🔐 Chiffrement local |
| 📈 Coûts qui augmentent avec l'usage | 📉 Coût fixe, scalabilité gratuite |

### Pourquoi Rust ?

**Rust** est le langage idéal pour cette nouvelle ère :

```
┌─────────────────────────────────────────────────────────────────┐
│                    AVANTAGES DE RUST                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  🚀 PERFORMANCE        Comparable au C/C++, zero-cost          │
│                        abstractions, pas de GC                  │
│                                                                 │
│  🛡️ SÉCURITÉ MÉMOIRE   Ownership, borrowing, pas de null       │
│                        pointer, pas de data races               │
│                                                                 │
│  📦 BINAIRE UNIQUE     Compilation statique, pas de DLL,       │
│                        déploiement simplifié                    │
│                                                                 │
│  🔧 TOOLING MODERNE    Cargo, rustfmt, clippy, rust-analyzer   │
│                                                                 │
│  🌍 CROSS-PLATFORM     Windows, macOS, Linux depuis le même    │
│                        code source                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### À qui s'adresse ce livre ?

| Profil | Ce que vous apprendrez |
|--------|------------------------|
| **Développeur Web fatigué** | Créer des apps performantes sans Electron, npm, ou infrastructure |
| **Développeur Rust débutant** | Appliquer Rust à des projets concrets avec UI |
| **Développeur desktop expérimenté** | Moderniser votre stack avec Rust et egui |
| **Entrepreneur/Indie hacker** | Construire et monétiser un logiciel desktop |
| **Architecte logiciel** | Patterns et architecture pour apps maintenables |

---

## Structure du livre

### [Partie I — Le Mindset Rust Desktop](./partie-1-mindset-rust-desktop/)

> **Objectif** : Comprendre pourquoi Rust est le choix idéal pour le desktop et adopter la bonne philosophie produit.

Cette partie pose les **fondations philosophiques et stratégiques**. Avant d'écrire une seule ligne de code, il est crucial de comprendre *pourquoi* nous faisons ces choix et *comment* penser notre travail.

**Vous apprendrez à :**
- Argumenter le choix de Rust pour un projet desktop
- Identifier les cas d'usage où le desktop surpasse le web
- Adopter une mentalité produit orientée utilisateur
- Évaluer les compromis entre performance, UX et time-to-market

#### [Chapitre 1 : Pourquoi Rust pour le logiciel moderne](./partie-1-mindset-rust-desktop/01-pourquoi-rust/)

> Explore les raisons fondamentales qui font de Rust le choix idéal : fatigue de l'écosystème JS/Electron, performance native, sécurité mémoire, et souveraineté des données.

| Section | Description | Concepts clés |
|---------|-------------|---------------|
| [La fatigue JS / Electron / SaaS](./partie-1-mindset-rust-desktop/01-pourquoi-rust/01-fatigue-js-electron.md) | Problèmes de l'écosystème actuel | Mémoire Electron, node_modules, instabilité npm |
| [Performance native](./partie-1-mindset-rust-desktop/01-pourquoi-rust/02-performance-native.md) | Pourquoi Rust est rapide | Zero-cost abstractions, comparaison JS/JVM |
| [Sécurité mémoire](./partie-1-mindset-rust-desktop/01-pourquoi-rust/03-securite-memoire.md) | Bugs évités à la compilation | Ownership, borrowing, Option vs null |
| [Binaire unique](./partie-1-mindset-rust-desktop/01-pourquoi-rust/04-binaire-unique.md) | Déploiement simplifié | Compilation statique, pas de DLL hell |
| [Offline-first](./partie-1-mindset-rust-desktop/01-pourquoi-rust/05-offline-first.md) | Architecture locale | Données locales, sync optionnelle |
| [Souveraineté des données](./partie-1-mindset-rust-desktop/01-pourquoi-rust/06-souverainete-donnees.md) | Contrôle utilisateur | Chiffrement local, RGPD, vie privée |

#### [Chapitre 2 : Desktop is back](./partie-1-mindset-rust-desktop/02-desktop-is-back/)

> Le retour en force des applications locales : pourquoi et quand le desktop surpasse le web.

| Section | Description | Concepts clés |
|---------|-------------|---------------|
| [Le retour des apps locales](./partie-1-mindset-rust-desktop/02-desktop-is-back/01-retour-apps-locales.md) | Tendances du marché | Obsidian, Notion local, Linear |
| [UX supérieure au navigateur](./partie-1-mindset-rust-desktop/02-desktop-is-back/02-ux-superieure.md) | Avantages UX natifs | Raccourcis globaux, intégration OS |
| [IA locale, SQLite, fichiers](./partie-1-mindset-rust-desktop/02-desktop-is-back/03-ia-locale-sqlite.md) | Technologies clés | LLM locaux, SQLite embarqué |
| [Cas concrets](./partie-1-mindset-rust-desktop/02-desktop-is-back/04-cas-concrets.md) | Exemples d'applications | ERP, prise de notes, automation |
| [Quand ne pas utiliser le web](./partie-1-mindset-rust-desktop/02-desktop-is-back/05-quand-pas-web.md) | Critères de décision | Matrice de choix web vs desktop |

#### [Chapitre 3 : Penser produit, pas repo GitHub](./partie-1-mindset-rust-desktop/03-penser-produit/)

> Adopter une mentalité produit orientée utilisateur plutôt qu'une approche purement technique.

| Section | Description | Concepts clés |
|---------|-------------|---------------|
| [Ship plutôt que démo](./partie-1-mindset-rust-desktop/03-penser-produit/01-ship-plutot-que-demo.md) | Livrer de la valeur | MVP, itération, feedback loop |
| [Logiciel utilisable 8h/jour](./partie-1-mindset-rust-desktop/03-penser-produit/02-logiciel-8h-jour.md) | Ergonomie professionnelle | Fatigue visuelle, raccourcis, workflow |
| [Dette UX vs dette technique](./partie-1-mindset-rust-desktop/03-penser-produit/03-dette-ux-vs-technique.md) | Prioriser l'expérience | Impact utilisateur, coût de la friction |
| [Le design comme avantage](./partie-1-mindset-rust-desktop/03-penser-produit/04-design-avantage-concurrentiel.md) | Différenciation | Design system, cohérence, polish |

---

### [Partie II — Fondations Rust Solides](./partie-2-fondations-rust/)

> **Objectif** : Maîtriser les concepts Rust essentiels pour construire des applications robustes.

Cette partie couvre les **concepts Rust nécessaires au développement desktop**. L'objectif n'est pas d'enseigner Rust depuis zéro, mais de maîtriser les patterns spécifiques aux applications réelles.

**Vous apprendrez à :**
- Écrire du code Rust idiomatique et maintenable
- Structurer un projet multi-crates professionnel
- Gérer la concurrence sans bloquer l'interface
- Diagnostiquer et optimiser les performances

#### [Chapitre 4 : Rust pour développeurs pragmatiques](./partie-2-fondations-rust/01-rust-pragmatique/)

> Les concepts Rust essentiels expliqués de manière pratique, avec focus sur le développement desktop.

| Section | Description | Concepts clés |
|---------|-------------|---------------|
| [Ownership expliqué simplement](./partie-2-fondations-rust/01-rust-pragmatique/01-ownership-simplement.md) | Les 3 règles fondamentales | Move, Copy, Drop, ownership transfer |
| [Borrow checker sans douleur](./partie-2-fondations-rust/01-rust-pragmatique/02-borrow-checker.md) | Références maîtrisées | &T, &mut T, lifetimes basiques |
| [Erreurs idiomatiques](./partie-2-fondations-rust/01-rust-pragmatique/03-erreurs-idiomatiques.md) | Gestion d'erreurs Rust | Result, Option, ?, thiserror, anyhow |
| [Patterns utiles](./partie-2-fondations-rust/01-rust-pragmatique/04-patterns-utiles.md) | Patterns courants | Builder, Newtype, State Machine, From/Into |

#### [Chapitre 5 : Organiser un vrai projet](./partie-2-fondations-rust/02-organiser-projet/)

> Structurer une application Rust professionnelle, maintenable et évolutive.

| Section | Description | Concepts clés |
|---------|-------------|---------------|
| [Cargo Workspace](./partie-2-fondations-rust/02-organiser-projet/01-cargo-workspace.md) | Multi-crates | Workspace, dépendances partagées |
| [Crates Modulaires](./partie-2-fondations-rust/02-organiser-projet/02-crates-modulaires.md) | Séparation des responsabilités | core, app, ui, infrastructure |
| [Architecture de Dossiers](./partie-2-fondations-rust/02-organiser-projet/03-architecture-dossiers.md) | Organisation du code | Conventions, modules, visibilité |
| [Tests et Benchmarks](./partie-2-fondations-rust/02-organiser-projet/04-tests-benchmarks.md) | Qualité du code | #[test], integration tests, Criterion |

#### [Chapitre 6 : Async et concurrence desktop](./partie-2-fondations-rust/03-async-concurrence/)

> Gérer la concurrence sans bloquer l'interface utilisateur.

| Section | Description | Concepts clés |
|---------|-------------|---------------|
| [Tokio](./partie-2-fondations-rust/03-async-concurrence/01-tokio.md) | Runtime async | spawn, select!, multi-thread runtime |
| [Channels (crossbeam)](./partie-2-fondations-rust/03-async-concurrence/02-channels-crossbeam.md) | Communication inter-threads | mpsc, bounded/unbounded, select |
| [Arc et ArcSwap](./partie-2-fondations-rust/03-async-concurrence/03-arc-arcswap.md) | Partage thread-safe | Arc<Mutex<T>>, ArcSwap, RCU pattern |
| [Thread UI vs workers](./partie-2-fondations-rust/03-async-concurrence/04-thread-ui-workers.md) | Séparation UI/calcul | Main thread, background workers |
| [Éviter les freezes](./partie-2-fondations-rust/03-async-concurrence/05-eviter-freezes.md) | UI toujours réactive | Chunked processing, cancellation |

#### [Chapitre 7 : Logs, debug et profiling](./partie-2-fondations-rust/04-logs-debug-profiling/)

> Diagnostiquer et optimiser les performances de manière rigoureuse.

| Section | Description | Concepts clés |
|---------|-------------|---------------|
| [Tracing](./partie-2-fondations-rust/04-logs-debug-profiling/01-tracing.md) | Logs structurés | spans, levels, subscribers, #[instrument] |
| [Logs visuels](./partie-2-fondations-rust/04-logs-debug-profiling/02-logs-visuels.md) | Panel de logs intégré | egui log viewer, filtrage |
| [Flamegraphs](./partie-2-fondations-rust/04-logs-debug-profiling/03-flamegraphs.md) | Profiling visuel | cargo-flamegraph, tracing-chrome |
| [Optimiser sans se mentir](./partie-2-fondations-rust/04-logs-debug-profiling/04-optimiser-sans-mentir.md) | Mesures rigoureuses | Benchmarks, profiling avant optimisation |

---

### [Partie III — Interfaces Modernes en Rust](./partie-3-interfaces-modernes/)

> **Objectif** : Construire des interfaces utilisateur professionnelles et réactives.

Cette partie est consacrée à la **création d'UI en Rust**. Nous couvrons les frameworks disponibles, les patterns de design system, et les techniques pour construire des interfaces modernes.

**Vous apprendrez à :**
- Choisir le framework UI adapté à votre projet
- Construire des interfaces réactives avec egui
- Créer et maintenir un design system cohérent
- Implémenter des composants réutilisables de qualité professionnelle

#### [Chapitre 8 : Panorama des frameworks UI](./partie-3-interfaces-modernes/01-panorama-frameworks/)

> Comparaison détaillée des frameworks UI Rust pour choisir le bon outil.

| Section | Description | Concepts clés |
|---------|-------------|---------------|
| [egui](./partie-3-interfaces-modernes/01-panorama-frameworks/01-egui.md) | Immediate mode GUI | Simple, rapide, prototypage |
| [Tauri](./partie-3-interfaces-modernes/01-panorama-frameworks/02-tauri.md) | Web frontend + Rust | HTML/CSS/JS, WebView, IPC |
| [wgpu](./partie-3-interfaces-modernes/01-panorama-frameworks/03-wgpu.md) | GPU rendering | Bas niveau, performance max |
| [iced](./partie-3-interfaces-modernes/01-panorama-frameworks/04-iced.md) | Declaratif Elm-style | État immutable, messages |
| [Quand choisir quoi](./partie-3-interfaces-modernes/01-panorama-frameworks/05-quand-choisir-quoi.md) | Matrice de décision | Critères, recommandations |

#### [Chapitre 9 : Construire une UI avec egui](./partie-3-interfaces-modernes/02-construire-ui-egui/)

> Guide pratique complet pour maîtriser egui, le framework choisi pour ce livre.

| Section | Description | Concepts clés |
|---------|-------------|---------------|
| [Immediate mode](./partie-3-interfaces-modernes/02-construire-ui-egui/01-immediate-mode.md) | Mental model | Différence avec retained mode |
| [Layouts](./partie-3-interfaces-modernes/02-construire-ui-egui/02-layouts.md) | Organisation visuelle | Vertical, horizontal, grid, panels |
| [Widgets](./partie-3-interfaces-modernes/02-construire-ui-egui/03-widgets.md) | Éléments d'interface | Labels, buttons, inputs, sliders |
| [Gestion de l'état](./partie-3-interfaces-modernes/02-construire-ui-egui/04-gestion-etat.md) | State management | Local vs global, persistence |

#### [Chapitre 10 : Créer son Design System](./partie-3-interfaces-modernes/03-design-system/)

> Construire un système de design cohérent et maintenable.

| Section | Description | Concepts clés |
|---------|-------------|---------------|
| [Tokens](./partie-3-interfaces-modernes/03-design-system/01-tokens.md) | Variables centralisées | Constantes de design, JSON tokens |
| [Couleurs](./partie-3-interfaces-modernes/03-design-system/02-couleurs.md) | Palette cohérente | Primary, secondary, semantic colors |
| [Spacing](./partie-3-interfaces-modernes/03-design-system/03-spacing.md) | Grille et espacement | 4px/8px grid, margins, paddings |
| [Typographie](./partie-3-interfaces-modernes/03-design-system/04-typographie.md) | Hiérarchie textuelle | Font sizes, weights, line heights |
| [Thèmes](./partie-3-interfaces-modernes/03-design-system/05-themes.md) | Dark/light mode | Theme switching, persistence |
| [DPI scaling](./partie-3-interfaces-modernes/03-design-system/06-dpi-scaling.md) | Multi-résolutions | HiDPI, scaling factors |

#### [Chapitre 11 : Composants réutilisables](./partie-3-interfaces-modernes/04-composants-reutilisables/)

> Bibliothèque de composants UI professionnels prêts à l'emploi.

| Section | Description | Concepts clés |
|---------|-------------|---------------|
| [Button](./partie-3-interfaces-modernes/04-composants-reutilisables/01-button.md) | Boutons | Primary, secondary, danger, disabled |
| [Input](./partie-3-interfaces-modernes/04-composants-reutilisables/02-input.md) | Champs de saisie | Text, password, validation |
| [Table](./partie-3-interfaces-modernes/04-composants-reutilisables/03-table.md) | Tableaux de données | Tri, pagination, sélection |
| [Sidebar](./partie-3-interfaces-modernes/04-composants-reutilisables/04-sidebar.md) | Navigation latérale | Collapse, nested items |
| [Modals](./partie-3-interfaces-modernes/04-composants-reutilisables/05-modals.md) | Dialogues | Confirmation, formulaires |
| [Toasts](./partie-3-interfaces-modernes/04-composants-reutilisables/06-toasts.md) | Notifications | Success, error, warning |
| [Inspector panels](./partie-3-interfaces-modernes/04-composants-reutilisables/07-inspector-panels.md) | Panneaux de propriétés | Détails, édition |
| [Command palette](./partie-3-interfaces-modernes/04-composants-reutilisables/08-command-palette.md) | Recherche d'actions | Ctrl+K, fuzzy search |

#### [Chapitre 12 : Patterns UX professionnels](./partie-3-interfaces-modernes/05-patterns-ux/)

> Les patterns UX qui font la différence entre une app amateur et professionnelle.

| Section | Description | Concepts clés |
|---------|-------------|---------------|
| [Loading](./partie-3-interfaces-modernes/05-patterns-ux/01-loading.md) | États de chargement | Spinners, progress bars |
| [Empty states](./partie-3-interfaces-modernes/05-patterns-ux/02-empty-states.md) | Écrans vides | Illustrations, call-to-action |
| [Erreurs](./partie-3-interfaces-modernes/05-patterns-ux/03-erreurs.md) | Messages d'erreur | Clairs, actionnables |
| [Undo et redo](./partie-3-interfaces-modernes/05-patterns-ux/04-undo-redo.md) | Historique d'actions | Command pattern, stack |
| [Autosave](./partie-3-interfaces-modernes/05-patterns-ux/05-autosave.md) | Sauvegarde automatique | Debouncing, indicateur |
| [Feedback instantané](./partie-3-interfaces-modernes/05-patterns-ux/06-feedback-instantane.md) | Réponse < 100ms | Optimistic updates |

#### [Chapitre 13 : Du wireframe SVG au code Rust](./partie-3-interfaces-modernes/06-wireframe-vers-code/)

> Workflow efficace pour transformer des maquettes en code.

| Section | Description | Concepts clés |
|---------|-------------|---------------|
| [Maquettes SVG](./partie-3-interfaces-modernes/06-wireframe-vers-code/01-maquettes-svg.md) | Création de wireframes | Figma, SVG export |
| [Mapping IDs](./partie-3-interfaces-modernes/06-wireframe-vers-code/02-mapping-ids.md) | Lier design et code | Nommage, convention |
| [Design vers egui](./partie-3-interfaces-modernes/06-wireframe-vers-code/03-design-vers-egui.md) | Traduction pratique | Extraction, composants |
| [Workflow rapide](./partie-3-interfaces-modernes/06-wireframe-vers-code/04-workflow.md) | Itérations design/code | Hot reload, preview |

---

### [Partie IV — Architecture d'un Vrai Logiciel](./partie-4-architecture-logiciel/)

> **Objectif** : Structurer une application maintenable, testable et évolutive.

Cette partie couvre l'**architecture logicielle**. Nous appliquons les principes de clean architecture adaptés au contexte desktop.

**Vous apprendrez à :**
- Structurer une application avec une clean architecture
- Implémenter une communication inter-composants efficace
- Gérer l'état de manière prévisible
- Persister les données localement de façon fiable

#### [Chapitre 14 : Clean Architecture pour apps desktop](./partie-4-architecture-logiciel/01-clean-architecture/)

> Organiser le code en couches indépendantes et testables.

| Section | Description | Concepts clés |
|---------|-------------|---------------|
| [Couches](./partie-4-architecture-logiciel/01-clean-architecture/01-couches.md) | Organisation en couches | Core, App, UI, Infrastructure |
| [Séparation logique](./partie-4-architecture-logiciel/01-clean-architecture/02-separation-logique.md) | Responsabilités claires | Single responsibility |
| [Découplage](./partie-4-architecture-logiciel/01-clean-architecture/03-decouplage.md) | Interfaces et abstractions | Dependency injection, traits |
| [Dépendances](./partie-4-architecture-logiciel/01-clean-architecture/04-dependances.md) | Direction des dépendances | Vers l'intérieur uniquement |

#### [Chapitre 15 : Event Bus et communication interne](./partie-4-architecture-logiciel/02-event-bus/)

> Patterns de communication découplée entre composants.

| Section | Description | Concepts clés |
|---------|-------------|---------------|
| [Channels](./partie-4-architecture-logiciel/02-event-bus/01-channels.md) | Communication via messages | mpsc, broadcast |
| [Pub/sub](./partie-4-architecture-logiciel/02-event-bus/02-pub-sub.md) | Abonnements multiples | Subscribe, publish, filter |
| [Messages](./partie-4-architecture-logiciel/02-event-bus/03-messages.md) | Types de messages | Command, Event, Query |
| [Flux de données](./partie-4-architecture-logiciel/02-event-bus/04-flux-donnees.md) | Flux unidirectionnel | Single source of truth |

#### [Chapitre 16 : Gestion d'état robuste](./partie-4-architecture-logiciel/03-gestion-etat/)

> Maintenir un état applicatif cohérent et prévisible.

| Section | Description | Concepts clés |
|---------|-------------|---------------|
| [Store global](./partie-4-architecture-logiciel/03-gestion-etat/01-store-global.md) | Source unique de vérité | AppState, reducers |
| [État local](./partie-4-architecture-logiciel/03-gestion-etat/02-etat-local.md) | UI state vs app state | Séparation, synchronisation |
| [Cache](./partie-4-architecture-logiciel/03-gestion-etat/03-cache.md) | Performance | LRU, TTL, invalidation |
| [Undo stack](./partie-4-architecture-logiciel/03-gestion-etat/04-undo-stack.md) | Historique des changements | Command pattern |

#### [Chapitre 17 : Stockage local souverain](./partie-4-architecture-logiciel/04-stockage-local/)

> Persister les données localement de façon fiable et sécurisée.

| Section | Description | Concepts clés |
|---------|-------------|---------------|
| [SQLite](./partie-4-architecture-logiciel/04-stockage-local/01-sqlite.md) | Base de données embarquée | rusqlite, requêtes, transactions |
| [Migrations](./partie-4-architecture-logiciel/04-stockage-local/02-migrations.md) | Évolution du schéma | Versioning, rollback |
| [Offline-first](./partie-4-architecture-logiciel/04-stockage-local/03-offline-first.md) | Sync optionnelle | Queue de sync, conflits |
| [Chiffrement](./partie-4-architecture-logiciel/04-stockage-local/04-chiffrement.md) | Sécurité des données | AES-GCM, Argon2, key derivation |

#### [Chapitre 18 : Fichiers, import, export et PDF](./partie-4-architecture-logiciel/05-fichiers-import-export/)

> Générer des documents et échanger des données.

| Section | Description | Concepts clés |
|---------|-------------|---------------|
| [Rapports](./partie-4-architecture-logiciel/05-fichiers-import-export/01-rapports.md) | Génération PDF | printpdf, templates |
| [Factures](./partie-4-architecture-logiciel/05-fichiers-import-export/02-factures.md) | Documents commerciaux | Layout, logo, conformité |
| [CSV et JSON](./partie-4-architecture-logiciel/05-fichiers-import-export/03-csv-json.md) | Import/export | csv crate, serde_json |
| [Intégration système](./partie-4-architecture-logiciel/05-fichiers-import-export/04-integration-systeme.md) | Fichiers et OS | native-dialog, associations |

#### [Chapitre 19 : Plugins et extensibilité](./partie-4-architecture-logiciel/06-plugins-extensibilite/)

> Concevoir une architecture extensible par des tiers.

| Section | Description | Concepts clés |
|---------|-------------|---------------|
| [Modules dynamiques](./partie-4-architecture-logiciel/06-plugins-extensibilite/01-modules-dynamiques.md) | Chargement runtime | libloading, FFI |
| [Feature flags](./partie-4-architecture-logiciel/06-plugins-extensibilite/02-feature-flags.md) | Activation conditionnelle | Compile-time, runtime flags |
| [Architecture plugin](./partie-4-architecture-logiciel/06-plugins-extensibilite/03-architecture-plugin.md) | API stable | Trait objects, sandboxing |

---

### [Partie V — Blocs Métiers Réels](./partie-5-blocs-metiers/)

> **Objectif** : Implémenter des fonctionnalités business concrètes et réutilisables.

Cette partie présente l'**implémentation de fonctionnalités business**. Chaque chapitre est un module réutilisable pour vos applications.

**Vous apprendrez à :**
- Construire un système ERP complet
- Implémenter un moteur de recherche performant
- Créer des dashboards avec visualisations
- Intégrer l'IA locale dans vos applications

#### [Chapitre 20 : Construire un mini-ERP Rust](./partie-5-blocs-metiers/01-mini-erp/)

> Système complet de gestion commerciale : clients, factures, TVA, exports.

| Section | Description | Concepts clés |
|---------|-------------|---------------|
| [Clients](./partie-5-blocs-metiers/01-mini-erp/01-clients.md) | Gestion des clients | CRUD, validation, recherche |
| [Factures](./partie-5-blocs-metiers/01-mini-erp/02-factures.md) | Facturation | Lignes, calculs, numérotation |
| [TVA](./partie-5-blocs-metiers/01-mini-erp/03-tva.md) | Fiscalité | Taux, calcul, conformité |
| [PDF](./partie-5-blocs-metiers/01-mini-erp/04-pdf.md) | Documents | Génération, templates |
| [Exports](./partie-5-blocs-metiers/01-mini-erp/05-exports.md) | Données | CSV, Excel, comptabilité |

#### [Chapitre 21 : Moteur de recherche local](./partie-5-blocs-metiers/02-moteur-recherche/)

> Recherche full-text performante avec Tantivy.

| Section | Description | Concepts clés |
|---------|-------------|---------------|
| [Tantivy](./partie-5-blocs-metiers/02-moteur-recherche/01-tantivy.md) | Moteur de recherche | Architecture, installation |
| [Indexation](./partie-5-blocs-metiers/02-moteur-recherche/02-indexation.md) | Création d'index | Schema, documents, async |
| [Full-text search](./partie-5-blocs-metiers/02-moteur-recherche/03-full-text-search.md) | Requêtes | Query parser, highlighting |
| [Performance](./partie-5-blocs-metiers/02-moteur-recherche/04-performance.md) | Optimisation | Cache, incremental indexing |

#### [Chapitre 22 : Dashboard et visualisation](./partie-5-blocs-metiers/03-dashboard-visualisation/)

> Tableaux de bord avec KPIs et graphiques en temps réel.

| Section | Description | Concepts clés |
|---------|-------------|---------------|
| [KPI cards](./partie-5-blocs-metiers/03-dashboard-visualisation/01-kpi-cards.md) | Métriques clés | Cards, trends, comparaison |
| [Charts](./partie-5-blocs-metiers/03-dashboard-visualisation/02-charts.md) | Graphiques | egui_plot, line, bar, pie |
| [Temps réel](./partie-5-blocs-metiers/03-dashboard-visualisation/03-temps-reel.md) | Live updates | Polling, streaming |
| [UX data](./partie-5-blocs-metiers/03-dashboard-visualisation/04-ux-data.md) | Présentation | Lisibilité, interaction |

#### [Chapitre 23 : IA locale dans une app Rust](./partie-5-blocs-metiers/04-ia-locale/)

> Intégrer l'intelligence artificielle sans cloud avec Candle.

| Section | Description | Concepts clés |
|---------|-------------|---------------|
| [Embeddings](./partie-5-blocs-metiers/04-ia-locale/01-embeddings.md) | Vecteurs de texte | BERT, sentence transformers |
| [Recherche sémantique](./partie-5-blocs-metiers/04-ia-locale/02-recherche-semantique.md) | Similarité | Cosine similarity, HNSW |
| [Inférence CPU](./partie-5-blocs-metiers/04-ia-locale/03-inference-cpu.md) | Sans GPU | Quantization, batch processing |
| [Souveraineté IA](./partie-5-blocs-metiers/04-ia-locale/04-souverainete-ia.md) | Vie privée | Données locales, RGPD |

#### [Chapitre 24 : Cas d'étude complet](./partie-5-blocs-metiers/05-cas-etude-complet/)

> Application NoteVault : de zéro à finie, code réel inclus.

| Section | Description | Concepts clés |
|---------|-------------|---------------|
| [De zéro à finie](./partie-5-blocs-metiers/05-cas-etude-complet/01-zero-a-finie.md) | Progression complète | Planning, milestones |
| [Code réel](./partie-5-blocs-metiers/05-cas-etude-complet/02-code-reel.md) | Implémentation | Architecture, modules |
| [Design system](./partie-5-blocs-metiers/05-cas-etude-complet/03-design-system.md) | Thème appliqué | Tokens, composants |
| [Packaging](./partie-5-blocs-metiers/05-cas-etude-complet/04-packaging.md) | Distribution | Build, installer, release |

---

### [Partie VI — Finition et Qualité Produit](./partie-6-finition-qualite/)

> **Objectif** : Polir l'application pour une qualité professionnelle.

Cette partie couvre les aspects qui transforment une application fonctionnelle en **produit professionnel** : performance perçue, accessibilité, packaging et tests.

**Vous apprendrez à :**
- Optimiser la performance perçue
- Créer une application accessible
- Packager pour Windows, macOS et Linux
- Mettre en place une stratégie de tests efficace

#### [Chapitre 25 : Performance perçue](./partie-6-finition-qualite/01-performance-percue/)

> Techniques pour améliorer la perception de performance par l'utilisateur.

| Section | Description | Concepts clés |
|---------|-------------|---------------|
| [Skeleton](./partie-6-finition-qualite/01-performance-percue/01-skeleton.md) | Placeholders animés | Loading states |
| [Lazy loading](./partie-6-finition-qualite/01-performance-percue/02-lazy-loading.md) | Chargement différé | Pagination, virtual scroll |
| [Démarrage instantané](./partie-6-finition-qualite/01-performance-percue/03-demarrage-instantane.md) | First paint rapide | Splash screen, progressive |
| [Caches](./partie-6-finition-qualite/01-performance-percue/04-caches.md) | Mise en cache | LRU, TTL, invalidation |

#### [Chapitre 26 : Accessibilité et ergonomie](./partie-6-finition-qualite/02-accessibilite-ergonomie/)

> Créer une application utilisable par tous.

| Section | Description | Concepts clés |
|---------|-------------|---------------|
| [Raccourcis clavier](./partie-6-finition-qualite/02-accessibilite-ergonomie/01-raccourcis-clavier.md) | Système de raccourcis | Global, contextuel, personnalisable |
| [Navigation clavier](./partie-6-finition-qualite/02-accessibilite-ergonomie/02-navigation-souris.md) | Focus management | Tab order, focus visible |
| [Contrastes](./partie-6-finition-qualite/02-accessibilite-ergonomie/03-contrastes.md) | WCAG compliance | AA/AAA, ratios |
| [Tailles et lisibilité](./partie-6-finition-qualite/02-accessibilite-ergonomie/04-tailles-lisibilite.md) | Zones cliquables | 44px minimum, touch targets |

#### [Chapitre 27 : Packaging multiplateforme](./partie-6-finition-qualite/03-packaging-multiplateforme/)

> Distribuer sur Windows, macOS et Linux.

| Section | Description | Concepts clés |
|---------|-------------|---------------|
| [Windows](./partie-6-finition-qualite/03-packaging-multiplateforme/01-windows.md) | MSI installer | cargo-wix, code signing |
| [macOS](./partie-6-finition-qualite/03-packaging-multiplateforme/02-macos.md) | App bundle | DMG, notarization, universal binary |
| [Linux](./partie-6-finition-qualite/03-packaging-multiplateforme/03-linux.md) | Packages | AppImage, .deb, Flatpak |
| [Installers](./partie-6-finition-qualite/03-packaging-multiplateforme/04-installers.md) | UX d'installation | Welcome, license, folder |
| [Mises à jour](./partie-6-finition-qualite/03-packaging-multiplateforme/05-mises-a-jour.md) | Auto-update | self_update, GitHub releases |

#### [Chapitre 28 : Tests UI et assurance qualité](./partie-6-finition-qualite/04-tests-ui-qualite/)

> Stratégies de test pour une qualité constante.

| Section | Description | Concepts clés |
|---------|-------------|---------------|
| [Tests unitaires](./partie-6-finition-qualite/04-tests-ui-qualite/01-tests-unitaires.md) | Logique métier | Mocks, fixtures, coverage |
| [Snapshots UI](./partie-6-finition-qualite/04-tests-ui-qualite/02-snapshots-ui.md) | Régression visuelle | Screenshot comparison |
| [Tests manuels](./partie-6-finition-qualite/04-tests-ui-qualite/03-tests-manuels.md) | Checklists | Scénarios, exploratory testing |

---

### [Partie VII — Production et Business](./partie-7-production-business/)

> **Objectif** : Transformer le code en produit viable et rentable.

Cette dernière partie couvre les **aspects business** et de mise en production d'une application desktop professionnelle.

**Vous apprendrez à :**
- Choisir et implémenter un modèle de licence
- Créer une présence web efficace
- Gérer le support et la maintenance
- Développer des revenus complémentaires

#### [Chapitre 29 : Licences et modèles économiques](./partie-7-production-business/01-licences-modeles-economiques/)

> Choisir et implémenter un modèle de monétisation.

| Section | Description | Concepts clés |
|---------|-------------|---------------|
| [Types de licences](./partie-7-production-business/01-licences-modeles-economiques/01-types-licences.md) | Modèles | Perpétuelle, abonnement, freemium |
| [Génération et validation](./partie-7-production-business/01-licences-modeles-economiques/02-generation-validation.md) | Système de clés | Ed25519, machine ID, HMAC |
| [Activation offline](./partie-7-production-business/01-licences-modeles-economiques/03-activation-offline.md) | Sans serveur | Hardware binding, tokens |
| [Modèle freemium](./partie-7-production-business/01-licences-modeles-economiques/04-modele-freemium.md) | Conversion | Free tier limits, upgrade prompts |

#### [Chapitre 30 : Distribution et marketing](./partie-7-production-business/02-distribution-marketing/)

> Faire connaître et distribuer votre application.

| Section | Description | Concepts clés |
|---------|-------------|---------------|
| [Site web produit](./partie-7-production-business/02-distribution-marketing/01-site-web.md) | Landing page | Hero, features, pricing, CTA |
| [SEO](./partie-7-production-business/02-distribution-marketing/02-seo.md) | Visibilité | Keywords, blog, long tail |
| [Page de téléchargement](./partie-7-production-business/02-distribution-marketing/03-page-telechargement.md) | Conversion | OS detection, checksums |
| [Canaux](./partie-7-production-business/02-distribution-marketing/04-canaux.md) | Distribution | Stores, Homebrew, Chocolatey |

#### [Chapitre 31 : Support et maintenance](./partie-7-production-business/03-support-maintenance/)

> Accompagner les utilisateurs et maintenir le produit.

| Section | Description | Concepts clés |
|---------|-------------|---------------|
| [Documentation](./partie-7-production-business/03-support-maintenance/01-documentation.md) | In-app help | Contextuelle, searchable |
| [Feedback in-app](./partie-7-production-business/03-support-maintenance/02-feedback.md) | Widget | Screenshot, logs, email |
| [Télémétrie](./partie-7-production-business/03-support-maintenance/03-telemetrie.md) | Analytics | Opt-in, anonymisée, RGPD |
| [Gestion des bugs](./partie-7-production-business/03-support-maintenance/04-gestion-bugs.md) | Crash reports | Panic handler, issue tracking |

#### [Chapitre 32 : Monétisation avancée](./partie-7-production-business/04-monetisation-avancee/)

> Stratégies de revenus complémentaires.

| Section | Description | Concepts clés |
|---------|-------------|---------------|
| [Modules complémentaires](./partie-7-production-business/04-monetisation-avancee/01-modules-complementaires.md) | Add-ons | Marketplace, licensing |
| [Personnalisation](./partie-7-production-business/04-monetisation-avancee/02-personnalisation.md) | White-label | Branding, custom features |
| [Services associés](./partie-7-production-business/04-monetisation-avancee/03-services.md) | Premium | Support, formation, consulting |
| [Affiliation](./partie-7-production-business/04-monetisation-avancee/04-affiliation.md) | Partenariats | Referral program, tracking |

#### [Chapitre 33 : Vision long terme](./partie-7-production-business/05-vision-long-terme/)

> Faire évoluer le produit sur le long terme.

| Section | Description | Concepts clés |
|---------|-------------|---------------|
| [Roadmap](./partie-7-production-business/05-vision-long-terme/01-roadmap.md) | Planification | Prioritization, public roadmap |
| [Communauté](./partie-7-production-business/05-vision-long-terme/02-communaute.md) | Engagement | Feature voting, Discord, forums |
| [Open source](./partie-7-production-business/05-vision-long-terme/03-open-source.md) | Stratégie hybride | Core open, addons paid |
| [Évolution technique](./partie-7-production-business/05-vision-long-terme/04-evolution-technique.md) | Maintenance | Dependency updates, migrations |
| [Métriques](./partie-7-production-business/05-vision-long-terme/05-metriques.md) | KPIs | DAU, MAU, NPS, churn, LTV |

---

### [Annexes](./annexes/)

> Ressources de référence rapide pour consultation quotidienne.

| Annexe | Description | Utilisation |
|--------|-------------|-------------|
| [A. Cargo.toml complet](./annexes/A-cargo-toml-complet.md) | Configuration avec dépendances | Copier-coller pour démarrer |
| [B. Cheatsheet egui](./annexes/B-cheatsheet-egui.md) | Référence rapide widgets | Aide-mémoire pendant le développement |
| [C. Patterns Rust](./annexes/C-patterns-rust.md) | Résumé des patterns | Rappel des bonnes pratiques |
| [D. Checklist de lancement](./annexes/D-checklist-lancement.md) | Todo-list release | Vérification avant publication |
| [E. Ressources et liens](./annexes/E-ressources-liens.md) | Documentation externe | Pour aller plus loin |
| [F. Glossaire](./annexes/F-glossaire.md) | Définitions | Référence terminologique |

---

## Pour commencer

### Prérequis

```
┌─────────────────────────────────────────────────────────────────┐
│                    PRÉREQUIS RECOMMANDÉS                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ✅ Rust basique      Variables, fonctions, structs, enums     │
│  ✅ Ligne de commande Cargo, terminal basics                    │
│  ✅ Git               Clone, commit, push (optionnel)          │
│  ❌ Pas requis        Expérience GUI, async avancé             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Installation rapide

```bash
# Installer Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Vérifier l'installation
rustc --version
cargo --version

# Créer un nouveau projet desktop
cargo new mon-app-desktop
cd mon-app-desktop

# Ajouter egui/eframe
cargo add eframe
```

### Comment utiliser ce livre

| Mode | Description | Recommandé pour |
|------|-------------|-----------------|
| **Lecture linéaire** | Parties I → VII dans l'ordre | Débutants, apprentissage complet |
| **Par projet** | Partie V + chapitres nécessaires | Développeurs pressés |
| **Référence** | Chapitres spécifiques selon besoins | Développeurs expérimentés |
| **Projet fil rouge** | Suivre NoteVault (Chapitre 24) | Apprentissage pratique |

### Premier exemple

```rust
// main.rs - Votre première application desktop en Rust
use eframe::egui;

fn main() -> eframe::Result<()> {
    let options = eframe::NativeOptions {
        viewport: egui::ViewportBuilder::default()
            .with_inner_size([400.0, 300.0])
            .with_title("Ma Première App Rust"),
        ..Default::default()
    };
    
    eframe::run_native(
        "mon_app",
        options,
        Box::new(|_cc| Ok(Box::new(MonApp::default()))),
    )
}

#[derive(Default)]
struct MonApp {
    nom: String,
    compteur: i32,
}

impl eframe::App for MonApp {
    fn update(&mut self, ctx: &egui::Context, _frame: &mut eframe::Frame) {
        egui::CentralPanel::default().show(ctx, |ui| {
            ui.heading("Bienvenue dans Rust Desktop!");
            
            ui.horizontal(|ui| {
                ui.label("Votre nom:");
                ui.text_edit_singleline(&mut self.nom);
            });
            
            if !self.nom.is_empty() {
                ui.label(format!("Bonjour, {}!", self.nom));
            }
            
            ui.separator();
            
            ui.horizontal(|ui| {
                if ui.button("-").clicked() {
                    self.compteur -= 1;
                }
                ui.label(format!("Compteur: {}", self.compteur));
                if ui.button("+").clicked() {
                    self.compteur += 1;
                }
            });
        });
    }
}
```

Exécutez avec `cargo run` et vous avez votre première application desktop fonctionnelle!

---

## Contribuer

Les contributions sont les bienvenues! Si vous trouvez des erreurs, avez des suggestions ou souhaitez améliorer le contenu :

1. Ouvrez une issue pour discuter du changement
2. Fork le repository
3. Créez une branche pour votre modification
4. Soumettez une pull request

---

## Licence

Ce livre est publié sous licence **MIT**. Vous êtes libre de l'utiliser, le modifier et le distribuer.

---

<p align="center">
  <strong>Commençons le voyage vers la maîtrise du développement desktop en Rust.</strong>
</p>

<p align="center">
  <a href="./partie-1-mindset-rust-desktop/">Commencer la lecture →</a>
</p>
