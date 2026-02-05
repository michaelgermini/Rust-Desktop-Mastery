# Chapitre 16 : Gestion d'État Robuste

Ce chapitre couvre les techniques avancées de gestion d'état dans une application desktop.

## Sous-sections

1. **[Store global](./01-store-global.md)**
   - État global avec ArcSwap
   - Accès thread-safe
   - Patterns de mise à jour

2. **[État local](./02-etat-local.md)**
   - Quand utiliser l'état local
   - Lifting state up
   - Props drilling

3. **[Cache](./03-cache.md)**
   - Stratégies de cache
   - LRU cache
   - Invalidation

4. **[Undo stack](./04-undo-stack.md)**
   - Implémentation d'un undo stack
   - Mementos et commandes
   - Limites et optimisations

## Objectifs d'apprentissage

À la fin de ce chapitre, vous serez capable de :

- Gérer l'état global efficacement
- Implémenter un système d'annulation
- Optimiser avec des caches intelligents
