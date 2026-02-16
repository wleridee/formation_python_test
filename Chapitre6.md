Voici le chapitre **Styliser les Composants React** pour la formation React 19.2.

```markdown
---
sidebar_label: Styliser les Composants React
sidebar_position: 6
---

# Chapitre 6 : Styliser les Composants React

Propriété `className`,Styles Inline,Fichiers CSS

Une application fonctionnelle mais laide ne vend pas. Dans ce chapitre, nous allons voir comment habiller nos composants. React ne dicte pas une manière unique de gérer le CSS, mais il existe des conventions standard (Standard de l'Industrie 2026) pour garder vos styles maintenables et performants.

## Propriété `className` {#propriete-classname}

### 1. Quoi
C'est la méthode la plus courante et la plus performante. Elle consiste à attacher des classes CSS à vos éléments JSX, exactement comme vous le feriez en HTML avec l'attribut `class`.
La seule différence est syntaxique : en React, on utilise **`className`** (camelCase) car `class` est un mot réservé en JavaScript.

### 2. Pourquoi
L'utilisation de classes CSS permet de séparer la structure (JSX) de la présentation (CSS). Cela permet :
- De bénéficier de toute la puissance du CSS (Media Queries, pseudo-classes `:hover`, animations).
- D'utiliser des bibliothèques utilitaires modernes comme **Tailwind CSS**.
- De mettre en cache le fichier CSS par le navigateur pour des performances optimales.

### 3. Comment

#### A. Syntaxe de base

```tsx
// Dans le fichier .tsx
<button className="btn-primary">Cliquez-moi</button>

// Dans le fichier .css lié
/* .btn-primary { background-color: blue; } */
```

#### B. Cas concret : Classes conditionnelles
Souvent, une classe dépend de l'état du composant (ex: un bouton actif ou inactif).
La méthode moderne (2026) consiste à utiliser des "Template Literals" ou une librairie légère comme `clsx` (non couverte ici pour rester sur React pur).

```tsx
export function AlertMessage() {
  const isError = true;
  const isVisible = true;

  return (
    <div 
      // Si isError est vrai, on ajoute 'alert-error', sinon 'alert-success'
      // Si isVisible est vrai, on ajoute 'visible', sinon 'hidden'
      className={`alert ${isError ? 'alert-error' : 'alert-success'} ${isVisible ? 'visible' : 'hidden'}`}
    >
      {isError ? "Une erreur est survenue" : "Opération réussie"}
    </div>
  );
}
```

#### C. Exemples pratiques

1.  **Tailwind CSS (Standard Moderne)** :
    `<div className="bg-blue-500 text-white p-4 rounded-lg hover:bg-blue-600">...</div>`
    *Note : Tailwind est omniprésent en 2026. Il s'utilise exclusivement via `className`.*

2.  **Modules CSS (Isolation)** :
    `import styles from './Button.module.css';`
    `<button className={styles.primaryBtn}>...</button>`

### 4. Zone de Danger

:::danger className vs class
Si vous écrivez `<div class="box">` par habitude du HTML, React affichera un avertissement dans la console, mais cela **pourrait** fonctionner (mode compatibilité). Ne vous y fiez pas.
En React 19, `class` est strictement interdit pour éviter des comportements imprévisibles avec les Web Components. Utilisez toujours `className`.
:::

---

## Styles Inline {#styles-inline}

### 1. Quoi
Le style inline consiste à appliquer des règles CSS directement sur l'élément via l'attribut `style`.
Contrairement au HTML où `style` est une chaîne de caractères (`style="color: red;"`), en React, `style` accepte un **Objet JavaScript**.

### 2. Pourquoi
C'est utile pour des valeurs **dynamiques** calculées à la volée par le JavaScript, impossibles à prévoir dans un fichier CSS statique.
Exemples :
- Une image de fond chargée depuis une API.
- Une coordonnée X/Y qui suit la souris.
- Une couleur choisie par l'utilisateur via un color picker.

### 3. Comment

#### A. Syntaxe de base
Les propriétés CSS doivent être converties en **camelCase**.
`background-color` devient `backgroundColor`.
`margin-left` devient `marginLeft`.

```tsx
const divStyle = {
  color: 'blue',
  backgroundImage: 'url(...)', // camelCase
  fontSize: '20px' // N'oubliez pas l'unité (px, rem)
};

return <div style={divStyle}>Hello</div>;
```

#### B. Cas concret : Avatar dynamique avec image de fond

```tsx
export function UserAvatar() {
  const imageUrl = "https://i.pravatar.cc/150?u=a042581f4e29026704d";
  const size = 64; // px

  return (
    <div
      className="avatar-circle" // Classes pour le style statique (arrondi, ombre)
      style={{
        // Styles dynamiques injectés ici
        backgroundImage: `url(${imageUrl})`,
        width: `${size}px`,
        height: `${size}px`
      }}
    />
  );
}
```

### 🚨 Limitations de l'approche Inline
- **Performance** : Plus lent que les classes CSS car le moteur JS doit créer l'objet à chaque rendu.
- **Fonctionnalités manquantes** : Impossible d'utiliser les **Media Queries** (`@media`), les **pseudo-sélecteurs** (`:hover`, `:focus`, `::before`) ou les **keyframes** d'animation directement dans l'attribut `style`.
**Règle d'or** : Utilisez `className` pour tout ce qui est statique, et `style` uniquement pour les valeurs qui changent dynamiquement.

---

## Fichiers CSS {#fichiers-css}

### 1. Quoi
React ne gère pas nativement les fichiers CSS, mais votre outil de build (Vite, Next.js) le fait pour vous. Vous pouvez importer un fichier `.css` directement dans votre fichier JavaScript/TypeScript.

### 2. Pourquoi
Cela permet de garder les styles proches des composants qui les utilisent (colocation). Si je supprime le composant `Button.tsx`, je vois immédiatement que je peux supprimer `Button.css`.

### 3. Comment

#### A. Import global
Dans `App.tsx` ou `main.tsx` :
```tsx
import './index.css'; // S'applique à toute l'application
```

#### B. Import spécifique au composant
```tsx
// Card.tsx
import './Card.css'; // Les styles définis ici sont chargés quand Card est utilisé

export function Card() {
  return <div className="card">...</div>;
}
```

### 4. Zone de Danger

:::warning Conflits de noms (Global Scope)
Par défaut, un fichier CSS importé est **GLOBAL**.
Si vous définissez `.button { color: red; }` dans `Header.css` et `.button { color: blue; }` dans `Footer.css`, **la dernière règle chargée gagnera partout**. C'est le problème de la "Global Namespace Pollution".

✅ **Solution** : Utilisez des **CSS Modules** (fichiers nommés `Composant.module.css`) ou adoptez une convention de nommage stricte comme **BEM** (Block Element Modifier).
:::

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-6}

1.  **Pourquoi utilise-t-on `className` plutôt que `class` en React ?**
    Parce que JSX est une extension de JavaScript et que `class` est un mot réservé du langage JS.

2.  **Comment écrit-on `text-align: center` dans un objet de style inline React ?**
    `{ textAlign: 'center' }` (camelCase).

3.  **Peut-on utiliser `:hover` dans un style inline (`style={{...}}`) ?**
    Non, les pseudo-classes CSS ne sont pas supportées dans l'attribut `style`. Il faut utiliser une classe CSS ou gérer les événements JS (`onMouseEnter`).

4.  **Si j'importe `Button.css` dans `Button.tsx`, les styles sont-ils limités à ce composant uniquement ?**
    Non, par défaut, ils sont injectés globalement dans la page `<head>`. Pour les isoler, il faut utiliser des CSS Modules (`Button.module.css`).

---

## Exercices : {#exercices-6}

### Exercice 1 - Le Thème Sombre/Clair {#exercice-1---le-theme-sombre-clair}

🎯 **Objectif** : Manipuler les classes CSS conditionnelles via des Template Literals.

💼 **Mise en situation** : Vous créez une carte de profil pour un dashboard SaaS. L'utilisateur peut basculer entre un mode sombre et clair. Le composant doit s'adapter.

📝 **Énoncé** :
1. Créez un composant `ProfileCard`.
2. Définissez une variable booléenne `isDarkMode` (mettez-la à `true` ou `false` pour tester).
3. La `div` principale doit toujours avoir la classe `card`.
4. Si `isDarkMode` est vrai, ajoutez la classe `card-dark`, sinon `card-light`.
5. Utilisez un fichier CSS externe pour définir des couleurs différentes (ex: fond noir/texte blanc pour dark, fond blanc/texte noir pour light).

📺 **Résultat attendu** :
En changeant la variable `isDarkMode` dans le code, l'apparence de la carte change instantanément.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
// ProfileCard.tsx
import './ProfileCard.css';

export function ProfileCard() {
  const isDarkMode = true; // Changez ceci pour tester

  return (
    <div className={`card ${isDarkMode ? 'card-dark' : 'card-light'}`}>
      <h2>John Doe</h2>
      <p>Développeur Fullstack</p>
    </div>
  );
}
```

```css
/* ProfileCard.css */
.card {
  padding: 20px;
  border-radius: 8px;
  transition: all 0.3s ease;
  border: 1px solid #ccc;
}

.card-light {
  background-color: #fff;
  color: #333;
}

.card-dark {
  background-color: #222;
  color: #fff;
  border-color: #444;
}
```
</details>

### Exercice 2 - La Jauge de Performance {#exercice-2---la-jauge-de-performance}

🎯 **Objectif** : Utiliser le style inline pour une visualisation de données.

💼 **Mise en situation** : Vous affichez un indicateur de performance serveur (CPU). La couleur de la jauge doit changer dynamiquement : vert (<50%), orange (<80%), rouge (>80%).

📝 **Énoncé** :
1. Créez un composant `CpuGauge`.
2. Variable `usage` (entier entre 0 et 100).
3. Déterminez la couleur (`color`) via une logique JS (if/else ou ternaire).
4. Appliquez cette couleur via `style` sur la barre de jauge.
5. La largeur de la barre doit aussi dépendre de `usage`.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Une barre remplie à 85% en rouge.
> **Annotation** : Montrez que la couleur est calculée par le JS.
> **Alt Text suggéré** : Jauge CPU rouge indiquant une charge élevée.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
export function CpuGauge() {
  const usage = 85;

  // Logique pure JS pour déterminer la couleur
  let gaugeColor = '#4caf50'; // Vert par défaut
  if (usage > 80) gaugeColor = '#f44336'; // Rouge
  else if (usage > 50) gaugeColor = '#ff9800'; // Orange

  return (
    <div className="gauge-container" style={{ border: '1px solid #ccc', height: '20px', width: '300px' }}>
      <div
        className="gauge-fill"
        style={{
          width: `${usage}%`, // Largeur dynamique
          backgroundColor: gaugeColor, // Couleur dynamique
          height: '100%',
          transition: 'width 0.5s'
        }}
      />
      <span>{usage}%</span>
    </div>
  );
}
```
</details>

### Exercice 3 - Le Bouton BEM {#exercice-3---le-bouton-bem}

🎯 **Objectif** : Adopter une convention de nommage stricte (BEM) avec `className`.

💼 **Mise en situation** : Pour éviter les conflits CSS dans une grande équipe, votre Lead Dev impose la méthodologie BEM (Block Element Modifier). Vous devez créer un bouton qui respecte cette convention.

📝 **Énoncé** :
1. Créez un composant `Button`.
2. Il accepte deux variables (props simulées) : `variant` ('primary' ou 'secondary') et `size` ('small' ou 'large').
3. Générez une chaîne de caractères `className` qui respecte BEM :
   - Block : `btn`
   - Modifiers : `btn--primary`, `btn--large`...
4. Exemple de résultat attendu dans le DOM : `<button class="btn btn--primary btn--large">`.

📺 **Résultat attendu** :
Un bouton correctement stylisé avec des classes composées proprement, sans logique CSS complexe, tout est dans la construction de la chaîne de classe.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
export function Button() {
  // Simulation des props
  const variant = 'primary'; // 'primary' | 'secondary'
  const size = 'large';      // 'small' | 'large'

  // Construction de la classe BEM
  // Astuce : On utilise un tableau et .join(' ') pour éviter les espaces en trop
  const classNames = [
    'btn',                // Block
    `btn--${variant}`,    // Modifier Variant
    `btn--${size}`        // Modifier Size
  ].join(' ');

  return (
    <button className={classNames}>
      Action BEM
    </button>
  );
}

/* CSS Associé (pour info)
.btn { ... }
.btn--primary { background: blue; }
.btn--secondary { background: grey; }
.btn--large { padding: 1rem; }
.btn--small { padding: 0.5rem; }
*/
```
</details>
```