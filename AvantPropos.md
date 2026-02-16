---
sidebar_label: Avant-propos
sidebar_position: 0
---

# Chapitre 0 : Avant-propos

Introduction à la formation, Prérequis (JavaScript), Objectifs pédagogiques

Bienvenue dans cette formation **TypeScript (version 5.x)**.

Vous vous apprêtez à apprendre le standard industriel pour le développement JavaScript moderne. TypeScript n'est pas une simple "surcouche" syntaxique : c'est un architecte qui sécurise, documente et stabilise vos applications avant même qu'elles ne soient exécutées.

Ce cours a été conçu avec une philosophie **"No Magic"** : nous n'utiliserons pas de configurations obscures ou d'outils magiques sans les comprendre. Chaque concept sera décortiqué pour que vous maîtrisiez non seulement le *comment*, mais surtout le *pourquoi*.

---

## Introduction à la formation {#introduction-à-la-formation}

### 1. Quoi
Cette formation est un guide exhaustif pour passer de développeur JavaScript à expert TypeScript. Elle couvre la version **5.x** du langage, incluant les fonctionnalités les plus récentes (Decorators stage 3, `satisfies`, `const` type parameters, etc.).

Nous allons transformer votre manière d'écrire du code : de la découverte des bugs en production (runtime) à leur éradication dans l'éditeur (compile time).

### 2. Pourquoi
JavaScript est extrêmement flexible, ce qui est sa plus grande force et sa plus grande faiblesse.
- **Le problème :** Sur de gros projets, la flexibilité devient du chaos. `undefined` n'est pas une fonction, refactorer est risqué, et la documentation devient obsolète.
- **La solution :** TypeScript ajoute un système de types statique qui analyse votre code sans l'exécuter. C'est comme avoir un pair-programmer expert qui vérifie chaque ligne en temps réel.

### 3. Comment
La formation est structurée pour simuler une montée en compétence professionnelle :
1. **Les Fondations (Chapitres 1-12) :** Syntaxe, inférence, structures de base.
2. **Le Cœur (Chapitres 13-38) :** Generics, Union Types, Narrowing, Manipulation de types avancée.
3. **L'Expertise (Chapitres 39-61) :** POO avancée, Patterns, Configuration, Tooling.

Chaque chapitre suit le format : Théorie → Cas Pratique → Zone de Danger → Exercices.

---

## Prérequis (JavaScript) {#prérequis-javascript}

### 1. Quoi
TypeScript est un **superset** de JavaScript. Cela signifie que tout code JavaScript valide est (théoriquement) du TypeScript. **Vous ne pouvez pas maîtriser TypeScript si vous ne maîtrisez pas JavaScript.**

### 2. Pourquoi
TypeScript ne corrige pas votre ignorance du fonctionnement de JavaScript. Il type le comportement de JavaScript. Si vous ne comprenez pas le prototypage, les closures ou l'asynchronisme en JS, TypeScript ne fera qu'ajouter de la confusion.

### 3. Comment
Avant de commencer, assurez-vous d'être à l'aise avec les concepts suivants (ES6+) :

| Concept JS | Pourquoi c'est critique en TS |
| :--- | :--- |
| **Variables** (`let`, `const`) | TS s'appuie fortement sur la distinction entre muable et immuable. |
| **Arrow Functions** | Syntaxiquement omniprésentes en TS. |
| **Modules ES6** (`import`/`export`) | TS utilise ce système pour moduler le code et les types. |
| **Objets & Arrays** | Manipulation, destructuring, méthodes (`.map`, `.filter`). |
| **Async / Await** | TS type les `Promise` de manière très précise. |

### 4. Zone de Danger
❌ **À ne pas faire** :
- Commencer ce cours si vous confondez encore `var` et `let`.
- Penser que TypeScript va "optimiser" les performances d'exécution de votre JS (c'est faux, TS disparaît à l'exécution).

✅ **Bonne Pratique** :
- Si vous hésitez, prenez une semaine pour renforcer vos bases JS avant d'attaquer ce cours.

---

## Objectifs pédagogiques {#objectifs-pédagogiques}

### 1. Quoi
À la fin de ce cursus, vous serez capable de concevoir des architectures applicatives robustes et scalables. Vous ne serez pas seulement un "utilisateur" de TypeScript, mais un ingénieur capable de typer des bibliothèques complexes.

### 2. Pourquoi
Les entreprises recherchent des profils capables de maintenir du code sur le long terme. Savoir écrire un `interface` de base ne suffit plus. Il faut savoir utiliser les **Generics**, les **Utility Types** et configurer un environnement de build performant.

### 3. Comment
Nous allons travailler sur des cas d'usage réels :
- **Typage de données API** (e-commerce, SaaS).
- **Gestion d'états complexes** (Redux-like patterns).
- **Architecture orientée composants**.

### 🚨 Limitations
Ce cours se concentre sur **TypeScript** en tant que langage. Bien que nous mentions React, Node.js ou Vue, ce n'est PAS un cours sur ces frameworks. Les compétences acquises ici sont transversales et s'appliquent à tout l'écosystème.

---

## Questions clés (validation des acquis du chapitre) {#questions-clés-0}

1.  **TypeScript est-il un langage différent de JavaScript ?**
    Non, c'est un "superset" (sur-ensemble). Une fois compilé, il devient du JavaScript pur. Il n'existe pas au "runtime" dans le navigateur.

2.  **Puis-je utiliser TypeScript pour accélérer l'exécution de mon site ?**
    Non. Le "typâge" est supprimé lors de la compilation. TypeScript améliore la productivité et la fiabilité du développement, pas la vitesse d'exécution du moteur JS.

3.  **Quelle version de Node.js est recommandée ?**
    Pour suivre ce cours (TS 5.x), utilisez une version Node.js LTS active (v18 ou v20+).

---

## Exercices : {#exercices-0}

Ces exercices visent à valider vos prérequis en JavaScript et à illustrer les problèmes que nous allons résoudre avec TypeScript.

### Exercice 1 - Le bug silencieux {#exercice-1---le-bug-silencieux}

**🎯 Objectif :** Comprendre le danger du typage dynamique faible de JavaScript.

**💼 Mise en situation :** Vous gérez une plateforme e-commerce. Une fonction calcule le total du panier, mais les données viennent d'une API (souvent en format JSON/String).

**📝 Énoncé :**
Analysez le code JavaScript ci-dessous. Quel sera le résultat dans la console ? Pourquoi est-ce catastrophique financièrement ?

```javascript
function calculerTotal(prix, taxe) {
    // prix vient d'un input HTML, donc c'est une string
    return prix + taxe;
}

const total = calculerTotal("100", 20);
console.log("Prix total : " + total);
```

**📺 Résultat attendu :**
Le développeur s'attend à `120`. Le résultat réel est différent.

<details>
<summary>Voir la solution et l'explication</summary>

**Résultat :** `Prix total : 10020`

**Pourquoi ?**
En JavaScript, l'opérateur `+` sert à la fois à l'addition numérique et à la concaténation de chaînes.
Comme `prix` est la chaîne `"100"`, JavaScript convertit `20` en chaîne et les concatène.

**L'apport de TypeScript :**
TypeScript aurait souligné `"100"` en rouge en disant : *Argument of type 'string' is not assignable to parameter of type 'number'*.

</details>

### Exercice 2 - L'objet mystère {#exercice-2---l-objet-mystère}

**🎯 Objectif :** Ressentir le besoin de structure (Interfaces).

**💼 Mise en situation :** Vous récupérez le code d'un ancien collègue parti sans documentation. Il y a une fonction `envoyerEmail` qui prend un objet `user`.

**📝 Énoncé :**
Regardez cette fonction. Sans exécuter le code, pouvez-vous savoir **exactement** quelles propriétés l'objet `user` doit posséder pour que la fonction n'échoue pas ?

```javascript
function envoyerEmail(user) {
    if (user.preferences.emailEnabled) {
        console.log(`Envoi à ${user.contact.email.toUpperCase()}...`);
    }
}
```

**📺 Résultat attendu :**
Réflexion sur la difficulté de deviner la structure des données.

<details>
<summary>Voir la solution</summary>

Il est impossible d'être sûr à 100% sans lire tout le code ou exécuter la fonction (et risquer un crash).
Pour que cela fonctionne, `user` doit avoir cette forme complexe :

```javascript
const user = {
    preferences: {
        emailEnabled: true // ou false, ou truthy
    },
    contact: {
        email: "..." // Doit être une string à cause de .toUpperCase()
    }
}
```

**L'apport de TypeScript :**
Nous définirons une `interface User` qui décrira explicitement cette structure. Si vous passez un objet mal formé, le code ne compilera pas.

</details>

### Exercice 3 - Check Environment {#exercice-3---check-environment}

**🎯 Objectif :** Valider que votre poste est prêt.

**📝 Énoncé :**
Ouvrez votre terminal et vérifiez vos versions de Node et Npm.

```bash
node -v
npm -v
```

**📺 Résultat attendu :**
- Node : `v18.x.x` ou supérieur (Recommandé `v20+`).
- Npm : `9.x.x` ou supérieur.

<details>
<summary>Besoin d'aide pour installer ?</summary>

Si vous n'avez pas ces outils ou des versions trop anciennes :
1. Allez sur [nodejs.org](https://nodejs.org).
2. Téléchargez la version **LTS** (Long Term Support).
3. Installez-la.
4. Redémarrez votre terminal.

Nous verrons l'installation détaillée au Chapitre 2.

</details>