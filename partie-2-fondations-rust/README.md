# Partie II — Fondations Rust Solides

Cette partie couvre les concepts Rust essentiels pour construire des applications desktop robustes. L'objectif n'est pas d'enseigner Rust depuis zéro, mais de maîtriser les patterns spécifiques au développement d'applications réelles.

## Chapitres

1. **[Rust pour développeurs pragmatiques](./01-rust-pragmatique/)**
   - Ownership expliqué simplement
   - Borrow checker sans douleur
   - Erreurs idiomatiques (Result, anyhow, thiserror)
   - Patterns utiles

2. **[Organiser un vrai projet](./02-organiser-projet/)**
   - Cargo workspace
   - Crates modulaires
   - Architecture de dossiers claire
   - Tests et benchmarks

3. **[Async et concurrence desktop](./03-async-concurrence/)**
   - Tokio
   - Channels (crossbeam)
   - Arc et ArcSwap
   - Thread UI vs workers
   - Éviter les freezes

4. **[Logs, debug et profiling](./04-logs-debug-profiling/)**
   - Tracing
   - Logs visuels
   - Flamegraphs
   - Optimiser sans se mentir

## Objectifs d'apprentissage

À la fin de cette partie, vous serez capable de :

- Écrire du code Rust idiomatique et maintenable
- Structurer un projet multi-crates professionnel
- Gérer la concurrence sans bloquer l'interface
- Diagnostiquer et optimiser les performances

## Prérequis

- Familiarité basique avec la syntaxe Rust
- Avoir écrit au moins quelques programmes Rust simples
- Comprendre les concepts de variables, fonctions, structs

## Approche pédagogique

Chaque concept est présenté avec :
1. **Le problème** : Pourquoi ce concept existe
2. **La solution** : Comment Rust le résout
3. **En pratique** : Code réel et applicatif
4. **Pièges courants** : Ce qu'il faut éviter
