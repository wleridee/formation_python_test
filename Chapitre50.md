Voici le chapitre **Éléments de Formulaire en React** pour la formation React 19.2.

```markdown
---
sidebar_label: Éléments de Formulaire en React
sidebar_position: 50
---

# Chapitre 50 : Éléments de Formulaire en React

`controlled components`, `uncontrolled components`, Balises `<form>`, `<input>`, `<select>`, `<textarea>`, Gestion des soumissions

Les formulaires sont le principal moyen d'interaction entre l'utilisateur et votre application. En HTML standard, les éléments de formulaire (input, select, textarea) gèrent leur propre état interne : quand vous tapez dans un champ, le DOM met à jour la valeur affichée.

En React, nous avons deux philosophies pour gérer ces données :
1.  **Composants Contrôlés (Controlled)** : React est la seule source de vérité (via le State).
2.  **Composants Non-Contrôlés (Uncontrolled)** : Le DOM garde la vérité, React la lit au moment de la soumission.

---

## 1. Composants Contrôlés (Controlled Components) {#composants-controles}

### 1. Quoi
Un élément de formulaire est dit "contrôlé" lorsque sa valeur est pilotée **entièrement** par l'état React (`state`). L'élément DOM ne peut pas changer sa valeur de lui-même ; il doit demander à React de le faire.

### 2. Pourquoi
*   **Validation instantanée** : Vous pouvez valider la saisie caractère par caractère (ex: force du mot de passe).
*   **Formatage** : Vous pouvez imposer un format (ex: numéro de téléphone, majuscules).
*   **Synchronisation** : Plusieurs champs peuvent dépendre les uns des autres.

### 3. Comment

Le pattern est toujours le même : `value` est lié au State, et `onChange` met à jour ce State.

#### A. Syntaxe de base

```tsx
import { useState } from 'react';

function EmailInput() {
  const [email, setEmail] = useState('');

  return (
    <input
      type="email"
      value={email} // 1. La source de vérité
      onChange={(e) => setEmail(e.target.value)} // 2. La demande de mise à jour
    />
  );
}
```

#### B. Cas concret : Validation et Feedback

```tsx
import { useState } from 'react';

function PasswordField() {
  const [password, setPassword] = useState('');
  const isValid = password.length >= 8;

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    // On met à jour l'état avec la nouvelle saisie
    setPassword(e.target.value);
  };

  return (
    <div className="input-group">
      <label htmlFor="pwd">Mot de passe :</label>
      <input
        id="pwd"
        type="password"
        value={password}
        onChange={handleChange}
        style={{ borderColor: isValid ? 'green' : 'red' }}
      />
      {!isValid && <small>Le mot de passe doit faire 8 caractères.</small>}
    </div>
  );
}
```

### 4. Zone de Danger

:::danger Warning Console "A component is changing an uncontrolled input..."
Si vous passez `value={undefined}` ou `value={null}` à un input, React pense qu'il est non-contrôlé. Si ensuite vous lui passez une string, React vous avertira d'un changement de type.
✅ **Toujours initialiser le state** avec une chaîne vide `useState('')` et non `useState()`.
:::

---

## 2. Composants Non-Contrôlés (Uncontrolled Components) {#composants-non-controles}

### 1. Quoi
Dans cette approche, on laisse le DOM gérer l'état de l'input comme en HTML classique. React ne surveille pas chaque frappe de clavier. On récupère la valeur uniquement quand on en a besoin (généralement à la soumission via `FormData` ou via une `ref`).

### 2. Pourquoi
*   **Performance** : Pas de re-rendu à chaque frappe (utile pour les formulaires géants).
*   **Simplicité** : Moins de code boilerplate si aucune validation temps réel n'est requise.
*   **Intégration** : Plus facile à utiliser avec des bibliothèques JS non-React.

### 3. Comment

On utilise `defaultValue` (pour la valeur initiale) et l'attribut `name` pour récupérer les données via `FormData`.

#### A. Utilisation Moderne avec FormData (Recommandé)

C'est la méthode standard en React 19 pour les formulaires simples ou lors de l'utilisation des Server Actions (Chapitre 60).

```tsx
function SignupForm() {
  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault(); // Empêche le rechargement de page
    
    const formData = new FormData(e.currentTarget);
    const username = formData.get('username'); // Récupération par l'attribut name
    
    console.log("Soumission de :", username);
  };

  return (
    <form onSubmit={handleSubmit}>
      <label>
        Nom d'utilisateur :
        {/* Pas de value, pas de onChange, juste un name */}
        <input name="username" type="text" defaultValue="Invité" />
      </label>
      <button type="submit">Envoyer</button>
    </form>
  );
}
```

#### B. Utilisation avec `useRef`

Utile si vous devez lire la valeur impérativement *sans* soumettre le formulaire.

```tsx
import { useRef } from 'react';

function SearchBar() {
  const inputRef = useRef<HTMLInputElement>(null);

  const handleSearch = () => {
    // Lecture directe du DOM
    alert(`Recherche de : ${inputRef.current?.value}`);
  };

  return (
    <>
      <input ref={inputRef} type="text" />
      <button onClick={handleSearch}>Chercher</button>
    </>
  );
}
```

---

## 3. `<textarea>` et `<select>` : Normalisation React {#textarea-select-normalisation}

### 1. Quoi
En HTML, `<textarea>` utilise ses enfants pour le contenu, et `<select>` utilise l'attribut `selected` sur les options. React unifie tout cela : tous utilisent l'attribut `value`.

### 2. Pourquoi
Pour avoir une API cohérente. Que ce soit un input texte, une zone de texte ou une liste déroulante, vous contrôlez la valeur via la prop `value` sur l'élément parent.

### 3. Comment

#### A. `<textarea>`

```tsx
// ❌ HTML Classique (Ne pas faire en React)
// <textarea>Texte initial</textarea>

// ✅ React
<textarea 
  value={message} 
  onChange={e => setMessage(e.target.value)} 
/>
```

#### B. `<select>`

Au lieu de chercher quelle `<option>` a l'attribut `selected`, on dit au `<select>` quelle est la valeur active.

```tsx
const [fruit, setFruit] = useState('coco');

// Le select affiche "Noix de Coco" car la value correspond
<select value={fruit} onChange={e => setFruit(e.target.value)}>
  <option value="pomme">Pomme</option>
  <option value="banane">Banane</option>
  <option value="coco">Noix de Coco</option>
</select>
```

Pour un select multiple, la valeur est un tableau :
`<select multiple={true} value={['pomme', 'coco']} ...>`

---

## 4. Gestion des Inputs Spéciaux (Checkbox, Radio, File) {#checkbox-radio-file}

### 1. Checkbox et Radio
Ils n'utilisent pas `value` pour leur état, mais `checked`.

```tsx
const [accepted, setAccepted] = useState(false);

<input 
  type="checkbox" 
  checked={accepted} 
  onChange={e => setAccepted(e.target.checked)} // Notez le .checked au lieu de .value
/>
```

### 2. Input File
Un champ fichier est **toujours non-contrôlé** car sa valeur est en lecture seule pour des raisons de sécurité (on ne peut pas définir le fichier via JS).

```tsx
function FileUploader() {
  const handleFileChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    // On accède à la liste des fichiers via .files
    const file = e.target.files?.[0];
    if (file) console.log("Fichier sélectionné :", file.name);
  };

  return <input type="file" onChange={handleFileChange} />;
}
```

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-50}

1.  **Quelle est la différence fondamentale entre un composant contrôlé et non-contrôlé ?**
    Dans un composant contrôlé, l'état React (`state`) est la source de vérité. Dans un composant non-contrôlé, c'est le DOM qui détient la valeur, récupérée via `ref` ou `FormData`.

2.  **Pourquoi ne doit-on jamais passer `null` ou `undefined` à la prop `value` d'un input contrôlé ?**
    Parce que React interpréterait l'input comme non-contrôlé. Si la valeur change ensuite pour une string, React lèvera une erreur console. Il faut toujours initialiser avec une chaîne vide `""`.

3.  **Comment définir la valeur par défaut d'un input non-contrôlé ?**
    On utilise la prop `defaultValue` (ou `defaultChecked` pour les cases à cocher), et non `value`.

---

## Exercices : {#exercices-50}

### Exercice 1 - Le Formulaire Contrôlé de Profil {#exercice-1---formulaire-controle}

🎯 **Objectif** : Créer un formulaire complet avec validation en temps réel.

💼 **Mise en situation** : Vous créez la page "Mon Profil" d'un réseau social. L'utilisateur doit saisir son pseudo et sa biographie. Le pseudo ne doit pas contenir d'espaces.

📝 **Énoncé** :
1. Créez un composant `ProfileEditor`.
2. Utilisez deux états : `username` (input) et `bio` (textarea).
3. Empêchez la saisie d'espaces dans le `username` (contrôle strict).
4. Affichez une prévisualisation en temps réel en dessous.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Formulaire avec input et textarea remplis.
> **Annotation** : Montrez la prévisualisation qui reflète le state.
> **Alt Text suggéré** : Formulaire React contrôlé avec prévisualisation.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState } from 'react';

export function ProfileEditor() {
  const [username, setUsername] = useState('');
  const [bio, setBio] = useState('');

  const handleUsernameChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const val = e.target.value;
    // Contrôle strict : on refuse les espaces
    if (!val.includes(' ')) {
      setUsername(val);
    }
  };

  return (
    <div style={{ display: 'flex', gap: '20px' }}>
      <form style={{ display: 'flex', flexDirection: 'column', gap: '10px' }}>
        <label>
          Pseudo (sans espaces) :
          <input 
            type="text" 
            value={username} 
            onChange={handleUsernameChange} 
          />
        </label>
        
        <label>
          Biographie :
          {/* Textarea contrôlé via value */}
          <textarea 
            value={bio} 
            onChange={(e) => setBio(e.target.value)}
            rows={5}
          />
        </label>
      </form>

      <div style={{ border: '1px solid #ccc', padding: '10px', width: '200px' }}>
        <h3>Aperçu</h3>
        <p><strong>@{username || '...'}</strong></p>
        <p style={{ fontStyle: 'italic' }}>{bio || 'Aucune bio.'}</p>
      </div>
    </div>
  );
}
```
</details>

### Exercice 2 - Le Select Dynamique (Pizza Builder) {#exercice-2---select-dynamique}

🎯 **Objectif** : Manipuler les `<select>` et gérer l'affichage conditionnel.

💼 **Mise en situation** : Vous développez un configurateur de pizza. Si l'utilisateur choisit "Base Tomate", on propose certaines options. Si "Base Crème", d'autres options.

📝 **Énoncé** :
1. Un `<select>` pour la `base` ("tomate" ou "creme").
2. Un `<select>` pour l'`ingredient` dépendant de la base choisie.
3. Le tout en composants contrôlés.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState } from 'react';

export function PizzaBuilder() {
  const [base, setBase] = useState('tomate');
  const [ingredient, setIngredient] = useState('mozza');

  // Options dynamiques selon la base
  const ingredientsDispo = base === 'tomate' 
    ? ['Mozzarella', 'Anchois', 'Champignons'] 
    : ['Chèvre', 'Miel', 'Poulet'];

  const handleBaseChange = (e: React.ChangeEvent<HTMLSelectElement>) => {
    setBase(e.target.value);
    // Reset l'ingrédient quand la base change pour éviter les incohérences
    setIngredient(base === 'tomate' ? 'Chevre' : 'Mozzarella'); 
  };

  return (
    <form>
      <h2>Composez votre Pizza 🍕</h2>
      
      <label>
        Base :
        <select value={base} onChange={handleBaseChange}>
          <option value="tomate">Sauce Tomate</option>
          <option value="creme">Crème Fraîche</option>
        </select>
      </label>

      <br /><br />

      <label>
        Ingrédient Principal :
        <select value={ingredient} onChange={e => setIngredient(e.target.value)}>
          {ingredientsDispo.map(ing => (
            <option key={ing} value={ing}>{ing}</option>
          ))}
        </select>
      </label>

      <p>Résumé : Pizza Base {base} avec {ingredient}.</p>
    </form>
  );
}
```
</details>

### Exercice 3 - Formulaire de Login Non-Contrôlé (FormData) {#exercice-3---login-non-controle}

🎯 **Objectif** : Utiliser la méthode native et performante pour les formulaires simples.

💼 **Mise en situation** : Une page de connexion simple. Pas besoin de validation complexe en temps réel, on veut juste envoyer les données au "serveur".

📝 **Énoncé** :
1. Créez un formulaire `<form>` avec `onSubmit`.
2. Inputs : `email` et `password` avec l'attribut `name` (CRUCIAL).
3. Dans le handler, utilisez `new FormData(e.currentTarget)` pour extraire les valeurs.
4. Affichez les valeurs dans une `alert`.
5. N'oubliez pas le `e.preventDefault()`.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import React from 'react';

export function LoginForm() {
  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    // 1. Empêcher le rechargement natif du navigateur
    e.preventDefault();

    // 2. Extraire les données du formulaire DOM
    const formData = new FormData(e.currentTarget);
    
    const email = formData.get('email');
    const password = formData.get('password');

    // 3. Traitement (simulation d'envoi API)
    if (!email || !password) {
      alert("Veuillez remplir tous les champs");
      return;
    }

    alert(`Tentative de connexion pour : ${email}`);
    // Ici, on ferait un appel fetch('/api/login', { body: formData })
  };

  return (
    <form onSubmit={handleSubmit} style={{ display: 'flex', flexDirection: 'column', width: 300, gap: 10 }}>
      <h3>Connexion</h3>
      
      <label>
        Email :
        {/* Composant non-contrôlé : on utilise defaultValue si besoin */}
        <input name="email" type="email" required />
      </label>

      <label>
        Mot de passe :
        <input name="password" type="password" required />
      </label>

      <button type="submit">Se connecter</button>
    </form>
  );
}
```
</details>
```