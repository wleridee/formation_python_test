Voici le chapitre **`useReducer` : Extraire la Logique d'État** pour la formation React 19.2.

```markdown
---
sidebar_label: `useReducer` : Extraire la Logique d'État
sidebar_position: 29
---

# Chapitre 29 : `useReducer` : Extraire la Logique d'État

Hook `useReducer`, Fonctions reducer, Actions d'état complexes, Avantages de `useReducer`

Jusqu'à présent, nous avons géré l'état avec `useState`. C'est parfait pour des valeurs simples comme des compteurs, des champs de texte ou des booléens.
Cependant, lorsque votre composant grandit, la logique de mise à jour de l'état peut devenir dispersée et complexe. Si vous avez plusieurs variables d'état qui changent ensemble (ex: `isLoading`, `error`, `data`), `useState` peut devenir difficile à maintenir.

`useReducer` est une alternative puissante à `useState`, conçue pour gérer des états complexes et consolider la logique de mise à jour en un seul endroit.

## Le Hook `useReducer` et la Fonction Reducer {#le-hook-usereducer-et-la-fonction-reducer}

### 1. Quoi
`useReducer` est un Hook qui permet de gérer l'état d'un composant à travers un modèle "reducer".
Une **fonction reducer** est une fonction pure qui prend l'état actuel et une action, et retourne le nouvel état.

Signature du Hook :
```tsx
const [state, dispatch] = useReducer(reducer, initialState);
```

### 2. Pourquoi
Au lieu de dire à React "remplace cette valeur par celle-ci" (approche impérative de `useState`), vous dites à React "voici ce qui vient de se passer" (approche déclarative via une **Action**). Le **Reducer** décide alors comment l'état doit changer en réponse à cette action.

Cela permet de :
1.  **Centraliser la logique** : Tout est dans la fonction reducer, pas éparpillé dans les event handlers.
2.  **Faciliter les tests** : Le reducer est une fonction pure JS, testable sans React.
3.  **Gérer des états dépendants** : Modifier plusieurs propriétés en une seule action.

### 3. Comment

#### A. Syntaxe de base : Compteur

```tsx
import { useReducer } from 'react';

// 1. Définir l'état initial et les types
interface State { count: number }
type Action = { type: 'increment' } | { type: 'decrement' } | { type: 'reset' };

// 2. La fonction Reducer (Pure : (state, action) => newState)
function counterReducer(state: State, action: Action): State {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    case 'reset':
      return { count: 0 };
    default:
      return state;
  }
}

// 3. Le Composant
function Counter() {
  const [state, dispatch] = useReducer(counterReducer, { count: 0 });

  return (
    <>
      Count: {state.count}
      {/* On envoie des "messages" (actions) */}
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'reset' })}>Reset</button>
    </>
  );
}
```

#### B. Cas concret : Formulaire avec états multiples
Imaginez un formulaire qui doit gérer : la valeur saisie, le statut de chargement, et les erreurs éventuelles.

```tsx
import { useReducer } from 'react';

// État complexe
interface FormState {
  username: string;
  status: 'idle' | 'loading' | 'success' | 'error';
  error: string | null;
}

// Actions discriminantes (Discriminated Unions)
type FormAction =
  | { type: 'TYPING'; payload: string }
  | { type: 'SUBMIT' }
  | { type: 'SUCCESS' }
  | { type: 'ERROR'; payload: string };

const initialState: FormState = {
  username: '',
  status: 'idle',
  error: null,
};

function formReducer(state: FormState, action: FormAction): FormState {
  switch (action.type) {
    case 'TYPING':
      return { 
        ...state, 
        username: action.payload, 
        error: null // On nettoie l'erreur quand l'user tape
      }; 
    case 'SUBMIT':
      return { ...state, status: 'loading', error: null };
    case 'SUCCESS':
      return { ...state, status: 'success' };
    case 'ERROR':
      return { ...state, status: 'error', error: action.payload };
    default:
      return state;
  }
}

export function RegistrationForm() {
  const [state, dispatch] = useReducer(formReducer, initialState);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    dispatch({ type: 'SUBMIT' });
    
    try {
      await fakeApiCall(state.username);
      dispatch({ type: 'SUCCESS' });
    } catch (err: any) {
      dispatch({ type: 'ERROR', payload: err.message });
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={state.username}
        onChange={(e) => dispatch({ type: 'TYPING', payload: e.target.value })}
        disabled={state.status === 'loading'}
      />
      <button type="submit" disabled={state.status === 'loading'}>
        {state.status === 'loading' ? 'Envoi...' : "S'inscrire"}
      </button>
      
      {state.status === 'error' && <p style={{ color: 'red' }}>{state.error}</p>}
      {state.status === 'success' && <p style={{ color: 'green' }}>Bienvenue !</p>}
    </form>
  );
}

// Mock API helper
const fakeApiCall = (user: string) => new Promise((resolve, reject) => {
  setTimeout(() => user.length > 3 ? resolve(true) : reject(new Error("Nom trop court")), 1000);
});
```

### 4. Zone de Danger

:::danger Règle d'Immutabilité
Le reducer **doit être une fonction pure**.
Il ne doit **JAMAIS** modifier l'état directement. Il doit toujours retourner un **nouvel objet**.

❌ **Mauvais (Mutation) :**
```javascript
state.count = state.count + 1;
return state;
```

✅ **Bon (Copie) :**
```javascript
return { ...state, count: state.count + 1 };
```
:::

:::warning Pas d'Effets Secondaires
Ne faites **JAMAIS** d'appels API ou de `setTimeout` à l'intérieur du reducer.
Le reducer ne fait que calculer le prochain état. Les effets secondaires restent dans `useEffect` ou dans les gestionnaires d'événements.
:::

---

## `useState` vs `useReducer` : Tableau Comparatif {#usestate-vs-usereducer}

| Critère | `useState` | `useReducer` |
| :--- | :--- | :--- |
| **Complexité de l'état** | Simple (primitifs: nombre, string, bool) | Complexe (objets imbriqués, tableaux) |
| **Transitions** | Simples, indépendantes | Complexes, interdépendantes |
| **Logique métier** | Dans le composant (Event Handlers) | Séparée (Fonction Reducer) |
| **Testabilité** | Difficile d'isoler la logique | Facile (test unitaire de la fonction pure) |
| **Taille du code** | Plus concis pour les cas simples | Plus verbeux (boilerplate switch/case) |
| **Débogage** | On voit l'état changer | On voit **pourquoi** l'état change (via l'action) |

---

## Pattern : Immer pour simplifier l'immutabilité {#pattern-immer}

### 1. Quoi
Manipuler des objets imbriqués ou des tableaux de manière immuable (`...state, items: [...state.items, newItem]`) est verbeux et propice aux erreurs.
La bibliothèque `use-immer` (basée sur `useReducer`) permet d'écrire du code "mutant" qui est converti en mises à jour immuables sécurisées.

### 2. Comment

```tsx
// Exemple conceptuel (nécessite 'npm install use-immer')
import { useImmerReducer } from 'use-immer';

function taskReducer(draft, action) {
  switch (action.type) {
    case 'add':
      // 😌 On peut faire un "push" direct sur le brouillon (draft)
      draft.push({ id: action.id, text: action.text, done: false });
      break;
    case 'toggle':
      const task = draft.find(t => t.id === action.id);
      task.done = !task.done; // 😌 Mutation directe autorisée sur le draft
      break;
  }
}
```
*Note : React 19 n'inclut pas Immer par défaut, mais c'est une pratique très courante dans l'écosystème.*

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-29}

1.  **Quels sont les deux arguments que reçoit une fonction reducer ?**
    L'état actuel (`state`) et l'action (`action`). Elle retourne le nouvel état.

2.  **Que contient généralement l'objet `action` ?**
    Une propriété `type` (obligatoire par convention pour décrire l'événement) et souvent une propriété `payload` (pour les données associées).

3.  **Pourquoi est-il préférable d'utiliser `useReducer` pour un formulaire complexe ?**
    Pour éviter d'avoir 5 ou 6 variables `useState` séparées et garantir que les mises à jour interdépendantes (ex: mettre `isLoading` à true et effacer `error`) se font de manière atomique et cohérente.

4.  **Une fonction reducer peut-elle lancer une requête Fetch ?**
    **Non**. Le reducer doit être une fonction pure. Les appels API doivent être faits avant le `dispatch` (dans le handler) ou dans un `useEffect`.

---

## Exercices : {#exercices-29}

### Exercice 1 - Le Contrôleur de Lecteur Audio {#exercice-1---le-controleur-de-lecteur-audio}

🎯 **Objectif** : Implémenter une machine à états simple avec `useReducer`.

💼 **Mise en situation** : Vous créez un lecteur de podcast. Il a 3 états possibles : `STOPPED`, `PLAYING`, `PAUSED`.
Les transitions sont strictes :
- On ne peut pas passer de `STOPPED` à `PAUSED`.
- `Play` depuis `STOPPED` met le volume à 50.
- `Stop` remet le temps à 0.

📝 **Énoncé** :
1. Définissez les types `AudioState` (status, volume, time) et `AudioAction`.
2. Créez le reducer qui respecte les règles ci-dessus.
3. Créez l'UI avec 3 boutons (Play, Pause, Stop) et l'affichage de l'état.

📺 **Résultat attendu** :
Cliquer sur "Pause" quand on est "Stopped" ne doit rien faire. Cliquer sur "Stop" doit réinitialiser le temps.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useReducer } from 'react';

type AudioStatus = 'STOPPED' | 'PLAYING' | 'PAUSED';

interface AudioState {
  status: AudioStatus;
  volume: number;
  time: number;
}

type AudioAction = 
  | { type: 'PLAY' }
  | { type: 'PAUSE' }
  | { type: 'STOP' }
  | { type: 'TICK' }; // Simule l'avancement du temps

const initialState: AudioState = {
  status: 'STOPPED',
  volume: 50,
  time: 0
};

function audioReducer(state: AudioState, action: AudioAction): AudioState {
  switch (action.type) {
    case 'PLAY':
      if (state.status === 'PLAYING') return state;
      return { 
        ...state, 
        status: 'PLAYING',
        // Si on vient de STOPPED, on reset le volume (règle métier)
        volume: state.status === 'STOPPED' ? 50 : state.volume 
      };
    
    case 'PAUSE':
      // Règle métier : impossible de mettre en pause si on est stoppé
      if (state.status === 'STOPPED') return state;
      return { ...state, status: 'PAUSED' };

    case 'STOP':
      return { ...state, status: 'STOPPED', time: 0 };

    case 'TICK':
      if (state.status !== 'PLAYING') return state;
      return { ...state, time: state.time + 1 };
      
    default:
      return state;
  }
}

export function PodcastPlayer() {
  const [state, dispatch] = useReducer(audioReducer, initialState);

  return (
    <div style={{ border: '1px solid #ccc', padding: 20, borderRadius: 8 }}>
      <h3>Lecteur React 19 🎧</h3>
      <div style={{ marginBottom: 10 }}>
        Statut : <strong>{state.status}</strong> | Temps : {state.time}s | Volume : {state.volume}%
      </div>
      
      <div style={{ display: 'flex', gap: 10 }}>
        <button onClick={() => dispatch({ type: 'PLAY' })}>▶ Play</button>
        <button onClick={() => dispatch({ type: 'PAUSE' })}>⏸ Pause</button>
        <button onClick={() => dispatch({ type: 'STOP' })}>⏹ Stop</button>
        <button onClick={() => dispatch({ type: 'TICK' })}>+1s (Simulate)</button>
      </div>
    </div>
  );
}
```
</details>

### Exercice 2 - Le Panier E-commerce (CRUD Array) {#exercice-2---le-panier-e-commerce}

🎯 **Objectif** : Manipuler un tableau d'objets de manière immuable dans un reducer.

💼 **Mise en situation** : Un panier d'achat classique. On doit pouvoir ajouter un produit, supprimer un produit, et changer la quantité.

📝 **Énoncé** :
1. State : `{ items: Array<{id, name, price, qty}>, total: number }`.
2. Actions : `ADD_ITEM`, `REMOVE_ITEM`, `UPDATE_QTY`.
3. Le `total` doit être recalculé automatiquement à chaque action (C'est un avantage du reducer : données dérivées synchrones).
4. Attention : `ADD_ITEM` doit incrémenter la quantité si le produit existe déjà.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useReducer } from 'react';

type CartItem = { id: number; name: string; price: number; qty: number };
type CartState = { items: CartItem[]; total: number };

type CartAction =
  | { type: 'ADD_ITEM'; payload: { id: number; name: string; price: number } }
  | { type: 'REMOVE_ITEM'; payload: number } // id
  | { type: 'UPDATE_QTY'; payload: { id: number; qty: number } };

// Helper pour recalculer le total
const calculateTotal = (items: CartItem[]) => 
  items.reduce((acc, item) => acc + item.price * item.qty, 0);

function cartReducer(state: CartState, action: CartAction): CartState {
  let newItems: CartItem[];

  switch (action.type) {
    case 'ADD_ITEM':
      const existingIndex = state.items.findIndex(i => i.id === action.payload.id);
      if (existingIndex >= 0) {
        // Le produit existe, on update la quantité (copie immuable !)
        newItems = [...state.items];
        newItems[existingIndex] = {
          ...newItems[existingIndex],
          qty: newItems[existingIndex].qty + 1
        };
      } else {
        // Nouveau produit
        newItems = [...state.items, { ...action.payload, qty: 1 }];
      }
      return { items: newItems, total: calculateTotal(newItems) };

    case 'REMOVE_ITEM':
      newItems = state.items.filter(i => i.id !== action.payload);
      return { items: newItems, total: calculateTotal(newItems) };

    case 'UPDATE_QTY':
      newItems = state.items.map(item => 
        item.id === action.payload.id 
          ? { ...item, qty: Math.max(0, action.payload.qty) } // Pas de qty négative
          : item
      );
      return { items: newItems, total: calculateTotal(newItems) };
      
    default:
      return state;
  }
}

export function ShoppingCart() {
  const [cart, dispatch] = useReducer(cartReducer, { items: [], total: 0 });

  return (
    <div>
      <h2>Votre Panier ({cart.total} €)</h2>
      <button onClick={() => dispatch({ type: 'ADD_ITEM', payload: { id: 1, name: 'Livre React', price: 30 } })}>
        Ajouter Livre (30€)
      </button>
      <button onClick={() => dispatch({ type: 'ADD_ITEM', payload: { id: 2, name: 'Clavier', price: 50 } })}>
        Ajouter Clavier (50€)
      </button>

      <ul>
        {cart.items.map(item => (
          <li key={item.id}>
            {item.name} - {item.price}€ x 
            <input 
              type="number" 
              value={item.qty} 
              onChange={(e) => dispatch({ type: 'UPDATE_QTY', payload: { id: item.id, qty: parseInt(e.target.value) } })}
              style={{ width: 40, margin: '0 10px' }}
            />
            <button onClick={() => dispatch({ type: 'REMOVE_ITEM', payload: item.id })}>Suppr.</button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```
</details>

### Exercice 3 - Le Wizard d'Installation {#exercice-3---le-wizard-d-installation}

🎯 **Objectif** : Gérer une interface par étapes (Step 1 -> Step 2 -> Step 3).

💼 **Mise en situation** : Un assistant de configuration SaaS. On ne peut pas aller à l'étape suivante si les données de l'étape actuelle sont invalides. On peut toujours revenir en arrière.

📝 **Énoncé** :
1. State: `{ step: number, data: { name: string, plan: string }, error: string | null }`.
2. Actions: `NEXT_STEP`, `PREV_STEP`, `SET_NAME`, `SET_PLAN`.
3. Logique : `NEXT_STEP` vérifie si le `name` est rempli avant de passer à l'étape 2. Si non, remplit `error`.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Un formulaire multi-étapes simple.
> **Annotation** : Montrez l'état où l'utilisateur essaie de passer à l'étape suivante sans remplir le champ, déclenchant un message d'erreur rouge géré par le reducer.
> **Alt Text suggéré** : Interface de wizard montrant un message d'erreur de validation.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useReducer } from 'react';

type WizardState = {
  step: number;
  data: { name: string; plan: string };
  error: string | null;
};

type WizardAction =
  | { type: 'SET_NAME'; payload: string }
  | { type: 'SET_PLAN'; payload: string }
  | { type: 'NEXT_STEP' }
  | { type: 'PREV_STEP' };

function wizardReducer(state: WizardState, action: WizardAction): WizardState {
  switch (action.type) {
    case 'SET_NAME':
      return { ...state, data: { ...state.data, name: action.payload }, error: null };
    case 'SET_PLAN':
      return { ...state, data: { ...state.data, plan: action.payload }, error: null };
    
    case 'NEXT_STEP':
      // Validation Logique dans le reducer
      if (state.step === 1 && state.data.name.trim() === '') {
        return { ...state, error: 'Le nom est obligatoire !' };
      }
      if (state.step === 2 && state.data.plan === '') {
        return { ...state, error: 'Veuillez choisir un plan.' };
      }
      return { ...state, step: state.step + 1, error: null };

    case 'PREV_STEP':
      return { ...state, step: Math.max(1, state.step - 1), error: null };

    default:
      return state;
  }
}

export function SetupWizard() {
  const [state, dispatch] = useReducer(wizardReducer, {
    step: 1,
    data: { name: '', plan: '' },
    error: null
  });

  return (
    <div style={{ padding: 20, border: '2px solid #333', maxWidth: 400 }}>
      <h2>Étape {state.step} / 3</h2>
      
      {state.error && <div style={{ color: 'red', marginBottom: 10 }}>⚠️ {state.error}</div>}

      {state.step === 1 && (
        <input 
          placeholder="Votre Nom (Requis)"
          value={state.data.name}
          onChange={(e) => dispatch({ type: 'SET_NAME', payload: e.target.value })}
        />
      )}

      {state.step === 2 && (
        <select 
          value={state.data.plan}
          onChange={(e) => dispatch({ type: 'SET_PLAN', payload: e.target.value })}
        >
          <option value="">Choisir un plan...</option>
          <option value="free">Gratuit</option>
          <option value="pro">Pro</option>
        </select>
      )}

      {state.step === 3 && (
        <div>
          <h3>Récapitulatif</h3>
          <p>Nom : {state.data.name}</p>
          <p>Plan : {state.data.plan}</p>
          <p style={{ color: 'green' }}>Prêt à valider !</p>
        </div>
      )}

      <div style={{ marginTop: 20, display: 'flex', justifyContent: 'space-between' }}>
        <button 
          onClick={() => dispatch({ type: 'PREV_STEP' })}
          disabled={state.step === 1}
        >
          Précédent
        </button>
        
        {state.step < 3 && (
          <button onClick={() => dispatch({ type: 'NEXT_STEP' })}>
            Suivant
          </button>
        )}
      </div>
    </div>
  );
}
```
</details>
```