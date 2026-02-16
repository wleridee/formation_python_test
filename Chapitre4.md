Voici le chapitre **Écrire du Markup avec JSX** pour la formation React 19.2.

```markdown
---
sidebar_label: Écrire du Markup avec JSX
sidebar_position: 4
---

# Chapitre 4 : Écrire du Markup avec JSX

Syntaxe JSX,Règles JSX (fermeture de balises),Fragments (<>...< />)

Maintenant que vous comprenez ce qu'est un composant, il est temps de maîtriser le langage qu'ils parlent : le **JSX**. À première vue, cela ressemble à du HTML, mais sous le capot, c'est du JavaScript pur et dur. Ce chapitre va vous apprendre à écrire du markup valide, propre et optimisé pour React.

## Syntaxe JSX {#syntaxe-jsx}

### 1. Quoi
JSX (JavaScript XML) est une extension syntaxique du JavaScript. Elle permet d'écrire la structure de l'interface (balises, attributs) directement dans le code JS, avec une syntaxe familière proche du HTML.

### 2. Pourquoi
Historiquement, les technologies web séparaient le contenu (HTML), le style (CSS) et la logique (JS). React change de paradigme : il sépare les **préoccupations** (Concerns) par composant.
Le JSX permet de visualiser la hiérarchie de l'interface directement là où la logique est définie, rendant le code plus lisible et maintenable.

### 3. Comment

#### A. Syntaxe de base
Le JSX est transformé par votre outil de build (Vite/Babel) en appels de fonction JavaScript (`_jsx(...)`).

```tsx
// Ce qui ressemble à du HTML...
const element = <h1 className="title">Bonjour</h1>;

// ...est en réalité du sucre syntaxique pour du JavaScript.
```

#### B. Les différences majeures avec HTML
Le JSX est plus strict et suit les conventions de nommage JavaScript (camelCase).

| Concept | HTML (Vieux) | JSX (React) | Pourquoi ? |
| :--- | :--- | :--- | :--- |
| **Classes CSS** | `class="box"` | `className="box"` | `class` est un mot-clé réservé en JS |
| **Labels Form** | `for="email"` | `htmlFor="email"` | `for` est réservé pour les boucles |
| **Événements** | `onclick="..."` | `onClick={...}` | Convention camelCase du JS |
| **Attributs composés** | `tabindex="0"` | `tabIndex={0}` | Cohérence avec le DOM API JS |

#### C. Cas concret : Une carte utilisateur
Voici un exemple de structure JSX valide et sémantique.

```tsx
export function UserCard() {
  return (
    <article className="user-card shadow-md">
      <header className="card-header">
        <h2 className="text-xl">Profil Administrateur</h2>
      </header>
      <section className="card-body">
        <p>Statut : Actif</p>
        <button type="button" aria-label="Supprimer l'utilisateur">
          Supprimer
        </button>
      </section>
    </article>
  );
}
```

### 4. Zone de Danger

:::danger JSX n'est pas une chaîne de caractères
Ne mettez jamais de guillemets autour de votre JSX, sinon il sera traité comme une simple string.

❌ **Incorrect :**
```tsx
const element = "<div className='app'>...</div>";
```

✅ **Correct :**
```tsx
const element = <div className="app">...</div>;
```
:::

---

## Règles JSX (fermeture de balises) {#regles-jsx}

### 1. Quoi
Contrairement au HTML qui est permissif (le navigateur "devine" souvent où fermer une balise), le JSX est impitoyable. **Toutes** les balises doivent être explicitement fermées.

### 2. Pourquoi
Le compilateur React doit construire un Arbre (Tree) précis de composants. Une ambiguïté sur la fermeture d'une balise pourrait casser toute la structure de l'application ou générer des erreurs de mémoire.

### 3. Comment

#### A. Balises avec contenu
Si une balise contient des enfants (texte ou autres balises), elle doit avoir une balise fermante correspondante.

```tsx
// ✅ Correct
<div>Contenu</div>

// ❌ Erreur de compilation : balise non fermée
<div>Contenu
```

#### B. Balises auto-fermantes (Self-Closing)
Les balises "vides" (void elements) comme `<img>`, `<input>` ou `<br>` **doivent** se fermer avec `/>`.

```tsx
// En HTML standard, ceci est valide :
// <img src="logo.png">

// En JSX, cela provoque une erreur critique (SyntaxError).
// ✅ Il faut auto-fermer :
<img src="logo.png" alt="Logo" />
<input type="text" />
<br />
```

### 4. Zone de Danger

:::warning Copier-coller du HTML
Lorsque vous copiez du code HTML d'un template ou d'un site existant (exemple : un SVG), vous aurez souvent des erreurs.
Utilisez des convertisseurs en ligne (comme "HTML to JSX") ou l'extension VS Code pour formater automatiquement le HTML en JSX valide.
:::

---

## Fragments (`<>...< />`) {#fragments}

### 1. Quoi
Un Fragment est un composant spécial intégré à React qui permet de grouper une liste d'enfants sans ajouter de nœud supplémentaire (comme un `<div>`) dans le DOM.

Syntaxe courte : `<> ... </>`
Syntaxe longue : `<Fragment> ... </Fragment>`

### 2. Pourquoi
En React, une fonction composant ne peut retourner qu'**un seul élément parent**.
Souvent, on enroule le contenu dans une `div` inutile pour satisfaire cette règle. C'est ce qu'on appelle la "Div Soup".
Cela pose des problèmes :
1.  **CSS Flexbox/Grid** : Une `div` intermédiaire casse la mise en page (parent/enfant direct).
2.  **HTML Sémantique** : Mettre une `div` dans un `ul` ou un `table` est invalide.
3.  **Performance** : Cela alourdit le DOM inutilement.

### 3. Comment

#### A. Le problème (Div Soup)

```tsx
// ❌ Problème : Cette div extra peut casser un layout Flexbox
return (
  <div className="useless-wrapper">
    <h1>Titre</h1>
    <p>Texte</p>
  </div>
);
```

#### B. La solution (Fragment)

```tsx
// ✅ Solution : Aucun nœud n'est ajouté au DOM réel
return (
  <>
    <h1>Titre</h1>
    <p>Texte</p>
  </>
);
```

#### C. Cas concret : Structure de définition
Dans une liste de définition `<dl>`, on ne peut pas mettre de `div` pour grouper `dt` et `dd`.

```tsx
function DefinitionItem() {
  return (
    <>
      <dt className="font-bold">Terme</dt>
      <dd>Définition du terme...</dd>
    </>
  );
}

export default function Glossary() {
  return (
    <dl>
      <DefinitionItem />
      <DefinitionItem />
    </dl>
  );
}
```

### 🚨 Limitations des Fragments
La syntaxe courte `<>` ne peut pas recevoir d'attributs.
Si vous devez passer une propriété `key` (pour une boucle, voir Chapitre 11), vous **devez** utiliser la syntaxe longue :

```tsx
import { Fragment } from 'react';

// ... dans une boucle map ...
<Fragment key={item.id}>
  <h2>{item.title}</h2>
  <p>{item.text}</p>
</Fragment>
```

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-4}

1.  **Pourquoi utilise-t-on `className` au lieu de `class` en JSX ?**
    Car `class` est un mot-clé réservé en JavaScript (pour définir des classes POO). React a choisi d'utiliser l'API DOM `className` pour éviter les conflits.

2.  **Quelle est la différence dans le DOM final entre `<div>...</div>` et `<>...</>` ?**
    `<div>` crée un nœud HTML réel `div` dans la page. `<>...</>` (Fragment) ne crée **aucun** nœud conteneur ; seuls les enfants sont insérés dans le DOM.

3.  **Comment doit-on écrire une balise `<input>` en JSX ?**
    Elle doit être auto-fermante : `<input />`.

4.  **Peut-on mettre deux balises `h1` côte à côte en retour direct d'un composant ?**
    Non, elles doivent être enveloppées dans un parent unique (comme une `div` ou un Fragment).

---

## Exercices : {#exercices-4}

### Exercice 1 - Le Convertisseur HTML vers JSX {#exercice-1---le-convertisseur-html-vers-jsx}

🎯 **Objectif** : Identifier et corriger les erreurs de syntaxe lors de la migration HTML vers JSX.

💼 **Mise en situation** : Vous migrez la landing page d'une startup "Web 2.0" vers React 19. Vous avez copié-collé le header HTML, mais l'application crashe.

📝 **Énoncé** :
Corrigez le code suivant pour qu'il soit du JSX valide.

```javascript
// Code cassé
export function HeroSection() {
  return (
    <div class="hero">
      <h1 style="color: red;">Bienvenue</h1>
      <br>
      <img src="banner.jpg">
      <input type="email" placeholder="Votre email" autofocus>
    </div>
  )
}
```
*Note : Ignorez l'erreur de l'attribut `style` pour l'instant (nous le verrons au chapitre 6), concentrez-vous sur la structure et les attributs de base.*

📺 **Résultat attendu** :
Plus d'erreurs de compilation. `class` devient `className`, les balises sont fermées, `autofocus` devient camelCase.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
export function HeroSection() {
  return (
    // 1. class -> className
    <div className="hero">
      {/* 2. style doit être un objet (anticipation Chap 6) */}
      <h1 style={{ color: "red" }}>Bienvenue</h1>
      
      {/* 3. Fermeture explicite de <br /> */}
      <br />
      
      {/* 4. Fermeture explicite de <img /> */}
      <img src="banner.jpg" alt="Bannière" />
      
      {/* 5. Fermeture explicite + autofocus (camelCase) + autoFocus (React props) */}
      <input type="email" placeholder="Votre email" autoFocus />
    </div>
  );
}
```
</details>

### Exercice 2 - Le Fantôme du DOM (Fragments) {#exercice-2---le-fantome-du-dom}

🎯 **Objectif** : Utiliser les Fragments pour préserver une mise en page Flexbox.

💼 **Mise en situation** : Vous créez un composant `MenuButtons`. Le container parent (dans `App`) est en `display: flex; gap: 20px;`. Si vous entourez vos boutons d'une `div` dans votre composant, l'espacement (gap) ne s'appliquera pas entre les boutons, mais autour de la div.

📝 **Énoncé** :
1. Créez un composant `MenuButtons` qui retourne deux boutons : "Connexion" et "Inscription".
2. Faites en sorte que ce composant ne génère **aucun** wrapper HTML inutile pour que le parent puisse contrôler l'espacement direct entre les deux boutons.
3. Simulez le parent dans `App` avec un style flex.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Deux boutons alignés horizontalement avec un espace visible.
> **Annotation** : Inspecteur DOM ouvert montrant qu'il n'y a pas de `div` autour des boutons.
> **Alt Text suggéré** : Inspection Chrome montrant les boutons comme enfants directs du conteneur Flex.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
function MenuButtons() {
  // Utilisation du Fragment pour ne pas casser le flex du parent
  return (
    <>
      <button>Connexion</button>
      <button>Inscription</button>
    </>
  );
}

export default function App() {
  return (
    // Le parent Flexbox
    <nav style={{ display: 'flex', gap: '20px', padding: '20px' }}>
      <div>Logo</div>
      {/* Les boutons seront des enfants directs de <nav> grâce au Fragment */}
      <MenuButtons />
    </nav>
  );
}
```
</details>

### Exercice 3 - Structure Sémantique stricte {#exercice-3---structure-semantique-stricte}

🎯 **Objectif** : Maîtriser l'imbrication et la fermeture des balises complexes.

💼 **Mise en situation** : Vous développez un composant pour afficher un article de blog. La structure HTML5 doit être impeccable pour le SEO.

📝 **Énoncé** :
Créez un composant `BlogPost` qui retourne la structure suivante en JSX valide :
- Un `article`.
- Dedans : un `h2` (Titre), une `img` (auto-fermante), et un `p` (texte).
- Sous le `p`, un `hr` (séparateur horizontal, auto-fermant).
- Enfin, un `small` pour l'auteur.
N'oubliez pas d'utiliser `className` si vous voulez ajouter des styles fictifs, mais l'important est la validité des balises.

📺 **Résultat attendu** :
Le composant s'affiche. Si vous inspectez le code, la hiérarchie est respectée et toutes les balises void (`img`, `hr`) sont présentes.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
export function BlogPost() {
  return (
    <article className="blog-post">
      <h2>L'avenir de React 19</h2>
      
      {/* Balise auto-fermante obligatoire */}
      <img src="/react-logo.png" alt="Illustration React" />
      
      <p>
        React 19 apporte le compilateur automatique. 
        C'est une révolution pour la performance.
      </p>
      
      {/* Balise auto-fermante obligatoire */}
      <hr />
      
      <small>Écrit par l'équipe React</small>
    </article>
  );
}
```
</details>
```