Voici le chapitre **`useLayoutEffect`: Effets après le DOM** pour la formation React 19.2.

```markdown
---
sidebar_label: `useLayoutEffect`: Effets après le DOM
sidebar_position: 25
---

# Chapitre 25 : `useLayoutEffect`: Effets après le DOM

Différences avec `useEffect`, Mesures de layout, Mises à jour synchrones

Dans 99% des cas, `useEffect` est le bon Hook pour gérer les effets secondaires. Il s'exécute après que React a dessiné l'écran, permettant à l'utilisateur de voir l'interface immédiatement.
Mais parfois, vous devez mesurer la taille d'un élément *avant* que l'utilisateur ne le voie, ou synchroniser une animation très précisément.
C'est là qu'intervient `useLayoutEffect`.

## `useEffect` vs `useLayoutEffect` {#useeffect-vs-uselayouteffect}

### 1. Quoi
`useLayoutEffect` a exactement la même signature (syntaxe) que `useEffect`, mais il s'exécute à un moment différent.
*   `useEffect` : S'exécute **asynchrone**, *après* que le navigateur a peint l'écran (paint).
*   `useLayoutEffect` : S'exécute **synchrone**, *après* que React a mis à jour le DOM mais *avant* que le navigateur ne peigne l'écran.

### 2. Pourquoi
Si votre effet modifie l'apparence du DOM (ex: déplacer une info-bulle, redimensionner une div), le faire dans `useEffect` peut causer un **clignotement** (flicker). L'utilisateur voit l'élément à la position A, puis il saute instantanément à la position B.
`useLayoutEffect` bloque l'affichage visuel jusqu'à ce que votre code soit terminé, garantissant que l'utilisateur ne voit que la version finale (position B).

### 3. Comment

#### A. Syntaxe de base

```tsx
import { useLayoutEffect, useRef } from 'react';

function MyComponent() {
  const ref = useRef(null);

  useLayoutEffect(() => {
    // Ce code s'exécute avant que l'utilisateur ne voie le composant
    // Idéal pour mesurer le DOM
    const { height } = ref.current.getBoundingClientRect();
    console.log(height);
  }, []);

  return <div ref={ref}>Hello</div>;
}
```

#### B. Tableau Comparatif

| Caractéristique | `useEffect` | `useLayoutEffect` |
| :--- | :--- | :--- |
| **Moment d'exécution** | Après le Paint (Affichage) | Avant le Paint |
| **Blocage du rendu** | Non (l'UI reste fluide) | Oui (l'UI fige tant que l'effet n'est pas fini) |
| **Cas d'usage** | Fetching API, abonnements, logs | Mesures DOM, Tooltips, Animations synchrones |
| **Fréquence d'usage** | 99% des cas | < 1% des cas (Optimisation visuelle) |

---

## Mesures de Layout et Tooltips {#mesures-de-layout-et-tooltips}

### 1. Quoi
Positionner un élément flottant (comme une info-bulle ou "Tooltip") à côté d'un bouton nécessite de connaître la taille exacte de l'info-bulle. Or, vous ne pouvez connaître sa taille qu'après l'avoir rendue dans le DOM.

### 2. Pourquoi
Si vous affichez le Tooltip, puis mesurez sa taille, puis corrigez sa position avec `useEffect`, l'utilisateur verra le Tooltip "sauter" d'un endroit à un autre.

### 3. Comment

```tsx
import { useState, useRef, useLayoutEffect } from 'react';

export function Tooltip({ children, targetRect }) {
  const ref = useRef<HTMLDivElement>(null);
  const [tooltipHeight, setTooltipHeight] = useState(0);

  // 🚨 Avec useEffect, le tooltip s'afficherait mal positionné 
  // pendant une fraction de seconde avant de se corriger.
  useLayoutEffect(() => {
    const { height } = ref.current.getBoundingClientRect();
    setTooltipHeight(height);
    // On peut maintenant ajuster la position avant le premier paint
  }, []);

  let tooltipY = 0;
  if (targetRect !== null) {
    tooltipY = targetRect.top - tooltipHeight;
    if (tooltipY < 0) {
      // Si ça dépasse en haut, on le met en dessous
      tooltipY = targetRect.bottom;
    }
  }

  return (
    <div 
      ref={ref} 
      style={{ 
        position: 'absolute', 
        top: tooltipY, 
        left: targetRect?.left || 0,
        background: 'black',
        color: 'white'
      }}
    >
      {children}
    </div>
  );
}
```

### 4. Zone de Danger

:::danger Attention aux performances
`useLayoutEffect` est synchrone. Tant que votre effet tourne, le navigateur ne peut rien afficher.
Si vous faites un calcul lourd (ex: boucle de 100ms) dans `useLayoutEffect`, votre application semblera "freezer" ou lagger.
**Règle** : Utilisez toujours `useEffect` par défaut. Ne passez à `useLayoutEffect` que si vous observez un clignotement visuel.
:::

---

## Mises à jour Synchrones {#mises-a-jour-synchrones}

### 1. Quoi
Parfois, vous devez forcer une mise à jour de l'état "immédiatement" après une modification du DOM, avant que l'écran ne se rafraîchisse.

### 2. Pourquoi
Imaginez un carrousel infini virtuel. Quand vous arrivez à la fin, vous voulez téléporter l'utilisateur au début sans qu'il ne s'en rende compte (pas de scroll visible, pas de flash blanc).

### 3. Comment

```tsx
import { useLayoutEffect, useRef, useState } from 'react';

function VirtualCarousel() {
  const [count, setCount] = useState(0);
  
  useLayoutEffect(() => {
    if (count === 10) {
      // On reset à 0 instantanément
      // L'utilisateur ne verra JAMAIS le chiffre "10" à l'écran
      setCount(0);
    }
  }, [count]);

  return (
    <div>
      <p>Compteur : {count}</p>
      <button onClick={() => setCount(c => c + 1)}>+1</button>
    </div>
  );
}
```
Dans cet exemple, React rend le composant avec `count = 10` en mémoire (Virtual DOM), exécute `useLayoutEffect`, voit le `setCount(0)`, et re-rend immédiatement avec `0`. Le navigateur n'affichera que le résultat final (0). Avec `useEffect`, vous auriez vu un flash "10" puis "0".

### 🚨 Limitations : Server-Side Rendering (SSR)
`useLayoutEffect` ne fonctionne pas côté serveur (Next.js, Remix) car il n'y a pas de mise en page (layout) sur le serveur.
Vous verrez un avertissement : *"useLayoutEffect does nothing on the server"*.
**Solutions :**
1.  Utiliser `useEffect` si la mesure n'est pas critique pour le premier affichage.
2.  Conditionner l'affichage du composant uniquement côté client.

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-25}

1.  **Quelle est la différence majeure entre `useEffect` et `useLayoutEffect` ?**
    `useEffect` est asynchrone et s'exécute après l'affichage. `useLayoutEffect` est synchrone et s'exécute avant l'affichage (bloque le paint).

2.  **Quel problème visuel `useLayoutEffect` permet-il d'éviter ?**
    Le clignotement (flicker) ou le saut de contenu, qui se produit quand on modifie le DOM visible juste après un rendu.

3.  **Pourquoi faut-il éviter d'utiliser `useLayoutEffect` par défaut ?**
    Parce qu'il bloque le rendu visuel. Si le traitement est long, l'application semble figée. Cela nuit à la performance perçue (Time to Interactive).

4.  **Comment `useLayoutEffect` se comporte-t-il en SSR (Next.js) ?**
    Il ne s'exécute pas sur le serveur et provoque un avertissement.

---

## Exercices : {#exercices-25}

### Exercice 1 - Le Texte Auto-Ajustable {#exercice-1---le-texte-auto-ajustable}

🎯 **Objectif** : Ajuster la taille de police pour qu'un texte tienne toujours sur une seule ligne.

💼 **Mise en situation** : Vous créez des cartes de visite virtuelles. Le nom de l'utilisateur doit être le plus gros possible, mais ne jamais passer à la ligne suivante, quelle que soit sa longueur.

📝 **Énoncé** :
1. Créez un composant `FitText` prenant une prop `text`.
2. Initialisez `fontSize` à 100px.
3. Utilisez `useLayoutEffect` pour vérifier si le texte dépasse (`scrollWidth > clientWidth`).
4. Tant que ça dépasse, réduisez la police (boucle `while` ou dichotomie simple).
5. Affichez le résultat. Avec `useEffect`, on verrait le texte "sauter" de gros à petit. Avec `useLayoutEffect`, c'est invisible.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Un texte très long ("Jean-Pierre Polnareff") qui tient parfaitement dans une div de 300px, avec une police réduite.
> **Annotation** : Montrez que le texte occupe toute la largeur sans déborder.
> **Alt Text suggéré** : Texte dont la taille de police s'est adaptée automatiquement au conteneur.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState, useRef, useLayoutEffect } from 'react';

export function FitText({ text }: { text: string }) {
  const containerRef = useRef<HTMLDivElement>(null);
  const textRef = useRef<HTMLSpanElement>(null);
  const [fontSize, setFontSize] = useState(100);

  // useLayoutEffect garantit que l'utilisateur ne verra pas les étapes intermédiaires
  useLayoutEffect(() => {
    const container = containerRef.current;
    const textEl = textRef.current;
    if (!container || !textEl) return;

    // Réinitialisation pour mesurer
    textEl.style.fontSize = "100px";
    
    // Algorithme simple de réduction
    let currentSize = 100;
    while (textEl.offsetWidth > container.offsetWidth && currentSize > 10) {
      currentSize--;
      textEl.style.fontSize = `${currentSize}px`;
    }
    
    // Mise à jour finale si nécessaire pour la persistance React
    // Note : ici on a manipulé le DOM direct pour la vitesse, 
    // mais on pourrait aussi faire des passes avec state.
  }, [text]);

  return (
    <div 
      ref={containerRef} 
      style={{ width: 300, border: '1px solid red', overflow: 'hidden', whiteSpace: 'nowrap' }}
    >
      <span ref={textRef} style={{ fontSize: '100px' }}>
        {text}
      </span>
    </div>
  );
}
```
</details>

### Exercice 2 - La Modal sans Clignotement {#exercice-2---la-modal-sans-clignotement}

🎯 **Objectif** : Positionner une fenêtre contextuelle qui évite les bords de l'écran.

💼 **Mise en situation** : Un menu contextuel (clic droit). Si on clique tout en bas de la page, le menu doit s'ouvrir vers le haut pour ne pas être coupé.

📝 **Énoncé** :
1. Un bouton tout en bas de la page "Ouvrir Menu".
2. Un composant `Menu` qui s'affiche d'abord à `top: 0` (invisible ou hidden).
3. `useLayoutEffect` mesure la hauteur du menu et la position du clic.
4. Si `clickY + menuHeight > windowHeight`, on inverse la position (affichage vers le haut).
5. Comparez mentalement ce qui se passerait avec `useEffect` (le menu apparaîtrait en bas, créerait une barre de scroll, puis sauterait en haut).

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState, useRef, useLayoutEffect } from 'react';

export function ContextMenu() {
  const [isOpen, setIsOpen] = useState(false);
  const menuRef = useRef<HTMLDivElement>(null);
  const [position, setPosition] = useState('bottom'); // 'bottom' ou 'top'

  useLayoutEffect(() => {
    if (isOpen && menuRef.current) {
      const { bottom } = menuRef.current.getBoundingClientRect();
      const windowHeight = window.innerHeight;

      // Si le menu dépasse le bas de l'écran
      if (bottom > windowHeight) {
        setPosition('top'); // On le déplace au-dessus
      }
    }
  }, [isOpen]);

  return (
    <div style={{ height: '150vh', position: 'relative', border: '1px dashed grey' }}>
      <p>Scrollez tout en bas...</p>
      
      <div style={{ position: 'absolute', bottom: 10, left: 10 }}>
        <button onClick={() => { setIsOpen(!isOpen); setPosition('bottom'); }}>
          Options
        </button>
        
        {isOpen && (
          <div 
            ref={menuRef}
            style={{
              position: 'absolute',
              [position === 'top' ? 'bottom' : 'top']: '100%', // Inverse la direction
              left: 0,
              width: 150,
              height: 200,
              background: 'white',
              border: '1px solid black',
              boxShadow: '0 4px 6px rgba(0,0,0,0.1)'
            }}
          >
            Menu Item 1<br/>Menu Item 2<br/>Menu Item 3
          </div>
        )}
      </div>
    </div>
  );
}
```
</details>

### Exercice 3 - Le Compteur Turbo (Optimisation visuelle) {#exercice-3---le-compteur-turbo}

🎯 **Objectif** : Démontrer le blocage visuel de `useLayoutEffect`.

💼 **Mise en situation** : Exercice théorique pour visualiser la différence.

📝 **Énoncé** :
1. Créez un composant avec un compteur à 0.
2. Un bouton qui, au clic, met le compteur à une valeur aléatoire, puis IMMÉDIATEMENT à 0 dans un effet.
3. Version A : Utilisez `useEffect`. Vous devriez voir un bref flash du chiffre aléatoire avant le 0.
4. Version B : Utilisez `useLayoutEffect`. Vous ne verrez jamais le chiffre aléatoire, il restera toujours à 0 visuellement.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState, useLayoutEffect, useEffect } from 'react';

export function FlickerTest() {
  const [count, setCount] = useState(0);

  // Changez useEffect en useLayoutEffect pour voir la différence
  // Avec useEffect : Flash possible
  // Avec useLayoutEffect : Aucun flash, le 0 reste stable
  useLayoutEffect(() => {
    if (count !== 0) {
      setCount(0);
    }
  }, [count]);

  return (
    <button onClick={() => setCount(Math.random())}>
      Valeur : {count} (Cliquez vite !)
    </button>
  );
}
```
</details>
```