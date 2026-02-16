Voici le chapitre **API React: `act`, `lazy`, `memo`, `createContext`, `use`** pour la formation React 19.2.

```markdown
---
sidebar_label: API React: `act`, `lazy`, `memo`, `createContext`, `use`
sidebar_position: 42
---

# Chapitre 42 : API React: `act`, `lazy`, `memo`, `createContext`, `use`

Tests unitaires, Chargement paresseux, Mémoïsation, Contexte, Lecture de valeurs de Context/Promise

Au-delà des Hooks (`useState`, `useEffect`), React expose des API de haut niveau (Top-Level APIs) essentielles pour structurer, optimiser et tester vos applications.

React 19 introduit notamment l'API **`use`**, une révolution qui assouplit les règles strictes des Hooks pour la consommation de ressources (Contextes et Promesses).

---

## 1. `createContext` et `use` (Nouveau) {#createcontext-et-use}

### 1. Quoi
*   **`createContext`** : Crée un espace de données partagé (un "tuyau") qui traverse l'arbre des composants sans passer par les props.
*   **`use`** (React 19) : Une API universelle pour **lire** la valeur d'une ressource. Elle remplace `useContext` et permet aussi de lire le résultat d'une Promesse (`Promise`).

### 2. Pourquoi
L'ancien Hook `useContext` devait respecter les règles des Hooks (pas de conditions, pas de boucles). L'API `use` est plus flexible : elle peut être appelée **conditionnellement**.
De plus, `use` permet d'intégrer des données asynchrones (Promesses) directement dans le rendu, en suspendant le composant via `<Suspense>`.

### 3. Comment

#### A. Création et Fourniture (Classique)

```tsx
import { createContext } from 'react';

// 1. Définition du type
type Theme = 'light' | 'dark';

// 2. Création du contexte avec une valeur par défaut
export const ThemeContext = createContext<Theme>('light');

// 3. Provider (dans un composant parent)
function App() {
  return (
    <ThemeContext.Provider value="dark">
      <Toolbar />
    </ThemeContext.Provider>
  );
}
```

#### B. Consommation avec `use` (Moderne React 19)

Contrairement à `useContext`, `use` peut être placé dans un `if`.

```tsx
import { use } from 'react';
import { ThemeContext } from './App';

function Toolbar({ showTheme }: { showTheme: boolean }) {
  if (showTheme) {
    // ✅ VALIDE avec `use` (Interdit avec `useContext`)
    const theme = use(ThemeContext);
    return <div>Thème actuel : {theme}</div>;
  }
  
  return null;
}
```

#### C. Consommation de Promesses avec `use`

```tsx
import { use, Suspense } from 'react';

// Une promesse (idéalement créée en dehors du rendu ou via une lib/Server Component)
const dataPromise = fetch('/api/user').then(r => r.json());

function UserProfile() {
  // Suspend le composant tant que la promesse n'est pas résolue
  const user = use(dataPromise);
  return <h1>Bonjour {user.name}</h1>;
}

function Page() {
  return (
    <Suspense fallback="Chargement...">
      <UserProfile />
    </Suspense>
  );
}
```

### 🚨 Limitations de `use`
Si vous passez une Promesse à `use`, cette promesse doit avoir été créée **avant** le rendu du composant (par exemple dans un Server Component ou une bibliothèque de cache).
Si vous créez la promesse *dans* le composant :
```tsx
// ❌ DANGER : Boucle infinie
const data = use(fetch('/api').then(...)); 
```
Le composant re-rend -> nouvelle promesse -> `use` suspend -> re-rend -> nouvelle promesse...

---

## 2. `memo` {#memo}

### 1. Quoi
`memo` est un HOC (Higher-Order Component) qui permet de "mémoïser" un composant. Il modifie le comportement par défaut de React : le composant ne se re-rendra **que si ses props ont changé**.

### 2. Pourquoi
Par défaut, quand un parent se re-rend, **tous** ses enfants se re-rendent, même si leurs props sont identiques. Pour des composants lourds ou des listes immenses, c'est du gaspillage CPU. `memo` agit comme un bouclier.

### 3. Comment

#### A. Syntaxe de base

```tsx
import { memo, useState } from 'react';

// Ce composant ne se re-rendra que si `name` change
const Greeting = memo(function Greeting({ name }: { name: string }) {
  console.log("Rendu de Greeting");
  return <h1>Bonjour, {name} !</h1>;
});

export function App() {
  const [count, setCount] = useState(0);

  return (
    <div>
      {/* Le clic met à jour `count` dans App */}
      <button onClick={() => setCount(c => c + 1)}>Compteur : {count}</button>
      
      {/* Greeting ne se re-rend PAS, car la prop "Alice" n'a pas changé */}
      <Greeting name="Alice" />
    </div>
  );
}
```

### 4. Zone de Danger : La Comparaison Superficielle (Shallow Compare)

`memo` compare les props avec `prevProp === nextProp`. Attention aux objets et fonctions !

```tsx
// ❌ Le bouclier est brisé !
<Greeting user={{ name: "Alice" }} /> 
// L'objet `{ name: "Alice" }` est une NOUVELLE référence à chaque rendu du parent.
// `memo` voit que la référence a changé -> re-rendu.
```

✅ **Solution** : Utilisez `useMemo` pour les objets et `useCallback` pour les fonctions passés en props.

---

## 3. `lazy` {#lazy}

### 1. Quoi
`lazy` permet de différer le chargement du code d'un composant jusqu'à ce qu'il soit affiché pour la première fois. C'est du "Code Splitting".

### 2. Pourquoi
Pour réduire la taille du bundle JavaScript initial (`bundle.js`). Pourquoi charger le code du "Panneau Admin" pour un visiteur qui est sur la page d'accueil ?

### 3. Comment
`lazy` fonctionne obligatoirement avec `<Suspense>` pour gérer l'état de chargement pendant que le navigateur télécharge le fichier JS manquant.

```tsx
import { lazy, Suspense, useState } from 'react';

// Le fichier './AdminPanel.js' n'est pas téléchargé tout de suite
const AdminPanel = lazy(() => import('./AdminPanel'));

export function App() {
  const [isAdmin, setIsAdmin] = useState(false);

  return (
    <div>
      <button onClick={() => setIsAdmin(true)}>Ouvrir Admin</button>

      {isAdmin && (
        // Le téléchargement commence ici, au moment du rendu conditionnel
        <Suspense fallback={<div>Téléchargement du module Admin... 🧹</div>}>
          <AdminPanel />
        </Suspense>
      )}
    </div>
  );
}
```

---

## 4. `act` {#act}

### 1. Quoi
`act` est un utilitaire de **test** (fourni par `react` pour les environnements de test comme Jest/Vitest). Il garantit que toutes les mises à jour liées à React (effets, promesses, changements d'état) sont traitées et appliquées au DOM avant de faire vos assertions (expect).

### 2. Pourquoi
Sans `act`, vos tests seraient fragiles ("flaky"). Vous pourriez essayer de vérifier qu'un texte est apparu alors que React n'a pas encore fini son rendu.
*Note : Si vous utilisez **React Testing Library**, `act` est déjà intégré dans la plupart des fonctions (`render`, `fireEvent`, `userEvent`), vous l'utilisez donc rarement explicitement.*

### 3. Comment

```tsx
// Exemple de test unitaire (Jest/Vitest) sans librairie de haut niveau
import { act } from 'react';
import { createRoot } from 'react-dom/client';

it("incrémente le compteur", async () => {
  const container = document.createElement('div');
  const root = createRoot(container);
  
  // Prépare le DOM
  await act(async () => {
    root.render(<Counter />);
  });

  const button = container.querySelector('button');
  
  // Déclenche l'événement et attend que React traite le setState
  await act(async () => {
    button?.dispatchEvent(new MouseEvent('click', { bubbles: true }));
  });

  expect(container.textContent).toBe("Compteur : 1");
});
```

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-42}

1.  **Quelle est la différence fondamentale entre `useContext` et `use` ?**
    `use` peut être appelé conditionnellement (dans des boucles ou des `if`), alors que `useContext` doit respecter strictement l'ordre des Hooks. De plus, `use` peut lire des Promesses.

2.  **Quand `memo` est-il inefficace ?**
    Si les props passées au composant sont des nouvelles références à chaque rendu (objets littéraux, fonctions inline) et qu'elles ne sont pas stabilisées par `useMemo` ou `useCallback` dans le parent.

3.  **Pourquoi `lazy` nécessite-t-il `<Suspense>` ?**
    Car `lazy` charge un composant de manière asynchrone (via le réseau). React a besoin de savoir quoi afficher (le `fallback`) pendant ce délai.

---

## Exercices : {#exercices-42}

### Exercice 1 - Le Widget Météo Paresseux (`lazy`) {#exercice-1---le-widget-meteo-paresseux}

🎯 **Objectif** : Implémenter du Code Splitting pour un composant lourd.

💼 **Mise en situation** : Votre tableau de bord SaaS est rapide, mais le widget "Graphiques Avancés" pèse 2Mo. On ne veut le charger que si l'utilisateur clique sur "Voir les stats".

📝 **Énoncé** :
1. Simulez un composant lourd `HeavyChart` (export par défaut, avec un simple `<div>Graphique</div>`).
2. Dans `Dashboard`, importez-le avec `lazy`.
3. Utilisez un état `showChart` pour l'afficher.
4. Entourez-le d'un `Suspense` avec un fallback "Chargement du module...".
5. Utilisez l'inspecteur réseau du navigateur pour vérifier que le chunk JS n'est chargé qu'au clic.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { lazy, Suspense, useState } from 'react';

// Simulation de l'import dynamique
// Dans un vrai projet : const HeavyChart = lazy(() => import('./HeavyChart'));
const HeavyChart = lazy(() => {
  return new Promise<{ default: React.ComponentType }>(resolve => {
    setTimeout(() => {
      resolve({
        default: () => <div style={{ height: 200, background: '#eee' }}>📊 Gros Graphique 3D</div>
      });
    }, 1000); // Délai artificiel pour voir le fallback
  });
});

export function Dashboard() {
  const [show, setShow] = useState(false);

  return (
    <div style={{ padding: 20 }}>
      <h1>Dashboard SaaS</h1>
      <button onClick={() => setShow(true)}>
        Voir les Statistiques Avancées
      </button>

      <div style={{ marginTop: 20 }}>
        {show && (
          <Suspense fallback={<div style={{ color: 'blue' }}>Téléchargement du code JS... ⏳</div>}>
            <HeavyChart />
          </Suspense>
        )}
      </div>
    </div>
  );
}
```
</details>

### Exercice 2 - Optimisation de Liste (`memo`) {#exercice-2---optimisation-de-liste}

🎯 **Objectif** : Empêcher le re-rendu inutile d'enfants.

💼 **Mise en situation** : Une liste de produits e-commerce. Quand on change le filtre "Couleur" (état global), tous les composants `ProductCard` se re-rendent, même s'ils ne dépendent pas de ce filtre.

📝 **Énoncé** :
1. Créez un composant `ProductCard` qui prend `name` et affiche un `console.log("Rendu Card", name)`.
2. Dans le parent, une liste de produits et un bouton qui incrémente un `count` sans rapport.
3. Cliquez sur le bouton : observez les logs (tout se re-rend).
4. Enveloppez `ProductCard` avec `memo`.
5. Cliquez sur le bouton : les logs disparaissent.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Console développeur.
> **Annotation** : Montrez qu'après avoir cliqué sur le bouton, aucun nouveau log "Rendu Card" n'apparaît.
> **Alt Text suggéré** : Démonstration de React memo bloquant les re-rendus.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState, memo } from 'react';

// Composant optimisé
const ProductCard = memo(function ProductCard({ name }: { name: string }) {
  console.log(`♻️ Rendu de ${name}`); // Log pour preuve
  return (
    <div style={{ border: '1px solid #ccc', padding: 10, margin: 5 }}>
      {name}
    </div>
  );
});

export function ProductList() {
  const [likes, setLikes] = useState(0);
  const products = ["Chaise", "Table", "Lampe"];

  return (
    <div>
      <div style={{ marginBottom: 20 }}>
        {/* Ce changement d'état cause un re-rendu du parent ProductList */}
        <button onClick={() => setLikes(l => l + 1)}>
          Likes globaux : {likes} ❤️
        </button>
      </div>

      <div style={{ display: 'flex' }}>
        {products.map(p => (
          // Grâce à memo, ces composants ne se re-rendent PAS quand 'likes' change
          // car leurs props 'name' restent strictement égales (string primitive).
          <ProductCard key={p} name={p} />
        ))}
      </div>
    </div>
  );
}
```
</details>

### Exercice 3 - Contexte Conditionnel (`use`) {#exercice-3---contexte-conditionnel}

🎯 **Objectif** : Utiliser `use` dans une boucle ou une condition.

💼 **Mise en situation** : Un système de permissions. Vous avez une liste de fonctionnalités. Certaines nécessitent de lire le `UserContext` pour vérifier les droits, d'autres sont publiques.

📝 **Énoncé** :
1. Créez un `UserContext` avec `{ role: 'admin' }`.
2. Créez un composant `Feature` qui prend `requiresAuth` (bool).
3. Si `requiresAuth` est vrai, utilisez `use(UserContext)` pour afficher le rôle. Sinon, affichez "Public".
4. Tentez de faire la même chose avec `useContext` (ça devrait être impossible ou nécessiter une refonte).

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { createContext, use } from 'react';

const UserContext = createContext({ name: 'Bob', role: 'Admin' });

function FeatureItem({ label, requiresAuth }: { label: string, requiresAuth: boolean }) {
  let info = "Accès Public";

  // ✨ MAGIE DE REACT 19 : `use` dans une condition !
  // Impossible avec useContext
  if (requiresAuth) {
    const user = use(UserContext);
    info = `Restreint (${user.role})`;
  }

  return (
    <li style={{ padding: 5 }}>
      <strong>{label}</strong> : {info}
    </li>
  );
}

export function FeatureList() {
  return (
    <UserContext.Provider value={{ name: 'Alice', role: 'Manager' }}>
      <h3>Liste des fonctionnalités</h3>
      <ul>
        <FeatureItem label="Page Accueil" requiresAuth={false} />
        <FeatureItem label="Dashboard" requiresAuth={true} />
        <FeatureItem label="Contact" requiresAuth={false} />
        <FeatureItem label="Paramètres" requiresAuth={true} />
      </ul>
    </UserContext.Provider>
  );
}
```
</details>
```