Voici le chapitre **Composants React Intégrés: `<Fragment>`, `<Profiler>`, `<StrictMode>`, `<Suspense>`** pour la formation React 19.2.

```markdown
---
sidebar_label: Composants React Intégrés
sidebar_position: 40
---

# Chapitre 40 : Composants React Intégrés: `<Fragment>`, `<Profiler>`, `<StrictMode>`, `<Suspense>`

Regroupement d'éléments, Mesure de performance, Détection d'erreurs, Chargement asynchrone

React ne se limite pas à créer vos propres composants. La bibliothèque fournit plusieurs composants intégrés ("built-in") qui jouent des rôles cruciaux dans la structure, la performance et le débogage de vos applications.

Ce chapitre explore ces outils fondamentaux que vous utiliserez quotidiennement.

---

## 1. `<Fragment>` (ou `<>...</>`) {#fragment}

### 1. Quoi
`<Fragment>` permet de regrouper une liste d'enfants sans ajouter de nœud supplémentaire (comme une `<div>`) dans le DOM.

### 2. Pourquoi
En HTML, un composant doit retourner **un seul élément racine**.
*   **Sans Fragment** : On ajoutait souvent des `<div>` inutiles ("div soup") juste pour satisfaire React, ce qui brisait parfois les layouts CSS (ex: Flexbox/Grid).
*   **Avec Fragment** : On retourne plusieurs éléments "transparents".

### 3. Comment

#### A. Syntaxe de base

```tsx
import { Fragment } from 'react';

function List() {
  return (
    <Fragment>
      <li>Pomme</li>
      <li>Poire</li>
    </Fragment>
  );
}

// Syntaxe courte (recommandée)
function ShortList() {
  return (
    <>
      <li>Pomme</li>
      <li>Poire</li>
    </>
  );
}
```

#### B. Cas concret : Layout CSS

```tsx
function TableRow() {
  // Si on mettait une <div> ici, la structure <table> serait invalide !
  return (
    <>
      <td>Jean</td>
      <td>Dupont</td>
    </>
  );
}

function Table() {
  return (
    <table>
      <tbody>
        <tr><TableRow /></tr>
      </tbody>
    </table>
  );
}
```

### 4. Zone de Danger

:::warning Syntaxe courte et attributs
La syntaxe courte `<>...</>` ne supporte **aucun attribut**.
Si vous devez passer une `key` (par exemple dans une boucle), vous **devez** utiliser `<Fragment key={id}>...</Fragment>`.
:::

```tsx
// ✅ Correct
{items.map(item => (
  <Fragment key={item.id}>
    <dt>{item.term}</dt>
    <dd>{item.def}</dd>
  </Fragment>
))}
```

---

## 2. `<Suspense>` {#suspense}

### 1. Quoi
`<Suspense>` permet d'afficher un contenu de repli ("fallback") déclaratif (comme un spinner) en attendant que ses enfants aient fini de charger.
Il gère :
*   Le chargement de code (Lazy Loading via `React.lazy`).
*   Le chargement de données (Data Fetching compatible Suspense, ex: via `use` ou des librairies comme React Query/Relay).

### 2. Pourquoi
Avant Suspense, chaque composant gérait son propre `if (isLoading) return <Spinner />`. Cela menait à des spinners qui clignotent partout ("popcorn UI").
Suspense permet de **coordonner** le chargement de plusieurs composants et d'afficher un seul indicateur de chargement cohérent.

### 3. Comment

#### A. Syntaxe de base

```tsx
import { Suspense, lazy } from 'react';

// Chargement paresseux du composant (code splitting)
const HeavyChart = lazy(() => import('./HeavyChart'));

function Dashboard() {
  return (
    <div>
      <h1>Tableau de bord</h1>
      <Suspense fallback={<div>Chargement du graphique... ⏳</div>}>
        <HeavyChart />
      </Suspense>
    </div>
  );
}
```

#### B. Cas concret : Streaming SSR (React 19)

Avec les Server Components et le Streaming SSR, `<Suspense>` permet au serveur d'envoyer le HTML de la page **immédiatement**, avec des "trous" pour les parties lentes. Dès que la partie lente est prête, React injecte le HTML et remplace le fallback.

```tsx
// Server Component (conceptuel)
import { Suspense } from 'react';
import { UserProfile, UserPosts } from './components';

export default function ProfilePage() {
  return (
    <main>
      {/* Chargement rapide */}
      <UserProfile /> 
      
      {/* Chargement lent (BDD) */}
      <Suspense fallback={<p>Chargement des posts...</p>}>
        <UserPosts />
      </Suspense>
    </main>
  );
}
```

### 🚨 Limitations de `<Suspense>`
1.  Il ne détecte pas automatiquement les `useEffect` asynchrones. Il ne fonctionne qu'avec des APIs spécifiques (Lazy, `use`, frameworks compatibles).
2.  Si une erreur survient pendant le chargement, Suspense ne l'attrape pas. Il faut l'envelopper dans un `<ErrorBoundary>`.

---

## 3. `<StrictMode>` {#strictmode}

### 1. Quoi
`<StrictMode>` est un outil de développement qui active des vérifications et des avertissements supplémentaires pour ses descendants. Il n'a aucun effet en production.

### 2. Pourquoi
Pour détecter les problèmes potentiels avant qu'ils ne deviennent des bugs :
*   Effets de bord inattendus.
*   Utilisation d'API dépréciées.
*   Problèmes de cycle de vie.

### 3. Comment

Il est généralement ajouté à la racine de l'application (par défaut avec Vite/Create React App).

```tsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import App from './App';

createRoot(document.getElementById('root')!).render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

#### L'effet "Double Rendu"
En mode développement, `<StrictMode>` exécute **deux fois** le rendu de chaque composant et **deux fois** les effets (`useEffect` : mount -> unmount -> mount).
C'est intentionnel ! Cela permet de vérifier que vos composants sont purs et que vos effets ont une fonction de nettoyage (`cleanup`) correcte.

> "Si ça casse parce que ça s'exécute deux fois, c'est que votre code est buggé."

---

## 4. `<Profiler>` {#profiler}

### 1. Quoi
`<Profiler>` mesure la fréquence et le coût (temps d'exécution) du rendu d'une partie de l'arbre React.

### 2. Pourquoi
Pour identifier les goulots d'étranglement de performance. "Pourquoi cette liste est-elle si lente à s'afficher ?"

### 3. Comment

Il accepte une prop `id` et une prop `onRender` (callback).

```tsx
import { Profiler } from 'react';

function onRenderCallback(
  id: string, // prop "id" du Profiler
  phase: "mount" | "update", // phase de rendu
  actualDuration: number, // temps passé à rendre ce sous-arbre
  baseDuration: number, // temps estimé sans mémoïsation
  startTime: number, 
  commitTime: number
) {
  console.log(`Profiler [${id}] - ${phase} : ${actualDuration.toFixed(2)}ms`);
}

function App() {
  return (
    <Profiler id="Sidebar" onRender={onRenderCallback}>
      <Sidebar />
    </Profiler>
  );
}
```

:::info Utilisation en Production
Contrairement aux autres outils de dev, le Profiler peut être utilisé en production (souvent désactivé par défaut pour économiser des ressources, nécessite un build spécifique de React), ce qui est utile pour du monitoring de performance réel (RUM).
:::

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-40}

1.  **Pourquoi `StrictMode` exécute-t-il les composants deux fois en développement ?**
    Pour exposer les effets de bord impurs et vérifier que les fonctions de nettoyage des effets fonctionnent correctement.

2.  **Quelle est la différence entre `<Fragment>` et `<div>` ?**
    `<Fragment>` ne crée aucun nœud DOM, il est "invisible" dans le rendu final HTML, alors que `<div>` ajoute un élément bloc qui peut affecter le style et la structure.

3.  **Que fait `<Suspense>` lorsqu'un composant enfant n'est pas encore prêt ?**
    Il suspend le rendu de l'enfant et affiche le composant passé dans la prop `fallback` à la place, jusqu'à ce que l'enfant soit résolu.

---

## Exercices : {#exercices-40}

### Exercice 1 - Nettoyage de la "Div Soup" {#exercice-1---nettoyage-de-la-div-soup}

🎯 **Objectif** : Utiliser `<Fragment>` pour corriger un layout CSS brisé.

💼 **Mise en situation** : Vous avez une grille CSS (`display: grid`) de 3 colonnes. Votre composant `CardGroup` retourne 3 cartes. Mais il les enveloppe dans une `div`, ce qui casse la grille (la `div` devient l'enfant de la grille, et non les cartes).

📝 **Énoncé** :
1. Créez un conteneur Grid.
2. Créez un composant `CardGroup` qui retourne 3 `<div>Carte</div>`.
3. Essayez d'abord en enveloppant les cartes dans une `<div>` parent : constatez le bug visuel.
4. Remplacez par `<>` (Fragment) : constatez que la grille s'aligne correctement.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { Fragment } from 'react';

function CardGroup() {
  // ❌ MAUVAIS : La div casse le layout grid du parent
  // return (
  //   <div>
  //     <div className="card">Carte 1</div>
  //     <div className="card">Carte 2</div>
  //     <div className="card">Carte 3</div>
  //   </div>
  // );

  // ✅ BON : Les enfants sont directement dans la grille
  return (
    <>
      <div style={{ border: '1px solid black', padding: 10 }}>Carte 1</div>
      <div style={{ border: '1px solid black', padding: 10 }}>Carte 2</div>
      <div style={{ border: '1px solid black', padding: 10 }}>Carte 3</div>
    </>
  );
}

export function GridApp() {
  return (
    <div style={{ 
      display: 'grid', 
      gridTemplateColumns: 'repeat(3, 1fr)', 
      gap: 10 
    }}>
      <CardGroup />
      <div style={{ border: '1px solid blue', padding: 10 }}>Autre Élément</div>
    </div>
  );
}
```
</details>

### Exercice 2 - Chargement Différé avec Suspense {#exercice-2---chargement-differe-avec-suspense}

🎯 **Objectif** : Simuler un chargement lent et afficher un fallback.

💼 **Mise en situation** : Un widget "Météo" est très lourd à charger. On veut afficher "Chargement météo..." pendant ce temps.

📝 **Énoncé** :
1. Créez un composant `WeatherWidget` qui simule un délai (utilisez `lazy` avec un `setTimeout` artificiel pour l'import).
2. Affichez-le dans `App` sans `Suspense` -> Erreur.
3. Enveloppez-le dans `<Suspense fallback="...">`.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { Suspense, lazy, useState, useEffect } from 'react';

// Simulation d'un composant chargé dynamiquement avec délai
// Note: En vrai projet, on ferait lazy(() => import('./Weather'))
const WeatherWidget = lazy(() => {
  return new Promise<{ default: React.ComponentType }>(resolve => {
    setTimeout(() => {
      resolve({
        default: () => <div style={{ background: '#aaf', padding: 20 }}>☀️ Il fait beau !</div>
      });
    }, 2000); // 2 secondes de délai
  });
});

export function WeatherApp() {
  return (
    <div>
      <h2>Ma Météo</h2>
      {/* Sans Suspense ici, React lancerait une erreur car WeatherWidget est une promesse */}
      <Suspense fallback={<div style={{ color: 'gray' }}>Chargement météo... ☁️</div>}>
        <WeatherWidget />
      </Suspense>
    </div>
  );
}
```
</details>

### Exercice 3 - L'espion Profiler {#exercice-3---l-espion-profiler}

🎯 **Objectif** : Mesurer le temps de rendu d'une liste.

💼 **Mise en situation** : Vous soupçonnez qu'un composant de liste est lent. Vous voulez voir combien de temps il prend pour "monter".

📝 **Énoncé** :
1. Créez une liste de 1000 éléments.
2. Enveloppez-la dans `<Profiler id="Liste" onRender={...}>`.
3. Affichez les métriques dans la console.
4. Forcez un re-rendu (ex: bouton compteur dans le parent) et comparez temps de `mount` vs `update`.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Console du navigateur.
> **Annotation** : Surlignez la ligne de log montrant "actualDuration" pour la phase "mount".
> **Alt Text suggéré** : Logs du Profiler React dans la console.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { Profiler, useState } from 'react';

export function PerformanceList() {
  const [count, setCount] = useState(0);

  const logPerformance = (
    id: string,
    phase: "mount" | "update",
    actualDuration: number
  ) => {
    console.log(`[${id}] Phase: ${phase} | Durée: ${actualDuration.toFixed(4)}ms`);
  };

  return (
    <div style={{ padding: 20 }}>
      <button onClick={() => setCount(c => c + 1)}>
        Forcer Update ({count})
      </button>

      <Profiler id="MaListeLourde" onRender={logPerformance}>
        <ul style={{ maxHeight: 200, overflow: 'auto', border: '1px solid #ccc' }}>
          {Array.from({ length: 1000 }).map((_, i) => (
            <li key={i}>Item #{i} - {Math.random()}</li>
          ))}
        </ul>
      </Profiler>
    </div>
  );
}
```
</details>
```