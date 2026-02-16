Voici le chapitre **Anatomie d'un Composant React** pour la formation React 19.2.

```markdown
---
sidebar_label: Anatomie d'un Composant React
sidebar_position: 3
---

# Chapitre 3 : Anatomie d'un Composant React

Fonctions Composants,Rendu de Markup,Convention de Nommage

Dans ce chapitre, nous allons disséquer la brique fondamentale de toute application React : le **Composant**. Si vous savez écrire une fonction JavaScript, vous savez (presque) écrire un composant React. Cependant, il existe des règles strictes à respecter pour que React puisse les comprendre et les afficher correctement.

## Fonctions Composants {#fonctions-composants}

### 1. Quoi
Un composant React est, littéralement, une fonction JavaScript (ou TypeScript). Cette fonction a une particularité : elle retourne des éléments d'interface utilisateur (UI) décrits en JSX.

Depuis React 16.8 et confirmé en React 19, on utilise exclusivement des **Fonctions** (Function Components) et non plus des Classes.

### 2. Pourquoi
L'approche fonctionnelle offre plusieurs avantages :
- **Simplicité** : Moins de code ("boilerplate") qu'une classe.
- **Isolation** : Chaque composant gère sa propre logique, facilitant les tests unitaires.
- **Réutilisabilité** : Une fois défini, un composant peut être utilisé autant de fois que nécessaire, comme un tampon encreur.

### 3. Comment

#### A. Syntaxe de base

```tsx
// Définition du composant
function WelcomeMessage() {
  return <p>Bienvenue sur notre SaaS !</p>;
}

// Utilisation (Appel)
export default function App() {
  return (
    <div>
      <WelcomeMessage />
      <WelcomeMessage />
    </div>
  );
}
```

#### B. Cas concret : Un bouton réutilisable

Voici un composant robuste typé avec TypeScript.

```tsx
// Button.tsx
// On exporte le composant pour pouvoir l'importer ailleurs
export function PrimaryButton() {
  // Logique métier (ex: analytique, calculs) ici
  const buttonLabel = "Confirmer la commande";

  // Le retour visuel
  return (
    <button type="button" className="btn-primary">
      {buttonLabel}
    </button>
  );
}
```

#### C. Exemples pratiques

1.  **Header de Navigation** : Contient le logo et les liens.
2.  **ProductCard** : Affiche l'image, le titre et le prix d'un produit.
3.  **Avatar** : Affiche la photo de profil de l'utilisateur ou ses initiales.

### 4. Zone de Danger

:::danger Ne jamais définir un composant dans un autre
C'est une erreur classique de débutant.

❌ **À NE PAS FAIRE :**
```tsx
export default function Gallery() {
  // DANGER : Redéfinition à chaque rendu de Gallery !
  // Cela tue les performances et fait perdre le focus des inputs.
  function Image() {
    return <img src="..." />;
  }

  return <Image />;
}
```

✅ **BONNE PRATIQUE :**
Sortez toujours les composants à l'extérieur.
```tsx
function Image() {
  return <img src="..." />;
}

export default function Gallery() {
  return <Image />;
}
```
:::

---

## Rendu de Markup {#rendu-de-markup}

### 1. Quoi
Le "Rendu" est la valeur retournée par votre fonction composant. En React, ce retour est généralement écrit en **JSX** (JavaScript XML). Le navigateur ne comprend pas le JSX, c'est pourquoi nos outils (Vite/React Compiler) le transforment en JavaScript standard.

### 2. Pourquoi
Le rendu décrit **l'intention** de l'interface. React s'occupe ensuite de comparer cette intention avec la page actuelle et de faire les mises à jour minimales nécessaires (le fameux Virtual DOM).

### 3. Comment

#### A. Règle du retour unique
Un composant ne peut retourner qu'**un seul élément racine**.

```tsx
// ❌ Erreur : Deux éléments racines
return (
  <h1>Titre</h1>
  <p>Paragraphe</p>
);

// ✅ Valide : Enveloppé dans un div (ou un Fragment <>)
return (
  <div>
    <h1>Titre</h1>
    <p>Paragraphe</p>
  </div>
);
```

#### B. Règle des parenthèses
Si votre JSX s'étend sur plusieurs lignes, entourez-le de parenthèses `()`. Sans cela, le JavaScript insère automatiquement un point-virgule après `return`, et votre fonction ne renvoie rien (`undefined`).

```tsx
// ❌ Le code après return est ignoré
function BadReturn() {
  return
    <img src="logo.png" />; // Code mort (unreachable)
}

// ✅ Les parenthèses indiquent le bloc de retour
function GoodReturn() {
  return (
    <img src="logo.png" />
  );
}
```

### 🚨 Limitations de l'instruction return
Vous ne pouvez pas retourner une boucle `for` ou une condition `if` directement *dans* le JSX (nous verrons comment gérer cela aux chapitres 10 et 11). Le JSX doit être une expression.

---

## Convention de Nommage {#convention-de-nommage}

### 1. Quoi
En React, il existe une règle d'or : **Les composants doivent toujours commencer par une Majuscule (PascalCase)**.

Les balises HTML standard (`div`, `span`, `h1`) sont en minuscules. Les composants personnalisés (`UserProfile`, `Navbar`) sont en majuscules.

### 2. Pourquoi
Ce n'est pas juste une convention de style, c'est une exigence technique du compilateur JSX.
- Si React voit `<userProfile />`, il pense que c'est une balise HTML standard (comme `<div>`), cherche `document.createElement('userProfile')` et échoue.
- Si React voit `<UserProfile />`, il comprend qu'il doit appeler la fonction `UserProfile`.

### 3. Comment

#### A. Tableau comparatif

| Type | Convention | Exemple | Interprétation React |
| :--- | :--- | :--- | :--- |
| **Composant React** | **PascalCase** | `<SidebarMenu />` | Appelle la fonction JS `SidebarMenu` |
| **Balise HTML** | **lowercase** | `<button />` | Crée un élément DOM `<button>` |
| **Fonction utilitaire** | **camelCase** | `formatDate()` | Code JS standard (hors JSX) |

#### B. Noms de fichiers
Par convention, le nom du fichier correspond au nom du composant qu'il exporte.
- Composant : `PaymentModal`
- Fichier : `PaymentModal.tsx`

### 4. Zone de Danger

:::warning Attention à l'export
Si vous nommez votre fonction en minuscule lors de la définition, même l'export ne la sauvera pas.

```tsx
// ❌ Mauvais
function footer() { return <footer>...</footer>; }

// ✅ Bon
function Footer() { return <footer>...</footer>; }
```
:::

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-3}

1.  **Pourquoi un composant React doit-il impérativement commencer par une majuscule ?**
    Pour que React puisse faire la distinction entre vos composants personnalisés et les balises HTML natives (div, span) lors de la compilation.

2.  **Que se passe-t-il si vous écrivez du code JSX sur plusieurs lignes après un `return` sans mettre de parenthèses ?**
    JavaScript insère automatiquement un point-virgule après le `return`, la fonction retourne donc `undefined` et rien ne s'affiche.

3.  **Peut-on définir une fonction composant à l'intérieur d'une autre fonction composant ?**
    C'est techniquement possible mais **fortement déconseillé**. Cela force React à recréer le composant interne à chaque rendu, causant des problèmes de performance et de perte d'état (focus, inputs).

4.  **Quel est le type de retour d'une fonction composant (implicitement) ?**
    Elle retourne du **JSX** (techniquement un objet React Element).

---

## Exercices : {#exercices-3}

Mettez en pratique la création et l'assemblage de composants.

### Exercice 1 - Le Débugger de Casse {#exercice-1---le-debugger-de-casse}

🎯 **Objectif** : Identifier et corriger les erreurs de nommage et de syntaxe.

💼 **Mise en situation** : Un développeur junior a commis un code qui ne s'affiche pas. Votre tech lead vous demande de corriger le fichier `Profile.tsx` pour que l'interface apparaisse.

📝 **Énoncé** :
Le code suivant est cassé pour 3 raisons. Trouvez-les et corrigez-les.

```tsx
function userAvatar() {
  return <img src="/avatar.png" alt="User" />
}

export default function UserProfile() {
  return
    <div>
      <h1>Profil Utilisateur</h1>
      <userAvatar />
    </div>
}
```

📺 **Résultat attendu** :
Le composant doit s'afficher sans erreur console, avec l'image et le titre.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
// 1. Correction : PascalCase pour le nom du composant
function UserAvatar() {
  return <img src="/avatar.png" alt="User" />;
}

export default function UserProfile() {
  // 2. Correction : Ajout des parenthèses pour le return multi-lignes
  return (
    <div>
      <h1>Profil Utilisateur</h1>
      {/* 3. Correction : Appel avec la majuscule */}
      <UserAvatar />
    </div>
  );
}
```
</details>

### Exercice 2 - Architecture d'une Page SaaS {#exercice-2---architecture-d-une-page-saas}

🎯 **Objectif** : Créer une hiérarchie simple de composants.

💼 **Mise en situation** : Vous travaillez sur le dashboard d'un SaaS CRM. Vous devez créer la structure de base (layout) de la page d'accueil en séparant les responsabilités.

📝 **Énoncé** :
1. Créez trois composants : `Navbar`, `Sidebar`, et `DashboardContent`.
2. Chaque composant doit retourner une `div` simple contenant un texte décrivant sa zone (ex: "Ceci est la Sidebar").
3. Créez un composant principal `AppLayout` qui assemble ces trois morceaux.
4. `Navbar` doit être tout en haut.
5. `Sidebar` et `DashboardContent` doivent être affichés l'un à côté de l'autre (utilisez du CSS basique ou imaginez juste la structure HTML).

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : La page affichant les 3 zones de texte simples.
> **Annotation** : Mettez en évidence la séparation visuelle des 3 composants.
> **Alt Text suggéré** : Rendu navigateur montrant Navbar en haut, Sidebar à gauche et Contenu à droite.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
// Composant pour la barre de navigation
function Navbar() {
  return <nav style={{ borderBottom: "1px solid #ccc", padding: "10px" }}>🔵 Logo CRM</nav>;
}

// Composant pour le menu latéral
function Sidebar() {
  return <aside style={{ width: "200px", background: "#f4f4f4", padding: "10px" }}>📁 Menu</aside>;
}

// Composant pour le contenu principal
function DashboardContent() {
  return <main style={{ flex: 1, padding: "20px" }}>📊 Graphiques et Stats</main>;
}

// Composant Racine qui assemble le tout
export default function AppLayout() {
  return (
    <div>
      <Navbar />
      <div style={{ display: "flex", height: "100vh" }}>
        <Sidebar />
        <DashboardContent />
      </div>
    </div>
  );
}
```
</details>

### Exercice 3 - L'Atome Produit {#exercice-3---l-atome-produit}

🎯 **Objectif** : Comprendre la réutilisabilité.

💼 **Mise en situation** : Pour un site e-commerce, on veut afficher une grille de produits. Plutôt que de copier-coller le code HTML 10 fois, on va créer un composant unique.

📝 **Énoncé** :
1. Créez un composant `ProductBadge` qui affiche "Nouveau" dans un `span`.
2. Créez un composant `ProductCard`.
3. Dans `ProductCard`, affichez un titre de produit (statique pour l'instant, ex: "Baskets React") et utilisez le composant `ProductBadge` à côté.
4. Dans le composant principal `Shop`, affichez 3 fois le composant `ProductCard`.

📺 **Résultat attendu** :
Vous devez voir 3 fois la même carte produit avec le badge "Nouveau".

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
// Petit composant UI réutilisable
function ProductBadge() {
  return (
    <span style={{ backgroundColor: "gold", fontSize: "0.8em", padding: "2px 5px", marginLeft: "10px" }}>
      Nouveau
    </span>
  );
}

// Composant représentant une carte produit
function ProductCard() {
  return (
    <div style={{ border: "1px solid #ddd", padding: "15px", margin: "10px", borderRadius: "8px" }}>
      <h3>
        Baskets React 
        {/* Composition : on utilise le badge ici */}
        <ProductBadge />
      </h3>
      <p>Prix : 120 €</p>
      <button>Ajouter au panier</button>
    </div>
  );
}

// Page principale
export default function Shop() {
  return (
    <div>
      <h1>Nos Produits</h1>
      {/* Réutilisation du même composant 3 fois */}
      <ProductCard />
      <ProductCard />
      <ProductCard />
    </div>
  );
}
```
</details>
```