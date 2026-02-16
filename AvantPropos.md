Voici le chapitre **Avant-propos** complet pour la formation React 19.2.

```markdown
---
sidebar_label: Avant-propos
sidebar_position: 0
---

# Chapitre 0 : Avant-propos

Introduction,Prérequis JavaScript,Contexte du Cours

Bienvenue dans cette formation complète sur **React 19.2**. Ce cours n'est pas une simple mise à jour d'un tutoriel existant : il a été conçu dès le départ pour enseigner le paradigme moderne de React, incluant les **Server Components**, les **Server Actions** et le **React Compiler**.

## Introduction {#introduction}

### 1. Quoi
React est une bibliothèque JavaScript pour créer des interfaces utilisateurs. Contrairement aux frameworks traditionnels (MVC), React se concentre exclusivement sur la "Vue" (l'interface), en utilisant une approche **déclarative** et basée sur des **composants**.

La version 19.2 marque un tournant majeur :
- **React Compiler** : Automatise l'optimisation des performances (finis les `useMemo` manuels excessifs).
- **Server Components (RSC)** : Les composants peuvent désormais s'exécuter sur le serveur par défaut, envoyant moins de JavaScript au navigateur.
- **Actions** : Une nouvelle façon native de gérer les mutations de données (formulaires).

### 2. Pourquoi
React domine le marché du développement web pour plusieurs raisons :
- **Écosystème immense** : Des milliers de bibliothèques compatibles.
- **Universalité** : Le même code (ou presque) peut fonctionner sur le web, le mobile (React Native) et même en réalité virtuelle.
- **Stabilité** : Facebook (Meta) utilise React en production sur des produits massivement utilisés, garantissant une stabilité à long terme.

### 3. Comment
React utilise une syntaxe appelée **JSX** (JavaScript XML), qui permet d'écrire des balises HTML-like directement dans votre code JavaScript.

#### A. Exemple minimaliste (React 19)

```tsx
// Un composant simple
function Welcome({ name }: { name: string }) {
  return <h1>Bonjour, {name}</h1>; // JSX : HTML dans JS
}
```

#### B. La révolution React 19
Avant React 19, la gestion des états de chargement nécessitait beaucoup de code manuel. Voici comment React 19 simplifie la logique avec les Actions :

```tsx
// ✅ Moderne : Gestion automatique du pending state avec useActionState
import { useActionState } from "react";

// Server Action (simulée)
async function updateName(prevState: any, formData: FormData) {
  "use server";
  await new Promise(resolve => setTimeout(resolve, 1000));
  return { message: "Nom mis à jour : " + formData.get("name") };
}

export function ProfileForm() {
  const [state, formAction, isPending] = useActionState(updateName, null);

  return (
    <form action={formAction}>
      <input name="name" type="text" />
      <button type="submit" disabled={isPending}>
        {isPending ? "Mise à jour..." : "Mettre à jour"}
      </button>
      <p>{state?.message}</p>
    </form>
  );
}
```

### 4. Zone de Danger

:::danger Ce que ce cours n'est PAS
Ce cours ne couvre pas les **Composants de Classe** (`class MyComponent extends React.Component`) dans les premiers chapitres. Bien qu'ils existent toujours pour la rétrocompatibilité, React 19 encourage exclusivement les **Composants Fonctionnels** et les Hooks.
:::

---

## Prérequis JavaScript {#prerequis-javascript}

### 1. Quoi
Pour suivre ce cours efficacement, vous devez maîtriser les concepts modernes de JavaScript (ES6+). React s'appuie énormément sur les fonctionnalités natives du langage.

### 2. Pourquoi
Si vous ne comprenez pas le JavaScript sous-jacent, vous aurez l'impression que React est "magique". Comprendre la syntaxe permet de déboguer efficacement.

### 3. Liste de contrôle technique

Voici les concepts que nous utiliserons quotidiennement. Si un point est flou, révisez-le avant de continuer.

#### A. Déclaration de variables
Oubliez `var`. Utilisez `const` par défaut, et `let` si vous devez réassigner la variable.

```typescript
const url = "https://api.monsite.com"; // ✅ Ne changera pas
let count = 0; // ✅ Changera
count = 1;
```

#### B. Fonctions Fléchées (Arrow Functions)
Essentielles pour écrire des composants concis.

```typescript
// Fonction classique
function add(a: number, b: number) {
  return a + b;
}

// Fonction fléchée (return implicite)
const add = (a: number, b: number) => a + b;
```

#### C. Destructuring (Objets et Tableaux)
Utilisé partout dans React pour les "props" et les "hooks".

```typescript
const user = { id: 1, name: "Alice", role: "Admin" };

// Extraction propre
const { name, role } = user; 

// Avec alias
const { name: userName } = user;
```

#### D. Spread Operator (...)
Crucial pour l'immutabilité (ne jamais modifier un état directement).

```typescript
const oldList = [1, 2];
// ❌ Ne faites pas oldList.push(3)
// ✅ Créez une nouvelle liste
const newList = [...oldList, 3]; 

const user = { name: "Bob", age: 30 };
// ✅ Copie user et modifie l'âge
const updatedUser = { ...user, age: 31 };
```

#### E. Modules (Import / Export)
React est modulaire par nature.

```typescript
// file.ts
export const PI = 3.14;
export default function myFunc() {}

// main.ts
import myFunc, { PI } from './file';
```

#### F. Async / Await & Promises
Indispensable pour les Server Components et la récupération de données.

```typescript
async function getData() {
  const res = await fetch('/api/data');
  const data = await res.json();
  return data;
}
```

### 🚨 Limitations
Ce cours suppose que vous savez configurer un environnement de développement basique (terminal, dossiers). Nous utiliserons **Node.js** (version LTS recommandée, min v20.x pour React 19) et npm/yarn/pnpm.

---

## Contexte du Cours {#contexte-du-cours}

### 1. Quoi
Ce cours est structuré pour vous emmener de "Hello World" à une architecture "Full-stack React" moderne.

### 2. Méthodologie
- **No Magic** : Chaque concept "magique" (comme le rendu automatique) sera expliqué.
- **TypeScript First** : Tous les exemples sont typés. C'est le standard de l'industrie en 2026.
- **Approche Composant** : Vous apprendrez à penser en petits blocs réutilisables.

### 3. Outils nécessaires
- **Node.js** : v20.10.0 ou supérieur.
- **Éditeur** : VS Code est fortement recommandé avec les extensions :
  - *ES7+ React/Redux/React-Native snippets*
  - *Prettier - Code formatter*
  - *Tailwind CSS IntelliSense* (si utilisé)
- **Navigateur** : Chrome ou Firefox avec **React Developer Tools** installé.

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-0}

1. **React 19 favorise-t-il les classes ou les fonctions ?**
   Les **fonctions**. Les classes sont considérées comme legacy.

2. **Quel mot-clé JavaScript permet d'éviter de modifier directement un tableau existant lors d'un ajout ?**
   Le **Spread Operator** (`...`). Exemple : `[...oldArray, newItem]`.

3. **Quelle est la version minimale de Node.js recommandée pour ce cours ?**
   Node.js version **20.x (LTS)** ou supérieure.

4. **Qu'est-ce que JSX ?**
   Une extension de syntaxe JavaScript permettant d'écrire du balisage similaire à HTML directement dans le code JS.

---

## Exercices : {#exercices-0}

Bien que ce soit l'avant-propos, assurons-nous que votre environnement mental et technique est prêt.

### Exercice 1 - Check-up JavaScript {#exercice-1---check-up-javascript}

🎯 **Objectif** : Vérifier la maîtrise du destructuring et du spread operator.

💼 **Mise en situation** : Vous recevez un objet de configuration d'un API tierce pour votre SaaS, mais vous devez écraser certaines valeurs par défaut sans muter l'objet original.

📝 **Énoncé** :
1. Créez un objet `defaultConfig` avec `theme: "light"`, `version: 1`, et `isAdmin: false`.
2. Créez un objet `userOverrides` avec `theme: "dark"`.
3. Créez une constante `finalConfig` qui fusionne les deux (les overrides gagnent), tout en extrayant la propriété `theme` dans une variable à part nommée `activeTheme`.
4. Le tout en une seule ligne ou bloc logique concis.

📺 **Résultat attendu** :
`finalConfig` doit contenir `{ theme: "dark", version: 1, isAdmin: false }` (ou similaire selon l'ordre) et `activeTheme` doit valoir `"dark"`.

<details>
<summary>💡 Voir le code complet commenté</summary>

```typescript
const defaultConfig = { theme: "light", version: 1, isAdmin: false };
const userOverrides = { theme: "dark" };

// On fusionne defaultConfig d'abord, puis userOverrides écrase les valeurs communes
// On utilise le destructuring immédiat pour extraire 'theme'
const { theme: activeTheme, ...restConfig } = { ...defaultConfig, ...userOverrides };

// Si on veut garder l'objet entier finalConfig :
const finalConfig = { ...defaultConfig, ...userOverrides };
const { theme } = finalConfig; // Extraction simple

console.log(finalConfig); // { theme: "dark", version: 1, isAdmin: false }
console.log(activeTheme); // "dark"
```
</details>

### Exercice 2 - Environnement Node {#exercice-2---environnement-node}

🎯 **Objectif** : Valider l'installation des outils.

💼 **Mise en situation** : Avant de lancer le projet "Next Big Thing", votre CTO vous demande de confirmer la version de vos outils pour éviter les bugs de build.

📝 **Énoncé** :
Ouvrez votre terminal et exécutez les commandes pour afficher les versions de Node et npm.

📺 **Résultat attendu** :
```bash
$ node -v
v20.11.0 (ou supérieur)
$ npm -v
10.2.0 (ou supérieur)
```

<details>
<summary>💡 Voir la solution</summary>

Ouvrez simplement votre terminal préféré (PowerShell, Terminal, iTerm2) et tapez :
```bash
node -v && npm -v
```
Si la commande n'est pas reconnue, réinstallez Node.js depuis [nodejs.org](https://nodejs.org/).
</details>

### Exercice 3 - Mentalité Déclarative {#exercice-3---mentalite-declarative}

🎯 **Objectif** : Comprendre la différence conceptuelle Impératif vs Déclaratif.

💼 **Mise en situation** : Vous devez expliquer à un stagiaire pourquoi on n'utilise pas `document.getElementById` en React.

📝 **Énoncé** :
Analysez ces deux bouts de code (pseudo-code). Lequel est "React-style" (Déclaratif) ?

**Code A :**
```javascript
function onClick() {
  const box = document.getElementById('box');
  if (box.style.color === 'red') {
    box.style.color = 'blue';
  } else {
    box.style.color = 'red';
  }
}
```

**Code B :**
```javascript
// État : isRed = true/false
// Rendu : <div style={{ color: isRed ? 'red' : 'blue' }} />
```

📺 **Résultat attendu** :
Le Code B est l'approche React.

<details>
<summary>💡 Explication détaillée</summary>

Le **Code A** est **Impératif** : Vous décrivez *comment* faire (trouver l'élément, vérifier sa couleur actuelle, la changer manuellement). C'est fragile.

Le **Code B** est **Déclaratif** : Vous décrivez *quoi* afficher en fonction de l'état (Si `isRed` est vrai, alors rouge, sinon bleu). React s'occupe de mettre à jour le DOM pour qu'il corresponde à cette description. C'est l'essence de ce cours.
</details>
```