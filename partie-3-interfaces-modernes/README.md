# Partie III — Interfaces Modernes en Rust

Cette partie est consacrée à la création d'interfaces utilisateur professionnelles en Rust. Nous couvrons les frameworks disponibles, les patterns de design system, et les techniques pour construire des UI réactives et esthétiques.

## Chapitres

1. **[Panorama des frameworks UI](./01-panorama-frameworks/)**
   - egui
   - Tauri
   - wgpu
   - iced
   - Quand choisir quoi

2. **[Construire une UI avec egui](./02-construire-ui-egui/)**
   - Immediate mode mental model
   - Layouts
   - Widgets
   - Gestion propre de l'état

3. **[Créer son Design System](./03-design-system/)**
   - Tokens
   - Couleurs
   - Spacing
   - Typographie
   - Thèmes
   - Dark et light
   - DPI scaling

4. **[Composants réutilisables](./04-composants-reutilisables/)**
   - Button
   - Input
   - Table
   - Sidebar
   - Modals
   - Toasts
   - Inspector panels
   - Command palette

5. **[Patterns UX professionnels](./05-patterns-ux/)**
   - Loading
   - Empty states
   - Erreurs
   - Undo et redo
   - Autosave
   - Feedback instantané

6. **[Du wireframe SVG au code Rust](./06-wireframe-vers-code/)**
   - Maquettes SVG
   - Mapping IDs vers composants
   - Design vers egui
   - Workflow rapide design vers code

## Objectifs d'apprentissage

À la fin de cette partie, vous serez capable de :

- Choisir le framework UI adapté à votre projet
- Construire des interfaces réactives avec egui
- Créer et maintenir un design system cohérent
- Implémenter des composants réutilisables de qualité professionnelle
- Appliquer les patterns UX modernes

## Philosophie

L'interface est ce que l'utilisateur voit et utilise. Une bonne UI n'est pas qu'esthétique :

- **Réactive** : Feedback instantané, pas de lag
- **Cohérente** : Design system unifié
- **Accessible** : Utilisable par tous
- **Intuitive** : L'utilisateur sait quoi faire
- **Efficace** : Optimisée pour les tâches fréquentes
