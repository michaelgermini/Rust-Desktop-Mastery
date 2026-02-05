# 30.1 Site Web Produit

## Landing page efficace

```html
<!DOCTYPE html>
<html lang="fr">
<head>
    <title>Mon App - Solution de gestion pour PME</title>
    <meta name="description" content="Gérez vos clients et factures 
        en toute souveraineté avec notre logiciel desktop.">
    
    <!-- Open Graph pour partage social -->
    <meta property="og:title" content="Mon App">
    <meta property="og:description" content="Solution de gestion...">
    <meta property="og:image" content="https://example.com/preview.png">
</head>
<body>
    <!-- Hero Section -->
    <section class="hero">
        <h1>Gérez votre entreprise sans dépendre du cloud</h1>
        <p>Solution desktop complète : clients, factures, devis</p>
        <a href="/download" class="cta-primary">Télécharger gratuitement</a>
        <a href="/demo" class="cta-secondary">Voir la démo</a>
    </section>
    
    <!-- Social Proof -->
    <section class="social-proof">
        <p>Utilisé par plus de 5 000 entreprises</p>
        <div class="logos"><!-- Logos clients --></div>
    </section>
    
    <!-- Features -->
    <section class="features">
        <h2>Pourquoi choisir Mon App ?</h2>
        <div class="feature-grid">
            <div class="feature">
                <h3>🔒 Souveraineté des données</h3>
                <p>Toutes vos données restent sur votre ordinateur</p>
            </div>
            <div class="feature">
                <h3>⚡ Rapide et fiable</h3>
                <p>Application native, pas de latence réseau</p>
            </div>
            <div class="feature">
                <h3>💰 Prix transparent</h3>
                <p>Licence unique, pas d'abonnement caché</p>
            </div>
        </div>
    </section>
    
    <!-- Pricing -->
    <section class="pricing">
        <h2>Tarification simple</h2>
        <div class="pricing-grid">
            <div class="plan">
                <h3>Gratuit</h3>
                <p class="price">0€</p>
                <ul>
                    <li>Jusqu'à 10 clients</li>
                    <li>Jusqu'à 5 factures/mois</li>
                    <li>Export PDF avec watermark</li>
                </ul>
            </div>
            <div class="plan featured">
                <h3>Pro</h3>
                <p class="price">49€</p>
                <ul>
                    <li>Clients illimités</li>
                    <li>Factures illimitées</li>
                    <li>Export PDF sans watermark</li>
                    <li>Support prioritaire</li>
                </ul>
                <a href="/buy" class="cta-primary">Acheter</a>
            </div>
        </div>
    </section>
    
    <!-- FAQ -->
    <section class="faq">
        <h2>Questions fréquentes</h2>
        <details>
            <summary>Les données sont-elles stockées dans le cloud ?</summary>
            <p>Non, toutes les données sont stockées localement sur votre ordinateur.</p>
        </details>
        <!-- Plus de questions... -->
    </section>
</body>
</html>
```

## Structure recommandée

```
site-web/
├── index.html          # Landing page
├── features/           # Pages de fonctionnalités
│   ├── clients.html
│   ├── factures.html
│   └── rapports.html
├── pricing/            # Tarification
│   └── index.html
├── docs/               # Documentation
│   ├── getting-started/
│   └── api/
├── blog/               # Articles SEO
│   ├── index.html
│   └── posts/
├── download/           # Page de téléchargement
│   └── index.html
└── support/            # FAQ, Contact
    ├── faq.html
    └── contact.html
```

## Call-to-action

```html
<!-- CTA primaire (téléchargement) -->
<a href="/download" class="cta-primary">
    ⬇️ Télécharger gratuitement
</a>

<!-- CTA secondaire (démo) -->
<a href="/demo" class="cta-secondary">
    ▶️ Voir la démo
</a>

<!-- CTA d'achat -->
<a href="/buy" class="cta-purchase">
    💳 Acheter maintenant - 49€
</a>
```

## Résumé

- **Hero** : Message clair et CTA visible
- **Social proof** : Témoignages et logos clients
- **Features** : Avantages clés mis en avant
- **Pricing** : Transparent et simple
- **FAQ** : Répondre aux objections courantes
