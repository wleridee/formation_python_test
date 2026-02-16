Voici le chapitre **Passer des Props aux Composants** pour la formation React 19.2.

```markdown
---
sidebar_label: Passer des Props aux Composants
sidebar_position: 9
---

# Chapitre 9 : Passer des Props aux Composants

Définir des props, Utiliser des props, Props par défaut, Destructuring des props

Les composants React seraient bien inutiles s'ils ne pouvaient pas communiquer. Imaginez une fonction JavaScript qui n'accepterait aucun argument : elle ferait toujours exactement la même chose.
Les **Props** (raccourci pour *Propriétés*) sont l'équivalent des arguments de fonction pour vos composants. Elles permettent au composant **Parent** de passer des données ou de la configuration au composant **Enfant**.

## Définir et Utiliser des Props {#definir-et-utiliser-des-props}

### 1. Quoi
Les Props sont un objet JavaScript contenant des données envoyées par un composant parent à un composant enfant. Elles permettent de rendre les composants **réutilisables** et **dynamiques**.

### 2. Pourquoi
Sans props, vous devriez créer un composant `ButtonSubmit`, un composant `ButtonCancel`, un composant `ButtonLogin`, etc.
Avec les props, vous créez un seul composant `Button` et vous lui dites : "Toi, tu es bleu et tu sers à valider", "Toi, tu es gris et tu sers à annuler".

### 3. Comment

#### A. Le Parent envoie (Syntaxe JSX)
C'est similaire aux attributs HTML, mais vous pouvez passer n'importe quelle valeur JavaScript (pas juste des chaînes).

```tsx
// Le Parent passe des données à l'Enfant "UserProfile"
<UserProfile 
  username="Thomas"       // Chaîne de caractères
  age={32}                // Nombre (dans des accolades)
  isAdmin={true}          // Booléen
  tags={['dev', 'react']} // Tableau
/>
```

#### B. L'Enfant reçoit (Typage TypeScript)
Le composant reçoit un seul argument, généralement nommé `props`. En TypeScript, nous définissons une interface pour valider ce contrat.

```tsx
// 1. On définit le "Contrat" (Type)
type UserProfileProps = {
  username: string;
  age: number;
  isAdmin: boolean;
  tags: string[];
};

// 2. On utilise l'argument props
export function UserProfile(props: UserProfileProps) {
  return (
    <div className="card">
      <h2>Profil de {props.username}</h2>
      <p>Âge : {props.age}</p>
      {props.isAdmin && <span className="badge">Admin</span>}
    </div>
  );
}
```

#### C. Cas concret : Bouton réutilisable

```tsx
type ButtonProps = {
  label: string;
  onClick: () => void; // Fonction callback
  variant: 'primary' | 'secondary'; // Union type (très puissant)
};

export function Button(props: ButtonProps) {
  return (
    <button 
      className={`btn btn-${props.variant}`}
      onClick={props.onClick}
    >
      {props.label}
    </button>
  );
}
```

### 4. Zone de Danger

:::danger Les Props sont Immuables (Read-Only)
C'est la règle absolue de React : **Un composant ne doit jamais modifier ses propres props.**
Le flux de données est unidirectionnel (One-Way Data Flow) : du Parent vers l'Enfant. Si l'Enfant veut changer une valeur, il doit demander au Parent de le faire (via un événement/callback).

❌ **Interdit :**
```tsx
function Counter(props) {
  props.count = props.count + 1; // 💥 Erreur : Props est en lecture seule
}
```
:::

---

## Destructuring et Props par défaut {#destructuring-et-props-par-defaut}

### 1. Quoi
La destructuration (Destructuring) est une fonctionnalité ES6 qui permet d'extraire des propriétés d'un objet directement dans la signature de la fonction. C'est le standard moderne en React.
Cela permet aussi de définir facilement des **valeurs par défaut**.

### 2. Pourquoi
- **Lisibilité** : On évite de répéter `props.machin`, `props.truc` partout.
- **Robustesse** : Si le parent oublie de passer une prop optionnelle, le composant ne plante pas et utilise une valeur de repli.

### 3. Comment

#### A. Syntaxe Moderne

```tsx
type AvatarProps = {
  url: string;
  size?: number; // le '?' signifie optionnel en TS
  shape?: 'circle' | 'square';
};

// Destructuring direct dans les parenthèses
// size = 50 : Valeur par défaut si size est undefined
export function Avatar({ url, size = 50, shape = 'circle' }: AvatarProps) {
  return (
    <img 
      src={url} 
      className={`avatar-${shape}`} 
      width={size} 
      height={size} 
      alt="Avatar"
    />
  );
}
```

#### B. Tableau Comparatif

| Approche | Syntaxe | Avantages |
| :--- | :--- | :--- |
| **Legacy** | `function Comp(props) { return props.name }` | Aucun, verbeux. |
| **Legacy (Default)** | `Comp.defaultProps = { ... }` | Déprécié en React moderne. |
| **Moderne** | `function Comp({ name }) { return name }` | Clair, concis. |
| **Moderne (Default)** | `function Comp({ name = 'Invité' })` | Standard JavaScript ES6. |

### 4. Zone de Danger

:::warning Props par défaut vs Null
La valeur par défaut (`size = 50`) ne s'applique que si la prop est `undefined` (ou absente).
Si le parent passe explicitement `null` (`<Avatar size={null} />`), la valeur par défaut **ne sera pas** utilisée. En React, préférez ne pas passer la prop plutôt que de passer `null`.
:::

---

## La Prop spéciale `children` {#la-prop-speciale-children}

### 1. Quoi
React possède une prop magique nommée `children`. Elle contient tout ce qui est inclus **entre** la balise ouvrante et la balise fermante de votre composant.

### 2. Pourquoi
Elle permet la **Composition**.
Certains composants ne connaissent pas leur contenu à l'avance : une `Card`, une `Modal`, une `Sidebar`. Ils agissent comme des "boîtes" (wrappers) qui stylisent ou positionnent ce qu'on leur donne, sans se soucier de ce que c'est.

### 3. Comment

#### A. Définition du Wrapper
En TypeScript, le type pour "n'importe quel élément React valide" est `React.ReactNode`.

```tsx
import { ReactNode } from 'react';

type CardProps = {
  title: string;
  children: ReactNode; // Accepte du texte, du JSX, null, etc.
};

export function Card({ title, children }: CardProps) {
  return (
    <div className="card-container">
      <div className="card-header">
        <h3>{title}</h3>
      </div>
      <div className="card-content">
        {/* On injecte le contenu ici */}
        {children}
      </div>
    </div>
  );
}
```

#### B. Utilisation

```tsx
<Card title="Bienvenue">
  {/* Tout ceci est passé dans la prop 'children' */}
  <p>Ceci est un paragraphe.</p>
  <button>Cliquez-moi</button>
</Card>
```

---

## Prop Drilling (Introduction) {#prop-drilling}

### 1. Quoi
Le "Prop Drilling" (forage de props) se produit quand vous devez passer une donnée d'un Grand-Parent à un Petit-Petit-Enfant. Les composants intermédiaires doivent recevoir la prop et la repasser, même s'ils ne l'utilisent pas eux-mêmes.

### 2. Pourquoi en parler ?
C'est une limitation naturelle du flux de données unidirectionnel.
Pour un niveau ou deux, c'est normal. Mais sur 5 niveaux, cela rend le code difficile à maintenir.

### 3. Comment l'éviter (Aperçu)
1.  **Composition** : Passer des composants entiers via `children` au lieu de passer des données brutes.
2.  **Context API** : Pour des données globales (Thème, User), nous utiliserons `useContext` (Chapitre 30).

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-9}

1.  **Les props sont-elles modifiables par le composant qui les reçoit ?**
    Non, elles sont en lecture seule (read-only). Essayer de les modifier provoquera une erreur.

2.  **Quelle est la syntaxe moderne pour définir une valeur par défaut pour une prop ?**
    La destructuration avec affectation par défaut dans les arguments de la fonction : `function MyComp({ name = 'Default' })`.

3.  **À quoi sert la prop `children` ?**
    Elle permet de récupérer le contenu imbriqué à l'intérieur des balises du composant (`<Comp>CONTENU</Comp>`).

4.  **En TypeScript, quel type utilise-t-on généralement pour la prop `children` ?**
    `ReactNode` (importé depuis 'react').

---

## Exercices : {#exercices-9}

### Exercice 1 - Le Badge Configurable {#exercice-1---le-badge-configurable}

🎯 **Objectif** : Définir des props typées et utiliser la destructuration.

💼 **Mise en situation** : Vous créez le Design System de votre startup. Vous avez besoin d'un composant `Badge` pour afficher des statuts (Nouveau, Promo, Solde).

📝 **Énoncé** :
1. Créez un composant `Badge`.
2. Il accepte 3 props :
   - `label` (string, obligatoire) : le texte.
   - `color` (string, optionnel) : la couleur de fond (ex: 'red', 'blue').
   - `icon` (string, optionnel) : un émoji à afficher avant le texte.
3. Si `color` n'est pas fournie, la valeur par défaut doit être `'gray'`.
4. Utilisez le style inline pour appliquer la couleur (`backgroundColor`).

📺 **Résultat attendu** :
`<Badge label="New" color="red" />` affiche un badge rouge "New".
`<Badge label="Archive" />` affiche un badge gris "Archive".

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
type BadgeProps = {
  label: string;
  color?: string; // Optionnel
  icon?: string;  // Optionnel
};

// Destructuration avec valeur par défaut pour color
export function Badge({ label, color = 'gray', icon }: BadgeProps) {
  return (
    <span 
      style={{ 
        backgroundColor: color, 
        color: 'white', 
        padding: '4px 8px', 
        borderRadius: '4px',
        fontWeight: 'bold'
      }}
    >
      {/* Rendu conditionnel : si icon existe, on l'affiche avec un espace */}
      {icon && <span style={{ marginRight: '5px' }}>{icon}</span>}
      {label}
    </span>
  );
}
```
</details>

### Exercice 2 - La Modal Universelle (Children) {#exercice-2---la-modal-universelle}

🎯 **Objectif** : Maîtriser la prop `children` pour la composition.

💼 **Mise en situation** : Votre application a besoin de fenêtres modales (popups). Parfois pour confirmer une suppression, parfois pour un formulaire de login. Le cadre de la modal est toujours le même, seul le contenu change.

📝 **Énoncé** :
1. Créez un composant `Modal`.
2. Il prend `title` (string) et `children` (ReactNode).
3. Structure HTML suggérée : une `div` overlay (fond sombre), une `div` content (fond blanc).
4. Affichez le titre dans un `h2` et le `children` en dessous.
5. Instanciez cette Modal avec un formulaire de connexion factice à l'intérieur.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Une fenêtre blanche sur fond sombre avec un titre "Connexion" et deux inputs à l'intérieur.
> **Annotation** : Identifiez visuellement ce qui vient de la prop `children`.
> **Alt Text suggéré** : Modal React affichant un formulaire injecté via children.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { ReactNode } from 'react';

type ModalProps = {
  title: string;
  children: ReactNode;
};

export function Modal({ title, children }: ModalProps) {
  return (
    // Overlay sombre
    <div style={{
      position: 'fixed', inset: 0, backgroundColor: 'rgba(0,0,0,0.5)',
      display: 'flex', alignItems: 'center', justifyContent: 'center'
    }}>
      {/* Fenêtre blanche */}
      <div style={{ backgroundColor: 'white', padding: '20px', borderRadius: '8px', minWidth: '300px' }}>
        <div style={{ borderBottom: '1px solid #eee', marginBottom: '10px' }}>
          <h2>{title}</h2>
        </div>
        
        {/* Zone de contenu dynamique */}
        <div>{children}</div>
        
        <div style={{ marginTop: '10px', textAlign: 'right' }}>
          <button>Fermer</button>
        </div>
      </div>
    </div>
  );
}

// Utilisation (Exemple)
export function App() {
  return (
    <Modal title="Connexion requise">
      {/* Tout ceci est le "children" */}
      <p>Veuillez entrer vos identifiants.</p>
      <input type="text" placeholder="Email" style={{ display: 'block', margin: '5px 0' }} />
      <input type="password" placeholder="Mot de passe" style={{ display: 'block', margin: '5px 0' }} />
      <button style={{ backgroundColor: 'blue', color: 'white' }}>Se connecter</button>
    </Modal>
  );
}
```
</details>

### Exercice 3 - La Carte Produit Robuste {#exercice-3---la-carte-produit-robuste}

🎯 **Objectif** : Gérer les props manquantes et le typage strict.

💼 **Mise en situation** : Vous affichez une liste de produits E-commerce. Certains produits n'ont pas d'image, d'autres n'ont pas de prix promo. Votre composant ne doit pas planter.

📝 **Énoncé** :
1. Interface `ProductCardProps` :
   - `name`: string
   - `price`: number
   - `discountPrice`: number (optionnel)
   - `imageUrl`: string (optionnel)
   - `inStock`: boolean (par défaut `true`)
2. Si `imageUrl` manque, afficher une image par défaut ("https://placehold.co/200").
3. Si `discountPrice` est présent, afficher l'ancien prix barré et le nouveau en gras/rouge. Sinon, afficher juste le prix normal.
4. Si `inStock` est faux, griser toute la carte (opacité 0.5).

📺 **Résultat attendu** :
Une carte qui s'adapte visuellement selon les données disponibles, sans erreur console.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
type ProductCardProps = {
  name: string;
  price: number;
  discountPrice?: number; // Optionnel
  imageUrl?: string;      // Optionnel
  inStock?: boolean;      // Optionnel
};

export function ProductCard({ 
  name, 
  price, 
  discountPrice, 
  imageUrl = "https://placehold.co/200", // Image par défaut
  inStock = true // Stock par défaut
}: ProductCardProps) {
  
  return (
    <div style={{ 
      border: '1px solid #ddd', 
      padding: '10px', 
      opacity: inStock ? 1 : 0.5 // Gestion visuelle du stock
    }}>
      <img src={imageUrl} alt={name} width="100%" />
      <h3>{name}</h3>
      
      <div>
        {/* Rendu conditionnel du prix */}
        {discountPrice ? (
          <>
            <span style={{ textDecoration: 'line-through', color: 'gray', marginRight: '10px' }}>
              {price} €
            </span>
            <span style={{ color: 'red', fontWeight: 'bold' }}>
              {discountPrice} €
            </span>
          </>
        ) : (
          <span style={{ fontWeight: 'bold' }}>{price} €</span>
        )}
      </div>

      {!inStock && <p style={{ color: 'red' }}>Rupture de stock</p>}
    </div>
  );
}
```
</details>
```