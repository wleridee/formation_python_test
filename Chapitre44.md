Voici le chapitre **API ReactDOM: `createPortal`, `flushSync`** pour la formation React 19.2.

```markdown
---
sidebar_label: API ReactDOM: `createPortal`, `flushSync`
sidebar_position: 44
---

# Chapitre 44 : API ReactDOM: `createPortal`, `flushSync`

Rendu hors de la hiérarchie DOM, Mises à jour synchrones forcées, Gestion du DOM

La plupart du temps, vous utilisez React sans toucher directement au DOM. Vous retournez du JSX, et React s'occupe de mettre à jour la page. Cependant, pour des cas d'utilisation avancés comme les **modales**, les **tooltips** ou des mesures **immédiates** après une mise à jour, les règles standards de la hiérarchie des composants et du "batching" (regroupement des mises à jour) peuvent devenir limitantes.

Ce chapitre explore deux fonctions essentielles de `react-dom` pour contourner ces limitations : `createPortal` pour la structure, et `flushSync` pour le timing.

---

## 1. `createPortal` {#create-portal}

### 1. Quoi
`createPortal` permet de rendre des enfants (JSX) dans un nœud DOM qui existe **en dehors** de la hiérarchie DOM du composant parent.

C'est une sorte de "téléportation" visuelle : le composant reste logiquement dans l'arbre React (pour les props, le contexte, les événements), mais ses éléments HTML atterrissent ailleurs dans la page (souvent à la fin de `<body>`).

Signature :
```tsx
createPortal(children, domNode, key?)
```

### 2. Pourquoi
En CSS, certains styles comme `overflow: hidden` ou `z-index` sont contraints par le conteneur parent.
Imaginez une **Modale** ou une **Tooltip** à l'intérieur d'une barre latérale (`<Sidebar>`) qui a `overflow: hidden`. Si vous rendez la modale normalement, elle sera coupée par la sidebar.
Avec un portail, vous pouvez "sortir" la modale pour l'attacher directement au `<body>`, ce qui résout instantanément les problèmes de superposition et de découpage CSS.

### 3. Comment

#### A. Syntaxe de base

```tsx
import { createPortal } from 'react-dom';

function MyPortal() {
  return createPortal(
    <div className="modal">Je suis téléporté !</div>,
    document.body // Destination
  );
}
```

#### B. Cas concret : Composant Modale Réutilisable

```tsx
import { useEffect, useState, ReactNode } from 'react';
import { createPortal } from 'react-dom';

interface ModalProps {
  isOpen: boolean;
  onClose: () => void;
  children: ReactNode;
}

export function Modal({ isOpen, onClose, children }: ModalProps) {
  // On ne rend rien si la modale est fermée
  if (!isOpen) return null;

  return createPortal(
    <div className="modal-overlay" onClick={onClose}>
      <div 
        className="modal-content" 
        onClick={e => e.stopPropagation()} // Empêche la fermeture si on clique DANS la modale
      >
        <button className="close-btn" onClick={onClose}>×</button>
        {children}
      </div>
    </div>,
    document.body // On s'attache directement au body
  );
}

// Exemple d'utilisation
export function App() {
  const [show, setShow] = useState(false);
  
  return (
    <div style={{ overflow: 'hidden', height: 100, border: '1px solid red' }}>
      <p>Conteneur restreint</p>
      <button onClick={() => setShow(true)}>Ouvrir Modale</button>
      
      {/* 
        Bien que placée ici dans le JSX, la modale sera rendue dans le body.
        Elle ne sera donc pas coupée par le overflow: hidden du parent.
      */}
      <Modal isOpen={show} onClose={() => setShow(false)}>
        <h2>Je suis libre !</h2>
        <p>Je suis par-dessus tout le reste.</p>
      </Modal>
    </div>
  );
}
```

### 4. Zone de Danger : La Propagation des Événements (Event Bubbling)

C'est une spécificité unique de React : un événement déclenché dans un portail **remonte** (bubble) dans l'arbre **React**, même s'il ne remonte pas dans l'arbre DOM natif.

```tsx
<div onClick={() => console.log("Clic attrapé !")}>
  <Portal>
    <button>Cliquez-moi</button>
  </Portal>
</div>
```
Si vous cliquez sur le bouton (qui est physiquement dans `<body>`), le `div` parent (qui est ailleurs dans le DOM) attrapera quand même le clic ! C'est généralement ce qu'on veut, mais cela peut surprendre.

---

## 2. `flushSync` {#flush-sync}

### 1. Quoi
`flushSync` force React à exécuter les mises à jour d'état en attente (« pending state updates ») **de manière synchrone, immédiatement**, avant que le code suivant ne s'exécute.

### 2. Pourquoi
Depuis React 18, les mises à jour d'état sont groupées (**Automatic Batching**) pour la performance. Si vous faites 3 `setState`, React attend un peu et ne fait qu'un seul rendu.
Cependant, parfois, vous devez interagir avec le DOM **immédiatement** après un changement d'état, sans attendre que React ait fini son cycle optimisé.
Cas typiques :
*   Focus sur un input juste après l'avoir affiché.
*   Scroller tout en bas d'une liste de chat juste après l'ajout d'un message.
*   Imprimer la page (`window.print()`) juste après avoir changé son contenu.

### 3. Comment

#### A. Syntaxe de base

```tsx
import { flushSync } from 'react-dom';

function handleClick() {
  flushSync(() => {
    setCount(c => c + 1);
  });
  // À cette ligne précise, le DOM est DÉJÀ mis à jour.
  console.log(document.getElementById('counter')?.textContent);
}
```

#### B. Cas concret : Scroll automatique (Chat)

```tsx
import { useState, useRef, useEffect } from 'react';
import { flushSync } from 'react-dom';

export function ChatRoom() {
  const [messages, setMessages] = useState<string[]>([]);
  const listRef = useRef<ul | null>(null);

  const addMessage = () => {
    const newMessage = `Message ${messages.length + 1}`;
    
    // 1. On force la mise à jour du DOM tout de suite
    flushSync(() => {
      setMessages(prev => [...prev, newMessage]);
    });

    // 2. Comme le DOM est à jour, le nouvel élément <li> existe déjà.
    // On peut scroller vers le bas immédiatement.
    if (listRef.current) {
      listRef.current.scrollTop = listRef.current.scrollHeight;
    }
  };

  return (
    <div>
      <ul ref={listRef} style={{ height: 100, overflow: 'auto', border: '1px solid gray' }}>
        {messages.map((m, i) => <li key={i}>{m}</li>)}
      </ul>
      <button onClick={addMessage}>Envoyer</button>
    </div>
  );
}
```

### 🚨 Limitations de `flushSync`
❌ **Performance** : `flushSync` casse le batching optimisé de React. Utiliser `flushSync` trop souvent peut ralentir votre application.
❌ **Boundary** : `flushSync` à l'intérieur d'un événement peut parfois forcer le rendu des Suspense boundaries en fallback.

Utilisez-le uniquement en dernier recours quand `useEffect` ou `useLayoutEffect` ne suffisent pas (souvent pour interagir avec des API impératives du navigateur).

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-44}

1.  **Si je rends un composant via `createPortal` dans le `<body>`, où se propage l'événement `onClick` déclenché sur ce composant ?**
    L'événement remonte dans l'arbre des composants React (vers le composant qui a appelé `createPortal`), et non simplement vers le parent DOM direct (`<body>`). Cela respecte la hiérarchie logique de votre application.

2.  **Pourquoi utiliser `createPortal` pour une modale plutôt que de la rendre directement dans le flux ?**
    Pour éviter les problèmes de CSS comme `overflow: hidden`, `z-index` ou les transformations CSS sur les parents qui pourraient couper la modale ou altérer son positionnement par rapport à la fenêtre.

3.  **Quelle est la conséquence principale de l'utilisation de `flushSync` sur les performances ?**
    Il désactive le "batching" automatique de React et force un cycle de rendu immédiat et synchrone, ce qui peut causer des saccades si utilisé intensivement.

---

## Exercices : {#exercices-44}

### Exercice 1 - La Tooltip Évadée (`createPortal`) {#exercice-1---la-tooltip-evadee}

🎯 **Objectif** : Créer une tooltip qui n'est pas coupée par son conteneur.

💼 **Mise en situation** : Vous avez une liste de cartes produits dans un carrousel avec `overflow: hidden`. Au survol d'un bouton "Info", une bulle d'aide doit s'afficher. Si vous ne sortez pas du flux, la bulle sera coupée.

📝 **Énoncé** :
1. Créez un conteneur avec `style={{ overflow: 'hidden', height: 100, border: '2px solid red' }}`.
2. À l'intérieur, placez un bouton près du bord.
3. Créez un composant `Tooltip` qui utilise `createPortal` pour s'afficher dans `document.body`.
4. Positionnez la tooltip de manière absolue (simulé, ex: `top: 50px, left: 50px`) pour qu'elle s'affiche par-dessus la bordure rouge.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Tooltip par-dessus une bordure.
> **Annotation** : Montrez que la tooltip dépasse physiquement du cadre rouge du parent.
> **Alt Text suggéré** : Tooltip React via Portal dépassant d'un conteneur overflow hidden.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState } from 'react';
import { createPortal } from 'react-dom';

function Tooltip({ text }: { text: string }) {
  // On rend la tooltip directement dans le body
  return createPortal(
    <div style={{
      position: 'fixed',
      top: '50%', // Position fixe pour l'exemple
      left: '50%',
      transform: 'translate(-50%, -50%)',
      backgroundColor: 'black',
      color: 'white',
      padding: '10px',
      borderRadius: '4px',
      zIndex: 9999
    }}>
      {text}
    </div>,
    document.body
  );
}

export function App() {
  const [show, setShow] = useState(false);

  return (
    <div style={{ padding: 50 }}>
      {/* Ce conteneur couperait la tooltip sans le portail */}
      <div style={{ 
        overflow: 'hidden', 
        height: 60, 
        border: '3px solid red',
        background: '#eee',
        position: 'relative'
      }}>
        <p>Conteneur Restreint</p>
        <button 
          onMouseEnter={() => setShow(true)}
          onMouseLeave={() => setShow(false)}
        >
          Survolez-moi
        </button>
        
        {/* La tooltip est déclarée ici, mais rendue ailleurs */}
        {show && <Tooltip text="Je ne suis pas coupé ! 🚀" />}
      </div>
    </div>
  );
}
```
</details>

### Exercice 2 - L'Espion de Clic (Bubbling) {#exercice-2---l-espion-de-clic}

🎯 **Objectif** : Prouver que les événements traversent les portails.

💼 **Mise en situation** : Vous voulez tracker tous les clics dans une zone "Admin", y compris ceux effectués dans des modales qui sont techniquement hors de cette zone dans le DOM.

📝 **Énoncé** :
1. Créez une `div` "Zone Admin" avec un gestionnaire `onClick` qui incrémente un compteur "Clics Admin".
2. À l'intérieur, placez un bouton normal.
3. À l'intérieur aussi, placez un composant `PortalButton` qui se rend dans `document.body`.
4. Cliquez sur le bouton "portalisé" et vérifiez que le compteur "Clics Admin" augmente.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState } from 'react';
import { createPortal } from 'react-dom';

export function AdminArea() {
  const [clicks, setClicks] = useState(0);

  return (
    <div 
      onClick={() => setClicks(c => c + 1)} // Ce handler attrape TOUT
      style={{ padding: 20, background: '#e0f7fa', border: '1px solid blue' }}
    >
      <h3>Zone Admin (Clics détectés : {clicks})</h3>
      
      <button>Bouton Normal (Interne)</button>

      {/* Le portail */}
      {createPortal(
        <div style={{ 
          marginTop: 20, 
          padding: 10, 
          border: '1px dashed black',
          background: 'white' 
        }}>
          <p>Je suis physiquement dans le body...</p>
          <button>...mais mon clic remonte vers la Zone Admin !</button>
        </div>,
        document.body
      )}
    </div>
  );
}
```
</details>

### Exercice 3 - Impression Immédiate (`flushSync`) {#exercice-3---impression-immediate}

🎯 **Objectif** : Comprendre la nécessité d'une mise à jour synchrone.

💼 **Mise en situation** : Vous avez un bouton "Imprimer". Lorsque l'utilisateur clique, vous voulez changer l'état pour passer en "Mode Impression" (simplifier l'UI) et lancer l'impression navigateur **immédiatement**.

📝 **Énoncé** :
1. Un état `isPrinting` (booléen).
2. Si `isPrinting` est vrai, affichez "VERSION IMPRIMABLE PROPRE", sinon "VERSION ÉCRAN COMPLEXE".
3. Un bouton "Imprimer".
4. Dans le handler :
   - Utilisez `flushSync` pour mettre `isPrinting` à true.
   - Appelez `window.print()`.
   - Remettez `isPrinting` à false (standard).
5. Sans `flushSync`, le navigateur imprimerait la version "COMPLEXE" car le rendu React n'aurait pas encore eu lieu.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState } from 'react';
import { flushSync } from 'react-dom';

export function Invoice() {
  const [isPrinting, setIsPrinting] = useState(false);

  const handlePrint = () => {
    // 1. On force React à mettre à jour le DOM MAINTENANT
    flushSync(() => {
      setIsPrinting(true);
    });
    
    // 2. Le DOM est à jour, on peut lancer l'impression native
    // Le navigateur verra "VERSION IMPRIMABLE"
    window.print();

    // 3. On remet l'état normal (pas besoin de flushSync ici, le batching est OK)
    setIsPrinting(false);
  };

  return (
    <div style={{ border: '1px solid #ccc', padding: 20 }}>
      <h1>Facture #001</h1>
      
      <div style={{ 
        background: isPrinting ? 'white' : '#f0f0f0', 
        padding: 20 
      }}>
        {isPrinting ? (
          <h2>📄 VERSION IMPRIMABLE (Noir & Blanc)</h2>
        ) : (
          <h2>💻 VERSION ÉCRAN (Avec animations, pubs, couleurs)</h2>
        )}
      </div>

      {!isPrinting && (
        <button onClick={handlePrint} style={{ marginTop: 20 }}>
          🖨️ Imprimer maintenant
        </button>
      )}
    </div>
  );
}
```
</details>
```