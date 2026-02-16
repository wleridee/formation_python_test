Voici le chapitre **API ReactDOM Client: `createRoot`, `hydrateRoot`** pour la formation React 19.2.

```markdown
---
sidebar_label: API ReactDOM Client: `createRoot`, `hydrateRoot`
sidebar_position: 45
---

# Chapitre 45 : API ReactDOM Client: `createRoot`, `hydrateRoot`

Montage d'applications React, Hydratation du SSR, Rendu côté client

Dans 99% des tutoriels, vous voyez ces fonctions à la première ligne du fichier `index.tsx` ou `main.tsx`, puis vous les oubliez. Pourtant, `createRoot` et `hydrateRoot` sont les portes d'entrée fondamentales de React dans le navigateur.

Depuis React 18 (et confirmé en 19), ces API situées dans `react-dom/client` sont obligatoires pour activer les fonctionnalités concurrentes (Concurrent Features) comme le batching automatique, les transitions et le Suspense.

---

## 1. `createRoot` {#create-root}

### 1. Quoi
`createRoot` est la fonction qui permet d'initialiser une application React dans un environnement **Client-Side Rendering (CSR)** pur. Elle prend un conteneur DOM existant (généralement vide) et renvoie un objet "Root" qui possède une méthode `render` pour afficher l'interface.

### 2. Pourquoi
Avant React 18, nous utilisions `ReactDOM.render`. Cette ancienne API fonctionnait en mode "Legacy", bloquant le thread principal lors des grosses mises à jour.
`createRoot` active le **Concurrent Mode** de React. Cela permet à React de :
*   Mettre en pause le rendu pour laisser le navigateur respirer (input utilisateur).
*   Préparer plusieurs versions de l'UI en arrière-plan.
*   Grouper automatiquement les mises à jour d'état (Automatic Batching).

### 3. Comment

#### A. Syntaxe de base

```tsx
import { createRoot } from 'react-dom/client';
import App from './App';

const container = document.getElementById('root');

// Vérification TypeScript stricte
if (container) {
  const root = createRoot(container);
  root.render(<App />);
}
```

#### B. Cas concret : Initialisation Robuste

Dans une application réelle, on s'assure souvent de nettoyer ou de gérer les erreurs d'initialisation.

```tsx
import { createRoot } from 'react-dom/client';
import { StrictMode } from 'react';
import App from './App';
import './index.css';

// 1. Sélection du nœud DOM
const domNode = document.getElementById('root');

if (!domNode) {
  throw new Error("Impossible de trouver l'élément #root. Vérifiez votre index.html");
}

// 2. Création de la racine React
const root = createRoot(domNode);

// 3. Rendu initial (montage)
root.render(
  <StrictMode>
    <App />
  </StrictMode>
);

/* 
   Note : root.unmount() existe aussi, 
   utile principalement pour les micro-frontends 
   ou les tests pour nettoyer la mémoire.
*/
```

### 4. Zone de Danger

:::danger Ne jamais faire
N'appelez jamais `createRoot` **à l'intérieur** d'un composant React. Cela recréerait une application entière à chaque rendu, détruisant tout l'état et le focus.
:::

```tsx
// ❌ HORRIBLE : Fuite de mémoire et perte d'état garanties
function BadComponent() {
  const container = useRef(null);
  
  useEffect(() => {
    // Ne faites pas ça !
    const root = createRoot(container.current); 
    root.render(<Child />);
  });
  
  return <div ref={container} />;
}
```

---

## 2. `hydrateRoot` {#hydrate-root}

### 1. Quoi
`hydrateRoot` est l'équivalent de `createRoot`, mais pour les applications rendues côté serveur (**SSR** - Server-Side Rendering) ou générées statiquement (**SSG**).

Au lieu de créer des nœuds DOM à partir de zéro, React va "s'attacher" au HTML existant (généré par le serveur) et y ajouter les écouteurs d'événements (click, submit, etc.). On appelle ce processus l'**Hydratation**.

### 2. Pourquoi
*   **Performance (FCP/LCP)** : L'utilisateur voit le contenu immédiatement (HTML statique) sans attendre que le JS se charge.
*   **SEO** : Les moteurs de recherche lisent le HTML complet.
*   `hydrateRoot` permet de transformer ce HTML inerte en application interactive.

### 3. Comment

#### A. Syntaxe de base

```tsx
import { hydrateRoot } from 'react-dom/client';
import App from './App';

// Le HTML dans 'root' doit correspondre EXACTEMENT au rendu de <App />
hydrateRoot(
  document.getElementById('root')!,
  <App />
);
```

#### B. Cas concret : Gestion des Mismatches (Différences Client/Serveur)

L'hydratation est fragile. Si le HTML du serveur diffère de ce que React génère sur le client (ex: un timestamp), React lancera un avertissement "Hydration Mismatch".

```tsx
import { hydrateRoot } from 'react-dom/client';
import App from './App';

const container = document.getElementById('root');

if (container) {
  // hydrateRoot renvoie aussi un objet root, mais on l'utilise rarement directement
  const root = hydrateRoot(container, <App />);
  
  // En dev, React affichera une erreur console si le HTML
  // serveur ne correspond pas au HTML client.
}
```

### 🚨 Limitations de l'Hydratation
Si React détecte une différence trop importante (ex: structure DOM différente), il abandonnera l'hydratation et refera un rendu complet (comme `createRoot`), annulant les bénéfices de performance du SSR. C'est ce qu'on appelle un "Hydration Bailout".

Les causes fréquentes d'erreurs d'hydratation :
1.  Utiliser `Math.random()`, `Date.now()` ou `new Date()` lors du rendu initial.
2.  Le HTML invalide (ex: un `<div>` dans un `<p>`) corrigé automatiquement par le navigateur mais pas par React.
3.  Des extensions de navigateur qui injectent du code dans la page avant l'hydratation.

Pour corriger les dates, utilisez un effet :

```tsx
function Timestamp() {
  const [isClient, setIsClient] = useState(false);

  useEffect(() => {
    setIsClient(true); // Ne s'exécute que sur le client
  }, []);

  return (
    <span>
      {isClient ? new Date().toLocaleTimeString() : 'Chargement...'}
    </span>
  );
}
```

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-45}

1.  **Quelle est la différence fondamentale entre `createRoot` et `hydrateRoot` ?**
    `createRoot` supprime tout contenu existant dans le conteneur et crée le DOM à partir de zéro (CSR). `hydrateRoot` préserve le DOM existant généré par le serveur et y attache simplement des écouteurs d'événements (SSR).

2.  **Pourquoi `ReactDOM.render` (l'ancienne API) est-il déconseillé en React 19 ?**
    Parce qu'il fonctionne en mode "Legacy" synchrone et bloquant. Il ne supporte pas les fonctionnalités modernes de React comme le Suspense, les Transitions ou le Streaming SSR.

3.  **Que se passe-t-il si le HTML du serveur et le rendu initial du client sont différents lors de l'utilisation de `hydrateRoot` ?**
    React émet un avertissement "Hydration failed" (mismatch). Il tente de corriger le DOM pour correspondre au client, ce qui peut coûter cher en performance et causer des sauts visuels.

---

## Exercices : {#exercices-45}

### Exercice 1 - Le Montage SPA Classique {#exercice-1---le-montage-spa-classique}

🎯 **Objectif** : Configurer proprement le point d'entrée d'une Single Page Application.

💼 **Mise en situation** : Vous configurez un nouveau projet React "from scratch" sans framework. Vous devez attacher l'application au DOM.

📝 **Énoncé** :
1. Créez un fichier HTML simulé avec une div `<div id="app-mount"></div>`.
2. Créez un composant `HelloWorld` simple.
3. Écrivez le code TypeScript pour monter l'application.
4. Gérez le cas où l'élément `#app-mount` n'existe pas (erreur explicite).

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { createRoot } from 'react-dom/client';
import { StrictMode } from 'react';

function HelloWorld() {
  return <h1>Hello React 19! 🚀</h1>;
}

// Simulation du DOM (normalement dans index.html)
// document.body.innerHTML = '<div id="app-mount"></div>';

function mountApplication() {
  // 1. Récupération sécurisée
  const container = document.getElementById('app-mount');

  // 2. Gestion d'erreur critique
  if (!container) {
    console.error("ERREUR FATALE: L'élément #app-mount est introuvable.");
    return;
  }

  // 3. Création et rendu
  const root = createRoot(container);
  
  root.render(
    <StrictMode>
      <HelloWorld />
    </StrictMode>
  );
  
  console.log("Application montée avec succès en mode CSR.");
}

// Lancer le montage
mountApplication();
```
</details>

### Exercice 2 - Simulation d'Hydratation {#exercice-2---simulation-d-hydratation}

🎯 **Objectif** : Comprendre comment `hydrateRoot` reprend la main sur du HTML existant.

💼 **Mise en situation** : Votre serveur a déjà envoyé le HTML d'un bouton. Vous devez le rendre cliquable sans le recréer visuellement (pas de clignotement).

📝 **Énoncé** :
1. Simulez du HTML serveur : mettez `<button id="btn">0 clics</button>` dans le `body` (via `innerHTML` pour l'exercice).
2. Créez un composant `Counter` qui initialise son état à 0 et affiche le même bouton.
3. Utilisez `hydrateRoot` pour attacher React.
4. Vérifiez que le clic fonctionne.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Console du navigateur.
> **Annotation** : Montrez qu'il n'y a pas d'erreur d'hydratation.
> **Alt Text suggéré** : Console propre après hydratation React.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState } from 'react';
import { hydrateRoot } from 'react-dom/client';

function Counter() {
  const [count, setCount] = useState(0);
  // Le JSX doit correspondre EXACTEMENT au HTML initial du serveur
  return (
    <button id="btn" onClick={() => setCount(c => c + 1)}>
      {count} clics
    </button>
  );
}

// 1. Simulation : Le serveur a envoyé ça
document.body.innerHTML = '<div id="root"><button id="btn">0 clics</button></div>';

// 2. Hydratation côté client
const container = document.getElementById('root');

if (container) {
  console.log("Début de l'hydratation...");
  
  // React va voir que le <button> est déjà là et juste ajouter le listener onClick
  hydrateRoot(container, <Counter />);
  
  console.log("Hydratation terminée. Le bouton est interactif.");
}
```
</details>

### Exercice 3 - Le Piège du Mismatch (Date) {#exercice-3---le-piege-du-mismatch}

🎯 **Objectif** : Identifier et corriger une erreur d'hydratation classique.

💼 **Mise en situation** : Vous voulez afficher la date du jour dans le footer. Mais le serveur a généré la page à 10h00, et le client l'affiche à 10h01. `hydrateRoot` va crier.

📝 **Énoncé** :
1. Créez un composant `Footer` qui affiche `new Date().toLocaleTimeString()`.
2. Simulez un HTML serveur avec une heure fixe "10:00:00".
3. Tentez d'hydrater -> Erreur console (Mismatch).
4. **Correction** : Utilisez l'attribut `suppressHydrationWarning` sur la balise HTML concernée pour dire à React "C'est normal, ignore la différence".

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { hydrateRoot } from 'react-dom/client';

function Footer() {
  const time = new Date().toLocaleTimeString();

  return (
    <footer>
      Heure de génération : 
      {/* 
         ✅ SOLUTION : suppressHydrationWarning
         Indispensable pour les dates ou les ID générés aléatoirement 
         qui diffèrent entre serveur et client.
         Attention : ne fonctionne qu'à un seul niveau de profondeur.
      */}
      <span suppressHydrationWarning={true}>
        {time}
      </span>
    </footer>
  );
}

// HTML Serveur (généré il y a quelques secondes/minutes)
document.body.innerHTML = `
  <div id="root">
    <footer>
      Heure de génération : <span>10:00:00</span>
    </footer>
  </div>
`;

const container = document.getElementById('root');
if (container) {
  // Sans le suppressHydrationWarning, React afficherait :
  // "Warning: Text content did not match. Server: "10:00:00" Client: "10:05:32""
  hydrateRoot(container, <Footer />);
}
```
</details>
```