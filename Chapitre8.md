Voici le chapitre **Votre UI comme un Arbre de Composants** pour la formation React 19.2.

```markdown
---
sidebar_label: Votre UI comme un Arbre de Composants
sidebar_position: 8
---

# Chapitre 8 : Votre UI comme un Arbre de Composants

Hiérarchie des composants, Composants racines, Arbre d'éléments React

Une application React n'est pas une "soupe" de balises HTML mélangées. C'est une structure organisée, hiérarchique et logique. Imaginez votre application comme un arbre généalogique ou l'organigramme d'une entreprise : il y a un grand patron (la racine) et des sous-divisions qui se spécialisent. Dans ce chapitre, nous allons apprendre à "penser en arbre".

## Hiérarchie des composants {#hierarchie-des-composants}

### 1. Quoi
La hiérarchie des composants désigne la relation **Parent-Enfant** entre vos composants.
- Un **Parent** est un composant qui en appelle un autre dans son JSX.
- Un **Enfant** est le composant qui est appelé.

Cette imbrication (nesting) crée une structure arborescente.

### 2. Pourquoi
La hiérarchie permet de respecter le principe de **Séparation des Préoccupations (SoC)**.
Au lieu d'avoir un fichier `App.tsx` de 5000 lignes (Monolithe), on divise l'interface en petits morceaux gérables, testables et réutilisables. Le Parent gère la mise en page (Layout), les Enfants gèrent les détails.

### 3. Comment

#### A. Syntaxe de base

```tsx
// L'Enfant (La feuille de l'arbre)
function Avatar() {
  return <img src="..." alt="Avatar" />;
}

// Le Parent (La branche)
function UserCard() {
  return (
    <div className="card">
      <h1>Profil</h1>
      {/* Imbrication : UserCard devient parent de Avatar */}
      <Avatar />
    </div>
  );
}
```

#### B. Cas concret : Page de Dashboard
Voici comment une page complexe est découpée.

```tsx
// 1. Les briques de base (Atomes/Molécules)
function SearchBar() { /* ... */ }
function UserMenu() { /* ... */ }
function StatWidget() { /* ... */ }

// 2. Les conteneurs (Organismes)
function Header() {
  return (
    <header>
      <SearchBar />
      <UserMenu />
    </header>
  );
}

function Sidebar() { /* ... */ }

function DashboardContent() {
  return (
    <main>
      <StatWidget title="Ventes" />
      <StatWidget title="Visites" />
    </main>
  );
}

// 3. La Page (Template)
export default function DashboardPage() {
  return (
    <div className="layout">
      <Sidebar />
      <div className="main-area">
        <Header />
        <DashboardContent />
      </div>
    </div>
  );
}
```

### 4. Zone de Danger

:::danger Dépendances Circulaires
Attention à ne pas créer de boucles infinies dans vos imports.
Si `ComponentA` importe `ComponentB`, et que `ComponentB` importe `ComponentA`, votre outil de build (Vite/Webpack) va planter ou créer des bugs étranges.
La hiérarchie doit être unidirectionnelle : du haut vers le bas.
:::

---

## Composants racines {#composants-racines}

### 1. Quoi
Le composant racine (souvent nommé `App` ou `Root`) est l'ancêtre commun de tous les autres composants. C'est le tronc de l'arbre. Il est le tout premier composant monté dans le DOM par React.

### 2. Pourquoi
Le composant racine est le lieu idéal pour configurer le contexte global de l'application :
- **Routing** : Quelle page afficher selon l'URL ?
- **Thème** : Dark mode ou Light mode ?
- **Authentification** : L'utilisateur est-il connecté ?
- **Providers** : Configuration des librairies (Redux, React Query, etc.).

### 3. Comment

#### A. Le point d'entrée (main.tsx)
C'est ici que l'arbre React est greffé sur le DOM HTML (généralement dans `<div id="root">`).

```tsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import App from './App';

// Sélection du noeud DOM
const rootElement = document.getElementById('root');

if (rootElement) {
  const root = createRoot(rootElement);
  // Démarrage de l'arbre React avec App comme racine
  root.render(
    <StrictMode>
      <App />
    </StrictMode>
  );
}
```

#### B. Structure typique d'un composant App
```tsx
// App.tsx
import { Navbar } from './components/Navbar';
import { Footer } from './components/Footer';
import { HomePage } from './pages/HomePage';

export default function App() {
  return (
    // Configuration globale (ex: ThemeProvider fictif)
    <div className="app-container theme-dark">
      <Navbar />
      
      {/* Zone de contenu dynamique */}
      <main>
        <HomePage />
      </main>
      
      <Footer />
    </div>
  );
}
```

---

## Arbre d'éléments React {#arbre-d-elements-react}

### 1. Quoi
Il faut distinguer :
- **Le Composant** : La fonction (le plan, le blueprint).
- **L'Élément React** : L'objet retourné par la fonction (une instance, un noeud de l'arbre).

L'arbre d'éléments (Render Tree) est la représentation en mémoire que React construit à chaque rendu. C'est ce que React utilise pour savoir quoi modifier dans le vrai DOM.

### 2. Pourquoi
Comprendre l'arbre de rendu est crucial pour comprendre :
- **La performance** : Si un parent "re-rend" (s'exécute à nouveau), tous ses enfants dans l'arbre seront récursivement re-rendus par défaut.
- **La persistance de l'état** : Tant qu'un composant reste à la même place dans l'arbre, React conserve son état (ce que l'utilisateur a tapé dans un input, par exemple). Si vous détruisez la branche, l'état est perdu.

### 3. Comment

#### A. Visualisation de l'arbre
Pour le code suivant :
```tsx
<App>
  <Header>
    <Logo />
  </Header>
  <Content />
</App>
```

React construit cet arbre en mémoire :
```text
root
 └─ App
     ├─ Header
     │   └─ Logo
     └─ Content
```

#### B. La règle de la position dans l'arbre
Si vous remplacez un composant par un autre au même endroit, React détruit l'ancien arbre (et son état) et en construit un nouveau.

```tsx
export function ToggleTree() {
  const isCompany = true;

  // React voit ici un changement de TYPE de composant
  // Il détruit <PersonalProfile> et monte <CompanyProfile>
  // Tout état local (input rempli) est perdu lors de la bascule.
  return (
    <div>
      {isCompany ? <CompanyProfile /> : <PersonalProfile />}
    </div>
  );
}
```

### 4. Zone de Danger

:::danger Définition imbriquée = Arbre instable
C'est un rappel du Chapitre 3, mais vu sous l'angle de l'arbre.
Si vous définissez un composant *dans* un autre :

```tsx
function Parent() {
  // ❌ Nouvelle fonction créée à CHAQUE rendu
  function Child() { return <p>Enfant</p>; } 
  
  return <Child />;
}
```
Pour React, la fonction `Child` est différente à chaque milliseconde. Il détruit donc la branche et la recrée constamment. L'arbre clignote, les inputs perdent le focus. **Sortez toujours les composants.**
:::

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-8}

1.  **Qu'est-ce qu'un composant racine ?**
    C'est le composant tout en haut de la hiérarchie (généralement `App`), qui contient tous les autres composants et providers.

2.  **Si le composant `Navbar` est utilisé dans `App`, qui est le parent ?**
    `App` est le parent, `Navbar` est l'enfant.

3.  **Pourquoi est-il important de voir l'UI comme un arbre ?**
    Pour comprendre le flux de données (qui passe souvent du haut vers le bas) et comment les mises à jour de rendu se propagent (d'un parent vers ses enfants).

4.  **Que se passe-t-il pour l'état d'un composant s'il est retiré de l'arbre de rendu ?**
    Son état est détruit (unmount). S'il revient plus tard, il sera réinitialisé à zéro (mount).

---

## Exercices : {#exercices-8}

### Exercice 1 - L'Architecte UI {#exercice-1---l-architecte-ui}

🎯 **Objectif** : Analyser une interface visuelle et la traduire en arborescence de composants.

💼 **Mise en situation** : Vous clonez l'interface d'un post "Instagram". Vous devez définir les composants nécessaires sans écrire le CSS, juste la structure.

📝 **Énoncé** :
Créez une hiérarchie de composants pour représenter un Post social.
1. `SocialCard` (Parent)
2. `CardHeader` (Contient Avatar + Nom)
3. `CardImage` (L'image centrale)
4. `CardActions` (Boutons Like, Comment, Share)
5. `CardFooter` (Nombre de likes + Description)

Assemblez le tout dans le parent. Utilisez des `div` simples avec du texte pour simuler le contenu.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Rendu simple des 4 zones de texte empilées.
> **Annotation** : Montrez la structure hiérarchique claire.
> **Alt Text suggéré** : Pile de composants représentant un post réseau social.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
// Composants "Feuilles" (Enfants)
function CardHeader() {
  return <div style={{ borderBottom: '1px solid #eee', padding: '10px' }}>👤 Utilisateur123</div>;
}

function CardImage() {
  return <div style={{ background: '#ddd', height: '200px', display: 'flex', alignItems: 'center', justifyContent: 'center' }}>🖼️ PHOTO</div>;
}

function CardActions() {
  return <div style={{ padding: '10px' }}>❤️ 💬 ✈️</div>;
}

function CardFooter() {
  return <div style={{ padding: '0 10px 10px' }}><strong>102 J'aime</strong> - Superbe photo !</div>;
}

// Composant Racine de cette fonctionnalité
export default function SocialCard() {
  return (
    <div style={{ border: '1px solid #ccc', borderRadius: '8px', maxWidth: '300px', fontFamily: 'sans-serif' }}>
      <CardHeader />
      <CardImage />
      <CardActions />
      <CardFooter />
    </div>
  );
}
```
</details>

### Exercice 2 - Le Refactoring Monolithique {#exercice-2---le-refactoring-monolithique}

🎯 **Objectif** : Extraire des composants pour assainir l'arbre.

💼 **Mise en situation** : Un développeur pressé a tout mis dans `App`. C'est illisible. Découpez le code.

📝 **Énoncé** :
Le composant `MonolithicApp` contient tout : Header, Liste de produits, Footer.
Extrayez 3 composants : `Header`, `ProductList`, `Footer`.
Le rendu final doit rester identique.

```tsx
// Code Monolithique à refactoriser
export function MonolithicApp() {
  return (
    <div>
      <nav><h1>Ma Boutique</h1></nav>
      <ul>
        <li>Produit 1</li>
        <li>Produit 2</li>
      </ul>
      <footer>© 2026 Boutique</footer>
    </div>
  );
}
```

📺 **Résultat attendu** :
Le composant `MonolithicApp` (renommé `CleanApp`) ne doit plus contenir de balises HTML complexes, seulement des appels aux 3 nouveaux composants.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
// 1. Extraction du Header
function Header() {
  return <nav><h1>Ma Boutique</h1></nav>;
}

// 2. Extraction de la Liste
function ProductList() {
  return (
    <ul>
      <li>Produit 1</li>
      <li>Produit 2</li>
    </ul>
  );
}

// 3. Extraction du Footer
function Footer() {
  return <footer>© 2026 Boutique</footer>;
}

// 4. Assemblage propre
export default function CleanApp() {
  return (
    <div>
      <Header />
      <ProductList />
      <Footer />
    </div>
  );
}
```
</details>

### Exercice 3 - Visualiser l'Arbre {#exercice-3---visualiser-l-arbre}

🎯 **Objectif** : Comprendre l'imbrication profonde.

💼 **Mise en situation** : Vous construisez un explorateur de fichiers simple.

📝 **Énoncé** :
1. Créez un composant `File` qui affiche une icône 📄 et un nom.
2. Créez un composant `Folder` qui affiche une icône 📁 et un nom.
3. Le composant `Folder` doit accepter des **enfants** (nous verrons la prop `children` en détail plus tard, pour l'instant codez en dur).
4. Créez une structure : Dossier "Documents" -> Dossier "Projets" -> Fichier "React.pdf".

*Note : Pour cet exercice, vous pouvez imbriquer manuellement les composants dans le JSX du parent.*

📺 **Résultat attendu** :
Une structure visuelle indentée montrant la hiérarchie.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
function FileIcon() { return <span>📄</span>; }
function FolderIcon() { return <span>📁</span>; }

// Composant "Feuille"
function File({ name }: { name: string }) {
  return <div style={{ marginLeft: '20px' }}><FileIcon /> {name}</div>;
}

// Composant qui peut contenir d'autres composants
// Note: Dans les chapitres suivants, on rendra ça dynamique avec {children}
function FolderProjects() {
  return (
    <div style={{ marginLeft: '20px' }}>
      <div><FolderIcon /> Projets</div>
      {/* Imbrication manuelle pour l'exercice */}
      <File name="React.pdf" />
      <File name="Budget.xlsx" />
    </div>
  );
}

// Racine
export default function FileManager() {
  return (
    <div>
      <h3>Mon Disque</h3>
      <div><FolderIcon /> Documents</div>
      <FolderProjects />
      <File name="todo.txt" />
    </div>
  );
}
```
</details>
```