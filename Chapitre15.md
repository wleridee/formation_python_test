Voici le chapitre **L'État comme un Snapshot** pour la formation React 19.2.

```markdown
---
sidebar_label: L'État comme un Snapshot
sidebar_position: 15
---

# Chapitre 15 : L'État comme un Snapshot

Fonctions de rendu pures, Capture des valeurs, Mises à jour asynchrones

Une source de confusion majeure pour les débutants (et même les intermédiaires) en React est le timing des mises à jour de l'état.
Avez-vous déjà écrit un `console.log(state)` juste après un `setState(...)` et vu l'ancienne valeur s'afficher ?
Ce chapitre explique pourquoi. Pour maîtriser React, vous devez changer votre modèle mental : l'état n'est pas une variable que l'on modifie, c'est un **instantané (snapshot)** figé dans le temps pour chaque rendu.

## L'État comme variable locale immuable {#l-etat-comme-variable-locale-immuable}

### 1. Quoi
Lorsqu'un composant React s'exécute (se "rend"), c'est comme si on prenait une **photo** de l'interface à cet instant précis.
Les `props`, l'état (`state`) et les gestionnaires d'événements sont des constantes calculées **pour ce rendu spécifique**. Ils ne changent pas pendant que l'utilisateur regarde l'écran.

### 2. Pourquoi
Cela garantit la cohérence de l'interface. Si React modifiait les variables "sous vos pieds" pendant l'exécution d'une fonction, le rendu deviendrait imprévisible et difficile à déboguer.

### 3. Comment

#### A. Le piège classique
Regardez ce code attentivement.

```tsx
import { useState } from 'react';

export function Counter() {
  const [count, setCount] = useState(0);

  const handleClick = () => {
    setCount(count + 1); // On demande à passer à 1
    console.log(count);  // ❓ Qu'est-ce qui s'affiche ?
  };

  return <button onClick={handleClick}>+1</button>;
}
```

**Réponse :** La console affiche `0`.

**Explication :**
1. Au premier rendu, `count` vaut `0`.
2. Nous sommes dans la fonction `Counter` où `count` est une constante `const count = 0`.
3. Au clic, `setCount(count + 1)` dit à React : "Pour le *prochain* rendu, la valeur sera 1".
4. La ligne suivante s'exécute : `console.log(count)`. Comme `count` vaut toujours 0 dans *cette* exécution de fonction, cela affiche 0.

#### B. La mise à jour déclenche un NOUVEAU rendu
Ce n'est qu'après l'exécution de votre fonction que React va :
1. Comparer l'ancienne et la nouvelle UI.
2. Mettre à jour le DOM.
3. Rappeler votre fonction `Counter` pour le rendu suivant.
4. Cette fois, `useState(0)` retournera `[1, ...]`.

---

## Capture des Valeurs (Closures) {#capture-des-valeurs}

### 1. Quoi
Puisque l'état est constant pour un rendu donné, toute fonction créée à l'intérieur de ce rendu (comme un `setTimeout` ou un gestionnaire d'événement) "capture" la valeur de l'état à cet instant précis. C'est le concept de **Fermeture (Closure)** en JavaScript.

### 2. Pourquoi
C'est une fonctionnalité, pas un bug ! Cela permet de garantir que les actions de l'utilisateur se basent sur ce qu'il voyait à l'écran au moment de l'interaction.

### 3. Comment
Imaginons un formulaire d'envoi de message avec un délai de 3 secondes.

```tsx
import { useState } from 'react';

export function ChatForm() {
  const [message, setMessage] = useState('');

  const handleSend = () => {
    setTimeout(() => {
      // ⚠️ Ici, 'message' est celui du moment du CLIC
      alert(`Message envoyé : ${message}`);
    }, 3000);
  };

  return (
    <div>
      <input
        value={message}
        onChange={e => setMessage(e.target.value)}
      />
      <button onClick={handleSend}>Envoyer</button>
    </div>
  );
}
```

**Scénario :**
1. Vous tapez "Bonjour".
2. Vous cliquez sur "Envoyer".
3. *Pendant les 3 secondes d'attente*, vous changez le texte en "Au revoir".

**Résultat :** L'alerte affichera "Bonjour".
React a capturé l'état ("Bonjour") au moment du clic. Le fait que vous ayez changé l'état ensuite (déclenchant un nouveau rendu où `message` vaut "Au revoir") n'affecte pas le `setTimeout` qui tourne dans le "passé" (le rendu précédent).

---

## Mises à jour Asynchrones et Batching {#mises-a-jour-asynchrones}

### 1. Quoi
Modifier l'état ne change pas la variable locale immédiatement. `setState` est une **demande** de mise à jour. React planifie cette mise à jour.

### 2. Pourquoi
Pour des raisons de performance, React fait du **Batching** (regroupement). Si vous avez 3 appels `setState` à la suite, React attendra la fin de votre gestionnaire d'événement pour ne faire qu'un seul re-rendu, au lieu de redessiner l'écran 3 fois.

### 3. Comment

#### A. Le problème des multiples mises à jour
Si vous essayez d'incrémenter 3 fois en utilisant la valeur actuelle...

```tsx
export function Scoreboard() {
  const [score, setScore] = useState(0);

  const incrementThreeTimes = () => {
    // Rendu initial : score = 0
    setScore(score + 1); // setScore(0 + 1)
    setScore(score + 1); // setScore(0 + 1)
    setScore(score + 1); // setScore(0 + 1)
  };

  return (
    <div>
      <p>Score : {score}</p>
      <button onClick={incrementThreeTimes}>+3</button>
    </div>
  );
}
```

**Résultat :** Le score passe à `1`, pas à `3`.
Chaque appel utilise la valeur `score` de ce rendu (snapshot), qui est `0`. React reçoit trois ordres : "Mets le score à 1", "Mets le score à 1", "Mets le score à 1".

#### B. La solution locale
Si vous avez besoin d'utiliser la "nouvelle" valeur immédiatement dans la même fonction, ne lisez pas l'état. Calculez-le dans une variable locale.

```tsx
const incrementThreeTimes = () => {
  const nextScore = score + 3; // Variable locale normale
  setScore(nextScore); // On met à jour l'état pour l'affichage futur
  
  // Si on a besoin de faire autre chose avec la nouvelle valeur :
  console.log(nextScore); // Affiche 3 correctement
};
```

*(Note : Pour résoudre le problème des mises à jour multiples basées sur l'état précédent de manière atomique, nous utiliserons les "Mises à jour fonctionnelles" au chapitre suivant).*

### 4. Zone de Danger

:::danger Ne trichez pas avec des variables globales
Il est tentant de déclarer une variable `let globalCount = 0` en dehors du composant pour contourner le système de Snapshot.
**Ne le faites pas.**
Cela brise l'isolation des composants (si vous affichez deux compteurs, ils partageront la même variable) et rend l'application impossible à tester et imprévisible.
:::

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-15}

1.  **Pourquoi `console.log(count)` affiche-t-il l'ancienne valeur juste après `setCount(count + 1)` ?**
    Parce que `count` est une constante ("snapshot") créée au début du rendu actuel. `setCount` ne modifie pas cette constante, mais demande à React de lancer un *nouveau* rendu avec une nouvelle valeur.

2.  **Si je clique sur un bouton qui déclenche un `setTimeout` de 5 secondes utilisant une variable d'état, et que je change cette variable entre temps, quelle valeur sera utilisée ?**
    La valeur au moment du clic (snapshot). La fonction du `setTimeout` a "capturé" (closure) les variables telles qu'elles étaient lors de sa création.

3.  **React met-il à jour le DOM immédiatement après un `setState` ?**
    Non. React attend généralement que tout le code de votre gestionnaire d'événement soit exécuté (batching) avant de calculer le nouveau rendu et de mettre à jour le DOM.

---

## Exercices : {#exercices-15}

### Exercice 1 - Le Feu Tricolore (Snapshot & Delay) {#exercice-1---le-feu-tricolore}

🎯 **Objectif** : Observer le phénomène de "Snapshot" avec des actions asynchrones.

💼 **Mise en situation** : Un contrôleur de trafic. Vous changez le feu au rouge, mais un message de prévention (lancé juste avant) doit indiquer la couleur *au moment de l'action*.

📝 **Énoncé** :
1. Créez un composant `TrafficLight`.
2. État `color` (string : "vert", "orange", "rouge"). Initialisé à "vert".
3. Un bouton "Changer de couleur" qui :
   - Lance une `alert` : "Attention, le feu est [color]".
   - *Ensuite*, change la couleur à la suivante (vert -> orange -> rouge -> vert).
4. Observez que l'alerte affiche la couleur *avant* le changement visuel.

📺 **Résultat attendu** :
Si le feu est vert : clic -> Alerte "Le feu est vert" -> Clic OK -> Le feu devient Orange.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState } from 'react';

export function TrafficLight() {
  const [color, setColor] = useState("vert");

  const handleNextLight = () => {
    // 1. Snapshot : 'color' vaut la valeur actuelle (ex: "vert")
    // L'alerte va capturer cette valeur
    alert(`Attention, le feu est actuellement : ${color}`);

    // 2. Mise à jour planifiée
    if (color === "vert") setColor("orange");
    else if (color === "orange") setColor("rouge");
    else setColor("vert");
    
    // Le rendu visuel ne changera qu'APRÈS la fermeture de l'alerte et la fin de la fonction
  };

  return (
    <div>
      <div style={{ 
        width: 50, 
        height: 50, 
        backgroundColor: color === "vert" ? "green" : (color === "orange" ? "orange" : "red"),
        borderRadius: "50%",
        marginBottom: 10
      }} />
      <button onClick={handleNextLight}>Actionner le feu</button>
    </div>
  );
}
```
</details>

### Exercice 2 - Le Panier Magique (Variables Locales) {#exercice-2---le-panier-magique}

🎯 **Objectif** : Utiliser une variable locale pour utiliser la "future" valeur immédiatement.

💼 **Mise en situation** : Un bouton "Acheter" qui ajoute un article au panier et affiche une confirmation avec le *nouveau* total d'articles.

📝 **Énoncé** :
1. État `cartCount` initialisé à 0.
2. Fonction `handleBuy` déclenchée par un bouton.
3. Dans cette fonction :
   - Calculez le nouveau total.
   - Mettez à jour l'état.
   - Lancez une `alert` disant "Merci ! Vous avez maintenant X articles".
4. Assurez-vous que X est le bon nombre (si je passe de 0 à 1, l'alerte doit dire 1).

📺 **Résultat attendu** :
Au premier clic, l'alerte dit "1 articles" (et non 0).

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState } from 'react';

export function ShoppingCart() {
  const [cartCount, setCartCount] = useState(0);

  const handleBuy = () => {
    // ❌ Mauvaise approche :
    // setCartCount(cartCount + 1);
    // alert(cartCount); // Affiche l'ancien nombre (snapshot)

    // ✅ Bonne approche : Pré-calculer
    const nextCount = cartCount + 1;
    
    // 1. Mise à jour de l'état
    setCartCount(nextCount);
    
    // 2. Utilisation immédiate de la variable locale
    alert(`Merci ! Vous avez maintenant ${nextCount} articles.`);
  };

  return (
    <button onClick={handleBuy}>
      Acheter (Panier: {cartCount})
    </button>
  );
}
```
</details>

### Exercice 3 - Le Bug du Select (Event Handler Snapshot) {#exercice-3---le-bug-du-select}

🎯 **Objectif** : Comprendre qu'un événement asynchrone utilise l'état du moment où il a été déclenché.

💼 **Mise en situation** : Un utilisateur choisit un destinataire dans une liste déroulante (`<select>`), puis clique sur "Envoyer plus tard (5s)". Pendant les 5 secondes, il change le destinataire dans le select. À qui le message est-il envoyé ?

📝 **Énoncé** :
1. État `recipient` ("Alice" ou "Bob").
2. Un `<select>` pour changer le destinataire.
3. Un bouton "Envoyer dans 5s".
4. Le bouton lance un `setTimeout`. Dans le timeout, affichez `alert("Envoyé à " + recipient)`.
5. Testez : Sélectionnez Alice -> Cliquez -> Changez vite pour Bob.
6. Qui reçoit le message dans l'alerte ? (Réponse attendue : Alice).

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Interface avec le Select sur "Bob" mais une Alerte qui dit "Envoyé à Alice".
> **Annotation** : Mettre en évidence la divergence entre l'UI actuelle et l'alerte.
> **Alt Text suggéré** : Démonstration de la capture de valeur : l'alerte affiche l'ancienne sélection malgré le changement d'option.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState } from 'react';

export function DelayedSender() {
  const [recipient, setRecipient] = useState("Alice");

  const handleSend = () => {
    setTimeout(() => {
      // Snapshot ! Ce 'recipient' est celui présent au moment du clic.
      // Même si l'utilisateur a changé la valeur entre temps, React a "figé" cette variable pour cette exécution.
      alert(`Envoyé à ${recipient}`);
    }, 5000);
  };

  return (
    <div style={{ border: '1px solid #ccc', padding: '20px' }}>
      <label>
        Destinataire :
        <select 
          value={recipient} 
          onChange={e => setRecipient(e.target.value)}
          style={{ marginLeft: '10px' }}
        >
          <option value="Alice">Alice</option>
          <option value="Bob">Bob</option>
        </select>
      </label>
      <br /><br />
      <button onClick={handleSend}>
        Envoyer dans 5 secondes
      </button>
      <p style={{ fontSize: '0.8em', color: '#666' }}>
        Test : Cliquez, puis changez immédiatement de personne.
      </p>
    </div>
  );
}
```
</details>
```