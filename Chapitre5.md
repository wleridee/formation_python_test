Voici le chapitre **Intégrer du JavaScript dans JSX** pour la formation React 19.2.

```markdown
---
sidebar_label: Intégrer du JavaScript dans JSX
sidebar_position: 5
---

# Chapitre 5 : Intégrer du JavaScript dans JSX

Expressions JavaScript ({}),Attributs dynamiques,Objets JavaScript en JSX

Dans le chapitre précédent, nous avons appris à écrire du JSX statique. C'est bien, mais une application moderne vit grâce à la donnée. Dans ce chapitre, vous allez découvrir le "super-pouvoir" du JSX : les **accolades** `{}`. Elles permettent d'ouvrir une fenêtre vers le moteur JavaScript au milieu de votre balisage.

## Expressions JavaScript ({}) {#expressions-javascript}

### 1. Quoi
Les accolades `{ }` dans le JSX signalent à React : « Arrête de traiter ceci comme du texte/HTML, et traite-le comme du code JavaScript ».
Tout ce qui se trouve entre les accolades doit être une **expression JavaScript valide**.

### 2. Pourquoi
Sans cela, nous ne pourrions afficher que du texte dur (hardcoded). Grâce aux expressions, nous pouvons :
- Afficher des variables (nom d'utilisateur, prix).
- Calculer des valeurs (total d'un panier).
- Manipuler du texte (mise en majuscule).
- Appeler des fonctions de formatage.

### 3. Comment

#### A. Syntaxe de base

```tsx
export function Welcome() {
  const name = "Thomas";
  
  // Sans accolades : affiche littéralement "Bonjour name"
  // Avec accolades : affiche "Bonjour Thomas"
  return <h1>Bonjour {name}</h1>;
}
```

#### B. Cas concret : Affichage dynamique de données
Voici un composant affichant le résumé d'une commande.

```tsx
export function OrderSummary() {
  const subtotal = 100;
  const taxRate = 0.2;
  const shipping = 15;

  const formatPrice = (price: number) => `${price.toFixed(2)} €`;

  return (
    <div className="summary">
      <h2>Résumé de commande</h2>
      {/* Expression : Appel de variable */}
      <p>Sous-total : {subtotal} €</p>
      
      {/* Expression : Calcul mathématique */}
      <p>TVA (20%) : {subtotal * taxRate} €</p>
      
      {/* Expression : Appel de fonction */}
      <p>Livraison : {formatPrice(shipping)}</p>
      
      <hr />
      
      {/* Expression : Calcul complexe */}
      <strong>Total : {formatPrice(subtotal * (1 + taxRate) + shipping)}</strong>
    </div>
  );
}
```

#### C. Exemples pratiques

1.  **Concaténation de chaînes** :
    `<h3>{firstName + ' ' + lastName}</h3>` ou mieux : `<h3>{`${firstName} ${lastName}`}</h3>`
2.  **Opérations ternaires (Conditions)** :
    `<p>Status : {isOnline ? 'En ligne' : 'Hors ligne'}</p>`
3.  **Appel de méthodes** :
    `<p>Message envoyé le : {new Date().toLocaleDateString()}</p>`

### 4. Zone de Danger

:::danger Expression vs Instruction (Statement)
C'est la limitation n°1 qui piège les débutants. Dans `{ ... }`, vous pouvez mettre une **expression** (quelque chose qui retourne une valeur), mais PAS une **instruction** (quelque chose qui contrôle le flux).

❌ **Interdit (Instructions) :**
- `<h1>{ if (user) { return user.name } }</h1>`
- `<div>{ for (let i=0; i<10; i++) { ... } }</div>`
- `<div>{ const x = 5 }</div>`

✅ **Autorisé (Expressions) :**
- Ternaire : `{ user ? user.name : 'Anonyme' }`
- Map (pour les boucles) : `{ list.map(item => ...) }`
:::

---

## Attributs dynamiques {#attributs-dynamiques}

### 1. Quoi
De la même manière que nous insérons du contenu textuel dynamique, nous pouvons rendre les attributs des balises HTML (`src`, `href`, `id`, `disabled`, `className`) dynamiques en utilisant `{}` au lieu de `""`.

### 2. Pourquoi
Une interface réactive doit pouvoir changer l'apparence ou le comportement des éléments selon le contexte :
- Désactiver un bouton si un formulaire est invalide.
- Changer l'avatar selon l'utilisateur connecté.
- Appliquer une classe CSS « error » si une validation échoue.

### 3. Comment

#### A. Syntaxe : Guillemets vs Accolades

```tsx
// 1. Attribut Statique (String)
<img src="logo.png" className="logo" />

// 2. Attribut Dynamique (JavaScript)
const userAvatar = "/users/avatar-42.jpg";
const isActive = true;

<img 
  src={userAvatar} // ✅ Pas de guillemets autour des accolades !
  className={isActive ? "avatar active" : "avatar"} 
/>
```

#### B. Cas concret : Bouton d'action contextuel

```tsx
export function SendButton() {
  const isSending = true;
  const buttonId = "submit-btn-123";

  return (
    <button 
      // L'ID est dynamique
      id={buttonId} 
      // L'état désactivé dépend d'un booléen JS
      disabled={isSending}
      // Le titre (tooltip) change selon l'état
      title={isSending ? "Envoi en cours..." : "Envoyer le message"}
    >
      {isSending ? "Chargement..." : "Envoyer"}
    </button>
  );
}
```

### 4. Zone de Danger

:::warning Ne mélangez pas guillemets et accolades
Une erreur fréquente venant d'autres langages de templating (comme PHP ou Handlebars) :

❌ **Faux :** `<img src="{user.avatar}" />`
En React, cela signifie que l'URL de l'image est littéralement la chaîne de caractères `"{user.avatar}"`, ce qui ne fonctionnera pas.

✅ **Vrai :** `<img src={user.avatar} />`
:::

---

## Objets JavaScript en JSX {#objets-javascript-en-jsx}

### 1. Quoi
Parfois, vous verrez une syntaxe qui ressemble à des doubles accolades : `{{ ... }}`.
Ce n'est pas une syntaxe spéciale de React. C'est simplement :
1.  Une paire d'accolades pour entrer en mode JavaScript `{ ... }`.
2.  Un objet JavaScript littéral à l'intérieur `{ key: value }`.

### 2. Pourquoi
Le cas d'usage le plus courant est l'attribut `style`. En React, les styles inline ne sont pas des chaînes de caractères (comme en HTML), mais des **Objets**.

### 3. Comment

#### A. Syntaxe Style Inline
Les propriétés CSS doivent être écrites en **camelCase** (`backgroundColor` et non `background-color`).

```tsx
export function TrafficLight() {
  return (
    // La première paire {} ouvre le JS
    // La deuxième paire {} définit l'objet
    <div style={{ backgroundColor: "red", width: "100px", height: "100px" }}>
      STOP
    </div>
  );
}
```

#### B. Exemple lisible
Pour éviter la confusion des doubles accolades, il est souvent préférable de définir l'objet à l'extérieur.

```tsx
export function ProfileCard() {
  // Définition de l'objet de style
  const cardStyle = {
    padding: "20px",
    borderRadius: "8px",
    boxShadow: "0 4px 6px rgba(0,0,0,0.1)",
    backgroundColor: "#fff" // camelCase
  };

  const titleStyle = {
    color: "darkblue",
    fontSize: "1.5rem"
  };

  return (
    // On passe l'objet cardStyle dans l'attribut style
    <article style={cardStyle}>
      <h2 style={titleStyle}>Profil</h2>
    </article>
  );
}
```

### 🚨 Limitations des styles inline
Bien que les styles inline soient puissants pour des valeurs dynamiques (comme une barre de progression qui change selon un pourcentage), ils sont moins performants que les classes CSS externes et ne supportent pas les pseudo-sélecteurs (`:hover`, `:focus`) ni les Media Queries. Utilisez `className` pour la majorité de vos styles (voir Chapitre 6).

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-5}

1.  **Peut-on utiliser une boucle `for` directement à l'intérieur des accolades JSX `{ ... }` ?**
    Non, car `for` est une **instruction** (statement), pas une expression. Il faut utiliser `Array.map()` (expression) ou préparer les données avant le `return`.

2.  **Quelle est la différence entre `<div id="app">` et `<div id={appId}>` ?**
    Le premier assigne la chaîne littérale "app". Le second évalue la variable JavaScript `appId` et assigne sa valeur à l'attribut id.

3.  **Pourquoi écrit-on `style={{ color: 'red' }}` avec deux paires d'accolades ?**
    La paire extérieure signale à JSX qu'on veut écrire du JavaScript. La paire intérieure définit un Objet JavaScript littéral.

4.  **Comment écrit-on la propriété CSS `font-size` dans un objet de style React ?**
    En camelCase : `fontSize`.

---

## Exercices : {#exercices-5}

### Exercice 1 - La Biographie Dynamique {#exercice-1---la-biographie-dynamique}

🎯 **Objectif** : Utiliser des expressions pour manipuler du texte.

💼 **Mise en situation** : Vous créez une page de profil pour un réseau social. Les données viennent d'une base de données (simulée ici par des variables).

📝 **Énoncé** :
1. Déclarez 3 variables : `firstName` ("Jean"), `lastName` ("Dupont"), et `birthYear` (1990).
2. Créez un composant `Bio` qui retourne :
   - Un `h1` affichant le nom complet (concaténation).
   - Un `p` calculant l'âge approximatif de la personne (Année courante - Année de naissance).
   - N'écrivez aucun chiffre "2026" en dur pour l'année courante, utilisez `new Date().getFullYear()`.

📺 **Résultat attendu** :
Le navigateur affiche :
**Jean Dupont**
Âge : 36 ans (selon l'année actuelle)

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
export function Bio() {
  const firstName = "Jean";
  const lastName = "Dupont";
  const birthYear = 1990;
  
  // Expression pour obtenir l'année actuelle dynamiquement
  const currentYear = new Date().getFullYear();

  return (
    <div className="bio-card">
      {/* Utilisation des Template Literals pour concaténer proprement */}
      <h1>{`${firstName} ${lastName}`}</h1>
      
      {/* Calcul arithmétique direct dans le JSX */}
      <p>Âge : {currentYear - birthYear} ans</p>
    </div>
  );
}
```
</details>

### Exercice 2 - L'Avatar Intelligent {#exercice-2---l-avatar-intelligent}

🎯 **Objectif** : Maîtriser les attributs dynamiques.

💼 **Mise en situation** : Sur un forum, l'avatar d'un utilisateur doit être rond s'il est un utilisateur standard, mais carré avec une bordure dorée s'il est administrateur. L'image doit aussi avoir un texte alternatif (`alt`) pertinent pour l'accessibilité.

📝 **Énoncé** :
1. Créez un composant `SmartAvatar`.
2. Variables : `avatarUrl` (une URL d'image quelconque), `username` ("AdminUser"), `isAdmin` (booléen `true`).
3. L'image doit avoir :
   - `src` lié à la variable.
   - `alt` affichant "Avatar de [username]".
   - `className` qui vaut "avatar-admin" si `isAdmin` est vrai, sinon "avatar-user".
   - `style` qui applique une bordure `2px solid gold` **seulement si** `isAdmin` est vrai (sinon `none` ou objet vide).

📺 **Résultat attendu** :
Une image s'affiche avec la classe correcte et une bordure dorée (si vous inspectez le DOM).

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
export function SmartAvatar() {
  const avatarUrl = "https://i.pravatar.cc/150";
  const username = "AdminUser";
  const isAdmin = true;

  return (
    <img
      src={avatarUrl}
      // Template string pour un alt dynamique (Accessibilité)
      alt={`Avatar de ${username}`}
      
      // Opérateur ternaire pour changer la classe CSS
      className={isAdmin ? "avatar-admin" : "avatar-user"}
      
      // Style inline conditionnel
      style={{
        border: isAdmin ? "2px solid gold" : "none",
        borderRadius: isAdmin ? "0px" : "50%" // Carré si admin, rond si user
      }}
    />
  );
}
```
</details>

### Exercice 3 - La Barre de Progression {#exercice-3---la-barre-de-progression}

🎯 **Objectif** : Utiliser le style inline pour des valeurs numériques dynamiques (le cas d'usage parfait).

💼 **Mise en situation** : Vous développez un composant d'upload de fichier. La barre de progression doit refléter visuellement le pourcentage d'avancement.

📝 **Énoncé** :
1. Créez un composant `ProgressBar`.
2. Déclarez une variable `progress` (nombre entre 0 et 100, ex: 65).
3. Créez une `div` conteneur (fond gris, largeur 100%).
4. À l'intérieur, créez une `div` de remplissage (fond bleu, hauteur 20px).
5. La **largeur** (`width`) de la div de remplissage doit correspondre à la variable `progress` + "%".
6. Affichez le pourcentage textuel à l'intérieur de la barre.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Une barre colorée remplie à 65%.
> **Annotation** : Montrez que le style `width` dans le DOM correspond à `65%`.
> **Alt Text suggéré** : Barre de progression bleue sur fond gris indiquant 65%.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
export function ProgressBar() {
  const progress = 65; // Imaginez que cette valeur change avec le temps

  return (
    <div style={{ backgroundColor: "#e0e0e0", borderRadius: "8px", width: "100%" }}>
      <div 
        style={{
          // On construit la chaîne "65%" dynamiquement
          width: `${progress}%`, 
          backgroundColor: "#3b82f6",
          height: "24px",
          borderRadius: "8px",
          textAlign: "center",
          color: "white",
          transition: "width 0.5s ease" // Animation fluide
        }}
      >
        {progress}%
      </div>
    </div>
  );
}
```
</details>
```