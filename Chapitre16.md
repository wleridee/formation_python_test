Voici le chapitre **Mettre en File d'Attente des Mises à Jour d'État** pour la formation React 19.2.

```markdown
---
sidebar_label: Mettre en File d'Attente des Mises à Jour d'État
sidebar_position: 16
---

# Chapitre 16 : Mettre en File d'Attente des Mises à Jour d'État

Mises à jour batching, Mises à jour fonctionnelles, Ordre des mises à jour

Dans le chapitre précédent, nous avons vu que l'état est un instantané (snapshot). Modifier l'état ne change pas la variable immédiatement, mais demande un nouveau rendu.
Mais que se passe-t-il si vous voulez effectuer plusieurs modifications à la suite sur la **même** variable d'état, avant même que le prochain rendu n'ait lieu ?
C'est ici que nous entrons dans les coulisses du moteur de React : la file d'attente (queue) et le regroupement (batching).

## Le Batching (Regroupement) {#le-batching}

### 1. Quoi
Le **Batching** est le mécanisme par lequel React regroupe plusieurs demandes de mises à jour d'état (`setState`) en **un seul rendu** pour des raisons de performance.

### 2. Pourquoi
Imaginez que vous êtes au restaurant. Vous ne demandez pas au serveur d'aller en cuisine juste pour commander une fourchette, puis de revenir, puis de repartir pour commander un couteau. Vous donnez toute votre liste, et il fait un seul voyage.
React fait pareil : il attend que tout votre code dans le gestionnaire d'événement soit terminé avant de mettre à jour l'écran. Cela évite que l'interface ne clignote avec des états "à moitié finis".

### 3. Comment

```tsx
import { useState } from 'react';

export function BatchingExample() {
  const [count, setCount] = useState(0);

  const handleClick = () => {
    // React ne redessine PAS l'écran ici
    setCount(count + 1);
    
    // Ni ici
    setCount(count + 1);
    
    // Ni ici
    setCount(count + 1);
    
    // -> React redessine UNE SEULE fois à la fin de cette fonction
  };

  return <button onClick={handleClick}>Compteur : {count}</button>;
}
```

Dans l'exemple ci-dessus, même s'il y a 3 appels, il n'y aura qu'un seul rendu. Cependant, comme vu au chapitre précédent, le compteur n'augmentera que de 1, car `count` vaut `0` pour les trois appels.

---

## Mises à jour Fonctionnelles (Functional Updates) {#mises-a-jour-fonctionnelles}

### 1. Quoi
Au lieu de passer une *valeur* à `setCount` (ex: `setCount(5)`), vous pouvez lui passer une **fonction** (ex: `setCount(n => n + 1)`).
Cette fonction est appelée "fonction de mise à jour" (updater function).

### 2. Pourquoi
C'est la solution pour effectuer plusieurs mises à jour sur la même variable d'état au sein d'un même événement.
Cela dit à React : "Ne remplace pas simplement la valeur. Prends la valeur **en attente** (calculée par l'opération précédente dans la file) et applique cette transformation."

### 3. Comment

#### A. Syntaxe
La convention est d'utiliser la première lettre de la variable d'état (ex: `n` pour `number`, `c` pour `count`) ou `prev` (pour `previous`).

```tsx
const [score, setScore] = useState(0);

// Style "Remplacement" (basé sur le snapshot)
setScore(score + 1);

// Style "Fonctionnel" (basé sur la valeur précédente calculée)
setScore(prevScore => prevScore + 1);
```

#### B. Résolution du problème d'incrémentation multiple
Reprenons notre compteur qui refusait de monter de 3.

```tsx
export function FixedCounter() {
  const [count, setCount] = useState(0);

  const handleTripleClick = () => {
    // 1. React met "n => n + 1" dans la file. (0 -> 1)
    setCount(n => n + 1);
    
    // 2. React met "n => n + 1" dans la file. Il prendra le 1 précédent -> 2
    setCount(n => n + 1);
    
    // 3. React met "n => n + 1" dans la file. Il prendra le 2 précédent -> 3
    setCount(n => n + 1);
  };

  return <button onClick={handleTripleClick}>+3 (Total: {count})</button>;
}
```

### 4. Zone de Danger

:::danger Fonctions pures obligatoires
La fonction que vous passez à `setCount(fn)` doit être **pure**.
Elle ne doit rien faire d'autre que calculer le résultat.
❌ `setCount(n => { console.log(n); return n + 1; })` (Évitez les effets de bord ici)
✅ `setCount(n => n + 1)`
:::

---

## L'Ordre de Traitement dans la File {#ordre-de-traitement}

### 1. Quoi
Vous pouvez mélanger des mises à jour "valeur" (remplacement) et des mises à jour "fonctionnelles". React traite la file d'attente **dans l'ordre exact** où le code est écrit.

### 2. Pourquoi
Comprendre cet ordre est vital pour prédire l'état final lors d'opérations complexes.

### 3. Comment

Analysons cette séquence :

```tsx
const [number, setNumber] = useState(0);

const handleComplexUpdate = () => {
  // 1. Remplacement : La file contient "remplacer par 5"
  setNumber(5);
  
  // 2. Fonctionnel : La file ajoute "prendre précédent (5) et ajouter 1" -> 6
  setNumber(n => n + 1);
  
  // 3. Remplacement : La file ajoute "remplacer par 42"
  setNumber(42);
};
```

**Résultat final :** `42`.
La dernière instruction de remplacement écrase tout le travail précédent.

Autre exemple :
```tsx
const handleLogic = () => {
  setNumber(number + 5); // Snapshot (0) + 5 = 5. File: [Remplacer par 5]
  setNumber(n => n + 1); // File: [Remplacer par 5, n => n + 1]. Résultat: 6
  setNumber(42);         // File: [..., Remplacer par 42]. Résultat: 42
};
```

---

## Cas Pratique : Toggle Booléen Sécurisé

Le cas d'utilisation le plus courant de la mise à jour fonctionnelle n'est pas le compteur, mais l'interrupteur (toggle).

### Approche Risquée
```tsx
const [isOpen, setIsOpen] = useState(false);
// Si l'utilisateur clique très vite ou si l'événement est asynchrone, 
// 'isOpen' pourrait être périmé.
const toggle = () => setIsOpen(!isOpen); 
```

### Approche Robuste (Best Practice)
```tsx
// On garantit qu'on prend toujours l'inverse de la valeur ACTUELLE en mémoire
const toggle = () => setIsOpen(prev => !prev);
```

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-16}

1.  **Qu'est-ce que le "Batching" en React ?**
    C'est le fait que React regroupe plusieurs mises à jour d'état (`setState`) en un seul rendu pour optimiser les performances.

2.  **Quelle est la différence entre `setCount(count + 1)` et `setCount(n => n + 1)` ?**
    La première utilise la valeur de `count` au moment du rendu (snapshot). La seconde utilise une fonction qui reçoit la valeur d'état *à jour* (celle traitée dans la file d'attente) juste avant d'appliquer la modification.

3.  **Si j'appelle `setCount(5)` puis `setCount(n => n + 1)`, quelle sera la valeur finale ?**
    La valeur finale sera 6. React remplace d'abord l'état par 5, puis applique la fonction `5 + 1`.

4.  **Quand devrais-je utiliser une mise à jour fonctionnelle ?**
    Chaque fois que le nouvel état dépend de l'ancien état (compteurs, toggles, ajouts à une liste), ou si vous faites plusieurs mises à jour dans le même événement.

---

## Exercices : {#exercices-16}

### Exercice 1 - Le Barman (Batching) {#exercice-1---le-barman}

🎯 **Objectif** : Comprendre le cumul des commandes via les mises à jour fonctionnelles.

💼 **Mise en situation** : Vous développez une application pour un bar. Un bouton "Tournée Générale" doit ajouter 4 bières d'un coup au compteur.

📝 **Énoncé** :
1. Créez un composant `BarTab`.
2. État `beers` initialisé à 0.
3. Un bouton "Ajouter 1 bière" (simple `setBeers(beers + 1)`).
4. Un bouton "Tournée Générale (+4)" qui appelle 4 fois `setBeers`.
5. **Défi** : Faites en sorte que le bouton "+4" fonctionne correctement en utilisant les mises à jour fonctionnelles.

📺 **Résultat attendu** :
Le bouton "+1" incrémente de 1. Le bouton "+4" incrémente bien de 4 (et pas de 1).

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState } from 'react';

export function BarTab() {
  const [beers, setBeers] = useState(0);

  const handleOneBeer = () => {
    // Ici, le snapshot suffit
    setBeers(beers + 1);
  };

  const handleRound = () => {
    // ❌ Ceci ne marcherait pas (résultat +1)
    // setBeers(beers + 1);
    // setBeers(beers + 1);
    // setBeers(beers + 1);
    // setBeers(beers + 1);

    // ✅ Utilisation de la mise à jour fonctionnelle
    // React empile les instructions : n+1, puis n+1, etc.
    setBeers(n => n + 1);
    setBeers(n => n + 1);
    setBeers(n => n + 1);
    setBeers(n => n + 1);
  };

  return (
    <div>
      <h1>Commandes : {beers} 🍺</h1>
      <button onClick={handleOneBeer}>+1 Bière</button>
      <button onClick={handleRound}>Tournée Générale (+4)</button>
    </div>
  );
}
```
</details>

### Exercice 2 - Le Jeu du Double (Ordre des opérations) {#exercice-2---le-jeu-du-double}

🎯 **Objectif** : Maîtriser l'ordre d'exécution dans la file d'attente.

💼 **Mise en situation** : Un jeu mathématique. Vous avez un score. Un bouton "Bonus" ajoute 5 points PUIS double le score total.

📝 **Énoncé** :
1. État `score` initialisé à 0.
2. Un bouton "Bonus Combo".
3. Au clic, le bouton doit exécuter deux actions dans l'ordre :
   - Ajouter 5 au score.
   - Multiplier le résultat par 2.
4. Exemple : Si score = 10 -> (10 + 5) * 2 = 30.
5. Implémentez cela dans un seul gestionnaire d'événement.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Un score affichant "10".
> **Annotation** : Montrez le bouton qui va transformer le 10 en 30.
> **Alt Text suggéré** : Interface de jeu mathématique montrant le score avant application du bonus.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState } from 'react';

export function ScoreGame() {
  const [score, setScore] = useState(10); // Valeur de départ pour l'exemple

  const handleBonus = () => {
    // 1. Ajouter 5 (on utilise la forme fonctionnelle pour la sûreté)
    setScore(s => s + 5);
    
    // 2. Doubler le tout
    // React prendra le résultat de la ligne précédente (15) et le multipliera
    setScore(s => s * 2);
  };

  return (
    <div>
      <h2>Score actuel : {score}</h2>
      {/* Résultat attendu depuis 10 : (10+5)*2 = 30 */}
      <button onClick={handleBonus}>Appliquer Bonus (+5 puis x2)</button>
      
      <button onClick={() => setScore(0)}>Reset</button>
    </div>
  );
}
```
</details>

### Exercice 3 - La File d'Attente Visuelle {#exercice-3---la-file-d-attente-visuelle}

🎯 **Objectif** : Comprendre que React ne s'arrête pas au premier `return` dans un setter.

💼 **Mise en situation** : Vous voulez limiter un compteur entre 0 et 10. Même si l'utilisateur clique 20 fois, le compteur ne doit pas dépasser 10.

📝 **Énoncé** :
1. État `count` à 0.
2. Un bouton "+5" qui appelle 5 fois une fonction d'incrémentation.
3. Utilisez `setCount(c => ...)` 5 fois.
4. Dans la fonction fléchée, ajoutez une logique : `si c >= 10, retourner c (ne rien changer), sinon retourner c + 1`.
5. Observez comment la file traite chaque demande séquentiellement en respectant la logique.

📺 **Résultat attendu** :
Si on clique 3 fois sur le bouton "+5", le compteur monte à 5, puis 10, puis reste bloqué à 10.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState } from 'react';

export function LimitedCounter() {
  const [count, setCount] = useState(0);

  const incrementSafe = () => {
    // La fonction de mise à jour reçoit l'état courant de la file
    // Elle peut contenir de la logique !
    setCount(c => {
      if (c >= 10) {
        return c; // On retourne l'état inchangé si la limite est atteinte
      }
      return c + 1;
    });
  };

  const handleAddFive = () => {
    // On empile 5 demandes d'incrémentation sécurisée
    incrementSafe();
    incrementSafe();
    incrementSafe();
    incrementSafe();
    incrementSafe();
  };

  return (
    <div>
      <h3>Compteur (Max 10) : {count}</h3>
      <button onClick={handleAddFive}>Ajouter 5</button>
      <button onClick={() => setCount(0)}>Reset</button>
    </div>
  );
}
```
</details>
```