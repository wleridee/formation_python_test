Voici le chapitre **L'État des Composants avec `useState`** pour la formation React 19.2.

```markdown
---
sidebar_label: L'État des Composants avec useState
sidebar_position: 13
---

# Chapitre 13 : L'État des Composants avec `useState`

Déclarer une variable d'état, Mettre à jour l'état, Le Hook `useState`, Initialisation de l'état

Jusqu'à maintenant, nos composants étaient comme des **fonctions pures** : ils recevaient des données (props) et affichaient un résultat. Mais une application réelle doit avoir de la "mémoire". Un formulaire doit retenir ce que l'utilisateur tape. Un carrousel doit savoir quelle image est affichée.
En React, cette mémoire interne au composant s'appelle l'**État (State)**.

## Le Hook `useState` {#le-hook-usestate}

### 1. Quoi
`useState` est un **Hook** (une fonction spéciale de React) qui permet d'ajouter une variable d'état à un composant fonctionnel.
Contrairement à une variable locale classique (`let x = 0`) qui disparaît à la fin de l'exécution de la fonction, une variable d'état est **préservée** par React entre les rendus.

### 2. Pourquoi
Si vous utilisez une variable locale :
```tsx
function Counter() {
  let count = 0; // ❌ Réinitialisé à 0 à chaque rendu (clic)
  function handleClick() {
    count = count + 1; // Le changement ne déclenche pas de mise à jour de l'UI
  }
  return <button onClick={handleClick}>{count}</button>;
}
```
React ne "surveille" pas les variables locales. Pour que l'interface change, il faut demander à React de **re-rendre** le composant. C'est le rôle de l'état.

### 3. Comment

#### A. Syntaxe de base : La destructuration de tableau
`useState` retourne un tableau contenant exactement deux éléments :
1.  La **valeur actuelle** de l'état (au début, la valeur initiale).
2.  Une **fonction setter** qui permet de mettre à jour cette valeur et de déclencher un nouveau rendu.

```tsx
import { useState } from 'react';

export function Counter() {
  // Déclaration : [valeur, fonctionDeModification] = useState(valeurInitiale)
  const [count, setCount] = useState(0); 

  return (
    <button onClick={() => setCount(count + 1)}>
      Compteur : {count}
    </button>
  );
}
```

#### B. Typage TypeScript
TypeScript infère souvent le type automatiquement (ici `number` grâce au `0`). Mais pour des types complexes ou optionnels, soyez explicites.

```tsx
// Type explicite : user peut être User ou null
const [user, setUser] = useState<User | null>(null);

// Type inféré (string)
const [name, setName] = useState("Anonyme");
```

#### C. Indépendance de l'état
L'état est **privé** et **isolé** à chaque instance du composant. Si vous affichez deux fois le composant `<Counter />`, chaque compteur aura son propre état indépendant.

```tsx
export function App() {
  return (
    <div>
      <Counter /> {/* État A : 0 */}
      <Counter /> {/* État B : 0 (indépendant de A) */}
    </div>
  );
}
```

### 4. Zone de Danger

:::danger Ne jamais modifier l'état directement
L'état en React doit être traité comme immuable.
❌ **Interdit :** `count = 5;` (React ne le saura pas, l'UI ne changera pas).
✅ **Obligatoire :** `setCount(5);` (React met à jour la valeur et relance le composant).

Notez que nous utilisons `const` pour déclarer `[count, setCount]`. Cela renforce l'idée qu'on ne réassigne pas la variable `count`. Lors du prochain rendu, React appellera à nouveau votre fonction et créera une *nouvelle* constante `count` avec la nouvelle valeur.
:::

---

## Initialisation de l'État {#initialisation-de-l-etat}

### 1. Quoi
La valeur passée à `useState(valeur)` est la valeur initiale. Elle n'est utilisée que lors du **tout premier rendu**. Lors des rendus suivants, React ignore cette valeur et utilise la valeur actuelle mémorisée.

### 2. Pourquoi
Parfois, calculer la valeur initiale est coûteux (lecture du LocalStorage, calcul mathématique lourd). On ne veut pas refaire ce calcul à chaque milliseconde.

### 3. Comment

#### A. Initialisation simple
Pour les types primitifs (nombres, chaînes, booléens), passez la valeur directement.

```tsx
const [isLoading, setIsLoading] = useState(true);
```

#### B. Initialisation paresseuse (Lazy Initialization)
Si l'initialisation est lourde, passez une **fonction** à `useState`. React n'exécutera cette fonction qu'une seule fois (au montage).

```tsx
function getComplexValue() {
  console.log("Calcul coûteux..."); 
  // Imaginez une boucle de 1 million d'itérations
  return 42; 
}

export function HeavyComponent() {
  // ✅ La fonction est appelée uniquement au premier rendu
  const [value, setValue] = useState(() => getComplexValue());
  
  // ❌ Si on faisait useState(getComplexValue()), la fonction serait exécutée à CHAQUE rendu (lent)
  
  return <span>{value}</span>;
}
```

---

## Cas Concret : Controlled Inputs {#cas-concret-controlled-inputs}

### 1. Quoi
En HTML standard, un `<input>` garde son propre état. En React, nous voulons souvent que l'état React soit la "Source Unique de Vérité". On lie la valeur de l'input (`value`) à l'état React, et l'événement `onChange` met à jour cet état.

### 2. Pourquoi
Cela permet de valider l'entrée en temps réel, de formater le texte pendant la frappe, ou de désactiver un bouton si le champ est vide.

### 3. Comment

```tsx
import { useState, ChangeEvent } from 'react';

export function TextInput() {
  const [text, setText] = useState(""); 

  // Typage de l'événement onChange
  const handleChange = (e: ChangeEvent<HTMLInputElement>) => {
    setText(e.target.value); // Met à jour l'état avec ce que l'utilisateur tape
  };

  return (
    <div>
      <input 
        type="text" 
        value={text}       // 1. Lecture : l'input affiche l'état
        onChange={handleChange} // 2. Écriture : l'input met à jour l'état
      />
      <p>Vous avez tapé : {text}</p>
      
      {/* Bouton Reset simple */}
      <button onClick={() => setText("")}>Effacer</button>
    </div>
  );
}
```

### 🚨 Limitations
Chaque frappe de clavier déclenche un re-rendu du composant. Sur des formulaires géants avec 50 champs, cela peut impacter les performances (bien que React 19 soit très rapide). Dans ces cas extrêmes, on utilisera des inputs "non contrôlés" (voir Chapitre 50).

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-13}

1.  **Pourquoi `let count = 0` ne suffit-il pas pour gérer un compteur en React ?**
    Parce que les variables locales sont réinitialisées à chaque exécution de la fonction composant et que leur modification ne déclenche pas de nouveau rendu (re-render) de l'interface.

2.  **Que retourne exactement `useState(0)` ?**
    Un tableau (tuple) de deux éléments : `[0, fonctionSetter]`. Le premier est la valeur (0), le second la fonction pour modifier cette valeur.

3.  **L'état est-il partagé entre plusieurs instances du même composant ?**
    Non. Chaque fois que vous utilisez un composant dans le JSX (`<Comp />`), React crée une nouvelle instance avec son propre état isolé.

4.  **Quand l'argument passé à `useState` est-il pris en compte ?**
    Uniquement lors du tout premier rendu (montage). Il est ignoré lors des mises à jour suivantes.

---

## Exercices : {#exercices-13}

### Exercice 1 - L'Interrupteur (Dark Mode) {#exercice-1---l-interrupteur-dark-mode}

🎯 **Objectif** : Gérer un état booléen simple.

💼 **Mise en situation** : Vous implémentez une fonctionnalité "Mode Sombre" pour un panneau de configuration.

📝 **Énoncé** :
1. Créez un composant `ThemeToggle`.
2. Déclarez un état `isDark` (boolean), initialisé à `false`.
3. Affichez une `div` qui change de style selon l'état :
   - Si `isDark` est vrai : fond noir (`#333`), texte blanc.
   - Si `isDark` est faux : fond blanc, texte noir.
4. Ajoutez un bouton qui inverse l'état à chaque clic ("Passer en mode sombre" / "Passer en mode clair").

📺 **Résultat attendu** :
Un rectangle dont les couleurs s'inversent au clic.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState } from 'react';

export function ThemeToggle() {
  // État booléen : false = Light, true = Dark
  const [isDark, setIsDark] = useState(false);

  return (
    <div 
      style={{ 
        padding: '20px', 
        border: '1px solid #ccc',
        // Application conditionnelle des styles basés sur l'état
        backgroundColor: isDark ? '#333' : '#fff',
        color: isDark ? '#fff' : '#000',
        transition: 'all 0.3s ease'
      }}
    >
      <h3>Mode actuel : {isDark ? 'Sombre 🌙' : 'Clair ☀️'}</h3>
      
      {/* Au clic, on passe la valeur inverse de l'état actuel */}
      <button onClick={() => setIsDark(!isDark)}>
        Basculer le thème
      </button>
    </div>
  );
}
```
</details>

### Exercice 2 - Compteur de caractères (Input) {#exercice-2---compteur-de-caracteres}

🎯 **Objectif** : Lier un input à un état (Controlled Component).

💼 **Mise en situation** : Vous créez une boîte de tweet. L'utilisateur doit savoir combien de caractères il a tapés.

📝 **Énoncé** :
1. Créez un composant `TweetBox`.
2. Un état `message` (string).
3. Un `<textarea>` lié à cet état.
4. Affichez le nombre de caractères en temps réel en dessous.
5. Si le nombre dépasse 50, affichez le compteur en rouge.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Zone de texte avec un compteur "55/50" en rouge en dessous.
> **Annotation** : Montrez la relation entre le texte tapé et le compteur.
> **Alt Text suggéré** : Interface de saisie de message avec indicateur de limite de caractères dépassée.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState } from 'react';

export function TweetBox() {
  const [message, setMessage] = useState("");
  const limit = 50;
  
  // Variable dérivée (pas besoin de useState pour la longueur !)
  const count = message.length;
  const isOverLimit = count > limit;

  return (
    <div style={{ maxWidth: '300px' }}>
      <textarea 
        rows={4}
        style={{ width: '100%', padding: '8px' }}
        placeholder="Quoi de neuf ?"
        value={message}
        // Mise à jour de l'état à chaque frappe
        onChange={(e) => setMessage(e.target.value)}
      />
      
      <div style={{ 
        textAlign: 'right', 
        // Style conditionnel basé sur la variable dérivée
        color: isOverLimit ? 'red' : 'gray',
        fontWeight: isOverLimit ? 'bold' : 'normal'
      }}>
        {count} / {limit}
      </div>
    </div>
  );
}
```
</details>

### Exercice 3 - La Galerie d'Images (Index) {#exercice-3---la-galerie-d-images}

🎯 **Objectif** : Utiliser un état numérique pour naviguer dans un tableau.

💼 **Mise en situation** : Un carrousel simple pour afficher des photos de vacances.

📝 **Énoncé** :
1. Données : Un tableau d'URLs d'images (utilisez des placeholders).
2. État : `index` (number), initialisé à 0.
3. Affichez l'image correspondant à `images[index]`.
4. Ajoutez deux boutons "Précédent" et "Suivant".
5. GESTION DES LIMITES :
   - "Précédent" doit être désactivé si on est à l'index 0.
   - "Suivant" doit être désactivé si on est à la dernière image.

📺 **Résultat attendu** :
Une image changeante. Les boutons se grisent quand on atteint les bords du tableau.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState } from 'react';

const images = [
  "https://placehold.co/300x200?text=Photo+1",
  "https://placehold.co/300x200?text=Photo+2",
  "https://placehold.co/300x200?text=Photo+3"
];

export function Gallery() {
  const [index, setIndex] = useState(0);

  // Vérification des bornes pour désactiver les boutons
  const hasPrev = index > 0;
  const hasNext = index < images.length - 1;

  function handlePrev() {
    if (hasPrev) setIndex(index - 1);
  }

  function handleNext() {
    if (hasNext) setIndex(index + 1);
  }

  return (
    <div style={{ textAlign: 'center' }}>
      <img 
        src={images[index]} 
        alt={`Slide ${index + 1}`} 
        style={{ borderRadius: '8px', marginBottom: '10px' }}
      />
      
      <div style={{ display: 'flex', justifyContent: 'center', gap: '10px' }}>
        <button onClick={handlePrev} disabled={!hasPrev}>
          ⬅️ Précédent
        </button>
        
        <span>{index + 1} / {images.length}</span>
        
        <button onClick={handleNext} disabled={!hasNext}>
          Suivant ➡️
        </button>
      </div>
    </div>
  );
}
```
</details>
```