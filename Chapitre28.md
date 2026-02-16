Voici le chapitre **Réutiliser la Logique avec des Hooks Personnalisés** pour la formation React 19.2.

```markdown
---
sidebar_label: Réutiliser la Logique avec des Hooks Personnalisés
sidebar_position: 28
---

# Chapitre 28 : Réutiliser la Logique avec des Hooks Personnalisés

Créer des Hooks, Partage de logique d'état, Convention de nommage

Vous savez déjà créer des composants pour réutiliser votre UI (l'interface visuelle).
Mais comment réutiliser la **logique** (le comportement) entre plusieurs composants ?
Avant les Hooks, nous utilisions des patterns complexes comme les *Higher-Order Components* ou les *Render Props*.
Aujourd'hui, les **Hooks Personnalisés** (Custom Hooks) sont la solution standard, élégante et puissante.

## Qu'est-ce qu'un Hook Personnalisé ? {#qu-est-ce-qu-un-hook-personnalise}

### 1. Quoi
Un Hook Personnalisé est simplement une **fonction JavaScript** qui :
1.  Commence par `use` (convention obligatoire).
2.  Appelle d'autres Hooks (comme `useState`, `useEffect`, ou d'autres custom hooks).
3.  Peut accepter des arguments et retourner n'importe quoi (valeur, objet, tableau, fonction).

### 2. Pourquoi
Imaginez que deux pages différentes de votre application (Profil et Paramètres) doivent toutes les deux savoir si l'utilisateur est en ligne ou hors ligne.
Au lieu de copier-coller le code `window.addEventListener('online', ...)` dans chaque composant, vous pouvez extraire cette logique dans un Hook `useOnlineStatus`.

### 3. Comment

#### A. Syntaxe de base

```tsx
// 1. Définition du Hook
import { useState, useEffect } from 'react';

function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);
  
  const increment = () => setCount(c => c + 1);
  const decrement = () => setCount(c => c - 1);

  // Retourne ce que le composant a besoin de consommer
  return { count, increment, decrement };
}

// 2. Utilisation
function MyComponent() {
  const { count, increment } = useCounter(10);
  return <button onClick={increment}>{count}</button>;
}
```

#### B. Cas concret : useOnlineStatus
Extrayons une logique plus complexe.

```tsx
import { useState, useEffect } from 'react';

// ✅ Hook Personnalisé : gère l'abonnement réseau
export function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(navigator.onLine);

  useEffect(() => {
    function handleOnline() { setIsOnline(true); }
    function handleOffline() { setIsOnline(false); }

    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);

    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);

  return isOnline;
}
```

Utilisation dans deux composants différents :

```tsx
function StatusBar() {
  const isOnline = useOnlineStatus(); // Réutilise la logique
  return <div>{isOnline ? '✅ Connecté' : '❌ Déconnecté'}</div>;
}

function SaveButton() {
  const isOnline = useOnlineStatus(); // Réutilise la même logique
  return <button disabled={!isOnline}>Sauvegarder</button>;
}
```

---

## Partage de Logique d'État (Stateful Logic) {#partage-de-logique-d-etat}

### 1. Quoi
C'est la distinction la plus importante : les Custom Hooks partagent la **logique d'état**, pas l'état lui-même.

### 2. Pourquoi
Chaque appel à un Hook est **indépendant**.
Si vous appelez `useCounter` dans le composant A et aussi dans le composant B, ils auront chacun leur propre état `count`. Cliquer sur le bouton A n'affectera pas le compteur B.

### 3. Comment
Visualisez le Hook comme un "moule". Chaque fois que vous l'utilisez, vous créez une nouvelle instance isolée de ce moule.

```tsx
function ComponentA() {
  const { count, increment } = useCounter(0); // État interne #1
  return <button onClick={increment}>A: {count}</button>;
}

function ComponentB() {
  const { count, increment } = useCounter(0); // État interne #2
  return <button onClick={increment}>B: {count}</button>;
}
```

Si vous vouliez partager *le même état* entre plusieurs composants, vous devriez utiliser un Contexte (`useContext`) ou un gestionnaire d'état global, pas juste un Custom Hook.

### 4. Zone de Danger

:::danger Convention de Nommage
Vous **DEVEZ** commencer le nom de votre fonction par `use` (ex: `useForm`, `useUser`).
C'est ce qui permet :
1.  Au plugin ESLint de vérifier que vous respectez les règles des Hooks (pas d'appel conditionnel, pas de boucle).
2.  Aux autres développeurs de comprendre immédiatement que cette fonction contient de l'état ou des effets.
:::

---

## Exemples Pratiques de Hooks Utiles {#exemples-pratiques}

### Cas 1 : `useForm` (Gestion de formulaires)
Gérer les `onChange` pour chaque input est répétitif.

```tsx
import { useState } from 'react';

export function useForm<T>(initialValues: T) {
  const [values, setValues] = useState(initialValues);

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const { name, value } = e.target;
    setValues(prev => ({ ...prev, [name]: value }));
  };

  const reset = () => setValues(initialValues);

  return { values, handleChange, reset };
}

// Utilisation
function SignupForm() {
  const { values, handleChange } = useForm({ username: '', email: '' });

  return (
    <input 
      name="username" 
      value={values.username} 
      onChange={handleChange} 
    />
  );
}
```

### Cas 2 : `useFetch` (Appels API)
Encapsuler le chargement, l'erreur et les données.

```tsx
import { useState, useEffect } from 'react';

export function useFetch<T>(url: string) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    let ignore = false;
    setLoading(true);

    fetch(url)
      .then(res => res.json())
      .then(json => {
        if (!ignore) {
          setData(json);
          setLoading(false);
        }
      })
      .catch(err => {
        if (!ignore) {
          setError(err);
          setLoading(false);
        }
      });

    return () => { ignore = true; };
  }, [url]);

  return { data, loading, error };
}
```

### Cas 3 : `useDebounce` (Optimisation)
Retarder une mise à jour de valeur (ex: barre de recherche).

```tsx
import { useState, useEffect } from 'react';

export function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => {
      clearTimeout(handler);
    };
  }, [value, delay]);

  return debouncedValue;
}
```

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-28}

1.  **Quelle est la règle absolue pour nommer un Hook personnalisé ?**
    Le nom doit commencer par le préfixe `use` (ex: `useAuth`).

2.  **Si j'utilise le même Hook `useCounter` dans deux composants, partagent-ils la même valeur de compteur ?**
    Non. Chaque appel à un Hook crée un état isolé. C'est de la réutilisation de *logique*, pas de partage de *données*.

3.  **Peut-on appeler un Hook à l'intérieur d'une condition `if` ou d'une boucle ?**
    Non, jamais. Les Hooks personnalisés doivent respecter les mêmes règles que les Hooks natifs.

4.  **Quelle est la différence entre une fonction utilitaire classique et un Custom Hook ?**
    Un Custom Hook peut appeler d'autres Hooks (`useState`, `useEffect`...) et conserver un état entre les rendus. Une fonction utilitaire est "stateless" et s'exécute de manière pure.

---

## Exercices : {#exercices-28}

### Exercice 1 - Le Hook Toggle {#exercice-1---le-hook-toggle}

🎯 **Objectif** : Créer votre premier Hook simple pour gérer des booléens.

💼 **Mise en situation** : Dans votre application SaaS, vous avez plein de modales, de menus déroulants et de switchs ON/OFF. Vous répétez tout le temps `const [isOpen, setIsOpen] = useState(false)` et les fonctions pour ouvrir/fermer.

📝 **Énoncé** :
1. Créez un hook `useToggle(initialValue: boolean)`.
2. Il doit retourner un tableau `[value, toggle]` (comme `useState`).
3. `toggle` doit être une fonction qui inverse la valeur actuelle (true -> false -> true).
4. Bonus : `toggle` peut aussi accepter un booléen pour forcer une valeur spécifique.

📺 **Résultat attendu** :
Un bouton "Afficher/Masquer" qui fonctionne en une ligne de code grâce au Hook.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState, useCallback } from 'react';

// Le Hook
export function useToggle(initialValue: boolean = false) {
  const [value, setValue] = useState(initialValue);

  // useCallback est une bonne pratique pour stabiliser la fonction
  const toggle = useCallback((forcedValue?: unknown) => {
    setValue(current => {
      // Si on passe un booléen explicite (ex: toggle(true)), on l'utilise
      if (typeof forcedValue === 'boolean') {
        return forcedValue;
      }
      // Sinon on inverse
      return !current;
    });
  }, []);

  return [value, toggle] as const; // 'as const' pour typer le tuple correctement
}

// Utilisation
export function ModalDemo() {
  const [isOpen, toggleOpen] = useToggle(false);

  return (
    <div>
      <button onClick={toggleOpen}>
        {isOpen ? 'Fermer' : 'Ouvrir'} le menu
      </button>
      
      {isOpen && (
        <div style={{ border: '1px solid black', padding: 10, marginTop: 10 }}>
          Contenu du menu...
          <button onClick={() => toggleOpen(false)}>Fermer explicitement</button>
        </div>
      )}
    </div>
  );
}
```
</details>

### Exercice 2 - Le Hook de Taille de Fenêtre {#exercice-2---le-hook-de-taille-de-fenêtre}

🎯 **Objectif** : Gérer un événement global et nettoyer proprement.

💼 **Mise en situation** : Vous construisez une interface responsive complexe en JS (pas seulement en CSS). Vous avez besoin de connaître la largeur de la fenêtre pour afficher une vue "Mobile" ou "Desktop".

📝 **Énoncé** :
1. Créez `useWindowSize()`.
2. Il doit retourner un objet `{ width, height }`.
3. Utilisez `useEffect` pour écouter l'événement `resize`.
4. N'oubliez pas le nettoyage (`removeEventListener`) !

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Un texte affichant "Largeur actuelle : 1024px" qui se met à jour quand on redimensionne.
> **Annotation** : Montrez la fenêtre du navigateur réduite à une taille mobile.
> **Alt Text suggéré** : Composant affichant les dimensions dynamiques de la fenêtre.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState, useEffect } from 'react';

// Le Hook
function useWindowSize() {
  // Initialisation avec les valeurs actuelles (attention au SSR où window n'existe pas)
  const [size, setSize] = useState({
    width: typeof window !== 'undefined' ? window.innerWidth : 0,
    height: typeof window !== 'undefined' ? window.innerHeight : 0,
  });

  useEffect(() => {
    function handleResize() {
      setSize({
        width: window.innerWidth,
        height: window.innerHeight,
      });
    }

    // Abonnement
    window.addEventListener('resize', handleResize);
    
    // Nettoyage impératif
    return () => window.removeEventListener('resize', handleResize);
  }, []); // Vide = exécuté une seule fois au montage

  return size;
}

// Utilisation
export function ResponsiveView() {
  const { width } = useWindowSize();
  const isMobile = width < 768;

  return (
    <div style={{ background: isMobile ? 'lightblue' : 'lightgreen', padding: 20 }}>
      <h2>Mode {isMobile ? '📱 Mobile' : '💻 Desktop'}</h2>
      <p>Largeur : {width}px</p>
    </div>
  );
}
```
</details>

### Exercice 3 - Le Hook `useLocalStorage` {#exercice-3---le-hook-uselocalstorage}

🎯 **Objectif** : Synchroniser un état avec le stockage du navigateur.

💼 **Mise en situation** : Vous voulez sauvegarder le thème préféré de l'utilisateur ("dark" ou "light") pour qu'il le retrouve à sa prochaine visite.

📝 **Énoncé** :
1. Créez `useLocalStorage<T>(key: string, initialValue: T)`.
2. Au montage, lisez la valeur dans `localStorage`. Si elle n'existe pas, utilisez `initialValue`.
3. Retournez `[storedValue, setValue]`.
4. Quand `setValue` est appelé, mettez à jour l'état React ET le `localStorage`.
5. Gérez les erreurs JSON (try/catch).

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState } from 'react';

// Le Hook
function useLocalStorage<T>(key: string, initialValue: T) {
  // 1. Initialiser l'état en lisant le storage (lazy init)
  const [storedValue, setStoredValue] = useState<T>(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(error);
      return initialValue;
    }
  });

  // 2. Fonction setter enrichie
  const setValue = (value: T | ((val: T) => T)) => {
    try {
      // Gérer le cas où value est une fonction (comme useState)
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      
      // Update State
      setStoredValue(valueToStore);
      
      // Update Storage
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error(error);
    }
  };

  return [storedValue, setValue] as const;
}

// Utilisation
export function ThemeSelector() {
  // Ce state survit au rafraîchissement de la page !
  const [theme, setTheme] = useLocalStorage('app-theme', 'light');

  return (
    <div style={{ background: theme === 'dark' ? '#333' : '#fff', color: theme === 'dark' ? '#fff' : '#000', padding: 20 }}>
      <p>Thème actuel : {theme}</p>
      <button onClick={() => setTheme('light')}>☀️ Light</button>
      <button onClick={() => setTheme('dark')}>🌙 Dark</button>
    </div>
  );
}
```
</details>
```