# Chapitre 6 : Async et Concurrence Desktop

Ce chapitre explique comment utiliser la concurrence en Rust pour éviter les freezes d'interface tout en gérant correctement les ressources partagées.

## Sous-sections

1. **[Tokio](./01-tokio.md)**
   - Configuration minimale
   - Runtime Tokio dans une app desktop
   - Spawn de tâches async

2. **[Channels (crossbeam)](./02-channels-crossbeam.md)**
   - Communication inter-threads
   - MPSC (Multiple Producer, Single Consumer)
   - Patterns de communication

3. **[Arc et ArcSwap](./03-arc-arcswap.md)**
   - Partage de données thread-safe
   - ArcSwap pour état mutable
   - Patterns de synchronisation

4. **[Thread UI vs Workers](./04-thread-ui-workers.md)**
   - Architecture thread UI / Workers
   - Règle des 16ms
   - Éviter les freezes

5. **[Éviter les freezes](./05-eviter-freezes.md)**
   - Détection des opérations bloquantes
   - Patterns de déblocage
   - Feedback utilisateur pendant les opérations longues

## Objectifs d'apprentissage

À la fin de ce chapitre, vous serez capable de :

- Utiliser Tokio pour les opérations async
- Communiquer entre threads avec des channels
- Partager des données de manière thread-safe
- Éviter les freezes d'interface utilisateur
