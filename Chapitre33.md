Voici le chapitre **`useMemo` : Mémoïsation des Valeurs Calculées** pour la formation React 19.2.

```markdown
---
sidebar_label: `useMemo` : Mémoïsation des Valeurs Calculées
sidebar_position: 33
---

# Chapitre 33 : `useMemo` : Mémoïsation des Valeurs Calculées

Mémoïsation des valeurs, Performances, Dépendances de valeur

Dans le chapitre précédent, nous avons vu `useCallback` pour stabiliser des **fonctions**.
`useMemo` est son frère jumeau, mais il sert à stabiliser et mettre en cache des **résultats de calculs** (valeurs, objets, tableaux).

React est rapide, mais si vous effectuez des calculs lourds (filtrage de milliers d'éléments, cryptographie, transformation de données complexes) à chaque rendu, votre interface va devenir lente ("laggy"). `useMemo` permet de dire à React : *"Ne refais ce calcul que si les données d'entrée ont changé"*.

:::info React Compiler (React 19)
Tout comme pour `useCallback`, le nouveau **React Compiler** automatise une grande partie de la mémoïsation. Cependant, `useMemo` reste un outil fondamental à maîtriser pour :
1.  Les calculs très coûteux.
2.  Garantir la stabilité référentielle d'objets passés à des dépendances (`useEffect`, `useContext`).
3.  Comprendre le code existant et les mécanismes internes de React.
:::

## Qu'est-ce que la Mémoïsation de Valeur ? {#qu-est-ce-que-la-memoisation-de-valeur}

### 1. Quoi
C'est une technique de mise en cache.
- **Sans `useMemo`** : Le calcul s'exécute à **chaque** rendu du composant.
- **Avec `useMemo`** : React mémorise le résultat du dernier rendu. Si les dépendances n'ont pas changé, il renvoie le résultat stocké sans recalculer.

Signature :
```tsx
const cachedValue = useMemo(() => calculateValue(), [dependencies]);
```

### 2. Pourquoi
Il y a deux raisons principales d'utiliser `useMemo` :
1.  **Performance CPU** : Éviter de bloquer le thread principal avec des boucles lourdes.
2.  **Stabilité Référentielle** : Éviter de créer une nouvelle référence d'objet/tableau à chaque rendu (ce qui déclencherait des re-rendus inutiles chez les enfants `memo`).

### 3. Comment

#### A. Syntaxe de base : Calcul Lourd

Imaginons une fonction lente qui simule un gros travail.

```tsx
import { useState, useMemo } from 'react';

// Fonction simulée lente
function expensiveCalculation(num: number) {
  console.log("🔄 Calcul en cours...");
  let result = 0;
  // Boucle artificielle pour ralentir
  for (let i = 0; i < 100000000; i++) {
    result += num;
  }
  return result;
}

export function HeavyComponent() {
  const [count, setCount] = useState(0);
  const [text, setText] = useState("");

  // ❌ Sans useMemo :
  // Taper dans l'input (setText) déclenche un render -> relance le calcul -> l'input lag
  // const heavyResult = expensiveCalculation(count);

  // ✅ Avec useMemo :
  // Le calcul ne se relance QUE si 'count' change.
  // Taper dans l'input est fluide car React réutilise le résultat précédent.
  const heavyResult = useMemo(() => {
    return expensiveCalculation(count);
  }, [count]);

  return (
    <div>
      <h2>Résultat : {heavyResult}</h2>
      <button onClick={() => setCount(c => c + 1)}>Incrémenter ({count})</button>
      <input 
        value={text} 
        onChange={e => setText(e.target.value)} 
        placeholder="Tapez ici (test de fluidité)" 
      />
    </div>
  );
}
```

#### B. Cas concret : Filtrage et Tri de Liste
C'est le cas d'usage le plus fréquent dans les applications métier (Tableaux de bord, E-commerce).

```tsx
import { useMemo, useState } from 'react';

interface Product { id: number; name: string; price: number; category: string }

function ProductList({ products }: { products: Product[] }) {
  const [filter, setFilter] = useState("");
  const [sortByPrice, setSortByPrice] = useState(false);

  // ✅ Mémoïsation du filtrage et du tri
  // Si le parent se re-rend pour une autre raison, on ne refait pas ce tri.
  const visibleProducts = useMemo(() => {
    console.log("🔄 Filtrage et Tri des produits");
    
    // 1. Filtrer
    let result = products.filter(p => 
      p.name.toLowerCase().includes(filter.toLowerCase())
    );

    // 2. Trier
    if (sortByPrice) {
      result.sort((a, b) => a.price - b.price);
    }

    return result;
  }, [products, filter, sortByPrice]); // Dépendances strictes

  return (
    <div>
      <input value={filter} onChange={e => setFilter(e.target.value)} placeholder="Filtrer..." />
      <label>
        <input type="checkbox" checked={sortByPrice} onChange={e => setSortByPrice(e.target.checked)} />
        Trier par prix
      </label>
      
      <ul>
        {visibleProducts.map(p => <li key={p.id}>{p.name} - {p.price}€</li>)}
      </ul>
    </div>
  );
}
```

### 4. Zone de Danger

:::danger Ne mémoïsez pas tout !
Utiliser `useMemo` a un coût : React doit stocker la valeur, comparer les dépendances à chaque rendu, et allouer de la mémoire.
**À éviter :**
❌ `const value = useMemo(() => a + b, [a, b]);`
L'addition est plus rapide que le coût de `useMemo`.

**À utiliser :**
✅ Filtrage de tableaux > 100 éléments.
✅ Calculs récursifs ou complexes.
✅ Objets passés dans un `Context` ou comme prop à un composant `memo`.
:::

---

## `useMemo` pour la Stabilité Référentielle {#usememo-pour-la-stabilite-referentielle}

### 1. Quoi
En JS, `{ a: 1 } !== { a: 1 }`. Chaque objet littéral créé dans un composant est une **nouvelle référence**.
Cela brise l'optimisation `React.memo` des enfants et déclenche les `useEffect` inutilement.

### 2. Pourquoi
Si vous passez un objet de configuration à un enfant optimisé sans le mémoïser, l'enfant se re-rendra toujours.

### 3. Comment

```tsx
function Parent({ theme }: { theme: string }) {
  // ❌ Créé à CHAQUE rendu -> Nouvelle référence mémoire
  // const config = { color: theme === 'dark' ? 'white' : 'black' };

  // ✅ La référence de l'objet reste la même tant que le thème ne change pas
  const config = useMemo(() => ({
    color: theme === 'dark' ? 'white' : 'black',
    padding: 20
  }), [theme]);

  // ChildOptimized ne se re-rendra pas si le parent se re-rend pour une raison étrangère au thème
  return <ChildOptimized config={config} />;
}
```

---

## Tableau Comparatif : `useMemo` vs `useCallback` {#usememo-vs-usecallback}

| Feature | `useMemo` | `useCallback` |
| :--- | :--- | :--- |
| **But** | Mettre en cache un **résultat** (valeur, objet, array) | Mettre en cache une **fonction** |
| **Syntaxe** | `useMemo(() => value, [deps])` | `useCallback(fn, [deps])` |
| **Exécution** | La fonction est exécutée **pendant** le rendu | La fonction n'est **pas** exécutée (elle est juste stockée) |
| **Équivalence** | - | `useMemo(() => fn, [deps])` |

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-33}

1.  **Quand le code à l'intérieur de `useMemo` est-il exécuté ?**
    Pendant le rendu (rendering phase). Jamais après. Ne faites pas d'effets secondaires (API calls, DOM mutation) dedans.

2.  **`useMemo` garantit-il que le calcul ne sera *jamais* refait si les dépendances ne changent pas ?**
    En théorie oui, mais React se réserve le droit de vider le cache pour libérer de la mémoire. Votre code doit fonctionner même sans `useMemo`.

3.  **Est-il utile d'utiliser `useMemo` pour une opération simple comme `items.length` ?**
    Non, c'est contre-productif. Le coût de la mécanique de mémoïsation dépasse le gain.

---

## Exercices : {#exercices-33}

### Exercice 1 - Le Dashboard Crypto (Calcul Lourd) {#exercice-1---le-dashboard-crypto}

🎯 **Objectif** : Ressentir la différence de fluidité avec et sans `useMemo`.

💼 **Mise en situation** : Vous créez un dashboard de minage de crypto. Une fonction calcule la "difficulté" (hash rate) actuelle. C'est lent. L'utilisateur veut changer le nom de son mineur dans un input sans que l'interface ne gèle.

📝 **Énoncé** :
1. Créez une fonction `calculateHash(difficulty: number)` qui fait une boucle lourde (ex: 50 millions d'itérations) et retourne un nombre aléatoire.
2. Dans le composant, deux états : `difficulty` (number) et `minerName` (string).
3. Affichez le résultat du hash et l'input pour le nom.
4. **Sans `useMemo`** : Taper le nom doit être lent/saccadé.
5. **Avec `useMemo`** : Taper le nom doit être fluide, le calcul ne se relance que si on change la difficulté.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Interface simple avec un input texte et un slider/input pour la difficulté.
> **Annotation** : Ajoutez un `console.log` dans le calcul pour prouver qu'il ne se lance pas quand on tape le nom.
> **Alt Text suggéré** : Dashboard crypto montrant l'optimisation des performances.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState, useMemo } from 'react';

// Simulation d'un calcul très lourd
function calculateHash(difficulty: number) {
  const start = performance.now();
  console.log(`🔨 Minage en cours (Difficulé: ${difficulty})...`);
  
  let hash = 0;
  // Boucle bloquante pour simuler la charge CPU
  for (let i = 0; i < difficulty * 1000000; i++) {
    hash += Math.random();
  }
  
  console.log(`✅ Minage terminé en ${(performance.now() - start).toFixed(2)}ms`);
  return Math.floor(hash);
}

export function CryptoDashboard() {
  const [difficulty, setDifficulty] = useState(10); // Essayez d'augmenter si votre PC est trop puissant
  const [minerName, setMinerName] = useState("Miner_01");
  const [darkTheme, setDarkTheme] = useState(false);

  // 1. Le calcul lourd
  // Grâce à useMemo, changer le nom ou le thème ne relance PAS la boucle for
  const currentHash = useMemo(() => {
    return calculateHash(difficulty);
  }, [difficulty]); // Seule dépendance réelle du calcul

  return (
    <div style={{ 
      padding: 20, 
      background: darkTheme ? '#333' : '#fff', 
      color: darkTheme ? '#fff' : '#000' 
    }}>
      <h3>Crypto Dashboard ⛏️</h3>
      
      <label>
        Difficulté (Million d'itérations): {difficulty}
        <input 
          type="range" min="1" max="100" 
          value={difficulty} 
          onChange={e => setDifficulty(Number(e.target.value))} 
        />
      </label>

      <div style={{ margin: '20px 0', padding: 20, border: '1px solid gray' }}>
        <strong>Hash Rate Actuel : </strong> 
        <span style={{ fontFamily: 'monospace', fontSize: 20 }}>{currentHash}</span>
      </div>

      <input 
        value={minerName} 
        onChange={e => setMinerName(e.target.value)} 
        placeholder="Nom du mineur (test de lag)" 
      />
      
      <br /><br />
      <button onClick={() => setDarkTheme(p => !p)}>Changer Thème</button>
    </div>
  );
}
```
</details>

### Exercice 2 - L'Analytique des Ventes (Objets Dérivés) {#exercice-2---l-analytique-des-ventes}

🎯 **Objectif** : Optimiser le traitement de tableaux complexes.

💼 **Mise en situation** : Vous avez une liste brute de 1000 ventes `{ id, amount, country }`. Vous devez afficher le **Total par Pays** et le **Total Global**.

📝 **Énoncé** :
1. Générez une liste statique de 1000 ventes aléatoires (hors du composant ou avec `useState`).
2. Créez un état `selectedCountry` (filtre).
3. Utilisez `useMemo` pour calculer :
   - `totalAmount` (somme de tout).
   - `countryStats` (somme filtrée par pays sélectionné).
4. Si on change le pays sélectionné, `totalAmount` ne doit pas être recalculé (astuce : séparez les mémos ou gérez intelligemment).

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState, useMemo } from 'react';

// Génération de fausses données (statique, pas besoin de useMemo)
const generateSales = () => Array.from({ length: 5000 }, (_, i) => ({
  id: i,
  amount: Math.floor(Math.random() * 100) + 1,
  country: ['France', 'USA', 'Germany', 'Spain'][Math.floor(Math.random() * 4)]
}));

// On stocke ça dans une variable globale ou un useState initialisé une seule fois
const ALL_SALES = generateSales();

export function SalesAnalytics() {
  const [selectedCountry, setSelectedCountry] = useState('France');
  const [darkMode, setDarkMode] = useState(false); // Juste pour forcer le re-render

  // 1. Calcul du total global
  // Ne dépend d'aucun état changeant ici (car ALL_SALES est statique), 
  // mais dans une vraie app, ça dépendrait de 'sales' reçu en props.
  const globalTotal = useMemo(() => {
    console.log("💰 Calcul du Total Global...");
    return ALL_SALES.reduce((sum, sale) => sum + sale.amount, 0);
  }, []); // [] car ALL_SALES est constant externe ici.

  // 2. Calcul filtré
  // Ne se recalcule QUE si on change de pays. 
  // Le toggle du DarkMode ne déclenche pas ce reduce.
  const countryTotal = useMemo(() => {
    console.log(`🇫🇷 Calcul du Total pour ${selectedCountry}...`);
    return ALL_SALES
      .filter(s => s.country === selectedCountry)
      .reduce((sum, s) => sum + s.amount, 0);
  }, [selectedCountry]);

  return (
    <div style={{ background: darkMode ? '#222' : '#fff', color: darkMode ? '#fff' : '#000', padding: 20 }}>
      <h2>Analytique des Ventes</h2>
      
      <p>Total Global : <strong>{globalTotal} €</strong></p>
      
      <select value={selectedCountry} onChange={e => setSelectedCountry(e.target.value)}>
        <option value="France">France</option>
        <option value="USA">USA</option>
        <option value="Germany">Germany</option>
        <option value="Spain">Spain</option>
      </select>

      <p>Total {selectedCountry} : <strong>{countryTotal} €</strong></p>

      <button onClick={() => setDarkMode(p => !p)}>
        {darkMode ? 'Mode Clair' : 'Mode Sombre'}
      </button>
      <small> (Ouvrez la console pour voir les recalculs)</small>
    </div>
  );
}
```
</details>

### Exercice 3 - Le Piège de la Dépendance d'Effet {#exercice-3---le-piege-de-la-dependance-d-effet}

🎯 **Objectif** : Comprendre comment `useMemo` stabilise les objets pour `useEffect`.

💼 **Mise en situation** : Un composant `Chart` reçoit un objet `options` et trace un graphique. Le `Chart` est dans un parent qui se re-rend souvent.

📝 **Énoncé** :
1. Créez un composant enfant `Chart` qui prend une prop `config` (objet).
2. Dans `Chart`, mettez un `useEffect` qui dépend de `config` et logue "Drawing Chart...".
3. Dans le Parent, créez l'objet `config` **sans** `useMemo`.
4. Ajoutez un bouton dans le Parent pour forcer le re-rendu (compteur).
5. Observez que "Drawing Chart..." apparaît à chaque clic.
6. Corrigez avec `useMemo` dans le Parent pour que le graphique ne se redessine pas inutilement.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState, useEffect, useMemo } from 'react';

// Composant Enfant (simulé)
function Chart({ config }: { config: { color: string; type: string } }) {
  useEffect(() => {
    console.log("📊 (Re)Dessin du graphique avec config :", config);
  }, [config]); // Dépendance sur l'objet !

  return <div style={{ color: config.color }}>Graphique de type {config.type}</div>;
}

// Composant Parent
export function AnalyticsDashboard() {
  const [count, setCount] = useState(0);
  const [chartType, setChartType] = useState('bar');

  // ❌ SANS useMemo :
  // const chartConfig = { color: 'blue', type: chartType };
  // À chaque clic sur "Compteur", une nouvelle référence d'objet est créée.
  // L'enfant Chart croit que la config a changé et relance son effet.

  // ✅ AVEC useMemo :
  // L'objet reste le MÊME (même référence mémoire) tant que chartType ne change pas.
  // Cliquer sur "Compteur" ne déclenche plus l'effet de l'enfant.
  const chartConfig = useMemo(() => ({
    color: 'blue',
    type: chartType
  }), [chartType]);

  return (
    <div style={{ border: '1px dashed grey', padding: 20 }}>
      <button onClick={() => setCount(c => c + 1)}>
        Forcer Rendu Parent ({count})
      </button>
      
      <select onChange={e => setChartType(e.target.value)}>
        <option value="bar">Barres</option>
        <option value="line">Lignes</option>
      </select>

      <Chart config={chartConfig} />
    </div>
  );
}
```
</details>
```