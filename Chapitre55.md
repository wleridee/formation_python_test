Voici le chapitre **React Compiler: Compilation des Bibliothèques** pour la formation React 19.2.

```markdown
---
sidebar_label: React Compiler: Compilation des Bibliothèques
sidebar_position: 55
---

# Chapitre 55 : React Compiler: Compilation des Bibliothèques

Pré-compilation, Distribution de code optimisé, Compatibilité

Dans les chapitres précédents, nous avons vu comment utiliser le React Compiler pour optimiser une **application**. Mais qu'en est-il si vous développez une **bibliothèque** (un UI Kit, un gestionnaire de formulaires, etc.) distribuée sur npm ?

Devez-vous livrer votre code déjà compilé (optimisé) ou laisser cette tâche à l'application consommatrice ?

Ce chapitre aborde les stratégies de distribution de bibliothèques à l'ère de React 19.

---

## 1. La Stratégie de Pré-compilation {#la-strategie-de-pre-compilation}

### 1. Quoi
La pré-compilation consiste à exécuter le React Compiler sur le code de votre bibliothèque lors de votre étape de build (avant le `npm publish`). Le code distribué dans le dossier `dist/` contient déjà les optimisations (appels à `useMemoCache`, etc.).

### 2. Pourquoi
*   **Performance garantie** : Les utilisateurs de votre bibliothèque bénéficient des optimisations même s'ils n'ont pas encore configuré le React Compiler dans leur propre application.
*   **Contrôle qualité** : Vous validez que vos composants sont parfaitement optimisés et respectent les règles de React avant la distribution.

### 3. Comment

Il faut configurer votre outil de build de bibliothèque (souvent Rollup, tsup ou Vite en mode lib) pour utiliser le plugin du compilateur.

#### A. Configuration typique (Vite/Rollup)

```javascript
// vite.config.ts (pour une bibliothèque)
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

const ReactCompilerConfig = {
  target: '19' // Cible explicite
};

export default defineConfig({
  plugins: [
    react({
      babel: {
        plugins: [
          ['babel-plugin-react-compiler', ReactCompilerConfig],
        ],
      },
    }),
  ],
  build: {
    lib: {
      entry: 'src/index.ts',
      name: 'MyUiKit',
      fileName: 'my-ui-kit',
    },
    rollupOptions: {
      // Externaliser React est crucial
      external: ['react', 'react-dom', 'react/compiler-runtime'],
    },
  },
});
```

#### B. Résultat dans le bundle (Conceptuel)

Votre code source :
```tsx
export function Button({ label }) {
  return <button className="btn">{label}</button>;
}
```

Votre code distribué (simplifié) :
```js
import { c as _c } from "react/compiler-runtime"; // Dépendance runtime !
export function Button({ label }) {
  const $ = _c(2); // Le hook de cache est injecté
  // ... logique optimisée ...
}
```

### 4. Zone de Danger
Si vous pré-compilez, votre bibliothèque **ne fonctionnera pas** sur des versions de React antérieures à 19 (ou celles ne supportant pas le runtime du compilateur). Vous créez un couplage fort.

---

## 2. La Stratégie "Compiler-Ready" (Recommandée) {#la-strategie-compiler-ready}

### 1. Quoi
Au lieu de livrer du code compilé, vous livrez du code JavaScript standard (transpilé en ESModules ou CommonJS), mais écrit de manière à être **facilement optimisable** par le compilateur de l'utilisateur.

### 2. Pourquoi
*   **Compatibilité maximale** : Votre bibliothèque fonctionne avec React 18, 19, et futurs, car elle ne dépend pas d'un runtime spécifique injecté au build.
*   **Flexibilité** : L'utilisateur final garde le contrôle. S'il active le compilateur, votre lib sera optimisée (car elle est incluse dans le `node_modules` et traitée si configurée).

### 3. Comment

Il s'agit ici de discipline de code plutôt que de configuration de build.

#### A. Code Source Propre

Assurez-vous que votre bibliothèque respecte strictement les règles de React (pas de mutation de props, hooks inconditionnels).

```tsx
// ✅ Compiler-Ready : Code pur et immuable
export function DataGrid({ data }) {
  // Pas de useMemo manuel nécessaire, mais pas interdit.
  // L'utilisateur pourra le compiler s'il le souhaite.
  const sorted = data.toSorted((a, b) => a.id - b.id);
  return <div>...</div>;
}
```

#### B. Indication de compatibilité

Vous pouvez ajouter le plugin ESLint `eslint-plugin-react-compiler` dans le processus de CI/CD de votre bibliothèque pour garantir qu'elle ne contient pas de violations qui empêcheraient l'optimisation chez l'utilisateur.

---

## 3. Gestion des Dépendances et Compatibilité {#gestion-des-dependances-et-compatibilite}

### 1. Quoi
Si vous choisissez la **pré-compilation** (Stratégie 1), vous devez déclarer correctement vos dépendances pour éviter que l'application de l'utilisateur ne crashe.

### 2. Pourquoi
Le code compilé par React Compiler insère des appels à des API internes (comme `useMemoCache` ou des imports depuis `react/compiler-runtime`). Si l'utilisateur a React 18, ces imports n'existent pas.

### 3. Comment

#### A. peerDependencies Strictes

Dans le `package.json` de votre bibliothèque pré-compilée :

```json
{
  "name": "super-fast-ui",
  "version": "2.0.0",
  "peerDependencies": {
    "react": "^19.0.0", 
    "react-dom": "^19.0.0"
  },
  "description": "Bibliothèque optimisée pour React 19+"
}
```

#### B. Le piège du "Double Compile"

Si vous livrez une bibliothèque pré-compilée ET que l'utilisateur compile aussi ses `node_modules` (ce qui est rare par défaut mais possible), le compilateur de l'utilisateur est assez intelligent pour voir que le code est déjà optimisé et ne pas y toucher. Il n'y a pas de risque de "double optimisation" qui casserait le code, mais cela ralentit le build de l'utilisateur inutilement.

### 🚨 Limitations de la Pré-compilation
Une bibliothèque pré-compilée est **difficile à déboguer** pour l'utilisateur. Si un bug survient dans votre composant, l'utilisateur verra dans son débogueur du code transformé avec des variables `_c`, `$`, `t0`, ce qui rend la stack trace cryptique.
👉 **Réservez la pré-compilation aux bibliothèques internes d'entreprise ou aux composants critiques de très haut niveau.**

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-55}

1.  **Quelle est la principale contrainte technique imposée par la distribution d'une bibliothèque pré-compilée ?**
    Elle force l'application consommatrice à utiliser une version de React compatible avec le code généré (React 19+), rendant la bibliothèque incompatible avec les anciennes versions (React 18 et moins).

2.  **Qu'est-ce qu'une bibliothèque "Compiler-Ready" ?**
    C'est une bibliothèque dont le code source respecte strictement les règles de React (immutabilité, hooks) et ne contient pas de code empêchant l'optimisation (bailout), permettant ainsi à l'application consommatrice de la compiler elle-même si elle le souhaite.

3.  **Pourquoi est-il déconseillé de pré-compiler une bibliothèque Open Source généraliste ?**
    Pour des raisons de compatibilité (limiter le support aux versions récentes de React) et de débogage (le code compilé est illisible pour l'utilisateur final en cas de problème).

---

## Exercices : {#exercices-55}

### Exercice 1 - Configuration du package.json pour une Lib Optimisée {#exercice-1---config-package-json}

🎯 **Objectif** : Préparer la publication d'une bibliothèque qui a été compilée avec React Compiler.

💼 **Mise en situation** : Vous maintenez `agency-charts`, une lib de graphiques internes très lourde. Vous avez décidé de la pré-compiler pour garantir la fluidité sur tous les projets de l'agence.

📝 **Énoncé** :
Écrivez la section `peerDependencies` et `engines` du fichier `package.json` pour interdire l'installation de cette bibliothèque sur des projets React 18.

<details>
<summary>💡 Voir le code complet commenté</summary>

```json
{
  "name": "@agency/charts",
  "version": "3.5.0",
  // Empêche l'installation si l'environnement Node/NPM est trop vieux
  "engines": {
    "node": ">=18.0.0"
  },
  // La partie CRUCIALE pour une lib pré-compilée
  "peerDependencies": {
    // Le code compilé utilise des APIs disponibles uniquement dans React 19
    "react": "^19.0.0",
    "react-dom": "^19.0.0"
  },
  // Optionnel mais recommandé : expliquer pourquoi
  "peerDependenciesMeta": {
    "react": {
      "optional": false
    }
  }
}
```
</details>

### Exercice 2 - Simulation de Crash de Compatibilité {#exercice-2---simulation-crash}

🎯 **Objectif** : Comprendre ce qui se passe quand on utilise une lib compilée dans un vieil environnement.

💼 **Mise en situation** : Un développeur junior essaie d'utiliser votre lib `super-fast-ui` (compilée React 19) dans un vieux projet React 17.

📝 **Énoncé** :
Le code compilé ressemble à ceci :
```js
import { c as _c } from "react/compiler-runtime";
// ... usage de _c()
```
Quelle erreur le développeur va-t-il rencontrer au lancement de l'application ? (Réponse théorique attendue).

<details>
<summary>💡 Voir la réponse détaillée</summary>

Le développeur rencontrera une erreur de résolution de module ou d'exécution (Runtime Error).

**Erreur probable :**
`Module not found: Error: Can't resolve 'react/compiler-runtime'`

Ou si le build passe mais que le runtime manque :
`TypeError: _c is not a function` ou `undefined is not a function`

**Pourquoi ?**
Parce que le paquet `react` en version 17 n'exporte pas le sous-chemin `/compiler-runtime` et ne contient pas les hooks internes nécessaires au code généré par le compilateur.
</details>

### Exercice 3 - Refactoring pour "Compiler-Ready" {#exercice-3---refactoring-compiler-ready}

🎯 **Objectif** : Nettoyer un composant de bibliothèque pour qu'il soit optimisable par le client.

💼 **Mise en situation** : Vous créez une bibliothèque de composants `AvatarGroup`. Vous voulez qu'elle soit "Compiler-Ready". Vous devez supprimer les `useMemo` manuels qui encombrent le code, car le compilateur du client s'en chargera mieux.

📝 **Énoncé** :
Transformez ce composant de bibliothèque pour qu'il soit plus propre (React 19 style), tout en gardant la même logique.

**Code actuel (Style React 18) :**
```tsx
import { useMemo } from 'react';

export function AvatarGroup({ users, max = 3 }) {
  const visibleUsers = useMemo(() => {
    return users.slice(0, max);
  }, [users, max]);

  const remaining = useMemo(() => {
    return users.length - max;
  }, [users.length, max]);

  return (
    <div className="avatar-group">
      {visibleUsers.map(u => <img key={u.id} src={u.avatar} />)}
      {remaining > 0 && <span>+{remaining}</span>}
    </div>
  );
}
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
// ✅ Version "Compiler-Ready" pour React 19
// Plus lisible, pas de dépendance manuelle.
// Si le consommateur active le compilateur, ce sera optimisé.
// Sinon, ça fonctionne comme du React standard.

export function AvatarGroup({ users, max = 3 }: { users: User[], max?: number }) {
  // Calculs simples, directs.
  // Le compilateur détectera la dépendance à [users, max] automatiquement.
  const visibleUsers = users.slice(0, max);
  const remaining = users.length - max;

  return (
    <div className="avatar-group">
      {visibleUsers.map(u => <img key={u.id} src={u.avatar} alt={u.name} />)}
      {remaining > 0 && <span>+{remaining}</span>}
    </div>
  );
}
```
</details>
```