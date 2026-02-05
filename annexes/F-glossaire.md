# Annexe F : Glossaire

Définitions des termes techniques utilisés dans ce livre.

---

## A

**Abstraction**
: Technique permettant de masquer les détails d'implémentation derrière une interface simple.

**AES-GCM**
: Advanced Encryption Standard - Galois/Counter Mode. Algorithme de chiffrement symétrique authentifié.

**Async/Await**
: Paradigme de programmation asynchrone permettant d'écrire du code non-bloquant de manière séquentielle.

**Arc (Atomic Reference Counted)**
: Type Rust permettant le partage thread-safe de données en lecture avec comptage de références atomique.

---

## B

**Borrow Checker**
: Composant du compilateur Rust qui vérifie les règles d'emprunt (borrowing) à la compilation pour garantir la sécurité mémoire.

**Builder Pattern**
: Pattern de conception pour construire des objets complexes étape par étape avec une API fluide.

---

## C

**Cargo**
: Gestionnaire de paquets et outil de build pour Rust.

**Channel**
: Mécanisme de communication inter-threads par passage de messages (MPSC: Multiple Producer, Single Consumer).

**Clean Architecture**
: Architecture logicielle séparant le code en couches (Core, Application, Infrastructure, UI) avec des dépendances unidirectionnelles.

**Closure**
: Fonction anonyme capable de capturer des variables de son environnement.

**Crate**
: Unité de compilation en Rust, équivalent à une bibliothèque ou un exécutable.

---

## D

**DPI (Dots Per Inch)**
: Densité de pixels d'un écran. Les écrans haute résolution (Retina, HiDPI) ont un DPI élevé nécessitant une mise à l'échelle de l'UI.

**Debounce**
: Technique limitant la fréquence d'exécution d'une fonction, n'exécutant qu'après une période d'inactivité.

---

## E

**egui**
: Bibliothèque Rust d'interface graphique en mode immédiat (immediate mode GUI).

**Enum (Enumeration)**
: Type Rust permettant de définir un type avec plusieurs variantes possibles, chacune pouvant contenir des données.

**Event Bus**
: Pattern de communication découplée où les composants publient et s'abonnent à des événements.

---

## F

**Feature Flag**
: Technique permettant d'activer/désactiver des fonctionnalités sans modifier le code.

**Freemium**
: Modèle économique offrant une version gratuite avec fonctionnalités limitées et une version payante complète.

**Full-Text Search**
: Recherche dans le contenu textuel complet des documents, pas seulement les métadonnées.

---

## G

**GUI (Graphical User Interface)**
: Interface utilisateur graphique, par opposition à l'interface en ligne de commande (CLI).

---

## H

**HiDPI**
: High Dots Per Inch. Écrans haute résolution nécessitant une mise à l'échelle spécifique.

---

## I

**Immediate Mode GUI**
: Paradigme d'interface graphique où l'UI est entièrement redessinée à chaque frame, sans état persistant des widgets.

**Interior Mutability**
: Pattern Rust permettant la mutation de données à travers une référence immutable, vérifié au runtime.

---

## L

**LRU (Least Recently Used)**
: Algorithme de cache évincant les éléments les moins récemment utilisés quand le cache est plein.

**Lifetime**
: Annotation Rust indiquant la durée de validité d'une référence.

---

## M

**Macro**
: Métaprogrammation en Rust permettant de générer du code à la compilation.

**Migration (Base de données)**
: Script de modification du schéma de base de données, versionné et appliqué séquentiellement.

**MPSC**
: Multiple Producer, Single Consumer. Type de channel où plusieurs émetteurs envoient vers un seul récepteur.

---

## N

**Newtype Pattern**
: Pattern Rust créant un nouveau type wrapper autour d'un type existant pour plus de type safety.

**Notarization**
: Processus Apple vérifiant qu'une application ne contient pas de malware avant distribution.

---

## O

**Offline-First**
: Architecture où l'application fonctionne d'abord localement, synchronisant avec le serveur quand disponible.

**Optimistic Update**
: Technique appliquant immédiatement un changement à l'UI avant confirmation du serveur, avec rollback si erreur.

**Ownership**
: Système Rust où chaque valeur a un propriétaire unique, transféré lors d'assignation ou de passage en paramètre.

---

## P

**Panic**
: Erreur irrécupérable en Rust causant l'arrêt du thread (ou du programme si non géré).

**Pattern Matching**
: Construction Rust (match) permettant de déstructurer et comparer des valeurs contre des patterns.

**Plugin**
: Extension ajoutant des fonctionnalités à une application sans modifier son code source.

**Port (Architecture)**
: Interface abstraite dans Clean Architecture définissant un contrat entre couches.

---

## R

**RAII (Resource Acquisition Is Initialization)**
: Pattern où l'acquisition de ressources est liée à la création d'objet, et la libération à sa destruction.

**Repository Pattern**
: Pattern d'abstraction de la persistance des données, séparant la logique métier du stockage.

**Result<T, E>**
: Type Rust représentant soit une valeur de succès (Ok(T)) soit une erreur (Err(E)).

---

## S

**Semver (Semantic Versioning)**
: Convention de numérotation de version (MAJOR.MINOR.PATCH) indiquant la nature des changements.

**Skeleton Loading**
: Technique d'UX affichant un placeholder animé pendant le chargement du contenu réel.

**SQLite**
: Base de données relationnelle embarquée, stockée dans un seul fichier.

**State Machine**
: Modèle où un système peut être dans un état parmi un ensemble fini, transitionnant selon des règles définies.

---

## T

**Tantivy**
: Bibliothèque Rust de recherche full-text, équivalent à Apache Lucene.

**Telemetry**
: Collecte automatique de données d'utilisation et de performance.

**Thread-Safe**
: Code pouvant être exécuté en parallèle par plusieurs threads sans corruption de données.

**Tokio**
: Runtime asynchrone le plus populaire en Rust, fournissant async I/O, timers, et scheduling.

**Trait**
: Interface Rust définissant un ensemble de méthodes qu'un type peut implémenter.

---

## U

**UI (User Interface)**
: Interface utilisateur, ensemble des éléments visuels permettant l'interaction avec l'application.

**UX (User Experience)**
: Expérience utilisateur, qualité globale de l'interaction avec l'application.

**Unsafe**
: Bloc Rust permettant des opérations normalement interdites, transférant la responsabilité de sécurité au développeur.

---

## V

**VAT (Value Added Tax)**
: Taxe sur la valeur ajoutée (TVA).

---

## W

**WCAG (Web Content Accessibility Guidelines)**
: Recommandations d'accessibilité web, applicables aussi aux applications desktop.

**White-Label**
: Produit rebrandable, vendu à d'autres entreprises qui le commercialisent sous leur propre marque.

**Widget**
: Composant d'interface utilisateur (bouton, champ texte, liste, etc.).

**Workspace (Cargo)**
: Collection de crates Rust partageant configuration et dépendances.

---

## Symboles et Acronymes

**`?` (Question Mark Operator)**
: Opérateur Rust propageant automatiquement les erreurs (équivalent à `match result { Ok(v) => v, Err(e) => return Err(e.into()) }`).

**API**
: Application Programming Interface. Ensemble de fonctions exposées par une bibliothèque ou service.

**CLI**
: Command Line Interface. Interface en ligne de commande.

**CRUD**
: Create, Read, Update, Delete. Opérations de base sur les données.

**DMG**
: Disk Image. Format d'archive utilisé sur macOS pour distribuer des applications.

**JSON**
: JavaScript Object Notation. Format de sérialisation de données textuelles.

**MSI**
: Microsoft Installer. Format d'installateur Windows.

**PDF**
: Portable Document Format. Format de document universel.

**TOML**
: Tom's Obvious Minimal Language. Format de configuration utilisé par Cargo.

**UUID**
: Universally Unique Identifier. Identifiant unique de 128 bits.
