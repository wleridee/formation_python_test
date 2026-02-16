Voici le chapitre **Rendre des Listes** pour la formation React 19.2.

```markdown
---
sidebar_label: Rendre des Listes
sidebar_position: 11
---

# Chapitre 11 : Rendre des Listes

Méthode `Array.map()`,La propriété `key`,Listes imbriquées

Dans 90% des applications (feed Instagram, liste de produits Amazon, boîte mail), vous ne codez pas les éléments un par un. Vous prenez un tableau de données (souvent reçu d'une API) et vous le transformez en une liste de composants.
En React, il n'y a pas de syntaxe "magique" comme `v-for` ou `ngFor`. Nous utilisons simplement JavaScript.

## Méthode `Array.map()` {#methode-array-map}

### 1. Quoi
`map()` est une méthode native des tableaux JavaScript. Elle prend une fonction de transformation en argument et retourne un **nouveau tableau** contenant les éléments transformés.
En React, nous transformons un tableau de **Données** (Objets/Strings) en un tableau de **JSX** (Composants/Elements).

### 2. Pourquoi
React est capable d'afficher directement un tableau d'éléments JSX.
Si vous lui donnez `[<p>A</p>, <p>B</p>]`, il affichera les deux paragraphes. `map()` est l'outil parfait pour générer ce tableau de manière déclarative.

### 3. Comment

#### A. Syntaxe de base

```tsx
const fruits = ['Pomme', 'Banane', 'Cerise'];

export function FruitList() {
  // Transformation : String -> JSX <li>
  const listItems = fruits.map((fruit) => <li>{fruit}</li>);

  return <ul>{listItems}</ul>;
}
```

#### B. Cas concret : Liste de Produits
Généralement, on itère directement dans le JSX (Inline).

```tsx
type Product = {
  id: number;
  name: string;
  price: number;
};

const products: Product[] = [
  { id: 1, name: 'Clavier Mécanique', price: 120 },
  { id: 2, name: 'Souris Gamer', price: 60 },
];

export function Shop() {
  return (
    <section>
      <h2>Boutique</h2>
      <div className="product-grid">
        {/* On ouvre les accolades pour le JS */}
        {products.map((product) => (
          // On retourne un composant pour chaque produit
          <div className="product-card" key={product.id}>
            <h3>{product.name}</h3>
            <span className="price">{product.price} €</span>
          </div>
        ))}
      </div>
    </section>
  );
}
```

### 🚨 Limitations de `forEach`
N'utilisez pas `forEach()` en React. `forEach` ne retourne rien (`void`). React a besoin d'un tableau d'éléments à rendre. Utilisez toujours `map`.

---

## La propriété `key` {#la-propriete-key}

### 1. Quoi
`key` est une propriété spéciale (réservée par React) que vous devez **obligatoirement** donner à chaque élément retourné directement par un `map()`. Elle doit être une chaîne de caractères ou un nombre **unique** parmi ses frères et sœurs.

### 2. Pourquoi
La `key` sert d'identité à l'élément. Elle permet à l'algorithme de React (la Réconciliation) de savoir quels éléments ont changé, ont été ajoutés ou supprimés.
- Sans `key`, si vous insérez un élément au début de la liste, React risque de repeindre toute la liste (mauvaise performance) ou de mélanger les états des composants (bug critique).
- Avec `key`, React sait que l'élément avec l'ID "42" a juste bougé de la position 3 à la position 1.

### 3. Comment

#### A. Utiliser des IDs de base de données (Best Practice)
C'est la solution idéale. Vos données viennent d'une DB, elles ont donc un ID unique (UUID, Primary Key).

```tsx
{users.map((user) => (
  <UserProfile key={user.id} name={user.name} />
))}
```

#### B. Où placer la key ?
La `key` doit être sur l'élément **le plus haut** à l'intérieur du `map`.

❌ **Faux :**
```tsx
{items.map(item => (
  <>
    <div key={item.id}>{item.name}</div> {/* La key est invisible pour React ici */}
  </>
))}
```

✅ **Correct :**
```tsx
import { Fragment } from 'react';

{items.map(item => (
  // La key doit être sur le Fragment ou le conteneur parent direct
  <Fragment key={item.id}>
    <div>{item.name}</div>
  </Fragment>
))}
```

### 4. Zone de Danger

:::danger Ne JAMAIS utiliser l'index comme Key
Il est tentant de faire `items.map((item, index) => <li key={index}>...)`.
C'est un anti-pattern dangereux si la liste peut changer (tri, filtrage, ajout/suppression).
Si vous supprimez l'élément 1, l'élément 2 devient l'index 1. React pensera que c'est le même composant et gardera son ancien état local (ex: le texte tapé dans un input).

**Règle** : Utilisez l'index uniquement si la liste est **statique** (ne changera jamais) et n'a pas d'IDs.
:::

---

## Listes Imbriquées {#listes-imbriquees}

### 1. Quoi
Vos données ne sont pas toujours plates. Souvent, vous avez des catégories contenant des produits, ou des posts contenant des commentaires. Il faut imbriquer les `map`.

### 2. Pourquoi
Pour représenter des structures arborescentes ou groupées.

### 3. Comment

#### A. Syntaxe double map

```tsx
type Category = {
  id: string;
  title: string;
  items: string[];
};

const menuData: Category[] = [
  { id: 'drinks', title: 'Boissons', items: ['Eau', 'Soda'] },
  { id: 'food', title: 'Plats', items: ['Burger', 'Pizza'] },
];

export function Menu() {
  return (
    <div>
      {menuData.map((category) => (
        // Key pour la catégorie (niveau 1)
        <div key={category.id} className="category-section">
          <h3>{category.title}</h3>
          
          <ul>
            {category.items.map((item, index) => (
              // Key pour l'item (niveau 2) - Ici index acceptable car liste statique de strings
              // Idéalement : item.id si disponible
              <li key={`${category.id}-${index}`}>
                {item}
              </li>
            ))}
          </ul>
        </div>
      ))}
    </div>
  );
}
```

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-11}

1.  **Quelle méthode JavaScript utilise-t-on pour transformer des données en liste de composants React ?**
    La méthode `Array.map()`.

2.  **Pourquoi la prop `key` est-elle obligatoire ?**
    Elle permet à React d'identifier chaque élément de manière unique pour optimiser les mises à jour (performance) et préserver l'état des composants lors des réarrangements.

3.  **Peut-on accéder à la valeur de `key` dans les props du composant enfant (`props.key`) ?**
    Non, `key` est consommé par React lui-même et n'est pas passé au composant. Si vous avez besoin de l'ID, passez-le dans une autre prop (ex: `id={user.id}`).

4.  **Pourquoi utiliser l'index du tableau comme key est-il déconseillé ?**
    Parce que si l'ordre des éléments change, l'index change aussi, ce qui trompe React sur l'identité réelle des éléments et peut causer des bugs d'affichage ou d'état.

---

## Exercices : {#exercices-11}

### Exercice 1 - La Liste de Tâches Prioritaires {#exercice-1---la-liste-de-taches-prioritaires}

🎯 **Objectif** : Utiliser `map` et le rendu conditionnel ensemble.

💼 **Mise en situation** : Vous développez une Todo List. Vous devez afficher une liste de tâches, mais celles qui sont "urgentes" doivent apparaître en gras et rouge.

📝 **Énoncé** :
1. Données : Tableau d'objets `{ id: number, text: string, isUrgent: boolean }`.
2. Affichez une liste `<ul>`.
3. Pour chaque tâche, générez un `<li>`.
4. La `key` doit être l'ID.
5. Si `isUrgent` est vrai, appliquez `style={{ fontWeight: 'bold', color: 'red' }}`.

📺 **Résultat attendu** :
Une liste à puces où certaines lignes ressortent visuellement.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
type Task = {
  id: number;
  text: string;
  isUrgent: boolean;
};

const tasks: Task[] = [
  { id: 101, text: "Acheter du lait", isUrgent: false },
  { id: 102, text: "Payer les impôts", isUrgent: true },
  { id: 103, text: "Appeler maman", isUrgent: false },
];

export function TodoList() {
  return (
    <ul>
      {tasks.map((task) => (
        <li 
          key={task.id} // ID unique stable
          style={{ 
            // Style conditionnel
            fontWeight: task.isUrgent ? 'bold' : 'normal',
            color: task.isUrgent ? 'red' : 'inherit'
          }}
        >
          {task.text}
          {/* Petit ajout conditionnel pour le fun */}
          {task.isUrgent && " 🔥"}
        </li>
      ))}
    </ul>
  );
}
```
</details>

### Exercice 2 - La Galerie Filtrable {#exercice-2---la-galerie-filtrable}

🎯 **Objectif** : Combiner `filter()` et `map()`.

💼 **Mise en situation** : Sur un site e-commerce, on veut afficher uniquement les produits disponibles en stock.

📝 **Énoncé** :
1. Données : Tableau de produits `{ id: string, name: string, inStock: boolean }`.
2. Créez un composant `AvailableProducts`.
3. Avant le rendu ou dans le JSX, filtrez le tableau pour ne garder que `inStock: true`.
4. Mappez le résultat filtré pour afficher des `<div>`.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Liste de 2 produits (sur 4 disponibles dans le code source).
> **Annotation** : Montrez que les produits hors stock sont masqués.
> **Alt Text suggéré** : Liste filtrée affichant uniquement les produits en stock.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
type Product = {
  id: string;
  name: string;
  inStock: boolean;
};

const inventory: Product[] = [
  { id: 'p1', name: 'PS5', inStock: false },
  { id: 'p2', name: 'Switch', inStock: true },
  { id: 'p3', name: 'Xbox', inStock: true },
  { id: 'p4', name: 'PC Gamer', inStock: false },
];

export function AvailableProducts() {
  // Bonne pratique : Filtrer avant le map pour la lisibilité
  const availableItems = inventory.filter(product => product.inStock);

  return (
    <div>
      <h3>Produits Disponibles ({availableItems.length})</h3>
      <div className="grid">
        {availableItems.map(product => (
          <div key={product.id} className="card">
            {product.name}
          </div>
        ))}
      </div>
    </div>
  );
}
```
</details>

### Exercice 3 - Le Forum (Listes Imbriquées) {#exercice-3---le-forum}

🎯 **Objectif** : Gérer des données hiérarchiques (Topic -> Commentaires).

💼 **Mise en situation** : Vous affichez une page de discussion. Chaque sujet contient une liste d'auteurs qui ont répondu.

📝 **Énoncé** :
1. Structure de données : Un tableau de `Topic`. Chaque `Topic` a un `id`, un `title`, et un tableau `comments`. Chaque commentaire a un `id` et un `author`.
2. Affichez le titre du topic en `h2`.
3. En dessous, affichez "Réponses de :" suivi d'une liste `<ul>` des auteurs.
4. Assurez-vous que toutes les clés sont uniques et correctement placées.

📺 **Résultat attendu** :
Plusieurs blocs de titres suivis de leurs listes respectives de noms d'auteurs.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
type Comment = {
  id: string;
  author: string;
};

type Topic = {
  id: string;
  title: string;
  comments: Comment[];
};

const forumData: Topic[] = [
  {
    id: 't1',
    title: "React 19 est sorti !",
    comments: [
      { id: 'c1', author: 'Alice' },
      { id: 'c2', author: 'Bob' }
    ]
  },
  {
    id: 't2',
    title: "Besoin d'aide sur CSS",
    comments: [
      { id: 'c3', author: 'Charlie' }
    ]
  }
];

export function Forum() {
  return (
    <div className="forum-container">
      {forumData.map((topic) => (
        // Key niveau 1 : ID du topic
        <article key={topic.id} style={{ marginBottom: '20px', border: '1px solid #ddd', padding: '10px' }}>
          <h2>{topic.title}</h2>
          
          <p>Réponses de :</p>
          <ul>
            {/* Map imbriqué pour les commentaires de CE topic */}
            {topic.comments.map((comment) => (
              // Key niveau 2 : ID du commentaire (doit être unique au sein de la liste sibling)
              <li key={comment.id}>
                {comment.author}
              </li>
            ))}
          </ul>
        </article>
      ))}
    </div>
  );
}
```
</details>
```