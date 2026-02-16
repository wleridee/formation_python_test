Voici le chapitre **`useSyncExternalStore`: Intégrer des Stores Externes** pour la formation React 19.2.

```markdown
---
sidebar_label: `useSyncExternalStore`: Intégrer des Stores Externes
sidebar_position: 38
---

# Chapitre 38 : `useSyncExternalStore`: Intégrer des Stores Externes

Intégration de bibliothèques d'état, Modèle de souscription, Compatibilité avec React 18

Ce chapitre aborde un Hook un peu particulier. La plupart du temps, vous utilisez `useState`, `useReducer` ou `useContext` pour gérer l'état **interne** à React. Mais qu'en est-il de l'état **externe** ?

Un "store externe" est une source de données qui vit en dehors de l'arbre des composants React et qui peut changer à tout moment, comme :
*   Des bibliothèques de gestion d'état (Redux, Zustand, MobX).
*   Des APIs du navigateur (DOM events, `window.innerWidth`, `navigator.onLine`).
*   Des variables globales mutables.

Avant React 18, on utilisait `useEffect` pour s'abonner à ces sources. Mais avec l'arrivée du **Rendu Concurrent**, cela peut provoquer un phénomène appelé "Tearing" (déchirure visuelle), où une partie de l'interface affiche une version de la donnée et une autre partie en affiche une autre.

`useSyncExternalStore` est la réponse officielle de React pour lire ces données externes de manière sûre et synchronisée.

## Comprendre le "Tearing" et le Rendu Concurrent {#comprendre-le-tearing}

### 1. Quoi
Le "Tearing" est une incohérence visuelle. Imaginez une liste rendue par React. Si une mise à jour externe survient *pendant* que React est en train de rendre la liste (et que React a mis le rendu en pause pour laisser passer une interaction prioritaire), le haut de la liste pourrait afficher la "Vieille Valeur" et le bas la "Nouvelle Valeur".

### 2. Pourquoi
Dans React 18+ (mode concurrent), le rendu n'est plus bloquant. Il peut être interrompu. Les stores externes, eux, ne sont pas contrôlés par React et continuent de muter. `useSyncExternalStore` force React à traiter les mises à jour de ce store de manière synchrone et atomique pour éviter ce décalage.

---

## Le Hook `useSyncExternalStore` {#le-hook-usesyncexternalstore}

### 1. Quoi
C'est un Hook qui souscrit à un store externe et retourne un instantané (snapshot) de sa valeur actuelle.

Signature :
```tsx
const snapshot = useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot?);
```

### 2. Paramètres
1.  **`subscribe`** : Une fonction qui accepte un callback (`onStoreChange`) et l'enregistre auprès du store. Elle doit retourner une fonction de nettoyage (unsubscribe).
2.  **`getSnapshot`** : Une fonction qui retourne la valeur **actuelle** de l'état.
3.  **`getServerSnapshot`** (Optionnel) : Une fonction qui retourne la valeur initiale utilisée lors du rendu côté serveur (SSR) ou de l'hydratation.

### 3. Comment

#### A. Exemple avec une API Navigateur (`navigator.onLine`)

C'est l'exemple le plus simple car le navigateur est votre "store".

```tsx
import { useSyncExternalStore } from 'react';

// 1. Définir comment s'abonner
function subscribe(callback: () => void) {
  window.addEventListener('online', callback);
  window.addEventListener('offline', callback);
  
  // Retourner la fonction de nettoyage
  return () => {
    window.removeEventListener('online', callback);
    window.removeEventListener('offline', callback);
  };
}

// 2. Définir comment lire la donnée
function getSnapshot() {
  return navigator.onLine;
}

// 3. Définir la valeur SSR (le serveur ne connaît pas l'état réseau du client)
function getServerSnapshot() {
  return true; // On suppose qu'on est en ligne par défaut
}

export function NetworkStatus() {
  const isOnline = useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot);

  return (
    <div style={{ color: isOnline ? 'green' : 'red' }}>
      {isOnline ? '✅ Connecté' : '❌ Déconnecté'}
    </div>
  );
}
```

---

## Créer un Store Externe Personnalisé {#creer-un-store-externe}

### 1. Quoi
Souvent, vous voudrez extraire la logique d'état complexe hors de React (pour la performance ou l'architecture), un peu comme Redux ou Zustand.

### 2. Comment

Créons un mini-store observable en pur TypeScript.

```tsx
// store.ts
// Pas de React ici ! Juste du JS/TS pur.

let nextId = 0;
// L'état initial
let todos = [{ id: nextId++, text: 'Apprendre useSyncExternalStore' }];
// Les abonnés
let listeners: Set<() => void> = new Set();

export const todoStore = {
  addTodo(text: string) {
    // Mutation de l'état
    todos = [...todos, { id: nextId++, text }];
    // Notification des abonnés
    emitChange();
  },
  
  subscribe(listener: () => void) {
    listeners.add(listener);
    return () => listeners.delete(listener); // Unsubscribe
  },
  
  getSnapshot() {
    return todos;
  }
};

function emitChange() {
  for (const listener of listeners) {
    listener();
  }
}
```

Consommation dans le composant :

```tsx
// TodoApp.tsx
import { useSyncExternalStore } from 'react';
import { todoStore } from './store';

export function TodoApp() {
  // Le composant se re-rendra AUTOMATIQUEMENT quand le store change
  const todos = useSyncExternalStore(todoStore.subscribe, todoStore.getSnapshot);

  return (
    <>
      <button onClick={() => todoStore.addTodo('Nouvelle Tâche')}>
        Ajouter Tâche
      </button>
      <ul>
        {todos.map(todo => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
    </>
  );
}
```

---

## Zone de Danger : Stabilité de `getSnapshot` {#zone-de-danger}

### ❌ L'erreur classique : Retourner un nouvel objet
React compare la valeur retournée par `getSnapshot` avec la précédente. Si elles sont différentes, il re-rend.

```tsx
// ❌ MAUVAIS
function getSnapshot() {
  // Retourne TOUJOURS un nouveau tableau ([...]), même si rien n'a changé
  // -> Boucle infinie de rendus ou erreurs "Too many re-renders"
  return [...globalArray]; 
}
```

### ✅ La bonne pratique : Retourner une référence stable
```tsx
// ✅ BON
function getSnapshot() {
  // Retourne la même référence tant que le store ne l'a pas lui-même remplacée
  return globalArray;
}
```

Si vous avez besoin de transformer des données (ex: `store.data.filter(...)`), vous ne pouvez pas le faire directement dans `getSnapshot`. Vous devez :
1.  Soit mémoïser le résultat dans le store lui-même.
2.  Soit retourner l'état complet dans `getSnapshot` et faire le filtrage dans le composant avec `useMemo`.

---

## SSR et Hydratation : `getServerSnapshot` {#ssr-et-hydratation}

Si vous utilisez Next.js, Remix ou SSR maison, l'argument `getServerSnapshot` est **obligatoire** si la donnée diffère entre le serveur et le client.

Exemple typique : `window.innerWidth`.
*   Serveur : Pas de window. `width` indéfini.
*   Client : `width` = 1920px.

Si vous ne fournissez pas `getServerSnapshot` ou s'il retourne une valeur différente du premier rendu client, React lèvera une erreur d'hydratation.

```tsx
const width = useSyncExternalStore(
  subscribe, 
  () => window.innerWidth, 
  () => 0 // Valeur par défaut côté serveur
);
```

:::warning Hydration Mismatch
Si `getServerSnapshot` retourne `0` et que le client lit `1920` au premier rendu, React va crier.
La solution classique est de ne rendre le composant dépendant de la taille qu'après le montage (via un `useEffect` qui set un flag `isMounted`).
:::

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-38}

1.  **Pourquoi préférer `useSyncExternalStore` à `useEffect` pour s'abonner à des données externes ?**
    Pour éviter le "Tearing" (incohérence visuelle) lors du rendu concurrent de React 18, et parce qu'il gère les souscriptions de manière plus stricte et performante.

2.  **Que se passe-t-il si la fonction `getSnapshot` retourne un nouvel objet à chaque appel ?**
    React entrera dans une boucle de re-rendu infinie (ou s'arrêtera avec une erreur), car il pensera que la donnée a changé à chaque fois qu'il la vérifie.

3.  **Quand l'argument `getServerSnapshot` est-il nécessaire ?**
    Uniquement si vous faites du Rendu Côté Serveur (SSR) et que vous avez besoin de fournir une valeur initiale pour la génération du HTML côté serveur.

---

## Exercices : {#exercices-38}

### Exercice 1 - Le Tracker de Scroll (API Navigateur) {#exercice-1---le-tracker-de-scroll}

🎯 **Objectif** : Créer un Hook personnalisé `useScrollY` utilisant `useSyncExternalStore`.

💼 **Mise en situation** : Vous créez une barre de progression de lecture en haut de page pour un blog. Vous avez besoin de la position Y du scroll de manière très performante.

📝 **Énoncé** :
1. Créez une fonction `subscribe` qui écoute l'événement `scroll` sur `window`.
2. Créez une fonction `getSnapshot` qui retourne `window.scrollY`.
3. Gérez le cas SSR (retournez 0 si `window` n'existe pas).
4. Utilisez ce Hook pour afficher la position actuelle dans une `div` fixe.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useSyncExternalStore } from 'react';

// Fonctions définies HORS du composant pour la stabilité
function subscribe(callback: () => void) {
  window.addEventListener('scroll', callback);
  return () => window.removeEventListener('scroll', callback);
}

function getSnapshot() {
  return window.scrollY;
}

function getServerSnapshot() {
  return 0;
}

function useScrollY() {
  return useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot);
}

export function ScrollTracker() {
  const scrollY = useScrollY();

  return (
    <div style={{ 
      position: 'fixed', 
      top: 0, 
      right: 0, 
      background: 'black', 
      color: 'white', 
      padding: 10 
    }}>
      Scroll Y: {Math.round(scrollY)}px
      <div style={{ height: '200vh' }}>
         (Scrollez pour tester)
      </div>
    </div>
  );
}
```
</details>

### Exercice 2 - Le Gestionnaire de Panier (Custom Store) {#exercice-2---le-gestionnaire-de-panier}

🎯 **Objectif** : Implémenter un store externe simple pour un panier d'achat.

💼 **Mise en situation** : Votre client veut que le panier soit accessible même en dehors de l'arbre React (par exemple, pour être lu par un script d'analytics tiers). Context API ne suffit pas.

📝 **Énoncé** :
1. Créez un objet `cartStore` (fichier séparé ou hors composant) avec :
   - Un état `items` (tableau).
   - Une méthode `addItem(item)`.
   - La logique de `subscribe` / `listeners`.
2. Dans le composant, utilisez `useSyncExternalStore`.
3. Affichez le nombre d'articles dans le panier et un bouton pour ajouter.
4. (Bonus) Ouvrez la console et tapez `cartStore.addItem(...)` pour voir l'UI se mettre à jour sans passer par React !

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Interface simple avec compteur de panier.
> **Annotation** : Montrez la console JS ouverte à côté exécutant une commande manuelle sur le store qui met à jour l'UI.
> **Alt Text suggéré** : Mise à jour du panier React via la console JS.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useSyncExternalStore } from 'react';

// --- LE STORE (Logique métier pure) ---
type Item = { id: number; name: string };
let cartItems: Item[] = [];
let listeners = new Set<() => void>();

export const cartStore = {
  addItem(name: string) {
    // Immutabilité : on crée un nouveau tableau pour que React détecte le changement
    cartItems = [...cartItems, { id: Date.now(), name }];
    emitChange();
  },
  
  subscribe(listener: () => void) {
    listeners.add(listener);
    return () => listeners.delete(listener);
  },
  
  getSnapshot() {
    return cartItems; // Référence stable tant que addItem n'est pas appelé
  }
};

function emitChange() {
  listeners.forEach(l => l());
}

// Pour le bonus : rendre le store accessible globalement
(window as any).myCartStore = cartStore;

// --- LE COMPOSANT ---
export function ShoppingCart() {
  const items = useSyncExternalStore(cartStore.subscribe, cartStore.getSnapshot);

  return (
    <div style={{ border: '1px solid #ccc', padding: 20 }}>
      <h3>Panier ({items.length})</h3>
      <button onClick={() => cartStore.addItem("Produit " + (items.length + 1))}>
        Ajouter un produit
      </button>
      <ul>
        {items.map(item => (
          <li key={item.id}>{item.name}</li>
        ))}
      </ul>
      <small>Essayez `myCartStore.addItem('Hack')` dans la console !</small>
    </div>
  );
}
```
</details>

### Exercice 3 - Le Piège du `getSnapshot` Instable {#exercice-3---le-piege-du-getsnapshot-instable}

🎯 **Objectif** : Comprendre et corriger une boucle de rendu causée par une mauvaise implémentation.

💼 **Mise en situation** : Un collègue a écrit un store pour suivre la taille de la fenêtre. Son composant plante le navigateur ("Too many re-renders"). Vous devez le réparer.

📝 **Énoncé** :
1. Observez le code "buggé" ci-dessous (ne l'exécutez pas sans protection !).
2. Identifiez pourquoi `getSnapshot` cause le crash.
3. Corrigez le code pour qu'il retourne une valeur stable ou primitive.

**Code buggé (à analyser) :**
```tsx
// ❌ Ce code plante
function getSnapshot() {
  // Retourne un nouvel objet { w, h } à CHAQUE appel
  return { w: window.innerWidth, h: window.innerHeight };
}
```

<details>
<summary>💡 Solution corrigée et explication</summary>

**Le problème :** React appelle `getSnapshot` fréquemment. Si vous retournez un objet littéral `{...}`, c'est une nouvelle référence mémoire à chaque fois (`ref A !== ref B`). React croit que la donnée a changé, relance le rendu, rappelle `getSnapshot`, obtient une nouvelle référence, et boucle à l'infini.

**Solution 1 : Retourner des primitives (Hooks séparés)**
```tsx
const width = useSyncExternalStore(subscribe, () => window.innerWidth);
const height = useSyncExternalStore(subscribe, () => window.innerHeight);
```

**Solution 2 : Mémoïser l'objet dans le store (plus complexe)**
Si vous tenez à retourner un objet, vous devez le cacher dans une variable et ne le mettre à jour que lors de l'événement 'resize'.

```tsx
// Solution corrigée robuste
let windowSize = { w: window.innerWidth, h: window.innerHeight };

function subscribe(cb: () => void) {
  const handleResize = () => {
    // On met à jour la référence UNIQUEMENT si ça change
    windowSize = { w: window.innerWidth, h: window.innerHeight };
    cb();
  };
  window.addEventListener('resize', handleResize);
  return () => window.removeEventListener('resize', handleResize);
}

function getSnapshot() {
  return windowSize; // Retourne toujours la même ref si pas de resize
}
```
</details>
```