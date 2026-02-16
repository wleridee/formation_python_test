Voici le chapitre **`useDeferredValue`: Déferrer la Mise à Jour d'une Valeur** pour la formation React 19.2.

```markdown
---
sidebar_label: `useDeferredValue`: Déferrer la Mise à Jour d'une Valeur
sidebar_position: 34
---

# Chapitre 34 : `useDeferredValue`: Déferrer la Mise à Jour d'une Valeur

Mises à jour non bloquantes, Priorisation de l'UI, Optimisation de l'expérience utilisateur

Dans les applications modernes, la réactivité de l'interface est critique.
Parfois, vous devez afficher des données fraîches instantanément (comme ce que l'utilisateur tape dans un champ de texte), tout en effectuant un rendu lourd en arrière-plan (comme filtrer une liste de 10 000 éléments).

Si vous faites tout en même temps, l'interface gèle. Le clavier ne répond plus. L'utilisateur s'énerve.
`useDeferredValue` est un Hook de performance qui permet de dire à React : *"Mets à jour cette partie de l'UI **plus tard**, quand le navigateur sera moins occupé"*.

## Le Concept de Valeur Différée {#le-concept-de-valeur-differee}

### 1. Quoi
`useDeferredValue` accepte une valeur en entrée et retourne une nouvelle copie de cette valeur qui sera mise à jour avec un **léger retard** par rapport à l'originale, sans bloquer le rendu principal.

Signature :
```tsx
const deferredValue = useDeferredValue(value, initialValue?);
```

### 2. Pourquoi
Pour garder l'interface **fluide** (responsive) même pendant des calculs lourds.
Contrairement au "debouncing" (qui attend que l'utilisateur arrête de taper), `useDeferredValue` se déclenche dès que possible, mais avec une priorité plus basse que les interactions utilisateur (clic, frappe clavier).

### 3. Comment

#### A. Syntaxe de base

```tsx
import { useState, useDeferredValue } from 'react';

function SearchPage() {
  const [query, setQuery] = useState('');
  
  // 1. Créer une version "en retard" de la requête
  // Si query change rapidement, deferredQuery ne se mettra à jour 
  // que quand le CPU aura le temps.
  const deferredQuery = useDeferredValue(query);

  return (
    <>
      {/* L'input est lié à 'query' : il reste super réactif */}
      <input value={query} onChange={e => setQuery(e.target.value)} />

      {/* La liste lourde utilise 'deferredQuery' */}
      <SlowList text={deferredQuery} />
    </>
  );
}
```

#### B. Cas Concret : Filtrage Lourd

Imaginez une liste de produits gigantesque.

```tsx
import { useState, useDeferredValue, memo } from 'react';

// Composant lent simulé (doit être memoïsé pour que l'optimisation fonctionne)
const HeavyList = memo(function HeavyList({ keyword }: { keyword: string }) {
  // Simulation de lenteur : on bloque le thread pendant 50ms par render
  const start = performance.now();
  while (performance.now() - start < 50) {
    // Boucle bloquante
  }

  return <p>Résultats pour "{keyword}" (Rendu lourd terminé)</p>;
});

export function ProductSearch() {
  const [text, setText] = useState('');
  
  // ✅ Sans useDeferredValue : L'input gèlerait à chaque frappe car HeavyList bloquerait tout.
  // ✅ Avec useDeferredValue : React met à jour l'input (text) IMMÉDIATEMENT,
  // puis tente de rendre HeavyList en arrière-plan avec la nouvelle valeur.
  const deferredText = useDeferredValue(text);

  return (
    <div>
      <input 
        value={text} 
        onChange={e => setText(e.target.value)} 
        placeholder="Tapez vite..." 
      />
      
      {/* Indicateur visuel optionnel : est-ce que la liste est périmée ? */}
      <div style={{ opacity: text !== deferredText ? 0.5 : 1 }}>
        <HeavyList keyword={deferredText} />
      </div>
    </div>
  );
}
```

---

## Pattern : Indicateur de Chargement "Stale" {#pattern-indicateur-de-chargement-stale}

### 1. Quoi
Quand `deferredValue` est différent de `value`, cela signifie que React est en train de calculer le nouveau rendu en arrière-plan. L'interface affichée est "périmée" (stale).

### 2. Pourquoi
C'est une bonne UX de montrer à l'utilisateur que "ça charge", sans pour autant afficher un spinner bloquant. On peut juste griser le contenu existant.

### 3. Comment

```tsx
const query = text;
const deferredQuery = useDeferredValue(query);
const isStale = query !== deferredQuery; // True si mise à jour en cours

return (
  <div style={{ 
    opacity: isStale ? 0.5 : 1, // Effet visuel immédiat
    transition: 'opacity 0.2s' 
  }}>
    <SlowComponent query={deferredQuery} />
  </div>
);
```

### 4. Zone de Danger

:::danger `useDeferredValue` vs `useEffect`
Ne faites pas ça :
```tsx
// ❌ MAUVAIS : Effet déclenché par deferredValue
useEffect(() => {
  fetchResults(deferredQuery);
}, [deferredQuery]);
```
`useDeferredValue` est fait pour le **rendu**, pas pour déclencher des effets asynchrones. Si vous voulez "debouncer" une requête réseau, utilisez une librairie de debounce classique (lodash) ou `setTimeout`.
`useDeferredValue` sert à débloquer le CPU pour le rendu UI, pas à économiser des requêtes réseau.
:::

---

## Tableau Comparatif : `useDeferredValue` vs `Debounce` {#usedeferredvalue-vs-debounce}

| Critère | Debounce (`setTimeout`) | `useDeferredValue` |
| :--- | :--- | :--- |
| **Mécanisme** | Attend un délai fixe (ex: 500ms) **après** la dernière frappe | Se lance **dès que possible** sans bloquer l'input |
| **Ressenti** | L'UI ne bouge pas, puis saute à la fin | L'UI reste fluide, le contenu lourd arrive au fil de l'eau |
| **Configuration** | Doit choisir un délai arbitraire | Automatique (React gère la priorité) |
| **Annulation** | Annule les appels intermédiaires | React abandonne le rendu intermédiaire si une nouvelle valeur arrive |

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-34}

1.  **Quelle est la différence principale entre `useDeferredValue` et un debounce classique ?**
    Le debounce impose un délai fixe arbitraire. `useDeferredValue` est adaptatif : il met à jour dès que le CPU est libre.

2.  **Pourquoi le composant qui reçoit la valeur différée doit-il être enveloppé dans `React.memo` ?**
    Si le composant n'est pas mémoïsé, il se re-rendra de toute façon quand le parent se re-rend (lors de la mise à jour de la valeur immédiate), annulant l'optimisation.

3.  **Comment savoir si React est en train de calculer la version différée en arrière-plan ?**
    En comparant la valeur originale (`value`) et la valeur différée (`deferredValue`). Si elles sont différentes, le rendu est en cours.

---

## Exercices : {#exercices-34}

### Exercice 1 - La Liste de Prénoms Gigantesque {#exercice-1---la-liste-de-prenoms-gigantesque}

🎯 **Objectif** : Ressentir la fluidité de l'input malgré un rendu lourd.

💼 **Mise en situation** : Vous affichez une liste de 20 000 prénoms. L'utilisateur doit pouvoir filtrer. Sans optimisation, chaque lettre tapée gèle le navigateur pendant 200ms.

📝 **Énoncé** :
1. Générez un tableau statique de 20 000 chaînes (ex: "Nom 1", "Nom 2"...).
2. Créez un composant `SlowList` qui filtre et affiche cette liste. **Forcez-le à être lent** (ajoutez `while(performance.now() - start < 20) {}` dans le corps du composant).
3. Dans le Parent, utilisez un `input` contrôlé.
4. Passez d'abord la valeur directe (`text`). Constatez le lag.
5. Implémentez `useDeferredValue` et passez la valeur différée. Constatez la fluidité de l'input.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Interface avec un input et une longue liste.
> **Annotation** : Montrez l'input rempli ("taper vite") alors que la liste en dessous affiche encore l'ancien état (ou est grisée).
> **Alt Text suggéré** : Démonstration de useDeferredValue gardant l'input réactif.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState, useDeferredValue, memo } from 'react';

// Génération de données
const NAMES = Array.from({ length: 20000 }, (_, i) => `Utilisateur ${i}`);

// Composant LENT (simulé)
const SlowList = memo(function SlowList({ query }: { query: string }) {
  // Ralentissement artificiel par item pour simuler un rendu complexe
  const start = performance.now();
  while (performance.now() - start < 50) {
    // Bloque le thread principal 50ms (énorme pour une UI)
  }

  const filtered = NAMES.filter(name => 
    name.toLowerCase().includes(query.toLowerCase())
  );

  return (
    <ul style={{ height: 200, overflow: 'auto', border: '1px solid #ccc' }}>
      {filtered.length === 0 ? <li>Aucun résultat</li> : null}
      {filtered.map(name => <li key={name}>{name}</li>)}
    </ul>
  );
});

export function BigListSearch() {
  const [query, setQuery] = useState('');
  
  // 🚀 L'astuce magique
  const deferredQuery = useDeferredValue(query);

  // UX : On montre que c'est "en train de réfléchir"
  const isStale = query !== deferredQuery;

  return (
    <div style={{ padding: 20 }}>
      <h3>Recherche Optimisée 🏎️</h3>
      <input 
        value={query} 
        onChange={e => setQuery(e.target.value)} 
        placeholder="Tapez très vite ici..." 
        style={{ width: '100%', padding: 8, marginBottom: 10 }}
      />
      
      <div style={{ opacity: isStale ? 0.3 : 1, transition: 'opacity 0.2s' }}>
        {/* On passe la valeur différée au composant lent */}
        <SlowList query={deferredQuery} />
      </div>

      <small>
        Statut : {isStale ? "⏳ Mise à jour en arrière-plan..." : "✅ À jour"}
      </small>
    </div>
  );
}
```
</details>

### Exercice 2 - Visualisation de Données (Chart) {#exercice-2---visualisation-de-donnees}

🎯 **Objectif** : Appliquer le pattern sur un composant graphique tiers.

💼 **Mise en situation** : Un slider contrôle la densité d'un nuage de points (Scatter Plot). Le graphique prend 300ms à se redessiner. Le slider doit rester fluide.

📝 **Énoncé** :
1. Créez un composant `ScatterPlot` (simulé par une div colorée qui change de taille ou un canvas). Faites-le ramer artificiellement.
2. Un slider (range input) contrôle le nombre de points (de 100 à 5000).
3. Utilisez `useDeferredValue` pour découpler la position du slider (instantanée) du rendu du graphique (différé).
4. Affichez la valeur du slider en temps réel à côté du graphique.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState, useDeferredValue, memo } from 'react';

// Simulation d'un graphique lourd
const Chart = memo(({ points }: { points: number }) => {
  // Rendu très lent
  const start = performance.now();
  while (performance.now() - start < 100) {}

  return (
    <div style={{ 
      width: '100%', height: 200, background: '#f0f0f0', 
      display: 'flex', alignItems: 'center', justifyContent: 'center',
      marginTop: 20
    }}>
      <div style={{ 
        width: points / 10, height: points / 10, 
        background: 'tomato', borderRadius: '50%' 
      }} />
      <span style={{ position: 'absolute' }}>{points} Points Dessinés</span>
    </div>
  );
});

export function ChartController() {
  const [points, setPoints] = useState(500);
  const deferredPoints = useDeferredValue(points);

  return (
    <div>
      <label>
        Nombre de points : {points}
        <input 
          type="range" min="100" max="2000" 
          value={points} 
          onChange={e => setPoints(Number(e.target.value))} 
          style={{ width: '100%' }}
        />
      </label>

      {/* Le graphique utilise la valeur différée */}
      <div style={{ opacity: points !== deferredPoints ? 0.5 : 1 }}>
        <Chart points={deferredPoints} />
      </div>
    </div>
  );
}
```
</details>

### Exercice 3 - Le Double Input (Comparaison) {#exercice-3---le-double-input}

🎯 **Objectif** : Comparer visuellement et techniquement deux approches.

💼 **Mise en situation** : Pour convaincre votre chef technique, vous faites une démo "Avec vs Sans" optimisation.

📝 **Énoncé** :
1. Créez deux sections côte à côte.
2. Gauche : "Mode Bloquant". L'input passe directement sa valeur à un composant lent.
3. Droite : "Mode Fluide". L'input passe une `useDeferredValue` au composant lent.
4. Tapez rapidement dans les deux inputs et observez la différence de réactivité du curseur.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState, useDeferredValue, memo } from 'react';

// Composant qui coûte cher
const HeavyComponent = memo(({ text }: { text: string }) => {
  const now = performance.now();
  while (performance.now() - now < 50) {} // 50ms lag
  return <div style={{ wordBreak: 'break-all' }}>Rendu : {text.length} chars</div>;
});

export function ComparisonDemo() {
  const [text1, setText1] = useState('');
  const [text2, setText2] = useState('');
  const deferredText2 = useDeferredValue(text2);

  return (
    <div style={{ display: 'flex', gap: 20 }}>
      
      {/* Colonne 1 : Bloquante */}
      <div style={{ flex: 1, border: '1px solid red', padding: 10 }}>
        <h3>❌ Bloquant</h3>
        <input 
          value={text1} 
          onChange={e => setText1(e.target.value)} 
          placeholder="Tapez ici..."
        />
        <p>Le curseur va laguer.</p>
        <HeavyComponent text={text1} />
      </div>

      {/* Colonne 2 : Différée */}
      <div style={{ flex: 1, border: '1px solid green', padding: 10 }}>
        <h3>✅ Différé (Fluid)</h3>
        <input 
          value={text2} 
          onChange={e => setText2(e.target.value)}
          placeholder="Tapez ici..."
        />
        <p>Le curseur reste fluide.</p>
        <div style={{ opacity: text2 !== deferredText2 ? 0.5 : 1 }}>
          <HeavyComponent text={deferredText2} />
        </div>
      </div>

    </div>
  );
}
```
</details>
```