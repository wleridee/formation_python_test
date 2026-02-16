Voici le chapitre **Gérer les Dépendances d'Effets** pour la formation React 19.2.

```markdown
---
sidebar_label: Gérer les Dépendances d'Effets
sidebar_position: 23
---

# Chapitre 23 : Gérer les Dépendances d'Effets

Tableau de dépendances, Re-exécution des effets, Dépendances manquantes

Dans le chapitre précédent, vous avez appris à utiliser `useEffect` pour synchroniser votre composant avec un système externe. Cependant, la gestion du **tableau de dépendances** est souvent la source principale de bugs et de boucles infinies pour les développeurs React.

Ce chapitre est crucial : comprendre les dépendances, c'est comprendre comment React "réfléchit" et évite les calculs inutiles.

## Le Tableau de Dépendances {#le-tableau-de-dependances}

### 1. Quoi
Le tableau de dépendances est le **deuxième argument** passé à `useEffect`.
Il contient la liste exhaustive de toutes les valeurs réactives (props, state, variables calculées dans le composant) utilisées à l'intérieur de l'effet.

### 2. Pourquoi
React utilise ce tableau pour décider s'il doit **re-lancer** l'effet après un nouveau rendu.
- Si les valeurs dans le tableau sont identiques au rendu précédent (`prevDeps === nextDeps`), React **saute** l'exécution de l'effet.
- Si au moins une valeur a changé, l'effet est relancé.

### 3. Comment

#### A. Syntaxe de base

```tsx
useEffect(() => {
  // Code utilisant 'userId'
  console.log("Synchronisation pour l'utilisateur", userId);
}, [userId]); // ✅ Tableau explicite
```

#### B. Cas concret : Filtrage de produits
Imaginons une liste de produits filtrée par catégorie.

```tsx
import { useState, useEffect } from 'react';

function ProductList({ categoryId }: { categoryId: string }) {
  const [products, setProducts] = useState([]);

  useEffect(() => {
    // L'effet dépend de 'categoryId'. 
    // Si categoryId change (ex: "shoes" -> "hats"), on doit refetcher.
    async function fetchProducts() {
      const data = await api.get(`/products?cat=${categoryId}`);
      setProducts(data);
    }
    fetchProducts();
  }, [categoryId]); // 🚨 Si on oublie categoryId ici, la liste ne se mettra pas à jour au changement de catégorie.

  return (
    <ul>
      {products.map(p => <li key={p.id}>{p.name}</li>)}
    </ul>
  );
}
```

### 4. Zone de Danger : Mentir à React

:::danger Ne mentez jamais sur vos dépendances
Il est tentant d'omettre une dépendance pour "empêcher l'effet de tourner trop souvent". **C'est une mauvaise pratique.**
Si votre effet utilise une variable `X` mais que `X` n'est pas dans le tableau `[]`, votre effet utilisera une **vieille version** de `X` (stale closure).

❌ **Mauvais :**
```tsx
useEffect(() => {
  console.log(count); // Utilise count
}, []); // [] dit "Ne jamais relancer". L'effet affichera toujours la valeur initiale (ex: 0).
```

✅ **Bon :**
```tsx
useEffect(() => {
  console.log(count);
}, [count]); // Relance l'effet quand count change.
```
Si vous voulez éviter que l'effet ne se relance, la solution n'est pas de supprimer la dépendance, mais de modifier la **logique** de l'effet (voir section suivante).
:::

---

## Dépendances Objets et Fonctions {#dependances-objets-et-fonctions}

### 1. Quoi
En JavaScript, les objets et les fonctions sont comparés par **référence**, pas par valeur.
`{ id: 1 } === { id: 1 }` est `false`.
`function() {} === function() {}` est `false`.

### 2. Pourquoi
Si vous créez un objet ou une fonction **dans le corps du composant** et que vous le mettez dans le tableau de dépendances, il sera différent à **chaque rendu**.
L'effet se relancera donc à l'infini ou inutilement à chaque rendu, brisant l'optimisation.

### 3. Comment résoudre

#### A. Problème : Boucle infinie potentielle

```tsx
function SearchResults({ query }) {
  // ⚠️ Cette config est recréée à CHAQUE rendu (nouvelle adresse mémoire)
  const options = { page: 1, sort: 'desc' }; 

  useEffect(() => {
    fetchData(query, options);
  }, [query, options]); // 'options' change tout le temps -> Effet relancé tout le temps
}
```

#### B. Solutions

**Option 1 : Sortir la variable du composant (si elle ne dépend pas des props/state)**
```tsx
// ✅ Stable car défini une seule fois en dehors
const options = { page: 1, sort: 'desc' }; 

function SearchResults({ query }) {
  useEffect(() => {
    fetchData(query, options);
  }, [query]); // 'options' n'est plus une dépendance réactive
}
```

**Option 2 : Déplacer la variable DANS l'effet (si elle n'est utilisée que là)**
```tsx
function SearchResults({ query }) {
  useEffect(() => {
    // ✅ Créé uniquement quand l'effet s'exécute
    const options = { page: 1, sort: 'desc' }; 
    fetchData(query, options);
  }, [query]);
}
```

**Option 3 : `useMemo` (si elle doit être partagée)**
Voir Chapitre 33 pour les détails.
```tsx
const options = useMemo(() => ({ page: 1, sort: 'desc' }), []);
```

---

## Supprimer des Dépendances Inutiles {#supprimer-des-dependances-inutiles}

Parfois, vous voulez utiliser une valeur dans un effet sans qu'elle ne déclenche l'effet.

### Cas classique : L'Event Listener
Vous voulez logger la position de la souris quand l'utilisateur appuie sur une touche, mais vous ne voulez pas recréer l'event listener à chaque mouvement de souris.

#### Mauvaise approche
```tsx
useEffect(() => {
  const handler = () => console.log(position); // Utilise 'position'
  window.addEventListener('click', handler);
  return () => window.removeEventListener('click', handler);
}, [position]); // 😱 L'écouteur est détruit et recréé à chaque pixel de mouvement !
```

#### La solution "useEffectEvent" (Expérimental / Canary)
React introduit un nouveau Hook (voir Chapitre 24) pour séparer la logique réactive de la logique non-réactive.
En attendant, la solution standard actuelle est d'utiliser une **Ref** pour stocker la valeur "fraîche" sans déclencher de rendu.

```tsx
const positionRef = useRef(position);
// On garde la ref à jour
useEffect(() => {
  positionRef.current = position;
});

useEffect(() => {
  const handler = () => console.log(positionRef.current); // ✅ Lit la valeur courante sans dépendance
  window.addEventListener('click', handler);
  return () => window.removeEventListener('click', handler);
}, []); // ✅ L'écouteur est créé une seule fois
```

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-23}

1.  **Pourquoi `useEffect` a-t-il besoin d'un tableau de dépendances ?**
    Pour optimiser les performances en évitant de ré-exécuter l'effet si les données n'ont pas changé, et pour éviter les boucles infinies.

2.  **Que se passe-t-il si j'omets le tableau de dépendances (ni `[]` ni variables) ?**
    L'effet s'exécutera après **chaque** rendu du composant.

3.  **Pourquoi mettre un objet `{ id: 1 }` créé dans le composant comme dépendance est une mauvaise idée ?**
    Parce qu'un nouvel objet est créé en mémoire à chaque rendu. Pour React, la dépendance a changé (`oldObj !== newObj`), donc l'effet se relance systématiquement.

4.  **Comment corriger une "dépendance manquante" signalée par le linter (ESLint) ?**
    Soit vous ajoutez la variable au tableau `[]`, soit vous modifiez votre code pour ne plus avoir besoin de cette variable dans l'effet (ex: en la déplaçant à l'intérieur de l'effet).

---

## Exercices : {#exercices-23}

### Exercice 1 - Le Timer Capricieux (Correction de bug) {#exercice-1---le-timer-capricieux}

🎯 **Objectif** : Identifier et corriger une dépendance manquante causant un comportement inattendu.

💼 **Mise en situation** : Un compteur qui doit s'incrémenter chaque seconde. Le stagiaire a écrit un code où le compteur reste bloqué à 1 ou se comporte bizarrement.

📝 **Énoncé** :
1. Analysez le code ci-dessous (le timer incrémente une fois puis s'arrête ou redémarre mal).
2. Corrigez le tableau de dépendances.
3. Proposez une version alternative utilisant la forme fonctionnelle de `setCount` (`c => c + 1`) pour vider le tableau de dépendances.

```tsx
// Code buggé fourni
useEffect(() => {
  const id = setInterval(() => {
    setCount(count + 1); // Dépend de count
  }, 1000);
  return () => clearInterval(id);
}, []); // ❌ count manque ici
```

📺 **Résultat attendu** :
Un compteur qui avance régulièrement : 1, 2, 3, 4... sans réinitialiser l'intervalle à chaque fois.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState, useEffect } from 'react';

export function FixedTimer() {
  const [count, setCount] = useState(0);

  // Solution 1 : Ajouter la dépendance
  // L'intervalle est détruit et recréé à chaque changement de 'count'.
  // C'est correct, mais pas optimal niveau performance.
  /*
  useEffect(() => {
    const id = setInterval(() => {
      setCount(count + 1);
    }, 1000);
    return () => clearInterval(id);
  }, [count]);
  */

  // Solution 2 (Recommandée) : Update fonctionnel
  // On supprime la dépendance à 'count' en utilisant le callback du setter.
  useEffect(() => {
    const id = setInterval(() => {
      setCount(prev => prev + 1); // ✅ Plus besoin de lire 'count' ici
    }, 1000);
    return () => clearInterval(id);
  }, []); // ✅ Tableau vide : l'intervalle est créé une seule fois au montage.

  return <h1>Timer : {count}</h1>;
}
```
</details>

### Exercice 2 - La Recherche API (Debouncing & Dépendances) {#exercice-2---la-recherche-api}

🎯 **Objectif** : Gérer les appels API dépendants d'une saisie utilisateur.

💼 **Mise en situation** : Un champ de recherche de films. On veut lancer la recherche quand `query` change, mais pas si `query` est vide.

📝 **Énoncé** :
1. État `query` (string) et `results` (array).
2. Effet qui se déclenche quand `query` change.
3. Si `query` est vide, vider `results` et ne rien faire.
4. Sinon, simuler un appel API (`console.log("Searching " + query)`).
5. Ajoutez une fonction de nettoyage pour ignorer les résultats si la requête change avant la réponse (pattern "ignore").

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Input avec le texte "Matrix" et une liste de résultats fictifs dessous.
> **Annotation** : Montrez que la recherche s'est déclenchée.
> **Alt Text suggéré** : Interface de recherche de films réactive.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState, useEffect } from 'react';

export function MovieSearch() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState<string[]>([]);

  useEffect(() => {
    // 1. Si vide, on nettoie et on sort
    if (query === '') {
      setResults([]);
      return;
    }

    let ignore = false;

    console.log(`📡 Fetching API for "${query}"...`);

    // Simulation API asynchrone
    setTimeout(() => {
      if (!ignore) {
        setResults([`Résultat 1 pour ${query}`, `Résultat 2 pour ${query}`]);
      }
    }, 500);

    // 2. Cleanup pour éviter les "Race Conditions"
    return () => {
      ignore = true;
      console.log(`❌ Request cancelled/ignored for "${query}"`);
    };
  }, [query]); // ✅ Dépendance explicite : on relance si le texte change

  return (
    <div>
      <input 
        value={query} 
        onChange={e => setQuery(e.target.value)} 
        placeholder="Chercher un film..." 
      />
      <ul>
        {results.map((r, i) => <li key={i}>{r}</li>)}
      </ul>
    </div>
  );
}
```
</details>

### Exercice 3 - L'Objet Piégé (Référence vs Valeur) {#exercice-3---l-objet-piege}

🎯 **Objectif** : Comprendre le problème des objets dans les dépendances.

💼 **Mise en situation** : Un composant `Profile` reçoit un objet `styleConfig` en prop. Il utilise cet objet dans un effet pour appliquer un thème. Mais le composant parent recrée cet objet à chaque rendu, causant des boucles infinies.

📝 **Énoncé** :
1. Créez un composant `ThemedBox` qui prend une prop `config` (objet `{ color: string }`).
2. Dans `ThemedBox`, un `useEffect` loggue "Theme Changed!" quand `config` change.
3. Dans le Parent, forcez un re-render (avec un compteur par exemple) et passez un objet littéral `{{ color: 'red' }}` à l'enfant.
4. Observez que "Theme Changed!" apparaît à chaque clic du compteur (BUG).
5. **Mission** : Corrigez le problème SANS changer le Parent (en utilisant une comparaison manuelle ou en déstructurant l'objet dans l'effet).

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState, useEffect } from 'react';

// Composant Enfant
function ThemedBox({ config }: { config: { color: string } }) {
  // ❌ MAUVAISE APPROCHE
  /*
  useEffect(() => {
    console.log("🎨 Theme Changed!");
  }, [config]); // config est un nouvel objet à chaque fois -> Log infini
  */

  // ✅ BONNE APPROCHE 1 : Déstructuration (Dépendre de la valeur primitive)
  const { color } = config;
  useEffect(() => {
    console.log(`🎨 Theme Changed to ${color}!`);
  }, [color]); // 'color' est une string, la comparaison par valeur fonctionne.

  return <div style={{ color: config.color }}>Je suis le texte coloré</div>;
}

// Composant Parent (Simule le problème)
export function App() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>
        Re-render Parent ({count})
      </button>
      
      {/* Ici, l'objet est recréé à chaque rendu du Parent */}
      <ThemedBox config={{ color: 'blue' }} />
    </div>
  );
}
```
</details>
```