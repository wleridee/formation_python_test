Voici le contenu complet du chapitre **AvantPropos.md**.

```markdown
---
sidebar_label: Avant-propos
sidebar_position: 0
---

# Chapitre 0 : Avant-propos

Introduction à la formation, Prérequis (JavaScript), Objectifs pédagogiques

## 1. Introduction à la formation {#introduction-a-la-formation-0}

### 1. Quoi {#quoi-introduction}
Bienvenue dans cette formation exhaustive dédiée à **TypeScript 5.x**. Ce cours est conçu pour transformer un développeur JavaScript compétent en un ingénieur TypeScript expert, capable de concevoir des applications robustes, scalables et maintenables.

TypeScript n'est pas simplement "JavaScript avec des types". C'est un ensemble d'outils de productivité et de sécurité qui modifie fondamentalement la manière de penser l'architecture logicielle frontend et backend.

### 2. Pourquoi TypeScript 5.x ? {#pourquoi-typescript-5}
La version 5 de TypeScript a marqué un tournant majeur en termes de performance (builds plus rapides), de simplification de la configuration et de fonctionnalités modernes (décorateurs conformes au standard ECMAScript, `const` type parameters, etc.).

Dans l'écosystème actuel (2026), TypeScript est le **standard de facto**. De React à Angular, en passant par NestJS ou les runtimes comme Deno et Bun, ne pas maîtriser TypeScript est devenu un frein professionnel majeur.

### 3. Philosophie du cours {#philosophie-du-cours}
Cette formation repose sur trois piliers :

1.  **No Magic** : Nous n'utilisons pas de copier-coller. Chaque syntaxe, chaque erreur de compilation et chaque configuration est expliquée.
2.  **TypeScript Strict** : Nous apprendrons directement avec le mode `strict: true`. Apprendre le mode laxiste est une perte de temps et une source de dette technique.
3.  **Approche Métier** : Les exemples sortent du classique "Foo/Bar". Nous modéliserons des paniers e-commerce, des réponses d'API SaaS, des systèmes de droits utilisateurs, etc.

---

## 2. Prérequis (JavaScript) {#prerequis-javascript-0}

### 1. Le niveau attendu {#niveau-attendu}
**Attention :** Ce cours n'est PAS un cours de JavaScript. TypeScript est un sur-ensemble (superset) de JavaScript. Si vous ne maîtrisez pas les fondations de JS, vous construirez sur du sable.

### 2. Checklist de compétences {#checklist-competences}
Avant de commencer le Chapitre 1, assurez-vous d'être à l'aise avec les concepts suivants (ES6+) :

*   **Variables** : Portée de `let` et `const` (et pourquoi ne plus utiliser `var`).
*   **Fonctions** : Fonctions fléchées (arrow functions), le mot-clé `this`.
*   **Tableaux & Objets** : Méthodes `map`, `filter`, `reduce`, déstructuration, spread operator (`...`).
*   **Asynchronisme** : `Promise`, `async/await`.
*   **Modules** : Syntaxe `import` et `export` (ES Modules).

### 3. Exemple de code prérequis {#exemple-code-prerequis}
Vous devez être capable de lire et comprendre ce code JavaScript natif sans difficulté :

```javascript
// Si ce code vous semble obscur, révisez le JavaScript moderne avant de continuer.

const fetchUserData = async (userId) => {
  try {
    const response = await fetch(`/api/users/${userId}`);
    if (!response.ok) throw new Error('Erreur réseau');
    
    const data = await response.json();
    // Déstructuration avec valeur par défaut
    const { name, roles = [] } = data; 
    
    return { 
      name, 
      isAdmin: roles.includes('ADMIN') 
    };
  } catch (error) {
    console.error("Échec:", error);
    return null;
  }
};
```

---

## 3. Objectifs Pédagogiques {#objectifs-pedagogiques-0}

### 1. Maîtrise de la syntaxe et de l'inférence {#maitrise-syntaxe}
À la fin de ce cours, vous saurez quand laisser TypeScript deviner les types (inférence) et quand les définir explicitement pour garantir la sécurité du code.

### 2. Modélisation de données complexe {#modelisation-donnees}
Vous apprendrez à utiliser les **Generics**, les **Utility Types** et les **Conditional Types** pour créer des types dynamiques qui s'adaptent à vos données, éliminant ainsi le besoin de maintenance manuelle des types.

### 3. Architecture et Outillage {#architecture-outillage}
Au-delà du code, nous verrons comment configurer un projet (`tsconfig.json`), gérer le packaging, publier des librairies typées sur NPM et intégrer des tests unitaires typés.

### 4. Zone de Danger : Ce que nous éviterons {#zone-de-danger-objectifs}

| ❌ À NE PAS FAIRE | ✅ BONNE PRATIQUE |
| :--- | :--- |
| Utiliser `any` pour faire taire le compilateur | Utiliser `unknown` et le *Narrowing* |
| Typer manuellement ce qui est évident | Faire confiance à l'inférence de type |
| Ignorer les erreurs de compilation en production | Traiter les erreurs TS comme des erreurs bloquantes |
| Utiliser des assertions `as` partout | Utiliser des *Type Guards* personnalisés |

---

## 4. Structure de la formation {#structure-formation-0}

La formation est découpée en **62 chapitres** progressifs :

1.  **Fondations (Chap 1-12)** : Installation, types primitifs, structures de contrôle.
2.  **Core API & POO (Chap 13-23)** : Unions, Interfaces, Classes, Generics.
3.  **Concepts Avancés (Chap 24-48)** : Asynchronisme, Types conditionnels, Mapped types, Utility types.
4.  **Écosystème & Production (Chap 49-61)** : Packaging, Tests, Décorateurs, Configuration monorepo.

> 💡 **Conseil de lecture** : Ne sautez pas les exercices. La théorie des Generics est simple, mais leur pratique sur des projets réels est complexe.

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-validation-des-acquis-du-chapitre-0}

1.  **TypeScript est-il un nouveau langage ou une extension de JavaScript ?**
    **C'est une extension (Superset)**. Il ajoute le typage statique optionnel au JavaScript existant et compile vers du JavaScript standard.

2.  **Quel est le rôle principal de TypeScript ?**
    **La sécurité de type au moment de la compilation** (Type safety at compile time). Il détecte les erreurs avant l'exécution du code.

3.  **Faut-il connaître JavaScript pour apprendre TypeScript ?**
    **Absolument**. TypeScript ne remplace pas la logique de JavaScript (boucles, fonctions, async), il la sécurise.

---

## Exercices : {#exercices-:-0}

Ces exercices servent à vérifier que votre environnement et vos connaissances JavaScript sont prêts pour la suite.

### Exercice 1 - Vérification de l'environnement {#exercice-1---verification-de-l-environnement}

**🎯 Objectif** : S'assurer que vous disposez des outils de base pour suivre la formation.

**💼 Mise en situation** :
Vous intégrez une nouvelle équipe Tech. La première étape est de vérifier que votre machine est configurée pour le développement moderne.

**📝 Énoncé** :
Ouvrez votre terminal et vérifiez les versions de Node.js et NPM.

**📺 Résultat attendu** :
Une version de Node.js >= 18.x (LTS) et NPM >= 9.x.

<details>
<summary>Voir la commande et le résultat attendu</summary>

```bash
# Vérifier la version de Node.js
node -v 
# Doit retourner v18.16.0 ou supérieur (v20+ recommandé en 2026)

# Vérifier la version de npm
npm -v
# Doit retourner 9.5.0 ou supérieur
```

</details>

### Exercice 2 - Test de niveau JS (Mental) {#exercice-2---test-de-niveau-js-mental}

**🎯 Objectif** : Valider la compréhension des méthodes de tableaux ES6.

**💼 Mise en situation** :
Vous devez filtrer une liste d'utilisateurs pour ne garder que les actifs, puis extraire uniquement leurs emails.

**📝 Énoncé** :
Soit le tableau suivant en JavaScript pur :
```javascript
const users = [
  { id: 1, email: "alice@example.com", isActive: true },
  { id: 2, email: "bob@example.com", isActive: false },
  { id: 3, email: "charlie@example.com", isActive: true },
];
```
Écrivez une seule ligne de code utilisant les méthodes de tableau (`map`, `filter`, etc.) pour obtenir `["alice@example.com", "charlie@example.com"]`.

**📺 Résultat attendu** :
Un tableau de chaînes de caractères contenant les emails des utilisateurs actifs.

<details>
<summary>Voir le code complet commenté</summary>

```javascript
// On chaîne les méthodes : d'abord on filtre, ensuite on transforme (map)
const activeEmails = users
  .filter(user => user.isActive) // Garde uniquement les utilisateurs actifs
  .map(user => user.email);      // Transforme chaque objet utilisateur en string (email)

console.log(activeEmails); 
// ["alice@example.com", "charlie@example.com"]
```

</details>

### Exercice 3 - Compréhension de l'Immutabilité {#exercice-3---comprehension-de-l-immutabilite}

**🎯 Objectif** : Comprendre la copie par référence vs copie par valeur (fondamental pour React/Redux/State Management en TS).

**💼 Mise en situation** :
Vous déboguez une fonctionnalité où la modification d'un objet "copié" modifie accidentellement l'original.

**📝 Énoncé** :
Analysez le code suivant. Que vaut `originalConfig.theme` à la fin ?

```javascript
const originalConfig = { theme: "dark", version: 1 };
const newConfig = originalConfig; // Attention ici
newConfig.theme = "light";
```

**📺 Résultat attendu** :
Comprendre que `originalConfig.theme` est devenu "light".

<details>
<summary>Voir l'explication et la correction</summary>

**Réponse** : `originalConfig.theme` vaut `"light"`.
En JS, les objets sont passés par **référence**. `newConfig` pointe vers le même emplacement mémoire que `originalConfig`.

**Correction (Copie superficielle / Shallow copy)** :
```javascript
const originalConfig = { theme: "dark", version: 1 };
// On crée un NOUVEL objet avec les propriétés de l'ancien
const newConfig = { ...originalConfig }; 
newConfig.theme = "light";

console.log(originalConfig.theme); // "dark" (Préservé)
console.log(newConfig.theme);      // "light"
```

</details>
```