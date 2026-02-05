# 13.1 Maquettes SVG comme Source de Vérité

## Pourquoi SVG ?

- **Éditable** : Figma, Sketch, Inkscape exportent en SVG
- **Versionnable** : Texte, donc compatible Git
- **Mesurable** : Dimensions et positions exactes
- **Inspectable** : Structure hiérarchique visible

## Convention de nommage des IDs

```xml
<!-- Maquette SVG avec IDs sémantiques -->
<svg viewBox="0 0 1200 800">
  <!-- Layout principal -->
  <g id="layout-sidebar">
    <rect id="sidebar-bg" x="0" y="0" width="240" height="800"/>
    <g id="sidebar-nav">
      <g id="nav-item-dashboard" class="nav-item">
        <rect x="16" y="80" width="208" height="40"/>
        <text x="56" y="105">Dashboard</text>
      </g>
      <g id="nav-item-invoices" class="nav-item">
        <rect x="16" y="128" width="208" height="40"/>
        <text x="56" y="153">Factures</text>
      </g>
    </g>
  </g>
  
  <g id="layout-main">
    <g id="header">
      <text id="header-title" x="280" y="40">Tableau de bord</text>
      <g id="header-actions">
        <rect id="btn-new-invoice" x="1050" y="16" width="120" height="36"/>
      </g>
    </g>
    
    <g id="content">
      <g id="stats-grid">
        <g id="stat-card-revenue" class="stat-card">
          <!-- ... -->
        </g>
      </g>
    </g>
  </g>
</svg>
```

## Convention de nommage

```
Format: [type]-[zone]-[element]-[variant]

Exemples:
- layout-sidebar       : Zone de layout
- nav-item-dashboard   : Item de navigation
- btn-new-invoice      : Bouton d'action
- card-stat-revenue    : Card de statistique
- input-search-main    : Champ de recherche
- modal-confirm-delete : Modale de confirmation
```

## Avantages

- **Traçabilité** : Lien direct entre design et code
- **Automatisation** : Génération de code possible
- **Cohérence** : Même vocabulaire partout
- **Maintenance** : Facile de trouver les éléments

## Résumé

- **SVG comme source** : Design versionné et mesurable
- **IDs sémantiques** : Convention de nommage claire
- **Structure hiérarchique** : Organisation logique
- **Automatisation** : Base pour génération de code
