Voici le chapitre **`useCallback` : Mémoïsation des Fonctions** pour la formation React 19.2.

```markdown
---
sidebar_label: `useCallback` : Mémoïsation des Fonctions
sidebar_position: 32
---

# Chapitre 32 : `useCallback` : Mémoïsation des Fonctions

Mémoïsation, Prévention des rendus inutiles, Dépendances de callback

Dans les chapitres précédents, nous avons vu comment React gère le rendu. Par défaut, quand un composant parent est rendu, tous ses enfants le sont aussi. C'est le comportement attendu.

Cependant, dans des applications complexes, vous voudrez parfois optimiser ces performances. Vous rencontrerez alors un problème subtil lié au fonctionnement de JavaScript : l'égalité des fonctions. Le Hook `useCallback` est l'outil précis pour résoudre ce problème.

:::info React Compiler (React 19)
Avec l'arrivée du **React Compiler** (introduit en React 19), la mémoïsation devient souvent automatique. Cependant, comprendre `useCallback` reste **indispensable** pour :
1.  Comprendre le fonctionnement interne de React.
2.  Travailler sur des projets existants ou sans le compilateur.
3.  Développer des bibliothèques ou des hooks complexes.
:::

## Le Problème de l'Égalité Référentielle {#le-probleme-de-l-egalite-referentielle}

### 1. Quoi
En JavaScript, deux fonctions qui se ressemblent ne sont pas "égales" si elles sont créées à deux moments différents.
À **chaque rendu** d'un composant, toutes les fonctions définies à l'intérieur sont **recréées**.

### 2. Pourquoi
Regardez ce code JavaScript standard :

```javascript
const f1 = () => console.log('Hello');
const f2 = () => console.log('Hello');

console.log(f1 === f2); // false (Adresse mémoire différente)
```

Dans React, si vous passez une fonction en prop à un composant enfant optimisé (avec `memo`), React comparera la nouvelle fonction avec l'ancienne. Comme elles ont des adresses mémoires différentes, React pensera que la prop a changé et forcera le re-rendu de l'enfant, annulant votre optimisation.

### 3. Comment fonctionne `useCallback`

`useCallback` demande à React de **garder la même instance de la fonction** en mémoire entre les rendus, tant que ses dépendances ne changent pas.

#### A. Syntaxe de base

```tsx
import { useCallback, useState } from 'react';

function MyComponent() {
  const [count, setCount] = useState(0);

  // ❌ Version classique : recréée à chaque clic (chaque rendu)
  const handleClickNormal = () => {
    console.log('Click', count);
  };

  // ✅ Version useCallback : stable entre les rendus
  const handleClickStable = useCallback(() => {
    console.log('Click', count);
  }, [count]); // Tableau de dépendances (comme useEffect)
}
```

---

## Optimiser les Composants Enfants (`React.memo`) {#optimiser-les-composants-enfants}

### 1. Quoi
L'usage principal de `useCallback` est de passer des fonctions stables à des composants enfants qui sont enveloppés dans `React.memo`.

### 2. Pourquoi
`React.memo` dit à un composant : "Ne te re-rends que si tes props changent".
Sans `useCallback`, la prop fonction change *toujours*, donc `React.memo` ne sert à rien.

### 3. Comment

#### B. Cas concret : Liste optimisée

Imaginons une liste d'articles où l'on peut supprimer un élément.

```tsx
import { useState, useCallback, memo } from 'react';

// 1. Composant Enfant Optimisé
// Il ne se re-rendra QUE si 'onDelete' ou 'id' change.
const TodoItem = memo(function TodoItem({ id, onDelete }: { id: number, onDelete: (id: number) => void }) {
  console.log(`Render Item ${id}`); // Trace pour vérifier
  return <button onClick={() => onDelete(id)}>Supprimer {id}</button>;
});

// 2. Composant Parent
export function TodoList() {
  const [todos, setTodos] = useState([1, 2, 3]);
  const [text, setText] = useState('');

  // ❌ SANS useCallback :
  // Chaque fois qu'on tape dans l'input (setText), 'handleDelete' est recréée.
  // Donc TOUS les <TodoItem /> se re-rendent inutilement.
  // const handleDelete = (id: number) => {
  //   setTodos(ts => ts.filter(t => t !== id));
  // };

  // ✅ AVEC useCallback :
  // La fonction reste la même tant que [ ] (aucune dépendance) ne change.
  // Tapez dans l'input ne déclenchera aucun re-rendu des TodoItem.
  const handleDelete = useCallback((id: number) => {
    setTodos(currentTodos => currentTodos.filter(t => t !== id));
  }, []); // Dépendances vides car on utilise le "functional update" de setState

  return (
    <div>
      <input value={text} onChange={e => setText(e.target.value)} placeholder="Ceci force un render du parent" />
      <div style={{ display: 'flex', gap: 10 }}>
        {todos.map(id => (
          <TodoItem key={id} id={id} onDelete={handleDelete} />
        ))}
      </div>
    </div>
  );
}
```

### 4. Zone de Danger

:::danger Ne pas optimiser prématurément
Utiliser `useCallback` a un coût (mémoire + coût d'exécution du hook).
**Ne l'utilisez pas par défaut.** Utilisez-le seulement si :
1.  La fonction est passée à un composant enfant **lourd** enveloppé dans `memo`.
2.  La fonction est utilisée comme dépendance dans un `useEffect` ou un autre hook.
:::

---

## Gestion des Dépendances (Le Piège des "Stale Closures") {#gestion-des-dependances}

### 1. Quoi
Comme pour `useEffect`, vous devez déclarer toutes les variables réactives utilisées dans votre callback. Si vous mentez à React (en omettant une dépendance), votre fonction verra des "vieux" états.

### 2. Pourquoi
C'est le concept de "Fermeture" (Closure) en JavaScript. La fonction "capture" les variables au moment de sa création.

### 3. Comment

```tsx
function Counter() {
  const [count, setCount] = useState(0);

  // ❌ MAUVAIS : Dépendance manquante
  const logBad = useCallback(() => {
    // Cette fonction est créée au 1er rendu, quand count valait 0.
    // Elle affichera TOUJOURS 0, même si count passe à 10.
    console.log("Count is: " + count); 
  }, []); // Il manque 'count' ici !

  // ✅ BON : Dépendance déclarée
  const logGood = useCallback(() => {
    console.log("Count is: " + count);
  }, [count]); // La fonction est recréée quand count change

  return <button onClick={logGood}>Log</button>;
}
```

---

## Tableau Comparatif : `useCallback` vs `useMemo` {#usecallback-vs-usememo}

Ils sont cousins, mais ne font pas la même chose.

| Critère | `useCallback` | `useMemo` |
| :--- | :--- | :--- |
| **Ce qu'il retourne** | La **fonction** elle-même (non exécutée) | Le **résultat** de l'exécution d'une fonction |
| **Syntaxe** | `useCallback(fn, deps)` | `useMemo(fn, deps)` |
| **Équivalence** | `useMemo(() => fn, deps)` | N/A |
| **Usage** | Stabiliser des event handlers (onClick...) | Stabiliser des calculs coûteux (filtrage...) |

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-32}

1.  **Que retourne exactement `useCallback(fn, [])` ?**
    Il retourne la fonction `fn` elle-même (mémoïsée), pas le résultat de son exécution.

2.  **Si je n'utilise pas `React.memo` sur le composant enfant, est-ce que `useCallback` sert à éviter son re-rendu ?**
    **Non**. Si l'enfant n'est pas "mémoïsé" (`memo`), il se re-rendra de toute façon quand le parent se re-rend, peu importe si la prop fonction est stable ou non.

3.  **Pourquoi préférer `setCount(c => c + 1)` à `setCount(count + 1)` dans un `useCallback` ?**
    La forme fonctionnelle (`c => c + 1`) ne dépend pas de la variable `count`. Cela permet de retirer `count` du tableau de dépendances (`[]`), rendant la fonction encore plus stable (elle ne change jamais).

---

## Exercices : {#exercices-32}

### Exercice 1 - La Grille de Pixels (Optimisation Massive) {#exercice-1---la-grille-de-pixels}

🎯 **Objectif** : Empêcher le re-rendu de 100 composants enfants quand on change une couleur.

💼 **Mise en situation** : Vous développez un éditeur de "Pixel Art". La grille contient 100 cellules (`<Pixel>`). Vous avez un sélecteur de couleur. Quand vous changez la couleur sélectionnée, la grille ne doit **PAS** se redessiner entièrement. Seul le clic sur un pixel doit déclencher une action.

📝 **Énoncé** :
1. Créez un composant `Pixel` qui accepte `id` et `onClick`. Il doit être enveloppé dans `memo` et logger "Render Pixel {id}".
2. Créez le parent `ArtBoard`. Il contient l'état `selectedColor` et une grille de 100 pixels.
3. Créez la fonction `handlePixelClick` avec `useCallback`.
4. **Astuce** : La fonction `handlePixelClick` doit appliquer la couleur actuelle au pixel. Si vous mettez `selectedColor` dans les dépendances, la fonction changera à chaque changement de couleur, brisant l'optimisation. Utilisez une ref ou une approche fonctionnelle pour contourner cela (ou acceptez que changer la couleur invalide la grille, mais essayez d'optimiser !).

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Console développeur.
> **Annotation** : Montrez que changer la couleur (input color) ne génère AUCUN log "Render Pixel".
> **Alt Text suggéré** : Console vide prouvant l'absence de re-rendus lors du changement d'état parent.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState, useCallback, memo } from 'react';

// 1. Composant Enfant Optimisé
const Pixel = memo(function Pixel({ id, onClick }: { id: number, onClick: (id: number) => void }) {
  console.log(`Render Pixel ${id}`); // Preuve de rendu
  return (
    <div 
      onClick={() => onClick(id)}
      style={{ width: 20, height: 20, border: '1px solid #ddd', cursor: 'pointer', background: 'white' }}
    />
  );
});

export function PixelArt() {
  const [color, setColor] = useState('#000000');
  // Utilisons un objet pour stocker les couleurs des pixels peints
  const [pixels, setPixels] = useState<Record<number, string>>({});

  // 3. Callback Optimisé
  // Problème : on a besoin de 'color' ici. Si on met [color], la grille se re-rend quand on change de couleur.
  // Solution : On passe la couleur en paramètre ou on utilise un state updater si possible.
  // Ici, pour l'exercice, acceptons une astuce : le onClick du Pixel ne fait que renvoyer l'ID.
  // C'est le parent qui sait quelle couleur appliquer.
  
  // Pour VRAIMENT optimiser et ne pas dépendre de 'color' dans le callback, 
  // une technique avancée est d'utiliser une `ref` pour la couleur courante 
  // ou de passer la couleur au moment du clic via l'enfant (moins propre).
  
  // Approche simple demandée : Stabiliser la fonction.
  // ATTENTION : Si on utilise 'color' dedans, on DOIT le mettre en dépendance.
  // Pour l'exercice, nous allons montrer comment stabiliser l'événement même si le parent change.
  
  const handlePixelClick = useCallback((id: number) => {
    // Ici, nous utilisons l'état fonctionnel pour ne pas dépendre de 'pixels'
    // Mais nous dépendons de 'color'. 
    setPixels(prev => ({ ...prev, [id]: color }));
  }, [color]); 
  // 👆 Notez : Si 'color' change, handlePixelClick change, et TOUT se re-rend.
  // C'est le comportement correct par défaut.
  // Pour aller plus loin (niveau expert), on utiliserait useRef pour 'color' pour avoir [] en dépendance.

  return (
    <div>
      <input type="color" value={color} onChange={e => setColor(e.target.value)} />
      <div style={{ display: 'grid', gridTemplateColumns: 'repeat(10, 20px)', gap: 2, marginTop: 10 }}>
        {Array.from({ length: 100 }).map((_, i) => (
          <div key={i} style={{ background: pixels[i] || 'transparent' }}>
             {/* Le wrapper div applique la couleur, le composant Pixel gère l'événement */}
             <Pixel id={i} onClick={handlePixelClick} />
          </div>
        ))}
      </div>
    </div>
  );
}
```
</details>

### Exercice 2 - La Boucle Infinie de l'Effet {#exercice-2---la-boucle-infinie-de-l-effet}

🎯 **Objectif** : Comprendre comment `useCallback` résout les dépendances d'effets.

💼 **Mise en situation** : Vous avez un hook `useEffect` qui appelle une fonction passée via les props ou définie localement. Si cette fonction n'est pas stable, l'effet se relance en boucle (ou trop souvent).

📝 **Énoncé** :
1. Créez un composant `SearchBox` qui prend une fonction `onSearch` en prop.
2. Dans `SearchBox`, un `useEffect` se déclenche quand `query` change et appelle `onSearch(query)`.
3. Ajoutez `onSearch` dans les dépendances de l'effet (règle ESLint obligatoire).
4. Dans le Parent, définissez `handleSearch` **sans** `useCallback`. Observez (via logs) que l'effet de l'enfant se lance à chaque render du parent, même si la recherche ne change pas.
5. Fixez avec `useCallback`.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState, useEffect, useCallback } from 'react';

// Enfant
function SearchResults({ fetchData }: { fetchData: () => void }) {
  useEffect(() => {
    console.log('⚡ Effet déclenché (fetchData a changé ou montage)');
    fetchData();
  }, [fetchData]); // ⚠️ fetchData est une dépendance critique

  return <div>Résultats (voir console)</div>;
}

// Parent
export function Dashboard() {
  const [text, setText] = useState('');
  const [counter, setCounter] = useState(0);

  // ❌ MAUVAIS : Cette fonction est recréée dès qu'on tape dans l'input (text change) ou qu'on clique (counter change)
  // const fetchData = () => {
  //   console.log('Fetching data...');
  // };

  // ✅ BON : Cette fonction ne change JAMAIS, peu importe les renders du parent
  const fetchData = useCallback(() => {
    console.log('Fetching data via API...');
  }, []); // [] = Stable pour toujours

  return (
    <div style={{ padding: 20 }}>
      <input value={text} onChange={e => setText(e.target.value)} placeholder="Taper ici..." />
      <button onClick={() => setCounter(c => c + 1)}>Re-render ({counter})</button>
      
      {/* Si fetchData n'est pas mémoïsé, SearchResults relance son effet à chaque frappe ! */}
      <SearchResults fetchData={fetchData} />
    </div>
  );
}
```
</details>

### Exercice 3 - Hooks Personnalisés et Callbacks {#exercice-3---hooks-personnalises-et-callbacks}

🎯 **Objectif** : Retourner des fonctions stables depuis un Custom Hook.

💼 **Mise en situation** : Vous créez une librairie interne. Vous écrivez un hook `useCounter`. Les fonctions `increment` et `decrement` qu'il retourne doivent être stables pour que les utilisateurs du hook puissent les utiliser dans des `useEffect` sans crainte.

📝 **Énoncé** :
1. Écrivez `useCounter(initialValue)`.
2. Il retourne `{ count, increment, reset }`.
3. Assurez-vous que `increment` et `reset` sont enveloppés dans `useCallback`.
4. Testez-le dans un composant qui a un `useEffect` dépendant de `reset`.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState, useCallback, useEffect } from 'react';

// 1. Le Hook optimisé
function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);

  // Utilisation de la forme fonctionnelle pour ne pas avoir de dépendance [count]
  const increment = useCallback(() => {
    setCount(c => c + 1);
  }, []);

  // Dépend de initialValue (si la prop change, on veut recréer la fonction, c'est logique)
  const reset = useCallback(() => {
    setCount(initialValue);
  }, [initialValue]);

  return { count, increment, reset };
}

// 2. Consommateur
export function CounterDemo() {
  const { count, increment, reset } = useCounter(10);
  const [renderCount, setRenderCount] = useState(0);

  // Pour forcer des re-rendus du composant parent
  const forceRender = () => setRenderCount(c => c + 1);

  useEffect(() => {
    console.log("✅ Reset function est stable, cet effet ne s'exécute qu'au montage.");
  }, [reset]); // Si reset changeait, cet effet se relancerait

  return (
    <div>
      <p>Compteur: {count}</p>
      <button onClick={increment}>+1</button>
      <button onClick={reset}>Reset</button>
      <hr />
      <button onClick={forceRender}>Forcer Render Parent ({renderCount})</button>
      <small> (Regardez la console : l'effet ne se déclenche pas lors du force render)</small>
    </div>
  );
}
```
</details>
```