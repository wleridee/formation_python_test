Voici le chapitre **Chargement des Ressources avec ReactDOM (Client)** pour la formation React 19.2.

```markdown
---
sidebar_label: Chargement des Ressources avec ReactDOM (Client)
sidebar_position: 48
---

# Chapitre 48 : Chargement des Ressources avec ReactDOM (Client)

`preconnect`, `prefetchDNS`, `preinit`, `preinitModule`, `preload`, `preloadModule`, Optimisation du chargement

Dans les versions précédentes de React, gérer les ressources externes (polices, scripts tiers, styles) nécessisait souvent de modifier manuellement le fichier `index.html` ou d'utiliser des bibliothèques tierces comme `react-helmet`.

React 19 introduit des APIs impératives directement dans `react-dom` pour gérer le chargement des ressources. La magie ? **React hisse (hoist) et dé-duplique automatiquement** ces demandes dans la balise `<head>` du document, peu importe où vous les appelez dans votre arbre de composants.

---

## 1. `prefetchDNS` et `preconnect` : L'Optimisation Réseau {#prefetch-dns-et-preconnect}

### 1. Quoi
Ces fonctions indiquent au navigateur de préparer la connexion réseau vers un domaine externe avant même qu'une requête ne soit réellement lancée.

*   **`prefetchDNS`** : Résout uniquement l'adresse IP (DNS Lookup).
*   **`preconnect`** : Résout le DNS + Établit la connexion TCP + Négociation TLS (HTTPS).

### 2. Pourquoi
La mise en place d'une connexion sécurisée (HTTPS) peut prendre plusieurs centaines de millisecondes (Latency).
Si votre composant `<PaymentForm>` doit charger un script Stripe, ou si `<UserProfile>` doit charger une image depuis AWS S3, utiliser `preconnect` permet d'éliminer ce délai initial. Quand le composant demandera la ressource, le "tuyau" sera déjà ouvert.

### 3. Comment

#### A. Syntaxe de base

```tsx
import { preconnect, prefetchDNS } from 'react-dom';

function PaymentPage() {
  // On prépare le terrain dès que ce composant est rendu
  prefetchDNS("https://api.stripe.com");
  preconnect("https://maps.googleapis.com");
  
  return <div>Formulaire de paiement...</div>;
}
```

#### B. Cas concret : Survol d'un lien (Event Handler)

On peut aussi appeler ces fonctions dans des événements pour anticiper une navigation.

```tsx
import { preconnect } from 'react-dom';
import { useNavigate } from 'react-router-dom';

export function SmartLink() {
  const navigate = useNavigate();

  const handleMouseEnter = () => {
    // L'utilisateur survole "Login", on sait qu'il va avoir besoin 
    // des assets du domaine d'authentification (ex: Auth0)
    preconnect("https://auth.mon-saas.com");
  };

  return (
    <button onMouseEnter={handleMouseEnter} onClick={() => navigate('/login')}>
      Se connecter
    </button>
  );
}
```

### 4. Zone de Danger
❌ **À ne pas faire** : `preconnect` vers tous les domaines possibles.
Maintenir une connexion ouverte coûte des ressources CPU et mémoire au navigateur. Limitez-vous aux domaines critiques utilisés dans les secondes qui suivent. Pour les autres, préférez `prefetchDNS` qui est moins coûteux.

---

## 2. `preload` et `preloadModule` : Téléchargement Prioritaire {#preload-et-preload-module}

### 1. Quoi
Ces fonctions ordonnent au navigateur de **télécharger** une ressource immédiatement, mais **sans l'exécuter** (pour les scripts) ni l'appliquer (pour les styles) tout de suite. Elle reste en cache mémoire, prête à l'emploi.

*   `preload` : Pour les polices, images, feuilles de style, scripts standards.
*   `preloadModule` : Spécifique pour les modules ESM (`.mjs` ou scripts `type="module"`).

### 2. Pourquoi
Pour éviter les effets de "saut" ou de retard visuel.
*   **Images (LCP)** : Précharger l'image "Hero" pour qu'elle s'affiche instantanément.
*   **Polices (CLS)** : Éviter le "Flash of Unstyled Text" en téléchargeant la police avant que le CSS ne soit complètement analysé.

### 3. Comment

#### A. Précharger une police critique

```tsx
import { preload } from 'react-dom';

function App() {
  // On veut cette police tout de suite, avec une haute priorité
  preload("https://fonts.example.com/my-font.woff2", { 
    as: "font", 
    crossOrigin: "anonymous" // Souvent requis pour les fonts
  });

  return <div style={{ fontFamily: 'MyFont' }}>Texte important</div>;
}
```

#### B. Précharger un module JS (Lazy Loading manuel)

```tsx
import { preloadModule } from 'react-dom';

function Dashboard() {
  // On commence à télécharger le gros fichier JS des graphiques
  // pendant que l'utilisateur lit le tableau de bord.
  // Quand il cliquera sur "Voir Graphiques", le fichier sera déjà là.
  preloadModule("/assets/charts-library.js", { as: "script" });

  return (
    <div>
      <h1>Tableau de bord</h1>
      {/* ... */}
    </div>
  );
}
```

### 🚨 Limitations de `preload`
Ne préchargez que ce qui sera utilisé **sur la page courante**. Si vous préchargez une ressource que le navigateur n'utilise pas dans les 3 secondes, Chrome affichera un avertissement dans la console ("The resource ... was preloaded but not used").

---

## 3. `preinit` et `preinitModule` : Charger et Exécuter {#preinit-et-preinit-module}

### 1. Quoi
Ces fonctions vont plus loin que `preload` : elles téléchargent la ressource **ET** l'injectent/exécutent immédiatement dans la page.

*   Pour un **Script** : Il est téléchargé et exécuté.
*   Pour un **Style** : Il est téléchargé et inséré dans le DOM (les règles CSS s'appliquent).

### 2. Pourquoi
C'est l'équivalent moderne d'ajouter dynamiquement une balise `<script src="...">` ou `<link rel="stylesheet">`.
React gère la **dé-duplication** : même si 10 composants appellent `preinit("style.css")`, React n'insérera la balise qu'une seule fois.

### 3. Comment

#### A. Charger un Script Tiers (ex: Google Maps)

```tsx
import { preinit } from 'react-dom';

function MapComponent() {
  // Charge et exécute le script immédiatement
  preinit("https://maps.googleapis.com/maps/api/js?key=MY_KEY", { 
    as: "script",
    priority: "low" // On peut définir la priorité
  });

  return <div id="map">La carte s'affichera ici</div>;
}
```

#### B. Charger une feuille de style CSS critique

```tsx
import { preinit } from 'react-dom';

function ThemeDark() {
  // Applique le thème sombre dès le rendu de ce composant
  preinit("/themes/dark-mode.css", { 
    as: "style", 
    precedence: "high" // Indique à React comment ordonner ce style
  });

  return <div className="dark-theme">Mode Sombre Activé</div>;
}
```

### 4. Tableau comparatif

| Fonction | Action Réseau | Exécution/Application ? | Cas d'usage typique |
| :--- | :--- | :--- | :--- |
| `prefetchDNS` | Resolve IP | Non | API externe probable |
| `preconnect` | Handshake TCP/TLS | Non | API critique (Stripe, Auth) |
| `preload` | Téléchargement complet | Non (Cache uniquement) | Fonts, Images LCP |
| `preinit` | Téléchargement complet | **Oui** (Immédiat) | CSS critique, Lib JS externe |

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-48}

1.  **Quelle est la différence fondamentale entre `preload` et `preinit` ?**
    `preload` télécharge la ressource et la garde en mémoire cache sans l'utiliser (pour un usage futur proche), tandis que `preinit` télécharge la ressource et l'applique/exécute immédiatement dans la page (injecte le CSS ou exécute le JS).

2.  **Si j'appelle `preconnect("https://api.xyz.com")` dans 5 composants différents affichés en même temps, combien de connexions le navigateur va-t-il ouvrir ?**
    Une seule (ou le nombre optimal géré par le navigateur). React dé-duplique automatiquement ces appels, donc il est sûr d'appeler ces fonctions au niveau du composant qui en a besoin.

3.  **Pourquoi `prefetchDNS` est-il moins coûteux que `preconnect` ?**
    `prefetchDNS` effectue juste une résolution de nom (quelques octets UDP), alors que `preconnect` ouvre une socket TCP réelle et effectue la négociation SSL/TLS, ce qui consomme de la mémoire et du CPU sur le client et le serveur.

---

## Exercices : {#exercices-48}

### Exercice 1 - Optimisation Checkout (DNS/Connect) {#exercice-1---optimisation-checkout}

🎯 **Objectif** : Réduire le temps de latence au moment du paiement.

💼 **Mise en situation** : Vous développez un panier d'achat. Lorsque l'utilisateur arrive sur le récapitulatif, il est très probable qu'il clique sur "Payer". Le script de paiement est hébergé sur `secure.payment-provider.com`.

📝 **Énoncé** :
1. Créez un composant `CartSummary`.
2. Utilisez `prefetchDNS` pour le domaine du CDN d'images (`cdn.shop.com`).
3. Utilisez `preconnect` pour le domaine de paiement (`secure.payment-provider.com`).
4. Affichez un simple bouton "Payer".

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { preconnect, prefetchDNS } from 'react-dom';

export function CartSummary() {
  // 1. On résout juste l'IP des images pour les futurs affichages
  // C'est léger, on peut le faire sans hésiter.
  prefetchDNS("https://cdn.shop.com");

  // 2. On établit une connexion complète (TCP+TLS) vers le provider de paiement
  // C'est plus lourd, mais critique pour que le clic sur "Payer" soit réactif.
  preconnect("https://secure.payment-provider.com");

  return (
    <div className="p-4 border rounded">
      <h2>Récapitulatif de la commande</h2>
      <p>Total: 150.00 €</p>
      
      <button 
        className="bg-blue-500 text-white p-2 rounded mt-2"
        onClick={() => console.log("Lancement du script de paiement...")}
      >
        Procéder au paiement
      </button>
    </div>
  );
}
```
</details>

### Exercice 2 - Chargement de Police (`preload`) {#exercice-2---chargement-de-police}

🎯 **Objectif** : Éviter le saut visuel (FOUT) sur un titre stylisé.

💼 **Mise en situation** : Votre page de "Promo de Noël" utilise une police spécifique très lourde. Vous voulez qu'elle commence à charger dès le rendu du composant, avant même que le navigateur ne parse le CSS qui la demande.

📝 **Énoncé** :
1. Créez un composant `ChristmasSale`.
2. Utilisez `preload` pour charger `christmas-font.woff2` en tant que `font`.
3. Assurez-vous de passer l'option `crossOrigin` (nécessaire pour les polices).

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { preload } from 'react-dom';

export function ChristmasSale() {
  // React va injecter <link rel="preload" href="..." as="font" ... /> dans le head
  preload("/fonts/christmas-font.woff2", {
    as: "font",
    crossOrigin: "anonymous" // Obligatoire pour les polices Web
  });

  return (
    <div className="sale-banner">
      {/* Supposons que ce style utilise fontFamily: 'ChristmasFont' */}
      <h1 style={{ fontFamily: 'ChristmasFont, serif' }}>
        🎄 PROMO SPÉCIALE -50% 🎄
      </h1>
    </div>
  );
}
```
</details>

### Exercice 3 - Injection de Script Tiers (`preinit`) {#exercice-3---injection-de-script-tiers}

🎯 **Objectif** : Charger un widget de chat support uniquement si l'utilisateur en a besoin.

💼 **Mise en situation** : Pour ne pas ralentir le site, le script lourd du Chatbot (`chat-widget.js`) ne doit être chargé que si l'utilisateur clique sur le bouton "Aide".

📝 **Énoncé** :
1. Créez un composant `HelpButton`.
2. Au clic sur le bouton, appelez une fonction qui déclenche `preinit` pour charger et exécuter `https://widget.chat.com/sdk.js`.
3. Simulez l'utilisation de l'API globale du chat une fois chargée (ex: `window.Chat.open()`) dans un `onload` ou juste après (attention à l'asynchronisme).

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { preinit } from 'react-dom';
import { useState } from 'react';

export function HelpButton() {
  const [isLoading, setIsLoading] = useState(false);

  const loadChat = () => {
    setIsLoading(true);

    // preinit injecte <script src="..." async> et l'exécute
    preinit("https://widget.chat.com/sdk.js", {
      as: "script",
      priority: "medium"
    });

    // Note : Dans un vrai cas, on écouterait l'événement 'load' du script 
    // ou on utiliserait un mécanisme de callback fourni par le SDK du chat.
    console.log("Script injecté !");
  };

  return (
    <button onClick={loadChat} disabled={isLoading}>
      {isLoading ? "Chargement du chat..." : "💬 Besoin d'aide ?"}
    </button>
  );
}
```
</details>
```