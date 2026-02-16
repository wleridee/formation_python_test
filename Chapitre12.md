Voici le chapitre **Gérer les Événements** pour la formation React 19.2.

```markdown
---
sidebar_label: Gérer les Événements
sidebar_position: 12
---

# Chapitre 12 : Gérer les Événements

Gestionnaires d'événements, Conventions `onEvent`, Événements synthétiques

Jusqu'à présent, nos composants étaient statiques : ils affichaient des données mais ne réagissaient pas. Une application web moderne doit répondre aux clics, aux frappes au clavier et aux soumissions de formulaires.
En React, la gestion des événements ressemble beaucoup au HTML standard, mais avec quelques super-pouvoirs (et règles strictes) en plus.

## Gestionnaires d'événements (Event Handlers) {#gestionnaires-d-evenements}

### 1. Quoi
Un gestionnaire d'événement (Event Handler) est une **fonction** que vous définissez à l'intérieur de votre composant et que vous attachez à un élément JSX (comme un bouton ou un input) pour qu'elle s'exécute quand l'utilisateur interagit avec cet élément.

### 2. Pourquoi
C'est le pont entre l'interface utilisateur (UI) et la logique métier. C'est ici que vous déclenchez des appels API, des calculs ou des changements de navigation suite à une action utilisateur.

### 3. Comment

#### A. Syntaxe de base
Contrairement au HTML où les événements sont en minuscules (`onclick="do()" `), en React :
1.  Les événements utilisent le **camelCase** (`onClick`, `onMouseEnter`).
2.  On passe la **fonction** elle-même entre accolades, pas une chaîne de caractères.

```tsx
export function SubscribeButton() {
  // 1. Définition de la fonction (le Handler)
  function handleClick() {
    alert("Merci pour votre inscription !");
  }

  return (
    // 2. Attachement à l'événement onClick
    // Notez l'absence de parenthèses après handleClick !
    <button onClick={handleClick}>
      S'abonner
    </button>
  );
}
```

#### B. Passer des arguments
Souvent, vous avez besoin de savoir *quel* élément a été cliqué (ex: ID d'un produit). Pour cela, on utilise une fonction fléchée anonyme.

```tsx
type Product = { id: number; name: string };

export function DeleteButton({ id }: { id: number }) {
  const handleDelete = (productId: number) => {
    console.log(`Suppression du produit ${productId}`);
  };

  return (
    <button 
      // ✅ On crée une fonction qui appelle notre handler avec l'ID
      onClick={() => handleDelete(id)}
      className="btn-danger"
    >
      Supprimer
    </button>
  );
}
```

### 4. Zone de Danger

:::danger Le piège de l'invocation immédiate
C'est l'erreur la plus fréquente chez les débutants React.

❌ **FAUX :**
```tsx
// La fonction est exécutée TOUT DE SUITE au rendu, pas au clic !
<button onClick={handleClick()}>Cliquez-moi</button>
```

✅ **VRAI :**
```tsx
// On passe la référence de la fonction. React l'appellera plus tard.
<button onClick={handleClick}>Cliquez-moi</button>
```

Si vous devez passer des arguments, n'invoquez pas directement. Enveloppez dans une fonction fléchée : `onClick={() => handleClick(id)}`.
:::

---

## Événements Synthétiques et Objet Event {#evenements-synthetiques}

### 1. Quoi
Lorsque votre fonction est appelée, React lui passe un argument : l'objet **Event** (souvent noté `e`).
En React, ce n'est pas l'événement natif du navigateur, mais un **SyntheticEvent**. C'est une enveloppe (wrapper) autour de l'événement natif.

### 2. Pourquoi
Les navigateurs (Chrome, Safari, Firefox) gèrent parfois les événements différemment. Le SyntheticEvent de React garantit que votre code fonctionne de manière **identique sur tous les navigateurs**. Il possède la même interface que l'événement natif (`e.target`, `e.preventDefault()`, etc.).

### 3. Comment

#### A. Empêcher le comportement par défaut (`preventDefault`)
Crucial pour les formulaires (pour éviter le rechargement de la page).

```tsx
import { FormEvent } from 'react';

export function LoginForm() {
  // Typage TypeScript de l'événement de formulaire
  function handleSubmit(e: FormEvent<HTMLFormElement>) {
    e.preventDefault(); // 🛑 Empêche le rechargement de la page
    console.log("Tentative de connexion...");
  }

  return (
    <form onSubmit={handleSubmit}>
      <input type="email" placeholder="Email" />
      <button type="submit">Se connecter</button>
    </form>
  );
}
```

#### B. Arrêter la propagation (`stopPropagation`)
L'événement "bulle" (bubble) du bas vers le haut. Si vous cliquez sur un bouton à l'intérieur d'une div cliquable, les deux événements se déclencheront.

```tsx
import { MouseEvent } from 'react';

export function CardWithButton() {
  const handleCardClick = () => alert("Navigation vers le détail");
  
  const handleButtonClick = (e: MouseEvent<HTMLButtonElement>) => {
    e.stopPropagation(); // 🛑 Arrête la bulle ici. handleCardClick ne sera pas appelé.
    alert("Ajouté au panier !");
  };

  return (
    <div onClick={handleCardClick} className="card p-4 border cursor-pointer">
      <h3>Produit Super</h3>
      <p>Cliquez sur la carte pour voir les détails.</p>
      
      <button onClick={handleButtonClick} className="bg-blue-500 text-white">
        Acheter
      </button>
    </div>
  );
}
```

---

## Conventions de Nommage `onEvent` {#conventions-onevent}

### 1. Quoi
Il existe une convention standard dans la communauté React :
- **Props** : Nommées `on[Action]` (ex: `onPlay`, `onUpload`).
- **Handlers** : Nommés `handle[Action]` (ex: `handlePlay`, `handleUpload`).

### 2. Pourquoi
Cela permet de distinguer instantanément le rôle de la fonction :
- `onPlay` dans les props ? -> "Le parent me demande de l'avertir quand ça joue".
- `handlePlay` dans le composant ? -> "C'est la logique qui s'exécute quand ça joue".

### 3. Comment

#### A. Composant Enfant (Définit l'API)

```tsx
type ToolbarProps = {
  onPlay: () => void;   // Le parent doit fournir cette fonction
  onUpload: () => void;
};

export function Toolbar({ onPlay, onUpload }: ToolbarProps) {
  return (
    <div>
      {/* On connecte l'événement DOM onClick à la prop onPlay */}
      <button onClick={onPlay}>Lire la vidéo</button>
      <button onClick={onUpload}>Uploader</button>
    </div>
  );
}
```

#### B. Composant Parent (Implémente la logique)

```tsx
export function App() {
  // Le Handler (l'implémentation)
  function handlePlayMovie() {
    console.log("Lecture du film...");
  }

  return (
    // Le Parent passe son handler à la prop 'onPlay' de l'enfant
    <Toolbar 
      onPlay={handlePlayMovie} 
      onUpload={() => alert("Upload!")} 
    />
  );
}
```

### 🚨 Limitations
Les événements personnalisés n'existent pas en React comme en Vue (`$emit`). On passe simplement des fonctions via les props. C'est du JavaScript pur : le parent donne une fonction à l'enfant, l'enfant l'appelle.

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-12}

1.  **Quelle est la différence entre `<button onClick={handleClick}>` et `<button onClick={handleClick()}>` ?**
    Le premier passe la fonction pour être exécutée *plus tard* (au clic). Le second exécute la fonction *immédiatement* au chargement de la page (ce qui est généralement un bug).

2.  **Qu'est-ce qu'un Événement Synthétique ?**
    C'est un objet créé par React qui enveloppe l'événement natif du navigateur pour garantir une compatibilité parfaite entre tous les navigateurs.

3.  **Comment empêcher un formulaire de recharger la page lors de la soumission ?**
    En appelant `e.preventDefault()` à l'intérieur du gestionnaire d'événement `onSubmit`.

4.  **Si je clique sur un bouton situé dans une div qui a aussi un `onClick`, que se passe-t-il ?**
    Les deux gestionnaires s'exécutent (propagation/bubbling). Pour empêcher la div de réagir au clic du bouton, il faut utiliser `e.stopPropagation()` sur le bouton.

---

## Exercices : {#exercices-12}

### Exercice 1 - Le Console Logger Paramétré {#exercice-1---le-console-logger-parametre}

🎯 **Objectif** : Passer des arguments à un gestionnaire d'événement.

💼 **Mise en situation** : Vous construisez un clavier virtuel pour une application de caisse enregistreuse.

📝 **Énoncé** :
1. Créez un composant `Numpad`.
2. Affichez des boutons pour les chiffres 1, 2 et 3.
3. Créez une fonction unique `handleNumberClick(num: number)` qui affiche dans la console : "Chiffre saisi : X".
4. Attachez cette fonction aux boutons.

📺 **Résultat attendu** :
Cliquer sur "1" logue "Chiffre saisi : 1", etc.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
export function Numpad() {
  // Fonction unique qui accepte un paramètre
  const handleNumberClick = (num: number) => {
    console.log(`Chiffre saisi : ${num}`);
  };

  return (
    <div style={{ display: 'flex', gap: '10px' }}>
      {/* Utilisation de la fonction fléchée pour passer l'argument */}
      <button onClick={() => handleNumberClick(1)}>1</button>
      <button onClick={() => handleNumberClick(2)}>2</button>
      <button onClick={() => handleNumberClick(3)}>3</button>
    </div>
  );
}
```
</details>

### Exercice 2 - L'Intercepteur de Formulaire {#exercice-2---l-intercepteur-de-formulaire}

🎯 **Objectif** : Utiliser `e.preventDefault()` et accéder aux valeurs basiques.

💼 **Mise en situation** : Une simple barre de recherche qui ne doit pas recharger la page.

📝 **Énoncé** :
1. Créez un composant `SearchBar`.
2. Il contient un formulaire (`<form>`) avec un `<input>` et un `<button>`.
3. Gérez l'événement `onSubmit` du formulaire.
4. Empêchez le rechargement de la page.
5. Affichez une alerte "Recherche lancée !" (Nous apprendrons à récupérer la valeur tapée proprement au prochain chapitre avec `useState` ou `Refs`).

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Barre de recherche simple.
> **Annotation** : Montrez l'alerte du navigateur qui apparaît sans recharger la page.
> **Alt Text suggéré** : Formulaire de recherche interceptant la soumission.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { FormEvent } from 'react';

export function SearchBar() {
  const handleSearch = (e: FormEvent<HTMLFormElement>) => {
    // 1. Étape cruciale : Bloquer le rechargement standard HTML
    e.preventDefault();
    
    // 2. Logique métier
    alert("Recherche lancée ! (Page non rechargée)");
  };

  return (
    // L'événement est mis sur le form, pas sur le bouton !
    // Cela permet de valider aussi en appuyant sur "Entrée" dans l'input
    <form onSubmit={handleSearch}>
      <input type="text" placeholder="Rechercher..." name="query" />
      <button type="submit">Go</button>
    </form>
  );
}
```
</details>

### Exercice 3 - Le Problème de la Galerie (StopPropagation) {#exercice-3---le-probleme-de-la-galerie}

🎯 **Objectif** : Maîtriser la propagation des événements.

💼 **Mise en situation** : Une galerie d'images. Cliquer sur l'image l'ouvre en grand (zoom). Mais il y a un petit bouton "❤️ Like" sur l'image. Cliquer sur "Like" ne doit PAS ouvrir le zoom.

📝 **Énoncé** :
1. Créez un composant `ImageCard`.
2. La div principale a un `onClick` qui fait `alert("Zoom sur l'image")`.
3. À l'intérieur, un bouton "Like" a un `onClick` qui fait `alert("J'aime !")`.
4. Testez : cliquer sur Like déclenche les DEUX alertes (bug).
5. Corrigez avec `stopPropagation`.

📺 **Résultat attendu** :
Cliquer sur "Like" affiche uniquement "J'aime !". Cliquer ailleurs sur l'image affiche "Zoom sur l'image".

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { MouseEvent } from 'react';

export function ImageCard() {
  const handleZoom = () => {
    alert("🔍 Zoom sur l'image ouverte");
  };

  const handleLike = (e: MouseEvent<HTMLButtonElement>) => {
    // Correction : On arrête la bulle ici.
    // L'événement ne remontera pas jusqu'à la div parent.
    e.stopPropagation();
    
    alert("❤️ Liké !");
  };

  return (
    // La zone cliquable principale
    <div 
      onClick={handleZoom} 
      style={{ 
        width: '200px', 
        height: '150px', 
        backgroundColor: '#eee', 
        position: 'relative',
        cursor: 'pointer',
        display: 'flex',
        alignItems: 'center',
        justifyContent: 'center'
      }}
    >
      <span>Image (Cliquez-moi)</span>
      
      {/* Le bouton interne */}
      <button 
        onClick={handleLike}
        style={{
          position: 'absolute',
          bottom: '10px',
          right: '10px',
          backgroundColor: 'white',
          border: '1px solid #ccc'
        }}
      >
        ❤️ Like
      </button>
    </div>
  );
}
```
</details>
```