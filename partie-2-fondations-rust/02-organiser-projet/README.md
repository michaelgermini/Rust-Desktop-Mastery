# Chapitre 5 : Organiser un Vrai Projet

Ce chapitre présente les meilleures pratiques pour structurer une application Rust professionnelle, maintenable et évolutive.

## Sous-sections

1. **[Cargo Workspace](./01-cargo-workspace.md)**
   - Pourquoi utiliser un workspace
   - Configuration et structure
   - Commandes workspace

2. **[Crates Modulaires](./02-crates-modulaires.md)**
   - Principe de séparation
   - Crate core (logique métier)
   - Crate storage (persistence)
   - Crate ui (interface)

3. **[Architecture de Dossiers Claire](./03-architecture-dossiers.md)**
   - Structure complète recommandée
   - Conventions de nommage
   - Organisation des modules

4. **[Tests et Benchmarks](./04-tests-benchmarks.md)**
   - Tests unitaires
   - Tests d'intégration
   - Benchmarks avec Criterion

## Objectifs d'apprentissage

À la fin de ce chapitre, vous serez capable de :

- Structurer un projet multi-crates avec Cargo workspace
- Séparer proprement la logique métier, l'UI et la persistence
- Organiser vos fichiers selon les conventions Rust
- Écrire des tests efficaces et des benchmarks
