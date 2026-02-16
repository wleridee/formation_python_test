Voici le chapitre **Scaler avec `useReducer` et `useContext`** pour la formation React 19.2.

```markdown
---
sidebar_label: Scaler avec `useReducer` et `useContext`
sidebar_position: 31
---

# Chapitre 31 : Scaler avec `useReducer` et `useContext`

Gestion d'état globale, Architecture d'état, Optimisation de rendu

Vous savez maintenant gérer des états complexes avec `useReducer` et passer des données en profondeur avec `useContext`.
Que se passe-t-il si nous combinons les deux ? Nous obtenons une architecture puissante pour gérer l'état global de l'application, souvent appelée le pattern **"Redux-like"**, mais nativement intégré à React.

C'est la méthode standard pour scaler une application React sans installer de librairies tierces (comme Redux ou Zustand) tant que la complexité reste modérée.

## Fusionner Reducer et Context {#fusionner-reducer-et-context}

### 1. Quoi
Il s'agit de placer l'état (`state`) et la fonction de dispatch (`dispatch`) issus d'un `useReducer` dans un Contexte pour les rendre accessibles partout.

### 2. Pourquoi
Dans le chapitre sur `useReducer`, l'état était coincé dans le composant qui l'avait créé. Pour qu'un composant enfant puisse déclencher une action, il fallait passer `dispatch` via les props.
En combinant les deux :
1.  **L'état est global** (ou partagé sur une grande section).
2.  **La logique est centralisée** (dans le reducer).
3.  **L'accès est facile** : N'importe quel composant peut lire l'état ou envoyer une action sans *prop drilling*.

### 3. Comment

#### A. Structure recommandée
On sépare généralement la définition du Contexte, du Reducer et du Provider dans un même fichier ou module.

```tsx
// TasksContext.tsx
import { createContext, useContext, useReducer, ReactNode } from 'react';

// 1. Définir les Types
type Task = { id: number; text: string; done: boolean };
type State = Task[];
type Action = 
  | { type: 'added'; id: number; text: string }
  | { type: 'deleted'; id: number };

// 2. Créer le Reducer (Logique pure)
function tasksReducer(tasks: State, action: Action): State {
  switch (action.type) {
    case 'added':
      return [...tasks, { id: action.id, text: action.text, done: false }];
    case 'deleted':
      return tasks.filter(t => t.id !== action.id);
    default:
      return tasks;
  }
}

// 3. Créer le Context
// On stocke state ET dispatch
const TasksContext = createContext<{
  tasks: State;
  dispatch: React.Dispatch<Action>;
} | null>(null);

// 4. Créer le Provider
export function TasksProvider({ children }: { children: ReactNode }) {
  const [tasks, dispatch] = useReducer(tasksReducer, []);

  return (
    <TasksContext.Provider value={{ tasks, dispatch }}>
      {children}
    </TasksContext.Provider>
  );
}

// 5. Hook personnalisé pour consommer
export function useTasks() {
  const context = useContext(TasksContext);
  if (!context) throw new Error("useTasks must be used within TasksProvider");
  return context;
}
```

#### B. Utilisation dans les composants

```tsx
// AddTask.tsx
import { useState } from 'react';
import { useTasks } from './TasksContext';

export function AddTask() {
  const [text, setText] = useState('');
  const { dispatch } = useTasks(); // Accès direct au dispatch

  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <button onClick={() => {
        dispatch({ type: 'added', id: Date.now(), text });
        setText('');
      }}>Ajouter</button>
    </>
  );
}

// TaskList.tsx
import { useTasks } from './TasksContext';

export function TaskList() {
  const { tasks } = useTasks(); // Accès direct à l'état
  return (
    <ul>
      {tasks.map(task => <li key={task.id}>{task.text}</li>)}
    </ul>
  );
}
```

---

## Optimisation de Rendu : Séparer État et Dispatch {#optimisation-de-rendu}

### 1. Quoi
Au lieu d'un seul contexte contenant `{ state, dispatch }`, on crée **deux contextes distincts** :
1.  `StateContext` : change à chaque modification de donnée.
2.  `DispatchContext` : ne change **jamais** (la fonction dispatch de `useReducer` est stable).

### 2. Pourquoi
Imaginez un composant `<AddTask />` qui a juste besoin de `dispatch`. Il n'a pas besoin de lire la liste des tâches.
Si vous mettez tout dans le même contexte, quand la liste des tâches change, le contexte change, et `<AddTask />` se re-rend inutilement.
En séparant les contextes, les composants qui ne font qu'émettre des actions ne se re-rendent pas quand l'état change.

### 3. Comment

```tsx
import { createContext, useContext, useReducer } from 'react';

// Deux contextes séparés
const TasksContext = createContext<Task[] | null>(null);
const TasksDispatchContext = createContext<React.Dispatch<Action> | null>(null);

export function TasksProvider({ children }: { children: React.ReactNode }) {
  const [tasks, dispatch] = useReducer(tasksReducer, []);

  return (
    <TasksContext.Provider value={tasks}>
      <TasksDispatchContext.Provider value={dispatch}>
        {children}
      </TasksDispatchContext.Provider>
    </TasksContext.Provider>
  );
}

// Hooks séparés
export function useTasks() {
  return useContext(TasksContext)!;
}

export function useTasksDispatch() {
  return useContext(TasksDispatchContext)!;
}
```

### 4. Zone de Danger

:::danger Context Hell vs Composition
N'abusez pas de ce pattern en créant 50 Providers imbriqués à la racine de votre `App`.
❌ `<UserProvider><ThemeProvider><LangProvider><CartProvider><NotifProvider>...`
Si vous avez trop de Providers globaux, regroupez-les dans un composant `AppProviders` ou reconsidérez si cet état doit vraiment être global.
:::

### 🚨 Limitations de cette Approche
1.  **Sélecteurs manquants** : Contrairement à Redux ou Zustand, `useContext` ne permet pas de s'abonner à *une partie* de l'état (ex: `state.user.name`). Si n'importe quoi change dans `state`, le composant se re-rend.
2.  **Debugging** : React DevTools est moins puissant que Redux DevTools pour voyager dans le temps (Time Travel Debugging), bien que le reducer reste prédictible.

---

## Architecture Modulaire {#architecture-modulaire}

Pour garder le code propre, suivez cette structure de dossiers :

```text
src/
  context/
    AuthContext.tsx      (Context + Provider + Hook)
    ThemeContext.tsx
  reducers/
    authReducer.ts       (Fonction reducer pure + Types Action)
    cartReducer.ts
  components/
    ...
  App.tsx                (Regroupe les Providers)
```

Cela sépare la *logique métier* (reducers) de la *tuyauterie React* (contexts).

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-31}

1.  **Pourquoi combiner `useReducer` et `useContext` ?**
    Pour créer une gestion d'état globale centralisée sans prop drilling, simulant une architecture de type Redux nativement.

2.  **Quel est l'avantage de séparer `StateContext` et `DispatchContext` ?**
    L'optimisation des performances. Les composants qui ont seulement besoin d'envoyer des actions (`dispatch`) ne se re-rendront pas lorsque l'état (`state`) change.

3.  **La fonction `dispatch` retournée par `useReducer` est-elle stable ?**
    Oui, React garantit que l'identité de la fonction `dispatch` reste la même entre les rendus. C'est pourquoi on peut la passer dans un Context sans déclencher de re-rendus inutiles.

---

## Exercices : {#exercices-31}

### Exercice 1 - Le "Redux-Lite" (Gestionnaire de Tâches) {#exercice-1---le-redux-lite}

🎯 **Objectif** : Mettre en place l'architecture complète (Types, Reducer, Context, Provider, Hooks).

💼 **Mise en situation** : Vous créez un widget de "To-Do List" collaboratif. L'état doit être accessible par la liste (affichage) et par le header (compteur de tâches restantes).

📝 **Énoncé** :
1. Créez `TodosContext.tsx`.
2. État : `{ todos: { id, text, done }[] }`.
3. Actions : `ADD`, `TOGGLE`, `REMOVE`.
4. Créez le Provider et le Hook `useTodos`.
5. Dans `App`, affichez un composant `Header` (affiche "X tâches restantes") et `TodoList` (la liste).
6. Les deux composants doivent utiliser le contexte.

📺 **Résultat attendu** :
Quand on coche une tâche dans `TodoList`, le compteur dans `Header` se met à jour instantanément.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { createContext, useContext, useReducer, ReactNode } from 'react';

// --- 1. Logique Métier (Reducer) ---
type Todo = { id: number; text: string; done: boolean };
type Action = 
  | { type: 'ADD'; text: string }
  | { type: 'TOGGLE'; id: number }
  | { type: 'REMOVE'; id: number };

function todoReducer(state: Todo[], action: Action): Todo[] {
  switch (action.type) {
    case 'ADD':
      return [...state, { id: Date.now(), text: action.text, done: false }];
    case 'TOGGLE':
      return state.map(t => t.id === action.id ? { ...t, done: !t.done } : t);
    case 'REMOVE':
      return state.filter(t => t.id !== action.id);
    default:
      return state;
  }
}

// --- 2. Tuyauterie Context ---
const TodoContext = createContext<{
  todos: Todo[];
  dispatch: React.Dispatch<Action>;
} | null>(null);

export function TodoProvider({ children }: { children: ReactNode }) {
  const [todos, dispatch] = useReducer(todoReducer, []);

  return (
    <TodoContext.Provider value={{ todos, dispatch }}>
      {children}
    </TodoContext.Provider>
  );
}

export const useTodos = () => {
  const ctx = useContext(TodoContext);
  if (!ctx) throw new Error("Missing TodoProvider");
  return ctx;
};

// --- 3. Composants UI ---
function Header() {
  const { todos } = useTodos();
  const remaining = todos.filter(t => !t.done).length;
  return <h2>Tâches à faire : {remaining}</h2>;
}

function TodoList() {
  const { todos, dispatch } = useTodos();
  return (
    <div>
      <button onClick={() => dispatch({ type: 'ADD', text: 'Nouvelle tâche' })}>
        + Ajouter
      </button>
      <ul>
        {todos.map(t => (
          <li key={t.id} style={{ textDecoration: t.done ? 'line-through' : 'none' }}>
            <span onClick={() => dispatch({ type: 'TOGGLE', id: t.id })}>{t.text}</span>
            <button onClick={() => dispatch({ type: 'REMOVE', id: t.id })}>X</button>
          </li>
        ))}
      </ul>
    </div>
  );
}

export function App() {
  return (
    <TodoProvider>
      <div style={{ padding: 20 }}>
        <Header />
        <TodoList />
      </div>
    </TodoProvider>
  );
}
```
</details>

### Exercice 2 - Optimisation des Rendus (Split Context) {#exercice-2---optimisation-des-rendus}

🎯 **Objectif** : Prouver et implémenter la séparation State/Dispatch.

💼 **Mise en situation** : Une application de compteur haute performance. Le composant `Display` doit se rafraîchir, mais le composant `Controls` (les boutons) ne doit **jamais** se re-rendre, même quand le compteur change.

📝 **Énoncé** :
1. Créez deux contextes : `CountContext` et `CountDispatchContext`.
2. Dans le Provider, passez `state` au premier et `dispatch` au second.
3. Créez deux hooks : `useCount` et `useCountDispatch`.
4. Ajoutez un `console.log('Render Controls')` dans le composant des boutons.
5. Vérifiez dans la console que cliquer sur "+" ne déclenche pas le log.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : La console du navigateur à côté de l'application.
> **Annotation** : Montrez que malgré plusieurs clics (compteur à 5, 6, 7...), le log "Render Controls" n'apparaît qu'une seule fois au chargement.
> **Alt Text suggéré** : Console montrant l'absence de re-rendus inutiles grâce à la séparation des contextes.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { createContext, useContext, useReducer, ReactNode } from 'react';

// Reducer simple
const counterReducer = (state: number, action: 'INC' | 'DEC') => 
  action === 'INC' ? state + 1 : state - 1;

// Deux Contextes
const CountStateContext = createContext<number | null>(null);
const CountDispatchContext = createContext<React.Dispatch<'INC'|'DEC'> | null>(null);

function CountProvider({ children }: { children: ReactNode }) {
  const [count, dispatch] = useReducer(counterReducer, 0);

  return (
    <CountStateContext.Provider value={count}>
      <CountDispatchContext.Provider value={dispatch}>
        {children}
      </CountDispatchContext.Provider>
    </CountStateContext.Provider>
  );
}

// Hooks séparés
const useCount = () => useContext(CountStateContext)!;
const useCountDispatch = () => useContext(CountDispatchContext)!;

// Composant qui lit l'état (se re-rend à chaque clic)
function Display() {
  const count = useCount();
  console.log('🎨 Render Display');
  return <h1>Compteur : {count}</h1>;
}

// Composant qui dispatch (ne se re-rend JAMAIS après le montage)
function Controls() {
  const dispatch = useCountDispatch();
  console.log('🛑 Render Controls (Should happen only once)');
  
  return (
    <div>
      <button onClick={() => dispatch('DEC')}>-</button>
      <button onClick={() => dispatch('INC')}>+</button>
    </div>
  );
}

export function OptimizedApp() {
  return (
    <CountProvider>
      <Display />
      <Controls />
    </CountProvider>
  );
}
```
</details>

### Exercice 3 - Le Dashboard Asynchrone {#exercice-3---le-dashboard-asynchrone}

🎯 **Objectif** : Gérer des données asynchrones (API) avec ce pattern.

💼 **Mise en situation** : Un tableau de bord affiche des données utilisateur chargées depuis une "API". On doit gérer les états `LOADING`, `SUCCESS`, `ERROR`.

📝 **Énoncé** :
1. State : `{ status: 'idle' | 'loading' | 'success' | 'error', data: any, error: string }`.
2. Le composant `UserList` déclenche un chargement au montage (`useEffect`).
3. Pour faire l'appel API, on ne peut pas le faire *dans* le reducer. On crée une fonction helper `fetchUsers(dispatch)` à l'extérieur.
4. Cette fonction dispatch `FETCH_START`, puis `FETCH_SUCCESS` ou `FETCH_ERROR`.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { createContext, useContext, useReducer, useEffect } from 'react';

// --- Types & Reducer ---
type State = { status: string; data: string[]; error: string | null };
type Action = 
  | { type: 'FETCH_START' }
  | { type: 'FETCH_SUCCESS'; payload: string[] }
  | { type: 'FETCH_ERROR'; payload: string };

const initialState: State = { status: 'idle', data: [], error: null };

function dataReducer(state: State, action: Action): State {
  switch (action.type) {
    case 'FETCH_START': return { ...state, status: 'loading', error: null };
    case 'FETCH_SUCCESS': return { status: 'success', data: action.payload, error: null };
    case 'FETCH_ERROR': return { status: 'error', data: [], error: action.payload };
    default: return state;
  }
}

// --- Context ---
const DataContext = createContext<{ state: State; dispatch: React.Dispatch<Action> } | null>(null);

// --- Helper Asynchrone (Simule un thunk Redux) ---
async function loadUsers(dispatch: React.Dispatch<Action>) {
  dispatch({ type: 'FETCH_START' });
  try {
    // Simulation API
    await new Promise(r => setTimeout(r, 1500));
    // Random fail pour tester l'erreur
    if (Math.random() > 0.8) throw new Error("Erreur réseau aléatoire !");
    
    dispatch({ type: 'FETCH_SUCCESS', payload: ['Alice', 'Bob', 'Charlie'] });
  } catch (err: any) {
    dispatch({ type: 'FETCH_ERROR', payload: err.message });
  }
}

// --- UI ---
export function Dashboard() {
  const [state, dispatch] = useReducer(dataReducer, initialState);

  // Déclenchement au montage
  useEffect(() => {
    loadUsers(dispatch);
  }, []);

  return (
    <DataContext.Provider value={{ state, dispatch }}>
      <div style={{ padding: 20 }}>
        <h3>Utilisateurs</h3>
        {state.status === 'loading' && <p>Chargement en cours...</p>}
        {state.status === 'error' && <p style={{ color: 'red' }}>Erreur : {state.error}</p>}
        
        {state.status === 'success' && (
          <ul>
            {state.data.map(u => <li key={u}>{u}</li>)}
          </ul>
        )}
        
        <button onClick={() => loadUsers(dispatch)}>Recharger</button>
      </div>
    </DataContext.Provider>
  );
}
```
</details>
```