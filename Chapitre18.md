Voici le chapitre **Mettre à Jour des Tableaux dans l'État** pour la formation React 19.2.

```markdown
---
sidebar_label: Mettre à Jour des Tableaux dans l'État
sidebar_position: 18
---

# Chapitre 18 : Mettre à Jour des Tableaux dans l'État

Immutabilité des tableaux, Méthodes de tableau non-mutantes, Ajouter, supprimer, modifier des éléments

Les tableaux (Arrays) sont omniprésents en JavaScript. Listes de tâches, flux de données, paniers d'achat... vous en utiliserez partout.
Cependant, comme pour les objets, les tableaux en React doivent être traités comme **immuables**. C'est souvent le point de blocage principal pour les développeurs venant de jQuery ou Angular 1.

Vous ne pouvez pas utiliser `push`, `pop` ou `splice` directement sur une variable d'état. Vous devez apprendre à manipuler les tableaux en créant de nouvelles copies.

## Immutabilité des Tableaux {#immutabilite-des-tableaux}

### 1. Quoi
En JavaScript, les tableaux sont des objets mutables.
```js
const arr = [1, 2];
arr.push(3); // arr est modifié sur place : [1, 2, 3]
```
En React, vous ne devez pas modifier le tableau existant. Vous devez créer un **nouveau** tableau contenant les changements, et passer ce nouveau tableau à la fonction `set`.

### 2. Pourquoi
React compare les tableaux par référence (`oldArray === newArray`).
Si vous faites `state.push(item)`, la référence mémoire du tableau ne change pas. React pense que rien n'a changé et **ne relance pas le rendu**. L'écran ne se met pas à jour.

### 3. Tableau Comparatif : Mutants vs Non-Mutants
Voici votre antisèche pour savoir quelles méthodes utiliser en React.

| Action | ❌ Méthodes interdites (Mutent) | ✅ Méthodes recommandées (Retournent une copie) |
| :--- | :--- | :--- |
| **Ajouter** | `push`, `unshift` | `[...arr, item]`, `concat` |
| **Supprimer** | `pop`, `shift`, `splice` | `filter`, `slice` |
| **Remplacer** | `arr[i] = x`, `splice` | `map` |
| **Trier** | `sort`, `reverse` | `toSorted`, `toReversed`, ou copier puis trier |

### 4. Zone de Danger

:::danger Ne jamais muter l'état
Même si cela "semble marcher" parfois, ne faites jamais ceci :
```tsx
const [list, setList] = useState([]);

// ⛔️ INTERDIT
list.push('nouvel item'); 
setList(list); 
```
Cela causera des bugs silencieux, des problèmes avec `PureComponent` ou `React.memo`, et interfère avec le mode concurrent de React 19.
:::

---

## Ajouter des Éléments {#ajouter-des-elements}

### 1. Quoi
Pour ajouter un élément, on crée un nouveau tableau qui contient les anciens éléments (via le spread operator `...`) plus le nouveau.

### 2. Comment

```tsx
import { useState } from 'react';

export function TodoList() {
  const [todos, setTodos] = useState<string[]>([]);
  const [text, setText] = useState('');

  const handleAdd = () => {
    // ✅ Création d'un nouveau tableau
    // [...anciens, nouveau]
    setTodos([...todos, text]);
    setText('');
  };

  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <button onClick={handleAdd}>Ajouter</button>
      <ul>
        {todos.map((todo, index) => <li key={index}>{todo}</li>)}
      </ul>
    </>
  );
}
```

#### Astuce : L'ordre d'insertion
*   Pour ajouter à la **fin** (push) : `[...list, newItem]`
*   Pour ajouter au **début** (unshift) : `[newItem, ...list]`

---

## Supprimer des Éléments {#supprimer-des-elements}

### 1. Quoi
Pour retirer un élément, la méthode standard est `.filter()`. Elle crée un nouveau tableau contenant uniquement les éléments qui répondent à une condition (ceux qu'on veut garder).

### 2. Comment
On supprime généralement par identifiant unique (`id`).

```tsx
interface Task {
  id: number;
  label: string;
}

const [tasks, setTasks] = useState<Task[]>([
  { id: 1, label: "Acheter du lait" },
  { id: 2, label: "Promener le chien" }
]);

const handleDelete = (idToDelete: number) => {
  // ✅ "Garde toutes les tâches dont l'ID est différent de celui à supprimer"
  setTasks(tasks.filter(t => t.id !== idToDelete));
};
```

---

## Modifier / Remplacer des Éléments {#modifier-remplacer-des-elements}

### 1. Quoi
C'est l'opération la plus complexe pour les débutants. Comment changer *une* propriété d'un objet situé *dans* un tableau ?
La réponse est `.map()`. On parcourt le tableau, et pour l'élément ciblé, on retourne une copie modifiée. Pour les autres, on les retourne tels quels.

### 2. Pourquoi
Cela permet de conserver l'intégrité du tableau et des objets non modifiés (optimisation mémoire par partage de structure) tout en changeant l'élément visé.

### 3. Comment

Imaginons une liste de produits où l'on veut incrémenter la quantité d'un produit spécifique.

```tsx
interface Product {
  id: number;
  name: string;
  count: number;
}

const [products, setProducts] = useState<Product[]>([
  { id: 1, name: 'Pomme', count: 0 },
  { id: 2, name: 'Poire', count: 0 }
]);

const handleIncrement = (idToUpdate: number) => {
  setProducts(products.map(product => {
    // 1. Est-ce l'élément que je veux modifier ?
    if (product.id === idToUpdate) {
      // ✅ OUI : Je retourne une COPIE avec la modification
      return { ...product, count: product.count + 1 };
    }
    // 2. NON : Je retourne l'objet original sans y toucher
    return product;
  }));
};
```

---

## Insérer, Trier et Inverser {#inserer-trier-et-inverser}

### 1. Insérer à une position précise
On utilise `.slice()` pour découper le tableau en deux (avant et après l'index d'insertion) et on intercale le nouvel élément.

```tsx
const insertAtIndex = (index: number, newItem: string) => {
  setList([
    ...list.slice(0, index), // Tout ce qui est avant
    newItem,                 // L'intrus
    ...list.slice(index)     // Tout ce qui est après
  ]);
};
```

### 2. Trier et Inverser
`.sort()` et `.reverse()` mutent le tableau. En React 19 (et JS moderne), vous pouvez utiliser `toSorted()` et `toReversed()` qui retournent de nouvelles copies.

```tsx
const handleSort = () => {
  // ✅ Méthode moderne (ES2023+)
  setList(list.toSorted()); 
  
  // 👴 Ancienne méthode (si support navigateur limité)
  // setList([...list].sort()); // Copie d'abord, trie ensuite
};
```

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-18}

1.  **Pourquoi `myArray.push('item')` ne déclenche-t-il pas de re-render ?**
    Parce que `push` modifie le tableau existant sans changer sa référence en mémoire. React, comparant l'ancienne et la nouvelle référence, ne voit aucune différence.

2.  **Quelle méthode de tableau utilise-t-on pour supprimer un élément en React ?**
    On utilise `.filter()`. Elle retourne un nouveau tableau excluant l'élément indésirable.

3.  **Comment modifier un élément spécifique dans un tableau d'objets ?**
    On utilise `.map()`. On itère sur le tableau et on retourne une copie modifiée (avec `...spread`) uniquement pour l'élément correspondant à l'ID recherché.

4.  **Peut-on utiliser `list.sort()` directement dans le setter d'état ?**
    Non, car `sort()` mute le tableau original. Il faut utiliser `list.toSorted()` ou copier le tableau avant (`[...list].sort()`).

---

## Exercices : {#exercices-18}

### Exercice 1 - Le Gestionnaire de Tags (Ajout/Suppression) {#exercice-1---le-gestionnaire-de-tags}

🎯 **Objectif** : Gérer un tableau de chaînes de caractères (ajout et suppression).

💼 **Mise en situation** : Vous construisez un éditeur de blog. L'utilisateur doit pouvoir ajouter des tags (ex: "React", "Tuto") et les supprimer en cliquant dessus.

📝 **Énoncé** :
1. État `tags` (tableau de strings) initialisé avec `["React", "JavaScript"]`.
2. Un input et un bouton pour ajouter un tag.
3. Affichage des tags sous forme de boutons.
4. Au clic sur un tag, il doit être supprimé de la liste.

📺 **Résultat attendu** :
Une liste de boutons. Cliquer sur un bouton le fait disparaître. Ajouter du texte crée un nouveau bouton.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState } from 'react';

export function TagManager() {
  const [tags, setTags] = useState(["React", "JavaScript"]);
  const [inputValue, setInputValue] = useState("");

  const handleAdd = () => {
    if (!inputValue) return;
    // Ajout : On crée un nouveau tableau avec l'ancien contenu + le nouveau
    setTags([...tags, inputValue]);
    setInputValue("");
  };

  const handleDelete = (tagToDelete: string) => {
    // Suppression : On garde tout ce qui n'est PAS le tag cliqué
    setTags(tags.filter(tag => tag !== tagToDelete));
  };

  return (
    <div style={{ padding: 20 }}>
      <div style={{ marginBottom: 10 }}>
        <input 
          value={inputValue} 
          onChange={e => setInputValue(e.target.value)}
          placeholder="Nouveau tag..."
        />
        <button onClick={handleAdd}>Ajouter</button>
      </div>
      
      <div style={{ display: 'flex', gap: 5 }}>
        {tags.map((tag, index) => (
          <button 
            key={index} // Idéalement utiliser un ID unique, ici index par défaut
            onClick={() => handleDelete(tag)}
            style={{ 
              backgroundColor: '#e0e0e0', 
              border: 'none', 
              padding: '5px 10px', 
              borderRadius: '15px',
              cursor: 'pointer' 
            }}
          >
            {tag} ✖
          </button>
        ))}
      </div>
    </div>
  );
}
```
</details>

### Exercice 2 - La Todo List Interactive (Modification via Map) {#exercice-2---la-todo-list-interactive}

🎯 **Objectif** : Modifier une propriété d'un objet dans un tableau.

💼 **Mise en situation** : Une application de productivité. On doit pouvoir cocher/décocher une tâche.

📝 **Énoncé** :
1. Interface `Todo` : `{ id: number, text: string, done: boolean }`.
2. État initial avec 2 tâches.
3. Affichez la liste. Si `done` est true, le texte est barré (`textDecoration: 'line-through'`).
4. Au clic sur une tâche, inversez son état `done`.
5. Utilisez `.map()` pour créer le nouvel état.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Une liste de 3 tâches, dont la deuxième est barrée.
> **Annotation** : Montrez l'état visuel distinct (barré/non barré).
> **Alt Text suggéré** : Liste de tâches dont certaines sont complétées et rayées.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState } from 'react';

interface Todo {
  id: number;
  text: string;
  done: boolean;
}

export function TodoList() {
  const [todos, setTodos] = useState<Todo[]>([
    { id: 1, text: "Apprendre React", done: true },
    { id: 2, text: "Maîtriser les Arrays", done: false },
    { id: 3, text: "Dormir", done: false },
  ]);

  const toggleTodo = (id: number) => {
    setTodos(todos.map(todo => {
      // Si c'est la tâche cliquée...
      if (todo.id === id) {
        // ...on retourne une copie avec 'done' inversé
        return { ...todo, done: !todo.done };
      }
      // Sinon, on retourne la tâche inchangée
      return todo;
    }));
  };

  return (
    <ul>
      {todos.map(todo => (
        <li 
          key={todo.id}
          onClick={() => toggleTodo(todo.id)}
          style={{ 
            textDecoration: todo.done ? 'line-through' : 'none',
            cursor: 'pointer',
            userSelect: 'none'
          }}
        >
          {todo.done ? '✅' : '⬜️'} {todo.text}
        </li>
      ))}
    </ul>
  );
}
```
</details>

### Exercice 3 - Le Panier de Fruits (Tri et Quantité) {#exercice-3---le-panier-de-fruits}

🎯 **Objectif** : Combiner modification et tri.

💼 **Mise en situation** : Un e-commerce. Vous avez une liste de produits avec un compteur. Vous devez pouvoir incrémenter le compteur ET trier les produits par quantité décroissante (les plus populaires en haut).

📝 **Énoncé** :
1. État : `[{ id: 1, name: "🍎", count: 0 }, { id: 2, name: "🍌", count: 0 }]`.
2. Bouton "+" à côté de chaque fruit pour augmenter `count`.
3. Bouton "Trier par popularité" qui réorganise la liste (plus grand `count` en premier).
4. Attention : `sort` ne doit pas muter l'état directement !

📺 **Résultat attendu** :
En cliquant sur "+" de la banane (qui passe à 1), puis sur "Trier", la banane doit passer au-dessus de la pomme (0).

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState } from 'react';

export function FruitBasket() {
  const [fruits, setFruits] = useState([
    { id: 1, name: "🍎 Pomme", count: 0 },
    { id: 2, name: "🍌 Banane", count: 0 },
    { id: 3, name: "🍒 Cerise", count: 0 },
  ]);

  const increment = (id: number) => {
    setFruits(fruits.map(f => 
      f.id === id ? { ...f, count: f.count + 1 } : f
    ));
  };

  const sortByPopularity = () => {
    // Utilisation de la méthode moderne toSorted (crée une copie)
    // Tri décroissant (b - a)
    const sorted = fruits.toSorted((a, b) => b.count - a.count);
    setFruits(sorted);
    
    // Si toSorted n'est pas dispo, on ferait :
    // setFruits([...fruits].sort((a, b) => b.count - a.count));
  };

  return (
    <div>
      <button onClick={sortByPopularity} style={{ marginBottom: 15 }}>
        🏆 Trier par popularité
      </button>
      
      <ul style={{ listStyle: 'none', padding: 0 }}>
        {fruits.map(fruit => (
          <li key={fruit.id} style={{ marginBottom: 5 }}>
            <button onClick={() => increment(fruit.id)}>
              +1
            </button>
            <span style={{ marginLeft: 10 }}>
              {fruit.name} : <strong>{fruit.count}</strong>
            </span>
          </li>
        ))}
      </ul>
    </div>
  );
}
```
</details>
```