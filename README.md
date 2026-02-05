# Rust Desktop Mastery

<p align="center">
  <img src="https://www.rust-lang.org/logos/rust-logo-512x512.png" alt="Rust Logo" width="150"/>
</p>

<h3 align="center">Architecture, UI, Design System et Mise en Production<br/>d'Applications Rust Reelles</h3>

<p align="center">
  <em>Le guide complet et pragmatique pour creer des applications desktop professionnelles en Rust.<br/>
  De l'idee au produit fini, du premier pixel au premier client.</em>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Rust-2021_Edition-orange?style=for-the-badge&logo=rust&logoColor=white" alt="Rust 2021"/>
  <img src="https://img.shields.io/badge/Chapitres-33-blue?style=for-the-badge" alt="33 Chapitres"/>
  <img src="https://img.shields.io/badge/Annexes-6-green?style=for-the-badge" alt="6 Annexes"/>
  <img src="https://img.shields.io/badge/Sections-100+-purple?style=for-the-badge" alt="100+ Sections"/>
</p>

<p align="center">
  <a href="#-vue-densemble">Vue d'ensemble</a> &#8226;
  <a href="#-pourquoi-ce-livre-">Pourquoi ce livre</a> &#8226;
  <a href="#-structure-du-livre">Structure</a> &#8226;
  <a href="#-pour-commencer">Commencer</a> &#8226;
  <a href="#-faq">FAQ</a>
</p>

<p align="center">
  <a href="#partie-i--le-mindset-rust-desktop">I. Mindset</a> &#8226;
  <a href="#partie-ii--fondations-rust-solides">II. Fondations</a> &#8226;
  <a href="#partie-iii--interfaces-modernes-en-rust">III. UI</a> &#8226;
  <a href="#partie-iv--architecture-dun-vrai-logiciel">IV. Architecture</a> &#8226;
  <a href="#partie-v--blocs-metiers-reels">V. Metiers</a> &#8226;
  <a href="#partie-vi--finition-et-qualite-produit">VI. Qualite</a> &#8226;
  <a href="#partie-vii--production-et-business">VII. Business</a>
</p>

---

## Vue d'ensemble

Ce livre est un **guide complet** pour creer des applications desktop professionnelles en Rust. Il couvre l'ensemble du cycle de developpement : de la conception architecturale au deploiement, en passant par la creation d'interfaces modernes et de systemes de design reutilisables.

> **Ce n'est pas un enieme tutoriel Rust.** C'est un guide de A a Z pour construire, designer, architecturer et **vendre** un vrai logiciel desktop.

### Ce que vous allez construire

En suivant ce livre, vous serez capable de creer des applications comme :

| Type d'application | Exemple concret | Chapitres cles |
|---|---|---|
| **ERP / Gestion commerciale** | Clients, factures, TVA, exports PDF | Parties IV + V |
| **Outil de productivite** | Notes, recherche full-text, IA locale | Parties III + V |
| **Dashboard professionnel** | KPIs, graphiques temps reel, data viz | Parties III + V |
| **Logiciel metier sur mesure** | Offline-first, chiffrement, plugins | Parties IV + VI |

### Le livre en chiffres

```
+-----------------------------------------------------------------------------+
|                         RUST DESKTOP MASTERY                                |
+-----------------------------------------------------------------------------+
|  7 Parties  |  33 Chapitres  |  100+ Sous-sections  |  6 Annexes           |
+-----------------------------------------------------------------------------+
|                                                                             |
|  PARTIE I      Mindset & Philosophie                    [Chapitres 1-3]    |
|  PARTIE II     Fondations Rust                          [Chapitres 4-7]    |
|  PARTIE III    Interfaces Utilisateur                   [Chapitres 8-13]   |
|  PARTIE IV     Architecture Logicielle                  [Chapitres 14-19]  |
|  PARTIE V      Fonctionnalites Metiers                  [Chapitres 20-24]  |
|  PARTIE VI     Finition & Qualite                       [Chapitres 25-28]  |
|  PARTIE VII    Production & Business                    [Chapitres 29-33]  |
|                                                                             |
+-----------------------------------------------------------------------------+
```

### Technologies couvertes

| Categorie | Technologies |
|-----------|--------------|
| **Langage** | Rust 2021 Edition |
| **UI Framework** | egui, eframe, Tauri, iced, wgpu |
| **Base de donnees** | SQLite (rusqlite), migrations |
| **Async/Concurrence** | Tokio, crossbeam, Arc, ArcSwap |
| **Recherche** | Tantivy (full-text search) |
| **IA locale** | Candle, embeddings, inference CPU |
| **PDF** | printpdf, generation de documents |
| **Logs/Profiling** | tracing, flamegraphs, criterion |
| **Packaging** | cargo-wix (Windows), DMG (macOS), AppImage (Linux) |
| **Tests** | cargo test, criterion (benchmarks), snapshots UI |
| **Securite** | AES-GCM, Argon2, Ed25519, chiffrement local |

---

## Pourquoi ce livre ?

### Le retour du desktop

Le developpement desktop connait une **renaissance**. Apres des annees de domination du web et des applications SaaS, les developpeurs et les entreprises redecouvrent les avantages des applications natives.

| Probleme Web/SaaS | Solution Desktop Rust |
|---|---|
| Latence reseau (100-500ms) | Reponse instantanee (<10ms) |
| Abonnement mensuel obligatoire | Achat unique possible |
| Donnees sur serveur tiers | Donnees 100% locales |
| Dependance internet | Fonctionne offline |
| Infrastructure complexe | Un seul fichier binaire |
| Risques de securite cloud | Chiffrement local |
| Couts qui augmentent avec l'usage | Cout fixe, scalabilite gratuite |

### Pourquoi Rust ?

**Rust** est le langage ideal pour cette nouvelle ere :

```
+-------------------------------------------------------------------+
|                    AVANTAGES DE RUST                                |
+-------------------------------------------------------------------+
|                                                                     |
|  PERFORMANCE        Comparable au C/C++, zero-cost                 |
|                     abstractions, pas de GC                         |
|                                                                     |
|  SECURITE MEMOIRE   Ownership, borrowing, pas de null              |
|                     pointer, pas de data races                      |
|                                                                     |
|  BINAIRE UNIQUE     Compilation statique, pas de DLL,              |
|                     deploiement simplifie                           |
|                                                                     |
|  TOOLING MODERNE    Cargo, rustfmt, clippy, rust-analyzer          |
|                                                                     |
|  CROSS-PLATFORM     Windows, macOS, Linux depuis le meme           |
|                     code source                                     |
|                                                                     |
+-------------------------------------------------------------------+
```

### Ce qui rend ce livre unique

D'autres ressources enseignent Rust. D'autres parlent d'UI. Ce livre est le **seul** a couvrir le parcours complet :

```
  Idee
   |
   v
  [Partie I]    Pourquoi Rust ? Pourquoi desktop ? Penser produit.
   |
   v
  [Partie II]   Ownership, async, projet multi-crates, profiling.
   |
   v
  [Partie III]  egui, design system, composants, UX pro.
   |
   v
  [Partie IV]   Clean archi, event bus, SQLite, plugins.
   |
   v
  [Partie V]    ERP, recherche, dashboards, IA locale.
   |
   v
  [Partie VI]   Performance, accessibilite, packaging, tests.
   |
   v
  [Partie VII]  Licences, distribution, marketing, monetisation.
   |
   v
  Produit fini, distribue, monetise
```

**Concretement, ce livre vous donne :**

- **Du code reel** -- pas des exemples jouets, mais des modules production-ready
- **Une vision produit** -- comment penser UX, business et technique ensemble
- **Un cas d'etude complet** -- NoteVault, une app construite de A a Z (Chapitre 24)
- **Des patterns eprouves** -- clean architecture, event bus, state management adaptes au desktop
- **Le chemin vers la monetisation** -- licences, distribution, marketing, support

### A qui s'adresse ce livre ?

| Profil | Ce que vous apprendrez |
|--------|------------------------|
| **Developpeur Web fatigue** | Creer des apps performantes sans Electron, npm, ou infrastructure |
| **Developpeur Rust debutant** | Appliquer Rust a des projets concrets avec UI |
| **Developpeur desktop experimente** | Moderniser votre stack avec Rust et egui |
| **Entrepreneur / Indie hacker** | Construire et monetiser un logiciel desktop |
| **Architecte logiciel** | Patterns et architecture pour apps maintenables |

### Ce que vous ne trouverez PAS ici

Pour etre honnete et vous faire gagner du temps :

- **Un cours Rust debutant** -- on suppose les bases acquises (variables, fonctions, structs, enums)
- **Du code copie-colle sans explication** -- chaque decision est expliquee et justifiee
- **Une couverture exhaustive de tous les frameworks** -- on se concentre sur egui avec des comparaisons
- **Du developpement web/serveur** -- c'est 100% desktop, 100% local-first

---

## Structure du livre

### Parcours de lecture recommande

```
  DEBUTANT RUST                    DEVELOPPEUR PRESSE           ENTREPRENEUR
  ============                     ==================           =============

  Partie I   (philosophie)         Partie III (UI)              Partie I   (vision)
      |                                |                            |
      v                                v                            v
  Partie II  (fondations)          Partie IV  (archi)           Partie V   (fonctionnalites)
      |                                |                            |
      v                                v                            v
  Partie III (UI)                  Partie V   (metiers)         Partie VII (business)
      |                                |                            |
      v                                v                            v
  Partie IV  (archi)               Partie VI  (qualite)         Partie VI  (polish)
      |                                |
      v                                v
  Partie V   (metiers)            Partie VII (business)
      |
      v
  Partie VI  (qualite)
      |
      v
  Partie VII (business)
```

---

### [Partie I -- Le Mindset Rust Desktop](./partie-1-mindset-rust-desktop/)

> **Objectif** : Comprendre pourquoi Rust est le choix ideal pour le desktop et adopter la bonne philosophie produit.

Cette partie pose les **fondations philosophiques et strategiques**. Avant d'ecrire une seule ligne de code, il est crucial de comprendre *pourquoi* nous faisons ces choix et *comment* penser notre travail.

**Vous apprendrez a :**
- Argumenter le choix de Rust pour un projet desktop
- Identifier les cas d'usage ou le desktop surpasse le web
- Adopter une mentalite produit orientee utilisateur
- Evaluer les compromis entre performance, UX et time-to-market

#### [Chapitre 1 : Pourquoi Rust pour le logiciel moderne](./partie-1-mindset-rust-desktop/01-pourquoi-rust/)

> Explore les raisons fondamentales qui font de Rust le choix ideal : fatigue de l'ecosysteme JS/Electron, performance native, securite memoire, et souverainete des donnees.

| Section | Description | Concepts cles |
|---------|-------------|---------------|
| [La fatigue JS / Electron / SaaS](./partie-1-mindset-rust-desktop/01-pourquoi-rust/01-fatigue-js-electron.md) | Problemes de l'ecosysteme actuel | Memoire Electron, node_modules, instabilite npm |
| [Performance native](./partie-1-mindset-rust-desktop/01-pourquoi-rust/02-performance-native.md) | Pourquoi Rust est rapide | Zero-cost abstractions, comparaison JS/JVM |
| [Securite memoire](./partie-1-mindset-rust-desktop/01-pourquoi-rust/03-securite-memoire.md) | Bugs evites a la compilation | Ownership, borrowing, Option vs null |
| [Binaire unique](./partie-1-mindset-rust-desktop/01-pourquoi-rust/04-binaire-unique.md) | Deploiement simplifie | Compilation statique, pas de DLL hell |
| [Offline-first](./partie-1-mindset-rust-desktop/01-pourquoi-rust/05-offline-first.md) | Architecture locale | Donnees locales, sync optionnelle |
| [Souverainete des donnees](./partie-1-mindset-rust-desktop/01-pourquoi-rust/06-souverainete-donnees.md) | Controle utilisateur | Chiffrement local, RGPD, vie privee |

#### [Chapitre 2 : Desktop is back](./partie-1-mindset-rust-desktop/02-desktop-is-back/)

> Le retour en force des applications locales : pourquoi et quand le desktop surpasse le web.

| Section | Description | Concepts cles |
|---------|-------------|---------------|
| [Le retour des apps locales](./partie-1-mindset-rust-desktop/02-desktop-is-back/01-retour-apps-locales.md) | Tendances du marche | Obsidian, Notion local, Linear |
| [UX superieure au navigateur](./partie-1-mindset-rust-desktop/02-desktop-is-back/02-ux-superieure.md) | Avantages UX natifs | Raccourcis globaux, integration OS |
| [IA locale, SQLite, fichiers](./partie-1-mindset-rust-desktop/02-desktop-is-back/03-ia-locale-sqlite.md) | Technologies cles | LLM locaux, SQLite embarque |
| [Cas concrets](./partie-1-mindset-rust-desktop/02-desktop-is-back/04-cas-concrets.md) | Exemples d'applications | ERP, prise de notes, automation |
| [Quand ne pas utiliser le web](./partie-1-mindset-rust-desktop/02-desktop-is-back/05-quand-pas-web.md) | Criteres de decision | Matrice de choix web vs desktop |

#### [Chapitre 3 : Penser produit, pas repo GitHub](./partie-1-mindset-rust-desktop/03-penser-produit/)

> Adopter une mentalite produit orientee utilisateur plutot qu'une approche purement technique.

| Section | Description | Concepts cles |
|---------|-------------|---------------|
| [Ship plutot que demo](./partie-1-mindset-rust-desktop/03-penser-produit/01-ship-plutot-que-demo.md) | Livrer de la valeur | MVP, iteration, feedback loop |
| [Logiciel utilisable 8h/jour](./partie-1-mindset-rust-desktop/03-penser-produit/02-logiciel-8h-jour.md) | Ergonomie professionnelle | Fatigue visuelle, raccourcis, workflow |
| [Dette UX vs dette technique](./partie-1-mindset-rust-desktop/03-penser-produit/03-dette-ux-vs-technique.md) | Prioriser l'experience | Impact utilisateur, cout de la friction |
| [Le design comme avantage](./partie-1-mindset-rust-desktop/03-penser-produit/04-design-avantage-concurrentiel.md) | Differenciation | Design system, coherence, polish |

---

### [Partie II -- Fondations Rust Solides](./partie-2-fondations-rust/)

> **Objectif** : Maitriser les concepts Rust essentiels pour construire des applications robustes.

Cette partie couvre les **concepts Rust necessaires au developpement desktop**. L'objectif n'est pas d'enseigner Rust depuis zero, mais de maitriser les patterns specifiques aux applications reelles.

**Vous apprendrez a :**
- Ecrire du code Rust idiomatique et maintenable
- Structurer un projet multi-crates professionnel
- Gerer la concurrence sans bloquer l'interface
- Diagnostiquer et optimiser les performances

#### [Chapitre 4 : Rust pour developpeurs pragmatiques](./partie-2-fondations-rust/01-rust-pragmatique/)

> Les concepts Rust essentiels expliques de maniere pratique, avec focus sur le developpement desktop.

| Section | Description | Concepts cles |
|---------|-------------|---------------|
| [Ownership explique simplement](./partie-2-fondations-rust/01-rust-pragmatique/01-ownership-simplement.md) | Les 3 regles fondamentales | Move, Copy, Drop, ownership transfer |
| [Borrow checker sans douleur](./partie-2-fondations-rust/01-rust-pragmatique/02-borrow-checker.md) | References maitrisees | &T, &mut T, lifetimes basiques |
| [Erreurs idiomatiques](./partie-2-fondations-rust/01-rust-pragmatique/03-erreurs-idiomatiques.md) | Gestion d'erreurs Rust | Result, Option, ?, thiserror, anyhow |
| [Patterns utiles](./partie-2-fondations-rust/01-rust-pragmatique/04-patterns-utiles.md) | Patterns courants | Builder, Newtype, State Machine, From/Into |

#### [Chapitre 5 : Organiser un vrai projet](./partie-2-fondations-rust/02-organiser-projet/)

> Structurer une application Rust professionnelle, maintenable et evolutive.

| Section | Description | Concepts cles |
|---------|-------------|---------------|
| [Cargo Workspace](./partie-2-fondations-rust/02-organiser-projet/01-cargo-workspace.md) | Multi-crates | Workspace, dependances partagees |
| [Crates Modulaires](./partie-2-fondations-rust/02-organiser-projet/02-crates-modulaires.md) | Separation des responsabilites | core, app, ui, infrastructure |
| [Architecture de Dossiers](./partie-2-fondations-rust/02-organiser-projet/03-architecture-dossiers.md) | Organisation du code | Conventions, modules, visibilite |
| [Tests et Benchmarks](./partie-2-fondations-rust/02-organiser-projet/04-tests-benchmarks.md) | Qualite du code | #[test], integration tests, Criterion |

#### [Chapitre 6 : Async et concurrence desktop](./partie-2-fondations-rust/03-async-concurrence/)

> Gerer la concurrence sans bloquer l'interface utilisateur.

| Section | Description | Concepts cles |
|---------|-------------|---------------|
| [Tokio](./partie-2-fondations-rust/03-async-concurrence/01-tokio.md) | Runtime async | spawn, select!, multi-thread runtime |
| [Channels (crossbeam)](./partie-2-fondations-rust/03-async-concurrence/02-channels-crossbeam.md) | Communication inter-threads | mpsc, bounded/unbounded, select |
| [Arc et ArcSwap](./partie-2-fondations-rust/03-async-concurrence/03-arc-arcswap.md) | Partage thread-safe | Arc<Mutex<T>>, ArcSwap, RCU pattern |
| [Thread UI vs workers](./partie-2-fondations-rust/03-async-concurrence/04-thread-ui-workers.md) | Separation UI/calcul | Main thread, background workers |
| [Eviter les freezes](./partie-2-fondations-rust/03-async-concurrence/05-eviter-freezes.md) | UI toujours reactive | Chunked processing, cancellation |

#### [Chapitre 7 : Logs, debug et profiling](./partie-2-fondations-rust/04-logs-debug-profiling/)

> Diagnostiquer et optimiser les performances de maniere rigoureuse.

| Section | Description | Concepts cles |
|---------|-------------|---------------|
| [Tracing](./partie-2-fondations-rust/04-logs-debug-profiling/01-tracing.md) | Logs structures | spans, levels, subscribers, #[instrument] |
| [Logs visuels](./partie-2-fondations-rust/04-logs-debug-profiling/02-logs-visuels.md) | Panel de logs integre | egui log viewer, filtrage |
| [Flamegraphs](./partie-2-fondations-rust/04-logs-debug-profiling/03-flamegraphs.md) | Profiling visuel | cargo-flamegraph, tracing-chrome |
| [Optimiser sans se mentir](./partie-2-fondations-rust/04-logs-debug-profiling/04-optimiser-sans-mentir.md) | Mesures rigoureuses | Benchmarks, profiling avant optimisation |

---

### [Partie III -- Interfaces Modernes en Rust](./partie-3-interfaces-modernes/)

> **Objectif** : Construire des interfaces utilisateur professionnelles et reactives.

Cette partie est consacree a la **creation d'UI en Rust**. Nous couvrons les frameworks disponibles, les patterns de design system, et les techniques pour construire des interfaces modernes.

**Vous apprendrez a :**
- Choisir le framework UI adapte a votre projet
- Construire des interfaces reactives avec egui
- Creer et maintenir un design system coherent
- Implementer des composants reutilisables de qualite professionnelle

#### [Chapitre 8 : Panorama des frameworks UI](./partie-3-interfaces-modernes/01-panorama-frameworks/)

> Comparaison detaillee des frameworks UI Rust pour choisir le bon outil.

| Section | Description | Concepts cles |
|---------|-------------|---------------|
| [egui](./partie-3-interfaces-modernes/01-panorama-frameworks/01-egui.md) | Immediate mode GUI | Simple, rapide, prototypage |
| [Tauri](./partie-3-interfaces-modernes/01-panorama-frameworks/02-tauri.md) | Web frontend + Rust | HTML/CSS/JS, WebView, IPC |
| [wgpu](./partie-3-interfaces-modernes/01-panorama-frameworks/03-wgpu.md) | GPU rendering | Bas niveau, performance max |
| [iced](./partie-3-interfaces-modernes/01-panorama-frameworks/04-iced.md) | Declaratif Elm-style | Etat immutable, messages |
| [Quand choisir quoi](./partie-3-interfaces-modernes/01-panorama-frameworks/05-quand-choisir-quoi.md) | Matrice de decision | Criteres, recommandations |

#### [Chapitre 9 : Construire une UI avec egui](./partie-3-interfaces-modernes/02-construire-ui-egui/)

> Guide pratique complet pour maitriser egui, le framework choisi pour ce livre.

| Section | Description | Concepts cles |
|---------|-------------|---------------|
| [Immediate mode](./partie-3-interfaces-modernes/02-construire-ui-egui/01-immediate-mode.md) | Mental model | Difference avec retained mode |
| [Layouts](./partie-3-interfaces-modernes/02-construire-ui-egui/02-layouts.md) | Organisation visuelle | Vertical, horizontal, grid, panels |
| [Widgets](./partie-3-interfaces-modernes/02-construire-ui-egui/03-widgets.md) | Elements d'interface | Labels, buttons, inputs, sliders |
| [Gestion de l'etat](./partie-3-interfaces-modernes/02-construire-ui-egui/04-gestion-etat.md) | State management | Local vs global, persistence |

#### [Chapitre 10 : Creer son Design System](./partie-3-interfaces-modernes/03-design-system/)

> Construire un systeme de design coherent et maintenable.

| Section | Description | Concepts cles |
|---------|-------------|---------------|
| [Tokens](./partie-3-interfaces-modernes/03-design-system/01-tokens.md) | Variables centralisees | Constantes de design, JSON tokens |
| [Couleurs](./partie-3-interfaces-modernes/03-design-system/02-couleurs.md) | Palette coherente | Primary, secondary, semantic colors |
| [Spacing](./partie-3-interfaces-modernes/03-design-system/03-spacing.md) | Grille et espacement | 4px/8px grid, margins, paddings |
| [Typographie](./partie-3-interfaces-modernes/03-design-system/04-typographie.md) | Hierarchie textuelle | Font sizes, weights, line heights |
| [Themes](./partie-3-interfaces-modernes/03-design-system/05-themes.md) | Dark/light mode | Theme switching, persistence |
| [DPI scaling](./partie-3-interfaces-modernes/03-design-system/06-dpi-scaling.md) | Multi-resolutions | HiDPI, scaling factors |

#### [Chapitre 11 : Composants reutilisables](./partie-3-interfaces-modernes/04-composants-reutilisables/)

> Bibliotheque de composants UI professionnels prets a l'emploi.

| Section | Description | Concepts cles |
|---------|-------------|---------------|
| [Button](./partie-3-interfaces-modernes/04-composants-reutilisables/01-button.md) | Boutons | Primary, secondary, danger, disabled |
| [Input](./partie-3-interfaces-modernes/04-composants-reutilisables/02-input.md) | Champs de saisie | Text, password, validation |
| [Table](./partie-3-interfaces-modernes/04-composants-reutilisables/03-table.md) | Tableaux de donnees | Tri, pagination, selection |
| [Sidebar](./partie-3-interfaces-modernes/04-composants-reutilisables/04-sidebar.md) | Navigation laterale | Collapse, nested items |
| [Modals](./partie-3-interfaces-modernes/04-composants-reutilisables/05-modals.md) | Dialogues | Confirmation, formulaires |
| [Toasts](./partie-3-interfaces-modernes/04-composants-reutilisables/06-toasts.md) | Notifications | Success, error, warning |
| [Inspector panels](./partie-3-interfaces-modernes/04-composants-reutilisables/07-inspector-panels.md) | Panneaux de proprietes | Details, edition |
| [Command palette](./partie-3-interfaces-modernes/04-composants-reutilisables/08-command-palette.md) | Recherche d'actions | Ctrl+K, fuzzy search |

#### [Chapitre 12 : Patterns UX professionnels](./partie-3-interfaces-modernes/05-patterns-ux/)

> Les patterns UX qui font la difference entre une app amateur et professionnelle.

| Section | Description | Concepts cles |
|---------|-------------|---------------|
| [Loading](./partie-3-interfaces-modernes/05-patterns-ux/01-loading.md) | Etats de chargement | Spinners, progress bars |
| [Empty states](./partie-3-interfaces-modernes/05-patterns-ux/02-empty-states.md) | Ecrans vides | Illustrations, call-to-action |
| [Erreurs](./partie-3-interfaces-modernes/05-patterns-ux/03-erreurs.md) | Messages d'erreur | Clairs, actionnables |
| [Undo et redo](./partie-3-interfaces-modernes/05-patterns-ux/04-undo-redo.md) | Historique d'actions | Command pattern, stack |
| [Autosave](./partie-3-interfaces-modernes/05-patterns-ux/05-autosave.md) | Sauvegarde automatique | Debouncing, indicateur |
| [Feedback instantane](./partie-3-interfaces-modernes/05-patterns-ux/06-feedback-instantane.md) | Reponse < 100ms | Optimistic updates |

#### [Chapitre 13 : Du wireframe SVG au code Rust](./partie-3-interfaces-modernes/06-wireframe-vers-code/)

> Workflow efficace pour transformer des maquettes en code.

| Section | Description | Concepts cles |
|---------|-------------|---------------|
| [Maquettes SVG](./partie-3-interfaces-modernes/06-wireframe-vers-code/01-maquettes-svg.md) | Creation de wireframes | Figma, SVG export |
| [Mapping IDs](./partie-3-interfaces-modernes/06-wireframe-vers-code/02-mapping-ids.md) | Lier design et code | Nommage, convention |
| [Design vers egui](./partie-3-interfaces-modernes/06-wireframe-vers-code/03-design-vers-egui.md) | Traduction pratique | Extraction, composants |
| [Workflow rapide](./partie-3-interfaces-modernes/06-wireframe-vers-code/04-workflow.md) | Iterations design/code | Hot reload, preview |

---

### [Partie IV -- Architecture d'un Vrai Logiciel](./partie-4-architecture-logiciel/)

> **Objectif** : Structurer une application maintenable, testable et evolutive.

Cette partie couvre l'**architecture logicielle**. Nous appliquons les principes de clean architecture adaptes au contexte desktop.

**Vous apprendrez a :**
- Structurer une application avec une clean architecture
- Implementer une communication inter-composants efficace
- Gerer l'etat de maniere previsible
- Persister les donnees localement de facon fiable

#### [Chapitre 14 : Clean Architecture pour apps desktop](./partie-4-architecture-logiciel/01-clean-architecture/)

> Organiser le code en couches independantes et testables.

| Section | Description | Concepts cles |
|---------|-------------|---------------|
| [Couches](./partie-4-architecture-logiciel/01-clean-architecture/01-couches.md) | Organisation en couches | Core, App, UI, Infrastructure |
| [Separation logique](./partie-4-architecture-logiciel/01-clean-architecture/02-separation-logique.md) | Responsabilites claires | Single responsibility |
| [Decouplage](./partie-4-architecture-logiciel/01-clean-architecture/03-decouplage.md) | Interfaces et abstractions | Dependency injection, traits |
| [Dependances](./partie-4-architecture-logiciel/01-clean-architecture/04-dependances.md) | Direction des dependances | Vers l'interieur uniquement |

#### [Chapitre 15 : Event Bus et communication interne](./partie-4-architecture-logiciel/02-event-bus/)

> Patterns de communication decouplee entre composants.

| Section | Description | Concepts cles |
|---------|-------------|---------------|
| [Channels](./partie-4-architecture-logiciel/02-event-bus/01-channels.md) | Communication via messages | mpsc, broadcast |
| [Pub/sub](./partie-4-architecture-logiciel/02-event-bus/02-pub-sub.md) | Abonnements multiples | Subscribe, publish, filter |
| [Messages](./partie-4-architecture-logiciel/02-event-bus/03-messages.md) | Types de messages | Command, Event, Query |
| [Flux de donnees](./partie-4-architecture-logiciel/02-event-bus/04-flux-donnees.md) | Flux unidirectionnel | Single source of truth |

#### [Chapitre 16 : Gestion d'etat robuste](./partie-4-architecture-logiciel/03-gestion-etat/)

> Maintenir un etat applicatif coherent et previsible.

| Section | Description | Concepts cles |
|---------|-------------|---------------|
| [Store global](./partie-4-architecture-logiciel/03-gestion-etat/01-store-global.md) | Source unique de verite | AppState, reducers |
| [Etat local](./partie-4-architecture-logiciel/03-gestion-etat/02-etat-local.md) | UI state vs app state | Separation, synchronisation |
| [Cache](./partie-4-architecture-logiciel/03-gestion-etat/03-cache.md) | Performance | LRU, TTL, invalidation |
| [Undo stack](./partie-4-architecture-logiciel/03-gestion-etat/04-undo-stack.md) | Historique des changements | Command pattern |

#### [Chapitre 17 : Stockage local souverain](./partie-4-architecture-logiciel/04-stockage-local/)

> Persister les donnees localement de facon fiable et securisee.

| Section | Description | Concepts cles |
|---------|-------------|---------------|
| [SQLite](./partie-4-architecture-logiciel/04-stockage-local/01-sqlite.md) | Base de donnees embarquee | rusqlite, requetes, transactions |
| [Migrations](./partie-4-architecture-logiciel/04-stockage-local/02-migrations.md) | Evolution du schema | Versioning, rollback |
| [Offline-first](./partie-4-architecture-logiciel/04-stockage-local/03-offline-first.md) | Sync optionnelle | Queue de sync, conflits |
| [Chiffrement](./partie-4-architecture-logiciel/04-stockage-local/04-chiffrement.md) | Securite des donnees | AES-GCM, Argon2, key derivation |

#### [Chapitre 18 : Fichiers, import, export et PDF](./partie-4-architecture-logiciel/05-fichiers-import-export/)

> Generer des documents et echanger des donnees.

| Section | Description | Concepts cles |
|---------|-------------|---------------|
| [Rapports](./partie-4-architecture-logiciel/05-fichiers-import-export/01-rapports.md) | Generation PDF | printpdf, templates |
| [Factures](./partie-4-architecture-logiciel/05-fichiers-import-export/02-factures.md) | Documents commerciaux | Layout, logo, conformite |
| [CSV et JSON](./partie-4-architecture-logiciel/05-fichiers-import-export/03-csv-json.md) | Import/export | csv crate, serde_json |
| [Integration systeme](./partie-4-architecture-logiciel/05-fichiers-import-export/04-integration-systeme.md) | Fichiers et OS | native-dialog, associations |

#### [Chapitre 19 : Plugins et extensibilite](./partie-4-architecture-logiciel/06-plugins-extensibilite/)

> Concevoir une architecture extensible par des tiers.

| Section | Description | Concepts cles |
|---------|-------------|---------------|
| [Modules dynamiques](./partie-4-architecture-logiciel/06-plugins-extensibilite/01-modules-dynamiques.md) | Chargement runtime | libloading, FFI |
| [Feature flags](./partie-4-architecture-logiciel/06-plugins-extensibilite/02-feature-flags.md) | Activation conditionnelle | Compile-time, runtime flags |
| [Architecture plugin](./partie-4-architecture-logiciel/06-plugins-extensibilite/03-architecture-plugin.md) | API stable | Trait objects, sandboxing |

---

### [Partie V -- Blocs Metiers Reels](./partie-5-blocs-metiers/)

> **Objectif** : Implementer des fonctionnalites business concretes et reutilisables.

Cette partie presente l'**implementation de fonctionnalites business**. Chaque chapitre est un module reutilisable pour vos applications.

**Vous apprendrez a :**
- Construire un systeme ERP complet
- Implementer un moteur de recherche performant
- Creer des dashboards avec visualisations
- Integrer l'IA locale dans vos applications

#### [Chapitre 20 : Construire un mini-ERP Rust](./partie-5-blocs-metiers/01-mini-erp/)

> Systeme complet de gestion commerciale : clients, factures, TVA, exports.

| Section | Description | Concepts cles |
|---------|-------------|---------------|
| [Clients](./partie-5-blocs-metiers/01-mini-erp/01-clients.md) | Gestion des clients | CRUD, validation, recherche |
| [Factures](./partie-5-blocs-metiers/01-mini-erp/02-factures.md) | Facturation | Lignes, calculs, numerotation |
| [TVA](./partie-5-blocs-metiers/01-mini-erp/03-tva.md) | Fiscalite | Taux, calcul, conformite |
| [PDF](./partie-5-blocs-metiers/01-mini-erp/04-pdf.md) | Documents | Generation, templates |
| [Exports](./partie-5-blocs-metiers/01-mini-erp/05-exports.md) | Donnees | CSV, Excel, comptabilite |

#### [Chapitre 21 : Moteur de recherche local](./partie-5-blocs-metiers/02-moteur-recherche/)

> Recherche full-text performante avec Tantivy.

| Section | Description | Concepts cles |
|---------|-------------|---------------|
| [Tantivy](./partie-5-blocs-metiers/02-moteur-recherche/01-tantivy.md) | Moteur de recherche | Architecture, installation |
| [Indexation](./partie-5-blocs-metiers/02-moteur-recherche/02-indexation.md) | Creation d'index | Schema, documents, async |
| [Full-text search](./partie-5-blocs-metiers/02-moteur-recherche/03-full-text-search.md) | Requetes | Query parser, highlighting |
| [Performance](./partie-5-blocs-metiers/02-moteur-recherche/04-performance.md) | Optimisation | Cache, incremental indexing |

#### [Chapitre 22 : Dashboard et visualisation](./partie-5-blocs-metiers/03-dashboard-visualisation/)

> Tableaux de bord avec KPIs et graphiques en temps reel.

| Section | Description | Concepts cles |
|---------|-------------|---------------|
| [KPI cards](./partie-5-blocs-metiers/03-dashboard-visualisation/01-kpi-cards.md) | Metriques cles | Cards, trends, comparaison |
| [Charts](./partie-5-blocs-metiers/03-dashboard-visualisation/02-charts.md) | Graphiques | egui_plot, line, bar, pie |
| [Temps reel](./partie-5-blocs-metiers/03-dashboard-visualisation/03-temps-reel.md) | Live updates | Polling, streaming |
| [UX data](./partie-5-blocs-metiers/03-dashboard-visualisation/04-ux-data.md) | Presentation | Lisibilite, interaction |

#### [Chapitre 23 : IA locale dans une app Rust](./partie-5-blocs-metiers/04-ia-locale/)

> Integrer l'intelligence artificielle sans cloud avec Candle.

| Section | Description | Concepts cles |
|---------|-------------|---------------|
| [Embeddings](./partie-5-blocs-metiers/04-ia-locale/01-embeddings.md) | Vecteurs de texte | BERT, sentence transformers |
| [Recherche semantique](./partie-5-blocs-metiers/04-ia-locale/02-recherche-semantique.md) | Similarite | Cosine similarity, HNSW |
| [Inference CPU](./partie-5-blocs-metiers/04-ia-locale/03-inference-cpu.md) | Sans GPU | Quantization, batch processing |
| [Souverainete IA](./partie-5-blocs-metiers/04-ia-locale/04-souverainete-ia.md) | Vie privee | Donnees locales, RGPD |

#### [Chapitre 24 : Cas d'etude complet](./partie-5-blocs-metiers/05-cas-etude-complet/)

> Application NoteVault : de zero a finie, code reel inclus.

| Section | Description | Concepts cles |
|---------|-------------|---------------|
| [De zero a finie](./partie-5-blocs-metiers/05-cas-etude-complet/01-zero-a-finie.md) | Progression complete | Planning, milestones |
| [Code reel](./partie-5-blocs-metiers/05-cas-etude-complet/02-code-reel.md) | Implementation | Architecture, modules |
| [Design system](./partie-5-blocs-metiers/05-cas-etude-complet/03-design-system.md) | Theme applique | Tokens, composants |
| [Packaging](./partie-5-blocs-metiers/05-cas-etude-complet/04-packaging.md) | Distribution | Build, installer, release |

---

### [Partie VI -- Finition et Qualite Produit](./partie-6-finition-qualite/)

> **Objectif** : Polir l'application pour une qualite professionnelle.

Cette partie couvre les aspects qui transforment une application fonctionnelle en **produit professionnel** : performance percue, accessibilite, packaging et tests.

**Vous apprendrez a :**
- Optimiser la performance percue
- Creer une application accessible
- Packager pour Windows, macOS et Linux
- Mettre en place une strategie de tests efficace

#### [Chapitre 25 : Performance percue](./partie-6-finition-qualite/01-performance-percue/)

> Techniques pour ameliorer la perception de performance par l'utilisateur.

| Section | Description | Concepts cles |
|---------|-------------|---------------|
| [Skeleton](./partie-6-finition-qualite/01-performance-percue/01-skeleton.md) | Placeholders animes | Loading states |
| [Lazy loading](./partie-6-finition-qualite/01-performance-percue/02-lazy-loading.md) | Chargement differe | Pagination, virtual scroll |
| [Demarrage instantane](./partie-6-finition-qualite/01-performance-percue/03-demarrage-instantane.md) | First paint rapide | Splash screen, progressive |
| [Caches](./partie-6-finition-qualite/01-performance-percue/04-caches.md) | Mise en cache | LRU, TTL, invalidation |

#### [Chapitre 26 : Accessibilite et ergonomie](./partie-6-finition-qualite/02-accessibilite-ergonomie/)

> Creer une application utilisable par tous.

| Section | Description | Concepts cles |
|---------|-------------|---------------|
| [Raccourcis clavier](./partie-6-finition-qualite/02-accessibilite-ergonomie/01-raccourcis-clavier.md) | Systeme de raccourcis | Global, contextuel, personnalisable |
| [Navigation clavier](./partie-6-finition-qualite/02-accessibilite-ergonomie/02-navigation-souris.md) | Focus management | Tab order, focus visible |
| [Contrastes](./partie-6-finition-qualite/02-accessibilite-ergonomie/03-contrastes.md) | WCAG compliance | AA/AAA, ratios |
| [Tailles et lisibilite](./partie-6-finition-qualite/02-accessibilite-ergonomie/04-tailles-lisibilite.md) | Zones cliquables | 44px minimum, touch targets |

#### [Chapitre 27 : Packaging multiplateforme](./partie-6-finition-qualite/03-packaging-multiplateforme/)

> Distribuer sur Windows, macOS et Linux.

| Section | Description | Concepts cles |
|---------|-------------|---------------|
| [Windows](./partie-6-finition-qualite/03-packaging-multiplateforme/01-windows.md) | MSI installer | cargo-wix, code signing |
| [macOS](./partie-6-finition-qualite/03-packaging-multiplateforme/02-macos.md) | App bundle | DMG, notarization, universal binary |
| [Linux](./partie-6-finition-qualite/03-packaging-multiplateforme/03-linux.md) | Packages | AppImage, .deb, Flatpak |
| [Installers](./partie-6-finition-qualite/03-packaging-multiplateforme/04-installers.md) | UX d'installation | Welcome, license, folder |
| [Mises a jour](./partie-6-finition-qualite/03-packaging-multiplateforme/05-mises-a-jour.md) | Auto-update | self_update, GitHub releases |

#### [Chapitre 28 : Tests UI et assurance qualite](./partie-6-finition-qualite/04-tests-ui-qualite/)

> Strategies de test pour une qualite constante.

| Section | Description | Concepts cles |
|---------|-------------|---------------|
| [Tests unitaires](./partie-6-finition-qualite/04-tests-ui-qualite/01-tests-unitaires.md) | Logique metier | Mocks, fixtures, coverage |
| [Snapshots UI](./partie-6-finition-qualite/04-tests-ui-qualite/02-snapshots-ui.md) | Regression visuelle | Screenshot comparison |
| [Tests manuels](./partie-6-finition-qualite/04-tests-ui-qualite/03-tests-manuels.md) | Checklists | Scenarios, exploratory testing |

---

### [Partie VII -- Production et Business](./partie-7-production-business/)

> **Objectif** : Transformer le code en produit viable et rentable.

Cette derniere partie couvre les **aspects business** et de mise en production d'une application desktop professionnelle.

**Vous apprendrez a :**
- Choisir et implementer un modele de licence
- Creer une presence web efficace
- Gerer le support et la maintenance
- Developper des revenus complementaires

#### [Chapitre 29 : Licences et modeles economiques](./partie-7-production-business/01-licences-modeles-economiques/)

> Choisir et implementer un modele de monetisation.

| Section | Description | Concepts cles |
|---------|-------------|---------------|
| [Types de licences](./partie-7-production-business/01-licences-modeles-economiques/01-types-licences.md) | Modeles | Perpetuelle, abonnement, freemium |
| [Generation et validation](./partie-7-production-business/01-licences-modeles-economiques/02-generation-validation.md) | Systeme de cles | Ed25519, machine ID, HMAC |
| [Activation offline](./partie-7-production-business/01-licences-modeles-economiques/03-activation-offline.md) | Sans serveur | Hardware binding, tokens |
| [Modele freemium](./partie-7-production-business/01-licences-modeles-economiques/04-modele-freemium.md) | Conversion | Free tier limits, upgrade prompts |

#### [Chapitre 30 : Distribution et marketing](./partie-7-production-business/02-distribution-marketing/)

> Faire connaitre et distribuer votre application.

| Section | Description | Concepts cles |
|---------|-------------|---------------|
| [Site web produit](./partie-7-production-business/02-distribution-marketing/01-site-web.md) | Landing page | Hero, features, pricing, CTA |
| [SEO](./partie-7-production-business/02-distribution-marketing/02-seo.md) | Visibilite | Keywords, blog, long tail |
| [Page de telechargement](./partie-7-production-business/02-distribution-marketing/03-page-telechargement.md) | Conversion | OS detection, checksums |
| [Canaux](./partie-7-production-business/02-distribution-marketing/04-canaux.md) | Distribution | Stores, Homebrew, Chocolatey |

#### [Chapitre 31 : Support et maintenance](./partie-7-production-business/03-support-maintenance/)

> Accompagner les utilisateurs et maintenir le produit.

| Section | Description | Concepts cles |
|---------|-------------|---------------|
| [Documentation](./partie-7-production-business/03-support-maintenance/01-documentation.md) | In-app help | Contextuelle, searchable |
| [Feedback in-app](./partie-7-production-business/03-support-maintenance/02-feedback.md) | Widget | Screenshot, logs, email |
| [Telemetrie](./partie-7-production-business/03-support-maintenance/03-telemetrie.md) | Analytics | Opt-in, anonymisee, RGPD |
| [Gestion des bugs](./partie-7-production-business/03-support-maintenance/04-gestion-bugs.md) | Crash reports | Panic handler, issue tracking |

#### [Chapitre 32 : Monetisation avancee](./partie-7-production-business/04-monetisation-avancee/)

> Strategies de revenus complementaires.

| Section | Description | Concepts cles |
|---------|-------------|---------------|
| [Modules complementaires](./partie-7-production-business/04-monetisation-avancee/01-modules-complementaires.md) | Add-ons | Marketplace, licensing |
| [Personnalisation](./partie-7-production-business/04-monetisation-avancee/02-personnalisation.md) | White-label | Branding, custom features |
| [Services associes](./partie-7-production-business/04-monetisation-avancee/03-services.md) | Premium | Support, formation, consulting |
| [Affiliation](./partie-7-production-business/04-monetisation-avancee/04-affiliation.md) | Partenariats | Referral program, tracking |

#### [Chapitre 33 : Vision long terme](./partie-7-production-business/05-vision-long-terme/)

> Faire evoluer le produit sur le long terme.

| Section | Description | Concepts cles |
|---------|-------------|---------------|
| [Roadmap](./partie-7-production-business/05-vision-long-terme/01-roadmap.md) | Planification | Prioritization, public roadmap |
| [Communaute](./partie-7-production-business/05-vision-long-terme/02-communaute.md) | Engagement | Feature voting, Discord, forums |
| [Open source](./partie-7-production-business/05-vision-long-terme/03-open-source.md) | Strategie hybride | Core open, addons paid |
| [Evolution technique](./partie-7-production-business/05-vision-long-terme/04-evolution-technique.md) | Maintenance | Dependency updates, migrations |
| [Metriques](./partie-7-production-business/05-vision-long-terme/05-metriques.md) | KPIs | DAU, MAU, NPS, churn, LTV |

---

### [Annexes](./annexes/)

> Ressources de reference rapide pour consultation quotidienne.

| Annexe | Description | Utilisation |
|--------|-------------|-------------|
| [A. Cargo.toml complet](./annexes/A-cargo-toml-complet.md) | Configuration avec dependances | Copier-coller pour demarrer |
| [B. Cheatsheet egui](./annexes/B-cheatsheet-egui.md) | Reference rapide widgets | Aide-memoire pendant le developpement |
| [C. Patterns Rust](./annexes/C-patterns-rust.md) | Resume des patterns | Rappel des bonnes pratiques |
| [D. Checklist de lancement](./annexes/D-checklist-lancement.md) | Todo-list release | Verification avant publication |
| [E. Ressources et liens](./annexes/E-ressources-liens.md) | Documentation externe | Pour aller plus loin |
| [F. Glossaire](./annexes/F-glossaire.md) | Definitions | Reference terminologique |

---

## Pour commencer

### Prerequis

```
+-------------------------------------------------------------------+
|                    PREREQUIS RECOMMANDES                            |
+-------------------------------------------------------------------+
|                                                                     |
|  [x] Rust basique      Variables, fonctions, structs, enums       |
|  [x] Ligne de commande Cargo, terminal basics                      |
|  [x] Git               Clone, commit, push (optionnel)            |
|  [ ] Pas requis        Experience GUI, async avance                |
|                                                                     |
+-------------------------------------------------------------------+
```

### Installation rapide

```bash
# Installer Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Verifier l'installation
rustc --version
cargo --version

# Creer un nouveau projet desktop
cargo new mon-app-desktop
cd mon-app-desktop

# Ajouter egui/eframe
cargo add eframe
```

### Comment utiliser ce livre

| Mode | Description | Recommande pour |
|------|-------------|-----------------|
| **Lecture lineaire** | Parties I a VII dans l'ordre | Debutants, apprentissage complet |
| **Par projet** | Partie V + chapitres necessaires | Developpeurs presses |
| **Reference** | Chapitres specifiques selon besoins | Developpeurs experimentes |
| **Projet fil rouge** | Suivre NoteVault (Chapitre 24) | Apprentissage pratique |

### Premier exemple

```rust
// main.rs - Votre premiere application desktop en Rust
use eframe::egui;

fn main() -> eframe::Result<()> {
    let options = eframe::NativeOptions {
        viewport: egui::ViewportBuilder::default()
            .with_inner_size([400.0, 300.0])
            .with_title("Ma Premiere App Rust"),
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

Executez avec `cargo run` et vous avez votre premiere application desktop fonctionnelle!

---

## FAQ

<details>
<summary><strong>Faut-il etre expert en Rust pour suivre ce livre ?</strong></summary>

Non. Il faut connaitre les bases (variables, fonctions, structs, enums, pattern matching), mais le livre explique en detail les concepts avances comme l'ownership, l'async et les lifetimes dans le contexte du developpement desktop. La Partie II est specialement concue pour cela.
</details>

<details>
<summary><strong>Pourquoi egui plutot que Tauri, iced ou Dioxus ?</strong></summary>

egui offre le meilleur equilibre entre **simplicite**, **performance** et **controle** pour du desktop natif pur. Pas de WebView, pas de JavaScript, pas de HTML/CSS. Le Chapitre 8 compare en detail tous les frameworks pour vous aider a choisir selon votre contexte. Les principes d'architecture (Parties IV-VII) sont applicables quel que soit le framework.
</details>

<details>
<summary><strong>Le code est-il disponible quelque part ?</strong></summary>

Chaque chapitre contient du code complet et fonctionnel. Le Chapitre 24 (NoteVault) est un cas d'etude complet de A a Z avec tout le code source. L'Annexe A fournit un `Cargo.toml` complet pret a copier-coller.
</details>

<details>
<summary><strong>Ce livre couvre-t-il le deploiement multiplateforme ?</strong></summary>

Oui. Le Chapitre 27 couvre en detail le packaging pour **Windows** (MSI via cargo-wix), **macOS** (DMG + notarization) et **Linux** (AppImage, .deb, Flatpak), ainsi que les mises a jour automatiques.
</details>

<details>
<summary><strong>Peut-on utiliser ce livre pour un projet commercial ?</strong></summary>

Absolument. C'est meme l'un des objectifs principaux. La Partie VII couvre les licences, la distribution, le marketing, la monetisation et la vision long terme. Le livre est concu pour vous amener jusqu'au premier client.
</details>

<details>
<summary><strong>Quelles versions de Rust sont supportees ?</strong></summary>

Le livre cible **Rust 2021 Edition** et toutes les versions stables recentes. Les dependances sont specifiees avec versions dans l'Annexe A.
</details>

---

## Licence

Ce livre est publie sous licence **MIT**. Vous etes libre de l'utiliser, le modifier et le distribuer.

---

<p align="center">
  <strong>Pret a maitriser le developpement desktop en Rust ?</strong>
</p>

<p align="center">
  <a href="./partie-1-mindset-rust-desktop/">Commencer la lecture</a>
</p>
#
