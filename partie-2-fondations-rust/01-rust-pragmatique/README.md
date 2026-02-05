# Chapitre 4 : Rust pour Développeurs Pragmatiques

Ce chapitre couvre les concepts Rust essentiels de manière pratique, en se concentrant sur ce qui est nécessaire pour développer des applications desktop.

## Sous-sections

1. **[Ownership expliqué simplement](./01-ownership-simplement.md)**
   - Les trois règles de l'ownership
   - Move vs Copy
   - Ownership dans les fonctions et structs

2. **[Borrow checker sans douleur](./02-borrow-checker.md)**
   - Références immutables et mutables
   - La règle d'or
   - Patterns pratiques et erreurs courantes

3. **[Erreurs idiomatiques](./03-erreurs-idiomatiques.md)**
   - Result<T, E> et Option<T>
   - L'opérateur ?
   - thiserror pour les bibliothèques
   - anyhow pour les applications

4. **[Patterns utiles](./04-patterns-utiles.md)**
   - Builder Pattern
   - Newtype Pattern
   - State Machine avec Enums
   - Extension Traits
   - From/Into et Default

## Objectifs d'apprentissage

À la fin de ce chapitre, vous serez capable de :

- Comprendre et utiliser le système d'ownership sans frustration
- Écrire du code qui compile du premier coup (ou presque)
- Gérer les erreurs de manière idiomatique
- Appliquer les patterns Rust courants dans vos projets
