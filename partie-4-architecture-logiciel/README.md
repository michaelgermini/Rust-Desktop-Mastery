# Partie IV — Architecture d'un Vrai Logiciel

Cette partie couvre l'architecture logicielle pour construire des applications desktop maintenables, évolutives et robustes. Nous appliquons les principes de clean architecture adaptés au contexte desktop.

## Chapitres

1. **[Clean architecture pour apps desktop](./01-clean-architecture/)**
   - Core / app / UI / infra
   - Séparation logique
   - Découplage

2. **[Event bus et communication interne](./02-event-bus/)**
   - Channels
   - Pub/sub
   - Messages
   - Flux de données clair

3. **[Gestion d'état robuste](./03-gestion-etat/)**
   - Store global
   - État local
   - Cache
   - Undo stack

4. **[Stockage local souverain](./04-stockage-local/)**
   - SQLite
   - Migrations
   - Offline-first
   - Chiffrement

5. **[Fichiers, import, export et PDF](./05-fichiers-import-export/)**
   - Rapports
   - Factures
   - CSV et JSON
   - Intégration système

6. **[Plugins et extensibilité](./06-plugins-extensibilite/)**
   - Modules dynamiques
   - Feature flags
   - Architecture plugin

## Objectifs d'apprentissage

À la fin de cette partie, vous serez capable de :

- Structurer une application avec une clean architecture
- Implémenter une communication inter-composants efficace
- Gérer l'état de manière prévisible
- Persister les données localement de façon fiable
- Générer des exports et des rapports PDF
- Concevoir un système de plugins extensible

## Principes directeurs

| Principe | Application |
|----------|-------------|
| Séparation des responsabilités | Chaque module a un rôle unique |
| Dépendances vers l'intérieur | Core ne dépend de rien |
| Testabilité | Chaque couche testable isolément |
| Évolutivité | Ajouter des features sans casser l'existant |
