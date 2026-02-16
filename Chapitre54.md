Voici le chapitre **React Compiler: Configuration et Directives** pour la formation React 19.2.

```markdown
---
sidebar_label: React Compiler: Configuration et Directives
sidebar_position: 54
---

# Chapitre 54 : React Compiler: Configuration et Directives

Options de configuration, Directives `"use memo"`, `"use no memo"`, Contrôle de la compilation

Dans le chapitre précédent, nous avons vu que le React Compiler agit comme un "autopilote", optimisant automatiquement votre code pour éviter les rendus inutiles. C'est magique, mais en ingénierie logicielle, la magie peut parfois être dangereuse si on ne peut pas la contrôler.

Que faire si le compilateur casse un composant legacy ? Comment déboguer si un comportement étrange apparaît ? Comment s'assurer que notre code est "compilable" ?

React 19 introduit des **directives** spécifiques (similaires à `"use strict"` ou `"use client"`) pour donner des ordres explicites au compilateur.

---

## 1. La Directive de Désactivation : `"use no memo"` {#la-directive-use-no-memo}

### 1. Quoi
C'est une chaîne de caractères spéciale à placer au tout début d'un composant ou d'un Hook. Elle ordonne au React Compiler de **ne pas toucher** à cette fonction et de la laisser telle quelle (comportement React 18 standard).

### 2. Pourquoi
*   **Débogage** : Si un bug apparaît, désactiver le compilateur sur un composant permet de vérifier si le problème vient de votre logique ou de l'optimisation automatique.
*   **Code Legacy** : Si vous avez un vieux composant "sale" qui mute des objets ou utilise des refs de manière non standard, le compilateur pourrait casser son fonctionnement.
*   **Incompatibilité** : Certaines bibliothèques très anciennes peuvent mal réagir à la mémoïsation agressive.

### 3. Comment

#### A. Syntaxe

```tsx
function LegacyComponent({ data }) {
  "use no memo"; // 🛑 Stop ! Le compilateur ignorera ce composant.
  
  // Ce code s'exécutera à chaque rendu parent, sans mémoïsation
  return <div>{data.label}</div>;
}
```

#### B. Cas concret : Composant instable volontaire

Imaginons un composant qui *doit* se re-rendre à chaque fois pour une raison obscure (ex: animation impérative mal gérée qui dépend du cycle de rendu).

```tsx
import { useRef } from 'react';

function BadAnimation({ items }: { items: string[] }) {
  "use no memo"; // On force le comportement standard

  const renderCount = useRef(0);
  renderCount.current++;

  console.log(`Rendu n°${renderCount.current}`);

  return (
    <div className="animate-pulse">
      {items.map(item => <span key={item}>{item}</span>)}
    </div>
  );
}
```

### 4. Zone de Danger
Utiliser `"use no memo"` ne doit être qu'une solution **temporaire** ou de **dernier recours**. Si vous l'utilisez pour masquer des erreurs de mutation, vous ne faites que repousser le problème.

---

## 2. Le Plugin ESLint : Le Gardien des Règles {#le-plugin-eslint}

### 1. Quoi
Le React Compiler ne travaille pas seul. Il est accompagné d'un plugin ESLint indispensable : `eslint-plugin-react-compiler`. Ce plugin analyse votre code en temps réel dans votre éditeur (VS Code, etc.).

### 2. Pourquoi
Le compilateur part du principe que vous respectez les **Règles de React** (immutabilité, ordre des Hooks, etc.).
*   Si vous violez ces règles, le compilateur peut "bailout" (abandonner l'optimisation) silencieusement.
*   Le plugin ESLint vous avertit explicitement : "Attention, ce code empêche l'optimisation".

### 3. Comment

Le plugin détecte les violations qui désactiveraient le compilateur.

#### A. Exemple de détection

```tsx
function UserProfile({ user }) {
  // ❌ ESLint Error: 
  // "React Compiler: Mutating a value that was passed into the component is not allowed."
  user.name = "Modifié"; 

  return <div>{user.name}</div>;
}
```

#### B. Configuration (Rappel contextuel)

Dans votre `eslint.config.js` (ou `.eslintrc`), assurez-vous que le plugin est actif :

```javascript
// eslint.config.js
import reactCompiler from 'eslint-plugin-react-compiler';

export default [
  {
    plugins: {
      'react-compiler': reactCompiler,
    },
    rules: {
      'react-compiler/react-compiler': 'error', // ou 'warn'
    },
  },
];
```

---

## 3. La Directive d'Activation : `"use memo"` (Expérimental) {#la-directive-use-memo}

### 1. Quoi
C'est l'inverse de `"use no memo"`. Cette directive force le compilateur à optimiser un composant, même si le compilateur n'est pas activé globalement pour tout le projet.

### 2. Pourquoi
*   **Adoption progressive** : Vous pouvez configurer le compilateur en mode "opt-in" (désactivé par défaut) et l'activer fichier par fichier.
*   **Tests de performance** : Comparer les perfs avec et sans compiler sur un composant précis.

### 3. Comment

```tsx
function ExpensiveChart({ data }) {
  "use memo"; // ✨ Force l'optimisation de ce composant spécifique
  
  // Tout le code ici sera mémoïsé automatiquement
  const processed = data.map(d => d.value * 2);
  
  return <Chart data={processed} />;
}
```

### 🚨 Limitations de l'approche "Opt-in"
En 2026, la recommandation standard est d'activer le compilateur **globalement** (via la config Vite/Webpack) et d'utiliser `"use no memo"` pour les exceptions. L'approche `"use memo"` est réservée aux migrations de très grosses bases de code legacy.

---

## 4. Comprendre le "Bailout" (Abandon d'Optimisation) {#comprendre-le-bailout}

### 1. Quoi
Lorsque le compilateur rencontre une syntaxe qu'il ne comprend pas ou qui viole la sécurité de React, il effectue un **Bailout** (il se retire). Le composant retombe alors en mode de rendu standard (React 18).

### 2. Pourquoi
Pour ne pas casser votre application. Mieux vaut un code non-optimisé qui fonctionne qu'un code optimisé qui bug.

### 3. Comment détecter un Bailout ?
Le plugin ESLint est votre meilleur allié. Cependant, vous pouvez aussi utiliser les **React DevTools**.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : React DevTools (onglet Components).
> **Annotation** : Montrez un composant avec le badge "Memo ✨" (optimisé) et un autre sans badge (ou avec une indication de bailout).
> **Alt Text suggéré** : React DevTools montrant l'état de compilation des composants.

Si un composant n'est pas optimisé, vérifiez :
1.  A-t-il une directive `"use no memo"` ?
2.  Y a-t-il une mutation de props ou de state ?
3.  L'utilisation des Hooks est-elle conditionnelle ?

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-54}

1.  **Quelle directive permet de désactiver le React Compiler pour un composant spécifique ?**
    `"use no memo"`. Elle doit être placée en toute première ligne du corps de la fonction du composant ou du Hook.

2.  **Si le compilateur rencontre une erreur de règles React (ex: mutation de props), que fait-il ?**
    Il effectue un "bailout" (abandon) pour ce composant ou ce hook spécifique. Il ne compile pas cette partie et laisse le code original s'exécuter pour éviter de casser l'application.

3.  **Pourquoi le plugin ESLint `react-compiler` est-il aussi important que le compilateur lui-même ?**
    Parce qu'il donne un retour immédiat au développeur (feedback loop). Il permet d'identifier pourquoi un composant n'est pas optimisé et d'enseigner les bonnes pratiques (comme l'immutabilité) en temps réel.

---

## Exercices : {#exercices-54}

### Exercice 1 - Le Débogueur Sceptique ("use no memo") {#exercice-1---le-debogueur-sceptique}

🎯 **Objectif** : Utiliser la directive d'exclusion pour diagnostiquer un problème.

💼 **Mise en situation** : Vous avez un composant `LiveClock` qui est censé afficher l'heure. Depuis l'activation du compilateur, l'heure ne se met plus à jour correctement. Vous suspectez que le compilateur cache un bug de closure.

📝 **Énoncé** :
1. Prenez le composant `LiveClock` ci-dessous.
2. Ajoutez la directive pour désactiver le compilateur.
3. Observez que le comportement change (ou redevient "normal" mais lent/buggy selon le cas).
4. *Bonus mental* : Comprenez que le bug venait probablement d'un `useEffect` mal déclaré que le compilateur a "figé".

```tsx
import { useState, useEffect } from 'react';

export function LiveClock() {
  // AJOUTEZ LA DIRECTIVE ICI POUR DÉSACTIVER L'OPTIMISATION
  "use no memo";

  const [time, setTime] = useState(new Date());

  useEffect(() => {
    const timer = setInterval(() => setTime(new Date()), 1000);
    return () => clearInterval(timer);
  }, []); // Dépendances vides : en React classique c'est OK pour un interval, 
          // mais le compilateur est très strict.

  return <h2>Il est : {time.toLocaleTimeString()}</h2>;
}
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState, useEffect } from 'react';

export function LiveClock() {
  // 1. Désactivation explicite pour débogage ou sécurité
  "use no memo";

  const [time, setTime] = useState(new Date());

  useEffect(() => {
    const timer = setInterval(() => setTime(new Date()), 1000);
    return () => clearInterval(timer);
  }, []);

  return <h2>Il est : {time.toLocaleTimeString()}</h2>;
}
```
</details>

### Exercice 2 - L'Inspecteur des Règles (Validation ESLint) {#exercice-2---l-inspecteur-des-regles}

🎯 **Objectif** : Identifier et corriger le code qui provoque un "Bailout" du compilateur.

💼 **Mise en situation** : Le plugin ESLint souligne une ligne en rouge dans votre composant `UserCard`. Le compilateur refuse d'optimiser.

📝 **Énoncé** :
Le code ci-dessous modifie une variable locale dérivée d'une prop de manière impure. Réécrivez-le pour satisfaire le compilateur.

**Code invalide (Bailout) :**
```tsx
function UserTags({ tags }) {
  // Le compilateur voit une mutation potentielle sur un tableau partagé
  const sortedTags = tags.sort(); 
  
  return <div>{sortedTags.join(', ')}</div>;
}
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
function UserTags({ tags }: { tags: string[] }) {
  // ✅ Correction : Utiliser toSorted() qui renvoie une COPIE
  // Le compilateur peut maintenant mémoïser le résultat de cet appel pur.
  const sortedTags = tags.toSorted(); 
  
  // Alternative classique : [...tags].sort()
  
  return <div>{sortedTags.join(', ')}</div>;
}
```
</details>

### Exercice 3 - Optimisation Sélective (Mix and Match) {#exercice-3---optimisation-selective}

🎯 **Objectif** : Simuler une architecture où l'on doit contrôler finement l'optimisation.

💼 **Mise en situation** : Vous travaillez sur une librairie de composants UI. Le composant `DataGrid` est très complexe et vous voulez garantir son optimisation, tandis que le composant `DebugPanel` doit rester brut.

📝 **Énoncé** :
Dans un même fichier :
1. Créez un composant `DataGrid` avec `"use memo"` (simulation d'activation forcée).
2. Créez un composant `DebugPanel` avec `"use no memo"`.
3. Affichez les deux.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
// Composant Optimisé de force
function DataGrid({ rows }: { rows: any[] }) {
  "use memo";
  
  // Ce calcul lourd sera mis en cache automatiquement
  const processedRows = rows.map(r => ({ ...r, active: true }));
  
  return (
    <table>
      <tbody>
        {processedRows.map(r => <tr key={r.id}><td>{r.name}</td></tr>)}
      </tbody>
    </table>
  );
}

// Composant Brut (Bypass)
function DebugPanel({ logs }: { logs: string[] }) {
  "use no memo";

  // Ce composant se re-rendra à chaque mise à jour du parent
  console.log("DebugPanel render");
  
  return (
    <pre style={{ background: '#eee' }}>
      {logs.join('\n')}
    </pre>
  );
}

export function AppHybrid() {
  return (
    <div>
      <DataGrid rows={[{ id: 1, name: "Item A" }]} />
      <DebugPanel logs={["Log 1", "Log 2"]} />
    </div>
  );
}
```
</details>
```