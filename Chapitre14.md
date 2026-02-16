Voici le chapitre **Les Hooks React: Introduction et Règles** pour la formation React 19.2.

```markdown
---
sidebar_label: Les Hooks React: Introduction et Règles
sidebar_position: 14
---

# Chapitre 14 : Les Hooks React: Introduction et Règles

Définition des Hooks, Règles des Hooks, Hooks intégrés

Dans le chapitre précédent, vous avez utilisé `useState` pour donner une mémoire à vos composants. `useState` n'était que le sommet de l'iceberg. C'est un **Hook**.
Introduits en React 16.8 et devenus la norme absolue, les Hooks sont des fonctions qui permettent de "s'accrocher" (to hook into) aux fonctionnalités de React (état, cycle de vie, contexte) depuis des composants fonctionnels.

Ce chapitre est **crucial** car les Hooks viennent avec un manuel d'utilisation strict. Si vous ne respectez pas ces règles, votre application plantera ou aura des bugs de comportement très difficiles à tracer.

## Définition des Hooks {#definition-des-hooks}

### 1. Quoi
Un Hook est une fonction JavaScript spéciale qui permet d'utiliser des fonctionnalités de React sans écrire de classe.
Par convention, **tous les Hooks commencent par le préfixe `use`** (ex: `useState`, `useEffect`, `useContext`).

### 2. Pourquoi
Avant les Hooks, la logique d'état et les effets de bord (appels API, timers) étaient dispersés dans des méthodes de cycle de vie de classes (`componentDidMount`, `componentDidUpdate`, etc.). Cela rendait le code complexe et difficile à réutiliser.
Les Hooks permettent :
1.  **Réutilisation** : Extraire la logique d'état dans des "Custom Hooks" pour la partager entre composants.
2.  **Organisation** : Grouper le code par fonctionnalité (ex: abonnement à un chat) plutôt que par cycle de vie.
3.  **Simplicité** : Plus de `this`, plus de classes, juste des fonctions.

### 3. Comment

#### A. Reconnaître un Hook
Si vous voyez une fonction qui commence par `use` et qui est appelée à l'intérieur d'un composant, c'est un Hook.

```tsx
import { useState } from 'react';

export function Counter() {
  // 🟢 Ceci est un appel de Hook
  const [count, setCount] = useState(0); 
  
  // ...
}
```

#### B. La puissance de la composition (Custom Hooks)
Vous pouvez créer vos propres Hooks ! Par exemple, si vous gérez souvent des formulaires, vous pouvez créer `useFormInput`. Nous verrons cela en détail au Chapitre 28, mais comprenez dès maintenant que les Hooks sont des briques LEGO composables.

---

## Les Règles des Hooks (Rules of Hooks) {#regles-des-hooks}

C'est la section la plus importante de ce chapitre. React impose deux règles inviolables pour que sa mécanique interne fonctionne.

### 1. Règle du Niveau Supérieur (Top Level Only)
**N'appelez jamais de Hooks à l'intérieur de boucles, de conditions ou de fonctions imbriquées.**
Vous devez toujours appeler les Hooks au niveau le plus haut de votre fonction React, avant tout retour (`return`) anticipé.

#### Pourquoi ?
React ne "scanne" pas votre code. Il s'appuie sur l'**ordre d'exécution** des Hooks pour savoir quel état correspond à quel `useState`.
Si vous mettez un Hook dans un `if`, et que la condition change au rendu suivant, l'ordre des appels change, et React mélange tout (il donnera l'état du Hook 2 au Hook 3, etc.).

#### Zone de Danger : Violation de l'ordre

❌ **Interdit :**
```tsx
function BadComponent({ user }: { user: User | null }) {
  if (!user) {
    return <div>Connectez-vous</div>;
  }

  // 💥 ERREUR : Ce Hook ne sera pas appelé si user est null.
  // L'ordre des Hooks change d'un rendu à l'autre -> Crash garanti.
  const [balance, setBalance] = useState(0); 

  return <div>Solde : {balance}</div>;
}
```

✅ **Correct :**
```tsx
function GoodComponent({ user }: { user: User | null }) {
  // 1. Toujours déclarer les Hooks en premier, inconditionnellement
  const [balance, setBalance] = useState(0);

  // 2. Ensuite, gérer les conditions et retours anticipés
  if (!user) {
    return <div>Connectez-vous</div>;
  }

  return <div>Solde : {balance}</div>;
}
```

### 2. Règle des Fonctions React
**N'appelez les Hooks que depuis des composants fonctionnels React ou depuis d'autres Hooks personnalisés.**
N'appelez jamais un Hook depuis une fonction JavaScript classique.

❌ **Interdit :**
```tsx
// Ceci est une fonction utilitaire JS normale, pas un composant ni un hook
function formatData() {
  const [val, setVal] = useState(0); // 💥 ERREUR
  return val;
}
```

### 3. Comment s'assurer du respect des règles ?
L'écosystème React utilise un plugin ESLint (`eslint-plugin-react-hooks`) qui est installé par défaut avec la plupart des configurations modernes (Vite, Next.js). Votre éditeur soulignera en rouge toute violation de ces règles. **Ne désactivez jamais ces avertissements.**

---

## Panorama des Hooks Intégrés {#panorama-des-hooks-integres}

React 19.2 fournit une bibliothèque standard de Hooks. Voici une vue d'ensemble pour vous repérer dans la formation.

### 1. Hooks d'État (State)
Pour que le composant "se souvienne" d'informations.
- `useState` (Chapitre 13) : Pour des valeurs simples.
- `useReducer` (Chapitre 29) : Pour des logiques d'état complexes (machine à états).

### 2. Hooks de Contexte
Pour accéder à des données globales (thème, utilisateur connecté) sans passer les props manuellement.
- `useContext` (Chapitre 30).

### 3. Hooks de Référence
Pour stocker des informations qui ne déclenchent pas de rendu (comme un ID de timer ou une référence directe au DOM).
- `useRef` (Chapitre 20).

### 4. Hooks d'Effet
Pour synchroniser le composant avec des systèmes externes (API, DOM, abonnements).
- `useEffect` (Chapitre 22) : Le standard.
- `useLayoutEffect` (Chapitre 25) : Rare, exécuté avant l'affichage visuel.

### 5. Hooks de Performance
Pour optimiser les calculs et éviter les rendus inutiles.
- `useMemo` (Chapitre 33) : Cache un résultat de calcul.
- `useCallback` (Chapitre 32) : Cache une définition de fonction.

### 6. Hooks de Transition et React 19
Pour gérer l'asynchronicité et la fluidité de l'interface.
- `useTransition` (Chapitre 35).
- `useOptimistic` (Chapitre 37).
- `useActionState` (Chapitre 39).

---

## Tableau Comparatif : Cycle de vie Classe vs Hooks {#tableau-comparatif-classe-vs-hooks}

Même si vous n'avez jamais fait de React avec des classes, il est utile de comprendre le changement de paradigme.

| Concept | Classes (Legacy) | Hooks (Moderne) |
| :--- | :--- | :--- |
| État local | `this.state`, `this.setState` | `useState` |
| Montage du composant | `componentDidMount` | `useEffect(() => {}, [])` |
| Mise à jour | `componentDidUpdate` | `useEffect(() => {})` |
| Nettoyage (Démontage) | `componentWillUnmount` | `useEffect` (fonction de retour) |
| Partage de logique | *Render Props* ou *HOC* (complexe) | **Custom Hooks** (simple) |

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-14}

1.  **Comment reconnait-on un Hook dans le code ?**
    C'est une fonction qui commence par le préfixe `use` (ex: `useState`, `useMyCustomLogic`).

2.  **Pourquoi est-il interdit d'appeler un Hook dans une condition `if` ?**
    Car React s'appuie sur l'ordre d'appel des Hooks pour maintenir la cohérence de l'état interne. Si l'ordre change, l'état se corrompt.

3.  **Peut-on utiliser des Hooks dans des composants de type Classe ?**
    Non, les Hooks ne fonctionnent qu'à l'intérieur de composants fonctionnels (Function Components).

4.  **Que fait le plugin ESLint `react-hooks` ?**
    Il analyse votre code en temps réel pour vérifier que vous respectez les règles des Hooks (pas de boucles, pas de conditions).

---

## Exercices : {#exercices-14}

### Exercice 1 - Le Debugger de Règles {#exercice-1---le-debugger-de-regles}

🎯 **Objectif** : Identifier et corriger une violation des règles des Hooks.

💼 **Mise en situation** : Un développeur junior a écrit un composant de profil utilisateur, mais l'application plante aléatoirement. Le linter signale une erreur "Rendered fewer hooks than expected".

📝 **Énoncé** :
Le code ci-dessous est buggé. Identifiez pourquoi et réécrivez-le correctement sans changer la logique métier (si pas d'ID, on affiche "Pas d'utilisateur", sinon on gère le compteur de vues).

```tsx
// ❌ CODE BUGGÉ
function UserProfile({ userId }: { userId: string | null }) {
  if (!userId) {
    return <div>Pas d'utilisateur sélectionné</div>;
  }

  // Ce hook est conditionnel !
  const [views, setViews] = useState(0);

  return <div>Vues du profil : {views}</div>;
}
```

📺 **Résultat attendu** :
Le `useState` doit être appelé à chaque rendu, quel que soit l'état de `userId`.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState } from 'react';

function UserProfile({ userId }: { userId: string | null }) {
  // ✅ CORRECTION : Le Hook est remonté tout en haut.
  // Il est exécuté à CHAQUE rendu, l'ordre est préservé.
  const [views, setViews] = useState(0);

  // La condition de sortie (Early Return) vient APRÈS les Hooks
  if (!userId) {
    return <div>Pas d'utilisateur sélectionné</div>;
  }

  return (
    <div>
      <h1>Profil {userId}</h1>
      <p>Vues du profil : {views}</p>
      <button onClick={() => setViews(views + 1)}>Simuler une vue</button>
    </div>
  );
}
```
</details>

### Exercice 2 - Création d'un mini Custom Hook {#exercice-2---creation-d-un-mini-custom-hook}

🎯 **Objectif** : Comprendre que les hooks sont justes des fonctions composables.

💼 **Mise en situation** : Vous avez plusieurs composants qui ont besoin d'un système de bascule (Ouvrir/Fermer, Afficher/Masquer, Activer/Désactiver). Au lieu de répéter `useState` et la fonction `toggle` partout, créez un Hook réutilisable.

📝 **Énoncé** :
1. Créez une fonction `useToggle(initialValue: boolean)`.
2. Dedans, utilisez `useState`.
3. Retournez un tableau `[valeur, fonctionPourInverser]`.
4. Utilisez ce Hook dans un composant `Spoiler` qui affiche ou masque du texte.

📺 **Résultat attendu** :
Un code propre où le composant `Spoiler` n'a pas besoin de définir la logique d'inversion manuellement.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState } from 'react';

// 1. Définition du Custom Hook
// Il commence par "use", il respecte les règles, il utilise d'autres hooks.
function useToggle(initialValue: boolean = false) {
  const [state, setState] = useState(initialValue);
  
  // Fonction utilitaire pour inverser (true -> false -> true)
  const toggle = () => setState(prev => !prev);
  
  // On retourne ce dont le composant aura besoin
  return [state, toggle] as const; // "as const" aide TypeScript à typer le tuple
}

// 2. Consommation dans un composant
export function Spoiler({ text }: { text: string }) {
  // Utilisation super simple !
  const [isVisible, toggleVisible] = useToggle(false);

  return (
    <div style={{ border: '1px dashed grey', padding: '10px' }}>
      <button onClick={toggleVisible}>
        {isVisible ? 'Masquer' : 'Afficher le spoil'}
      </button>
      
      {isVisible && (
        <p style={{ marginTop: '10px' }}>{text}</p>
      )}
    </div>
  );
}
```
</details>

### Exercice 3 - Le Piège de la Boucle {#exercice-3---le-piege-de-la-boucle}

🎯 **Objectif** : Comprendre pourquoi on ne met pas de hooks dans un `.map()`.

💼 **Mise en situation** : Vous voulez afficher une liste de 3 compteurs indépendants. Vous tentez de faire un `.map()` et de déclarer un `useState` à l'intérieur.

📝 **Énoncé** :
1. Essayez conceptuellement de comprendre pourquoi le code ci-dessous est interdit :
   ```tsx
   items.map(item => {
      const [count, setCount] = useState(0); // ❌ INTERDIT
      return <li onClick={() => setCount(c => c+1)}>{item}: {count}</li>
   })
   ```
2. La solution consiste à **extraire** le contenu du `map` dans un sous-composant.
3. Créez un composant `CounterItem` qui contient le hook.
4. Utilisez ce composant dans la boucle principale.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Liste de 3 items (A, B, C) avec chacun un chiffre à côté.
> **Annotation** : Montrez que cliquer sur B n'augmente que B.
> **Alt Text suggéré** : Liste de compteurs indépendants créés par extraction de composant.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState } from 'react';

// ✅ SOLUTION : On isole l'état dans un composant enfant.
// Chaque instance de <CounterItem> aura son propre useState, appelé au top level de ce composant.
function CounterItem({ label }: { label: string }) {
  const [count, setCount] = useState(0);

  return (
    <li style={{ marginBottom: '5px' }}>
      <strong>{label}</strong> : {count} 
      <button onClick={() => setCount(count + 1)} style={{ marginLeft: '10px' }}>
        +1
      </button>
    </li>
  );
}

export function CounterList() {
  const items = ['Compteur A', 'Compteur B', 'Compteur C'];

  return (
    <ul>
      {items.map((item, index) => (
        // On appelle le COMPOSANT dans la boucle, pas le HOOK.
        <CounterItem key={index} label={item} />
      ))}
    </ul>
  );
}
```
</details>
```