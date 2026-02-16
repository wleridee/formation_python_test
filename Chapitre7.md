Voici le chapitre **Les Composants Pures** pour la formation React 19.2.

```markdown
---
sidebar_label: Les Composants Pures
sidebar_position: 7
---

# Chapitre 7 : Les Composants Pures

Fonctions pures,Impératifs vs Déclaratifs,Rendu prévisible

React est construit autour d'un concept central emprunté à la programmation fonctionnelle : la **Pureté**. Comprendre ce principe est la différence entre une application qui a des bugs aléatoires impossibles à reproduire et une application stable comme le roc.

## Fonctions Pures {#fonctions-pures}

### 1. Quoi
Une fonction est dite "pure" si elle respecte deux règles strictes :
1.  **Même entrée, même sortie** : Si on lui donne les mêmes arguments, elle retourne toujours le même résultat.
2.  **Pas d'effets de bord (Side Effects)** : Elle ne change rien à l'extérieur d'elle-même (ne modifie pas de variable globale, ne fait pas d'appel réseau, ne modifie pas le DOM directement).

### 2. Pourquoi
React considère que **tous vos composants doivent être purs**.
Cela permet à React de :
- Ne pas recalculer un composant si ses données n'ont pas changé (Performance).
- Rendre le rendu côté serveur (SSR) sûr.
- Interrompre le rendu au milieu pour faire autre chose de plus urgent, puis reprendre, sans casser l'état.

### 3. Comment

#### A. Syntaxe de base

```tsx
// ✅ Fonction Pure : Dépend uniquement de ses arguments (a, b)
function add(a: number, b: number) {
  return a + b;
}

// ❌ Fonction Impure : Dépend d'une variable externe (compteur)
let compteur = 0;
function addImpure(a: number) {
  compteur++; // Effet de bord !
  return a + compteur; // Résultat imprévisible si appelé plusieurs fois
}
```

#### B. Cas concret : Composant React Pur
Un composant pur reçoit des données (props) et retourne du JSX. Il ne touche à rien d'autre.

```tsx
type PriceDisplayProps = {
  price: number;
  currency: string;
};

// Ce composant retournera TOUJOURS le même HTML pour le même prix/devise.
export function PriceDisplay({ price, currency }: PriceDisplayProps) {
  // Calcul local autorisé car il ne modifie rien à l'extérieur
  const formattedPrice = new Intl.NumberFormat('fr-FR', {
    style: 'currency',
    currency: currency
  }).format(price);

  return <span className="font-bold">{formattedPrice}</span>;
}
```

### 4. Zone de Danger

:::danger Mutation de variables externes
React appelle vos composants. Vous ne savez pas **quand** ni **combien de fois**.
Si vous modifiez une variable externe pendant le rendu, vous créez des bugs temporels.

❌ **NE FAITES JAMAIS CECI :**
```tsx
let guestCount = 0;

function GuestList() {
  guestCount = guestCount + 1; // 😱 Mutation externe pendant le rendu !
  return <h2>Invité #{guestCount}</h2>;
}
```
Si React rend ce composant 2 fois (ce qui arrive souvent en Strict Mode), votre compteur sera faux.
:::

---

## Impératif vs Déclaratif {#imperatif-vs-declaratif}

### 1. Quoi
- **Impératif** : Vous dites à la machine *comment* faire (étape par étape). C'est le style jQuery ou manipulation DOM classique.
- **Déclaratif** : Vous dites à la machine *ce que vous voulez* (le résultat final), et elle se débrouille pour le faire. React est déclaratif.

### 2. Pourquoi
Le code déclaratif est plus facile à prédire et à débugger. Vous décrivez l'état final de l'interface pour une donnée donnée, sans vous soucier des transitions complexes (ajouter une classe, puis la retirer, puis changer le texte...).

### 3. Comment

#### A. Comparaison

**Approche Impérative (JS Vanilla)** :
```javascript
// On manipule le DOM manuellement
const btn = document.getElementById('btn');
if (user.isAdmin) {
  btn.style.color = 'red';
  btn.innerText = 'Supprimer';
} else {
  btn.style.display = 'none';
}
```

**Approche Déclarative (React)** :
```tsx
// On décrit l'interface selon l'état
// React gère le DOM pour nous
if (!user.isAdmin) return null;

return (
  <button style={{ color: 'red' }}>
    Supprimer
  </button>
);
```

#### B. Cas concret : Liste de tâches

```tsx
type Task = { id: number; text: string; done: boolean };

// On déclare simplement : "Voici à quoi ressemble la liste pour ces données"
export function TaskList({ tasks }: { tasks: Task[] }) {
  return (
    <ul>
      {tasks.map(task => (
        <li 
          key={task.id} 
          style={{ textDecoration: task.done ? 'line-through' : 'none' }}
        >
          {task.text}
        </li>
      ))}
    </ul>
  );
}
```

### 🚨 Limitations
Parfois, l'impératif est nécessaire (ex: donner le focus à un input, scroller vers une position, lancer une animation complexe). Pour ces rares cas, React fournit des "échappatoires" comme les `Refs` (Chapitre 20) et `useEffect` (Chapitre 22). Mais le rendu principal doit rester pur et déclaratif.

---

## Rendu Prévisible et Strict Mode {#rendu-previsible-et-strict-mode}

### 1. Quoi
Pour vous aider à garder vos composants purs, React 19 (en développement) active le **Strict Mode** par défaut.
Le Strict Mode **appelle chaque composant deux fois** (uniquement en développement) pour détecter les impuretés.

### 2. Pourquoi
Si votre composant est pur (1 + 1 = 2), l'appeler deux fois ne change rien (1 + 1 = 2, et 1 + 1 = 2). Le résultat visuel est identique.
Si votre composant est impur (ex: il incrémente une variable globale), l'appeler deux fois rendra le bug évident (le compteur avancera de 2 au lieu de 1).

### 3. Comment

#### A. Le test de la double exécution
Imaginez que React fasse ceci en coulisse :

```js
// En Strict Mode Dev :
const resultatVersion1 = VotreComposant(props);
const resultatVersion2 = VotreComposant(props);

// Si VotreComposant est pur, resultatVersion1 et 2 sont identiques visuellement.
```

#### B. Cas concret : Le piège du `Date.now()`
Afficher l'heure actuelle dans un composant le rend techniquement impur, car l'heure change à chaque appel.

```tsx
// ⚠️ Légèrement impur : Le rendu change à chaque milliseconde
function Clock() {
  const time = new Date().toLocaleTimeString();
  return <p>Il est {time}</p>;
}
```
*Note : C'est généralement accepté pour l'affichage de l'heure, mais si cette heure servait à faire un calcul d'ID unique ou une requête API, ce serait un bug critique.*

### 4. Zone de Danger

:::warning Les console.log doubles
À cause du Strict Mode, vous verrez souvent vos `console.log` apparaître deux fois dans la console du navigateur.
**C'est normal.** Ne cherchez pas à "réparer" ça. C'est React qui vous protège en vérifiant la pureté de votre code. En production, cela n'arrivera qu'une seule fois.
:::

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-7}

1.  **Qu'est-ce qu'une fonction pure ?**
    Une fonction qui retourne toujours le même résultat pour les mêmes arguments et qui n'a pas d'effets de bord (ne modifie rien à l'extérieur).

2.  **Pourquoi React appelle-t-il les composants deux fois en mode développement ?**
    Pour vérifier qu'ils sont purs. Si le rendu est différent entre les deux appels ou si des variables externes sont modifiées, cela révèle un bug potentiel.

3.  **Quelle est la différence entre le code impératif et déclaratif ?**
    L'impératif décrit les étapes pour atteindre un résultat (manipulation DOM). Le déclaratif décrit le résultat voulu (JSX) et laisse le framework gérer les étapes.

4.  **Est-il interdit de modifier une variable locale à l'intérieur d'un composant ?**
    Non, modifier une variable définie *dans* la fonction (comme `let formatted = ...`) est autorisé car cela n'affecte pas l'extérieur. C'est la modification de variables *externes* (globales) qui est interdite pendant le rendu.

---

## Exercices : {#exercices-7}

### Exercice 1 - Le Détective de Pureté {#exercice-1---le-detective-de-purete}

🎯 **Objectif** : Identifier et corriger un composant impur.

💼 **Mise en situation** : Vous reprenez le code d'un stagiaire. Il a créé un composant pour afficher des badges avec des couleurs qui changent, mais les couleurs changent de manière erratique à chaque clic ailleurs sur la page.

📝 **Énoncé** :
Le code suivant est impur. Corrigez-le pour qu'il devienne pur. La couleur doit être déterminée de manière stable (par exemple, basée sur le texte du badge).

```tsx
// Code cassé (Impur)
const colors = ['red', 'blue', 'green'];
let index = 0; // 😱 Variable globale !

export function Badge({ text }: { text: string }) {
  const color = colors[index];
  index = index + 1; // 😱 Effet de bord !
  if (index >= colors.length) index = 0;
  
  return <span style={{ color }}>{text}</span>;
}
```

📺 **Résultat attendu** :
Le composant ne doit plus dépendre de la variable globale `index`.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
const colors = ['red', 'blue', 'green'];

export function Badge({ text }: { text: string }) {
  // Correction : On calcule l'index basé sur la longueur du texte.
  // "Admin" (5 lettres) donnera toujours la même couleur.
  // C'est pur : même entrée (text) = même sortie (couleur).
  const index = text.length % colors.length;
  const color = colors[index];
  
  return <span style={{ color }}>{text}</span>;
}
```
</details>

### Exercice 2 - La Tasse de Thé (Pureté et Props) {#exercice-2---la-tasse-de-the}

🎯 **Objectif** : Créer un composant purement visuel (Dumb Component).

💼 **Mise en situation** : Pour une application de salon de thé, créez un composant qui affiche la recette selon le nombre d'invités.

📝 **Énoncé** :
1. Créez un composant `TeaRecipe` qui prend `guestCount` en prop.
2. Il doit calculer la quantité d'eau (200ml par personne) et de thé (2g par personne).
3. Il ne doit **rien** lire ni écrire à l'extérieur de lui-même.
4. Affichez la phrase : "Pour X invités : Y ml d'eau et Z g de thé."

📺 **Résultat attendu** :
Si on passe `guestCount={2}`, on obtient toujours "Pour 2 invités : 400 ml d'eau et 4 g de thé".

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
type TeaRecipeProps = {
  guestCount: number;
};

// Ce composant est pur : il ne fait que calculer et afficher.
export function TeaRecipe({ guestCount }: TeaRecipeProps) {
  // Calculs locaux (autorisés)
  const waterAmount = guestCount * 200;
  const teaAmount = guestCount * 2;

  return (
    <div className="recipe-card">
      <h3>Recette de Thé</h3>
      <p>
        Pour {guestCount} invités : {waterAmount} ml d'eau et {teaAmount} g de thé.
      </p>
    </div>
  );
}
```
</details>

### Exercice 3 - Transformation Impérative vers Déclarative {#exercice-3---transformation-imperative-vers-declarative}

🎯 **Objectif** : Changer de mentalité (Paradigm Shift).

💼 **Mise en situation** : Un ancien développeur jQuery a écrit une fonction pour générer une liste de produits en construisant une chaîne HTML. Convertissez cette horreur en composant React déclaratif.

📝 **Énoncé** :
Transformez cette logique JS pure en composant React `ProductList`.
Données d'entrée : `products = [{ name: "Pomme", stock: 10 }, { name: "Poire", stock: 0 }]`.

*Code Impératif (à ne pas utiliser)* :
```js
function renderProducts(products) {
  let html = '<ul>';
  for (let i = 0; i < products.length; i++) {
    html += '<li>' + products[i].name;
    if (products[i].stock === 0) {
      html += ' (Rupture)';
    }
    html += '</li>';
  }
  html += '</ul>';
  return html;
}
```

📺 **Résultat attendu** :
Un composant React propre utilisant `map` et le rendu conditionnel (ternaire ou `&&`).

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
type Product = {
  name: string;
  stock: number;
};

// Approche déclarative : On décrit la structure finale
export function ProductList({ products }: { products: Product[] }) {
  return (
    <ul>
      {products.map((product) => (
        // N'oubliez pas la key (nom supposé unique ici)
        <li key={product.name} style={{ color: product.stock === 0 ? 'red' : 'black' }}>
          {product.name}
          
          {/* Rendu conditionnel déclaratif */}
          {product.stock === 0 && <strong> (Rupture)</strong>}
        </li>
      ))}
    </ul>
  );
}
```
</details>
```