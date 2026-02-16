Voici le chapitre **Mettre à Jour des Objets dans l'État** pour la formation React 19.2.

```markdown
---
sidebar_label: Mettre à Jour des Objets dans l'État
sidebar_position: 17
---

# Chapitre 17 : Mettre à Jour des Objets dans l'État

Immutabilité des objets, Opérateur de spread (`...`), Mises à jour imbriquées

Jusqu'à présent, nous avons travaillé avec des types primitifs (nombres, chaînes, booléens). Ces valeurs sont immuables par nature en JavaScript (`7` sera toujours `7`).
Mais en React, vous stockerez souvent des données complexes : profils utilisateurs, configurations, formulaires. Ces données sont stockées dans des **objets**.

La règle d'or de React s'applique encore plus strictement ici : **ne modifiez jamais un objet d'état directement**. Vous devez créer un *nouvel* objet avec les modifications souhaitées.

## Immutabilité des Objets {#immutabilite-des-objets}

### 1. Quoi
L'immutabilité signifie qu'une fois créé, un objet ne peut pas être changé. En React, au lieu de modifier une propriété d'un objet existant, vous devez remplacer l'objet entier par une nouvelle version.

### 2. Pourquoi
React compare les états pour décider s'il faut relancer un rendu. Il utilise une comparaison de référence ("shallow comparison") : `ancienObjet === nouvelObjet`.
*   Si vous modifiez l'intérieur de l'objet (`obj.x = 5`), la référence reste la même. **React ne voit pas le changement et ne met pas à jour l'écran.**
*   Si vous créez un nouvel objet, la référence change. React détecte le changement et met à jour l'UI.

### 3. Comment

#### A. L'erreur classique (Mutation)

❌ **NE FAITES JAMAIS CELA :**
```tsx
const [position, setPosition] = useState({ x: 0, y: 0 });

const handleMove = () => {
  // ⛔️ Mutation directe !
  position.x = 5; 
  
  // React pense que c'est le même objet (même adresse mémoire)
  // Le rendu ne sera pas déclenché.
  setPosition(position); 
};
```

#### B. La bonne approche (Remplacement)

✅ **FAITES CELA :**
```tsx
const handleMove = () => {
  // ✨ Création d'un NOUVEL objet
  const newPosition = { x: 5, y: 0 };
  
  // React voit une nouvelle référence mémoire -> Re-render
  setPosition(newPosition);
};
```

---

## L'Opérateur de Spread (`...`) {#operateur-de-spread}

### 1. Quoi
L'opérateur de décomposition (spread operator) `...` permet de copier toutes les propriétés d'un objet existant dans un nouvel objet. C'est l'outil standard pour effectuer des mises à jour partielles.

### 2. Pourquoi
Souvent, vous voulez modifier *un seul* champ (ex: le prénom) tout en gardant *les autres* intacts (ex: le nom, l'email, l'ID). Sans le spread, vous devriez recopier chaque champ manuellement, ce qui est long et source d'erreurs.

### 3. Comment

#### A. Syntaxe de copie
```tsx
const [person, setPerson] = useState({
  firstName: 'John',
  lastName: 'Doe',
  email: 'john@example.com'
});

const handleNameChange = (e: ChangeEvent<HTMLInputElement>) => {
  setPerson({
    ...person, // 1. Copie tout l'ancien objet (firstName, lastName, email)
    firstName: e.target.value // 2. Écrase UNIQUEMENT firstName
  });
};
```

#### B. L'importance de l'ordre
L'ordre compte ! Les propriétés définies *après* le spread écrasent celles copiées par le spread.

```tsx
// ✅ Correct : "b" écrase la valeur copiée
const obj1 = { ...oldObj, b: 5 }; 

// ❌ Incorrect : la valeur de "b" dans oldObj écrasera votre 5
const obj2 = { b: 5, ...oldObj }; 
```

#### C. Pattern : Le Formulaire Générique
Une technique très courante permet de gérer plusieurs inputs avec une seule fonction de gestion, en utilisant les "Computed Property Names" de ES6 (`[key]: value`).

```tsx
export function SignupForm() {
  const [form, setForm] = useState({
    username: '',
    email: '',
    password: ''
  });

  const handleChange = (e: ChangeEvent<HTMLInputElement>) => {
    setForm({
      ...form,
      // Utilise le 'name' de l'input comme clé dynamique
      [e.target.name]: e.target.value 
    });
  };

  return (
    <form>
      <input name="username" value={form.username} onChange={handleChange} />
      <input name="email" value={form.email} onChange={handleChange} />
      <input name="password" value={form.password} onChange={handleChange} />
    </form>
  );
}
```

### 4. Zone de Danger

:::danger Pas de fusion automatique (contrairement aux classes)
Si vous venez des composants de classe (`this.setState`), sachez que le Hook `useState` **ne fusionne pas** l'ancien et le nouvel état.
Si vous oubliez `...form`, les champs manquants seront perdus.

```tsx
// État initial : { name: 'A', email: 'a@a.com' }
setForm({ name: 'B' }); 
// Nouvel état : { name: 'B' } (L'email a disparu !)
```
:::

---

## Mises à jour imbriquées (Nested Updates) {#mises-a-jour-imbriquees}

### 1. Quoi
Les objets contiennent souvent d'autres objets. Le spread operator est "superficiel" (shallow) : il ne copie que le premier niveau. Si vous voulez modifier un objet imbriqué, vous devez copier toute la hiérarchie jusqu'à la propriété ciblée.

### 2. Pourquoi
C'est la conséquence directe de l'immutabilité. Si vous changez une donnée profonde, tous les objets parents doivent être recréés (nouvelles références) pour que React sache que "quelque chose a changé" dans cette branche de l'arbre.

### 3. Comment

Imaginez cet état :
```tsx
const [user, setUser] = useState({
  name: 'Niamh',
  address: {
    city: 'Dublin',
    zip: 'D01'
  }
});
```

Pour changer la ville (`city`), vous ne pouvez pas faire `user.address.city = 'Cork'`.
Vous devez faire :

```tsx
setUser({
  ...user, // Copie le niveau 1 (name)
  address: {
    ...user.address, // Copie le niveau 2 (zip)
    city: 'Cork' // Écrase la ville
  }
});
```

C'est ce qu'on appelle parfois "l'escalier de spread". C'est verbeux, mais c'est le prix de la clarté et de la prédictibilité en React standard.

### 🚨 Limitations
Pour des objets très profonds, cette syntaxe devient illisible.
Dans des applications complexes, on utilise souvent des bibliothèques comme **Immer** qui permettent d'écrire du code "mutant" (`obj.nested.x = 5`) tout en produisant un état immuable sous le capot. Cependant, pour cette formation, il est crucial de maîtriser la syntaxe manuelle.

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-17}

1.  **Pourquoi ne peut-on pas faire `state.value = 5` en React ?**
    Parce que React compare les références d'objet. Si vous modifiez l'objet existant, la référence ne change pas, donc React ne déclenche pas de nouveau rendu.

2.  **Que fait l'instruction `...objet` ?**
    Elle crée une "copie superficielle" : elle prend toutes les propriétés énumérables de l'objet source et les place dans le nouvel objet.

3.  **Si j'ai un état `{ a: 1, b: 2 }` et que je fais `setState({ a: 5 })`, que devient l'état ?**
    L'état devient `{ a: 5 }`. La propriété `b` est perdue car `useState` ne fusionne pas automatiquement les objets. Il fallait faire `setState({ ...state, a: 5 })`.

4.  **Comment mettre à jour une propriété imbriquée au 3ème niveau de profondeur ?**
    Il faut utiliser le spread operator à chaque niveau de la hiérarchie (niveau 1, niveau 2, niveau 3) pour recréer l'arbre complet des objets parents.

---

## Exercices : {#exercices-17}

### Exercice 1 - La Souris Traqueuse (Mise à jour simple) {#exercice-1---la-souris-traqueuse}

🎯 **Objectif** : Mettre à jour un objet d'état simple avec deux propriétés.

💼 **Mise en situation** : Vous créez un outil de dessin. Vous devez afficher les coordonnées actuelles de la souris dans une zone de débogage.

📝 **Énoncé** :
1. Créez un état `position` contenant `{ x: 0, y: 0 }`.
2. Attachez un événement `onPointerMove` à une `div` conteneur (donnez-lui une taille fixe, ex: 300x300).
3. À chaque mouvement, mettez à jour l'objet `position` avec `e.clientX` et `e.clientY`.
4. Affichez la position sous la div.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Un carré coloré et un texte en dessous "X: 123, Y: 456".
> **Annotation** : Montrez que les valeurs changent quand on survole le carré.
> **Alt Text suggéré** : Zone de capture de mouvement de souris affichant les coordonnées en temps réel.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState, PointerEvent } from 'react';

export function MouseTracker() {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  const handleMove = (e: PointerEvent<HTMLDivElement>) => {
    // Remplacement complet de l'objet
    // Note : Ici pas besoin de spread car on remplace TOUT l'objet (x et y)
    // Mais par habitude, on crée un nouvel objet littéral.
    setPosition({
      x: e.clientX,
      y: e.clientY
    });
  };

  return (
    <div 
      onPointerMove={handleMove}
      style={{ 
        width: '300px', 
        height: '300px', 
        backgroundColor: '#eee', 
        position: 'relative' 
      }}
    >
      <div style={{ 
        position: 'absolute', 
        left: 0, 
        bottom: -30 
      }}>
        X: {position.x}, Y: {position.y}
      </div>
    </div>
  );
}
```
</details>

### Exercice 2 - L'Éditeur de Profil (Spread partiel) {#exercice-2---l-editeur-de-profil}

🎯 **Objectif** : Utiliser le spread operator pour ne modifier qu'une partie d'un objet.

💼 **Mise en situation** : Un formulaire de profil utilisateur SaaS. L'objet utilisateur contient `firstName`, `lastName`, `email`. Vous ne voulez créer qu'un seul handler pour les 3 champs.

📝 **Énoncé** :
1. État initial : `{ firstName: 'Jane', lastName: 'Doe', email: 'jane@corp.com' }`.
2. Trois champs `<input>` correspondants.
3. Une seule fonction `handleChange` qui utilise `e.target.name` et `e.target.value`.
4. Affichez un résumé "Bonjour, [firstName] [lastName] ([email])" en temps réel.

📺 **Résultat attendu** :
Modifier le prénom ne doit pas effacer le nom ou l'email.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState, ChangeEvent } from 'react';

export function ProfileEditor() {
  const [user, setUser] = useState({
    firstName: 'Jane',
    lastName: 'Doe',
    email: 'jane@corp.com'
  });

  const handleChange = (e: ChangeEvent<HTMLInputElement>) => {
    setUser({
      ...user, // 1. On garde les anciennes valeurs (crucial !)
      [e.target.name]: e.target.value // 2. On écrase dynamiquement le champ modifié
    });
  };

  return (
    <div style={{ display: 'flex', flexDirection: 'column', gap: '10px', maxWidth: '300px' }}>
      <label>
        Prénom :
        <input name="firstName" value={user.firstName} onChange={handleChange} />
      </label>
      <label>
        Nom :
        <input name="lastName" value={user.lastName} onChange={handleChange} />
      </label>
      <label>
        Email :
        <input name="email" value={user.email} onChange={handleChange} />
      </label>

      <div style={{ marginTop: '20px', padding: '10px', border: '1px solid #ccc' }}>
        <strong>Aperçu :</strong><br/>
        {user.firstName} {user.lastName}<br/>
        <small>{user.email}</small>
      </div>
    </div>
  );
}
```
</details>

### Exercice 3 - La Customisation de Voiture (Imbrication) {#exercice-3---la-customisation-de-voiture}

🎯 **Objectif** : Mettre à jour un objet imbriqué (Nested Object).

💼 **Mise en situation** : Un configurateur de voiture. L'objet d'état contient des infos de base et un sous-objet `theme`.

📝 **Énoncé** :
1. État :
   ```ts
   {
     model: "Sport GT",
     theme: {
       color: "red",
       interior: "leather"
     }
   }
   ```
2. Créez un bouton "Peindre en Bleu" qui ne change QUE `theme.color`.
3. Créez un bouton "Intérieur Tissu" qui ne change QUE `theme.interior`.
4. Assurez-vous que cliquer sur l'un n'annule pas l'autre.

📺 **Résultat attendu** :
Si je peins en bleu, l'intérieur reste en cuir. Si je mets l'intérieur tissu, la voiture reste bleue.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState } from 'react';

export function CarConfigurator() {
  const [car, setCar] = useState({
    model: "Sport GT",
    theme: {
      color: "red",
      interior: "cuir" // "leather" traduit pour l'exercice
    }
  });

  const handlePaintBlue = () => {
    setCar({
      ...car, // Copie niveau 1 (model)
      theme: {
        ...car.theme, // Copie niveau 2 (interior) -> Sans ça, interior serait perdu !
        color: "blue" // Modification
      }
    });
  };

  const handleFabricInterior = () => {
    setCar({
      ...car,
      theme: {
        ...car.theme, // Copie niveau 2 (color)
        interior: "tissu"
      }
    });
  };

  return (
    <div>
      <h3>Votre {car.model}</h3>
      <p>Couleur : {car.theme.color}</p>
      <p>Intérieur : {car.theme.interior}</p>

      <div style={{ display: 'flex', gap: '10px' }}>
        <button onClick={handlePaintBlue}>Peindre en Bleu</button>
        <button onClick={handleFabricInterior}>Intérieur Tissu</button>
      </div>
    </div>
  );
}
```
</details>
```