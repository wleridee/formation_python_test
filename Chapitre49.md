Voici le chapitre **Composants HTML et SVG intégrés à React** pour la formation React 19.2.

```markdown
---
sidebar_label: Composants HTML et SVG intégrés à React
sidebar_position: 49
---

# Chapitre 49 : Composants HTML et SVG intégrés à React

Balises HTML courantes, Attributs HTML en JSX, Balises SVG

React ne réinvente pas le Web : il l'enveloppe. Au cœur de chaque application React, aussi complexe soit-elle, se trouvent les briques fondamentales du DOM : les balises HTML (`<div>`, `<span>`, `<h1>`) et SVG (`<svg>`, `<path>`).

Cependant, comme JSX est une extension de JavaScript et non du HTML pur, il existe quelques différences syntaxiques subtiles mais critiques qu'il faut maîtriser pour éviter les erreurs de compilation et garantir l'accessibilité.

---

## 1. Les Balises HTML Standards {#balises-html-standards}

### 1. Quoi
Dans JSX, toute balise commençant par une **lettre minuscule** est interprétée par React comme une balise HTML native.
React supporte pratiquement toutes les balises HTML5 standard (`<header>`, `<footer>`, `<article>`, `<video>`, etc.).

### 2. Pourquoi
React utilise ces primitives pour construire le Virtual DOM. Contrairement aux composants personnalisés (qui commencent par une Majuscule), ces balises ne sont pas "importées" : elles sont intrinsèques à `react-dom`.

### 3. Comment

#### A. Syntaxe de base

```tsx
// ✅ Correct : Minuscule = HTML natif
const element = <div className="container">Bonjour</div>;

// ❌ Incorrect : Majuscule = React cherche un composant "Div" qui n'existe pas
const badElement = <Div className="container">Bonjour</Div>;
```

#### B. Cas concret : Sémantique HTML5

Il est crucial d'utiliser les bonnes balises pour l'accessibilité et le SEO, pas juste des `<div>` partout.

```tsx
function BlogPost({ title, content }: { title: string; content: string }) {
  return (
    <article className="blog-post">
      <header>
        <h1>{title}</h1>
        <time dateTime="2023-10-12">12 Octobre 2023</time>
      </header>
      
      <section>
        <p>{content}</p>
      </section>

      <footer>
        <small>Écrit par l'équipe React</small>
      </footer>
    </article>
  );
}
```

### 4. Zone de Danger : L'auto-fermeture

En HTML5, certaines balises sont "void" (vides) et ne nécessitent pas de fermeture (ex: `<input>`, `<br>`, `<img>`).
En **JSX**, **toutes** les balises doivent être fermées.

*   ❌ HTML : `<br>`
*   ✅ JSX : `<br />`

*   ❌ HTML : `<img src="...">`
*   ✅ JSX : `<img src="..." />`

---

## 2. Les Attributs HTML en JSX (Props) {#attributs-html-en-jsx}

### 1. Quoi
Puisque JSX est du JavaScript, les attributs HTML deviennent des **clés d'objet**. Comme `class` et `for` sont des mots réservés en JavaScript, React utilise des équivalents en **camelCase**.

### 2. Pourquoi
Pour éviter les conflits de syntaxe JS (`class` pour les classes POO, `for` pour les boucles) et pour unifier l'accès aux propriétés du DOM (qui utilisent le camelCase, ex: `element.tabIndex`).

### 3. Comment

#### A. Les équivalences critiques

| Concept | HTML Standard | JSX (React) |
| :--- | :--- | :--- |
| Classes CSS | `class="box"` | `className="box"` |
| Labels Formulaire | `for="email"` | `htmlFor="email"` |
| Événements | `onclick="..."` | `onClick={...}` |
| Focus Clavier | `tabindex="0"` | `tabIndex={0}` |
| Contenu Editable | `contenteditable` | `contentEditable` |
| Lecture Seule | `readonly` | `readOnly` |

#### B. L'attribut `style`

L'attribut `style` en React n'accepte pas une chaîne de caractères ("color: red;"), mais un **objet JavaScript** avec des propriétés en camelCase.

```tsx
// ❌ Incorrect
// <div style="background-color: red; margin-top: 10px">...</div>

// ✅ Correct
<div style={{ 
  backgroundColor: 'red', // camelCase
  marginTop: '10px'       // valeur string avec unité
}}>
  Attention
</div>
```

#### C. Les attributs `data-*` et `aria-*`

React respecte la convention standard pour ces attributs :
*   `data-*` : Restent en kebab-case (ex: `data-testid="my-id"`).
*   `aria-*` : Restent en kebab-case (ex: `aria-label="Fermer"`).

#### D. Cas concret : Un bouton accessible

```tsx
interface ButtonProps {
  isDisabled: boolean;
  label: string;
  onClick: () => void;
}

function AccessibleButton({ isDisabled, label, onClick }: ButtonProps) {
  return (
    <button
      // Attributs standards camelCase
      className="btn-primary"
      onClick={onClick}
      disabled={isDisabled} // Booléen : si true, attribut présent
      tabIndex={0}
      
      // Attributs d'accessibilité (ARIA)
      aria-disabled={isDisabled}
      aria-label={label}
      
      // Attributs de données personnalisés
      data-tracker-id="btn_submit_123"
      
      // Style dynamique
      style={{ opacity: isDisabled ? 0.5 : 1, cursor: isDisabled ? 'not-allowed' : 'pointer' }}
    >
      {label}
    </button>
  );
}
```

### 🚨 Limitations : `dangerouslySetInnerHTML`

Parfois, vous devez insérer du HTML brut (venant d'un CMS par exemple). React vous force à utiliser une syntaxe explicite et "effrayante" pour vous rappeler le risque de failles XSS.

```tsx
<div dangerouslySetInnerHTML={{ __html: "<p>Contenu non échappé</p>" }} />
```

---

## 3. Balises SVG dans React {#balises-svg-dans-react}

### 1. Quoi
React permet d'utiliser des balises SVG (`<svg>`, `<path>`, `<circle>`, `<g>`) directement dans le JSX, sans passer par des fichiers images `.svg` externes.

### 2. Pourquoi
*   **Performance** : Pas de requête HTTP supplémentaire.
*   **Dynamisme** : Vous pouvez contrôler la couleur (`fill`), la taille ou la forme via des props React et des variables d'état.
*   **Animation** : Facile à animer avec CSS ou JS via React.

### 3. Comment

Comme pour le HTML, les attributs SVG doivent être convertis en **camelCase**.
*   `stroke-width` -> `strokeWidth`
*   `stroke-linecap` -> `strokeLinecap`
*   `fill-rule` -> `fillRule`

*Exception : Les attributs `data-*` et `aria-*` restent en kebab-case.*

#### A. Exemple : Icône dynamique

```tsx
interface IconProps {
  color?: string;
  size?: number;
  isFilled?: boolean;
}

function HeartIcon({ color = "red", size = 24, isFilled = false }: IconProps) {
  return (
    <svg 
      width={size} 
      height={size} 
      viewBox="0 0 24 24" // Attention : viewBox est sensible à la casse (V majuscule)
      xmlns="http://www.w3.org/2000/svg"
    >
      <path 
        d="M12 21.35l-1.45-1.32C5.4 15.36 2 12.28 2 8.5 2 5.42 4.42 3 7.5 3c1.74 0 3.41.81 4.5 2.09C13.09 3.81 14.76 3 16.5 3 19.58 3 22 5.42 22 8.5c0 3.78-3.4 6.86-8.55 11.54L12 21.35z"
        fill={isFilled ? color : "none"}
        stroke={color}
        strokeWidth="2"
      />
    </svg>
  );
}
```

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-49}

1.  **Pourquoi ne peut-on pas utiliser `class` pour définir une classe CSS en JSX ?**
    Parce que `class` est un mot-clé réservé en JavaScript (pour définir des classes ES6). React utilise `className` à la place pour correspondre à l'API DOM native.

2.  **Quelle est la règle générale pour les attributs composés en JSX (ex: `tabindex`) ?**
    Ils doivent être écrits en **camelCase** (ex: `tabIndex`). Les exceptions principales sont les attributs `data-*` et `aria-*` qui restent en kebab-case.

3.  **Comment passer un style inline dynamique à une balise en React ?**
    On passe un **objet JavaScript** à la prop `style`, avec les propriétés CSS en camelCase (ex: `{{ backgroundColor: 'blue' }}`), et non une chaîne de caractères CSS.

---

## Exercices : {#exercices-49}

### Exercice 1 - Le Traducteur HTML vers JSX {#exercice-1---le-traducteur-html-vers-jsx}

🎯 **Objectif** : Corriger un bloc de code HTML copié-collé pour le rendre valide en React/JSX.

💼 **Mise en situation** : Un designer vous a donné un bout de code HTML pour une carte "Utilisateur", mais il contient des erreurs syntaxiques JSX classiques.

📝 **Énoncé** :
Convertissez le code suivant en composant React valide. Corrigez les erreurs (`class`, `style` string, balise non fermée).

**Code HTML brut (à corriger) :**
```html
<div class="user-card" style="border: 1px solid #ccc; padding: 10px;">
  <img src="avatar.jpg" class="avatar">
  <h3 onclick="alert('Hello')">Jean Dupont</h3>
  <input type="text" maxlength="10" readonly>
  <br>
  <label for="bio">Biographie</label>
</div>
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
export function UserCard() {
  return (
    // 1. class -> className
    // 2. style="..." -> style={{ ... }} avec propriétés camelCase et valeurs string
    <div className="user-card" style={{ border: '1px solid #ccc', padding: '10px' }}>
      
      {/* 3. Fermeture automatique des balises orphelines (self-closing) */}
      <img src="avatar.jpg" className="avatar" alt="Avatar utilisateur" />
      
      {/* 4. onclick -> onClick (camelCase) + fonction fléchée */}
      <h3 onClick={() => alert('Hello')}>Jean Dupont</h3>
      
      {/* 5. maxlength -> maxLength, readonly -> readOnly, self-closing */}
      <input type="text" maxLength={10} readOnly />
      
      {/* 6. <br> -> <br /> */}
      <br />
      
      {/* 7. for -> htmlFor */}
      <label htmlFor="bio">Biographie</label>
    </div>
  );
}
```
</details>

### Exercice 2 - Le Loader SVG Progressif {#exercice-2---le-loader-svg-progressif}

🎯 **Objectif** : Manipuler des attributs SVG via des props React.

💼 **Mise en situation** : Vous créez un indicateur de chargement circulaire (Ring Progress). La progression doit être contrôlable via une prop.

📝 **Énoncé** :
1. Créez un composant `CircularProgress` prenant une prop `percentage` (0 à 100).
2. Utilisez un SVG avec deux cercles : un gris (fond) et un bleu (progression).
3. Utilisez `strokeDasharray` et `strokeDashoffset` pour afficher la portion du cercle correspondant au pourcentage.
   *   *Astuce Math* : Circonférence = 2 * PI * R. Offset = Circonférence - (pourcentage / 100 * Circonférence).
4. Le rayon (r) est de 40.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Un cercle SVG partiellement rempli.
> **Annotation** : Montrez que la longueur du trait coloré dépend de la prop.
> **Alt Text suggéré** : Loader SVG React 75%.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
interface ProgressProps {
  percentage: number; // 0 à 100
}

export function CircularProgress({ percentage }: ProgressProps) {
  const radius = 40;
  const stroke = 8;
  const normalizedRadius = radius - stroke * 2;
  const circumference = normalizedRadius * 2 * Math.PI;
  
  // Calcul de l'offset pour l'animation de remplissage
  const strokeDashoffset = circumference - (percentage / 100) * circumference;

  return (
    <svg height={radius * 2} width={radius * 2}>
      {/* Cercle de fond (gris) */}
      <circle
        stroke="#e6e6e6"
        strokeWidth={stroke}
        fill="transparent"
        r={normalizedRadius}
        cx={radius}
        cy={radius}
      />
      
      {/* Cercle de progression (bleu) */}
      <circle
        stroke="blue"
        fill="transparent"
        strokeWidth={stroke}
        // Conversion des attributs SVG kebab-case en camelCase React
        strokeDasharray={circumference + ' ' + circumference}
        style={{ 
          strokeDashoffset, 
          transition: 'stroke-dashoffset 0.5s ease-in-out',
          transform: 'rotate(-90deg)', // Pour commencer à midi (en haut)
          transformBox: 'fill-box',
          transformOrigin: 'center'
        }}
        r={normalizedRadius}
        cx={radius}
        cy={radius}
      />
      
      {/* Texte central */}
      <text 
        x="50%" 
        y="50%" 
        dy=".3em" 
        textAnchor="middle"
        style={{ fontSize: '12px', fontWeight: 'bold' }}
      >
        {percentage}%
      </text>
    </svg>
  );
}
```
</details>

### Exercice 3 - Formulaire Sémantique et Accessible {#exercice-3---formulaire-semantique}

🎯 **Objectif** : Utiliser les attributs `htmlFor`, `aria-*` et `type` correctement.

💼 **Mise en situation** : Créer un champ de mot de passe avec un bouton "Afficher/Masquer". L'accessibilité est prioritaire.

📝 **Énoncé** :
1. Un label lié à l'input via `htmlFor`.
2. Un input dont le `type` change entre "password" et "text".
3. Un bouton avec `aria-label` qui change ("Afficher le mot de passe" vs "Masquer...").
4. L'attribut `aria-pressed` sur le bouton pour indiquer l'état.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState } from 'react';

export function PasswordInput() {
  const [showPassword, setShowPassword] = useState(false);
  const inputId = "user-password"; // ID unique pour la liaison label/input

  return (
    <div style={{ display: 'flex', flexDirection: 'column', gap: 8, maxWidth: 300 }}>
      {/* Label lié via htmlFor */}
      <label htmlFor={inputId} style={{ fontWeight: 'bold' }}>
        Votre mot de passe
      </label>

      <div style={{ display: 'flex', gap: 4 }}>
        <input
          id={inputId}
          // Type dynamique
          type={showPassword ? "text" : "password"}
          className="form-control"
          placeholder="••••••••"
          style={{ flex: 1, padding: 8 }}
        />

        <button
          type="button" // Important pour ne pas submit le formulaire
          onClick={() => setShowPassword(!showPassword)}
          // Attributs d'accessibilité
          aria-label={showPassword ? "Masquer le mot de passe" : "Afficher le mot de passe"}
          aria-pressed={showPassword}
          style={{ cursor: 'pointer' }}
        >
          {showPassword ? "👁️‍🗨️" : "👁️"}
        </button>
      </div>
    </div>
  );
}
```
</details>
```