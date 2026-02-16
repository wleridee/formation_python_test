Voici le chapitre **Synchroniser avec les Effets: `useEffect`** pour la formation React 19.2.

```markdown
---
sidebar_label: Synchroniser avec les Effets: useEffect
sidebar_position: 22
---

# Chapitre 22 : Synchroniser avec les Effets: `useEffect`

Hook `useEffect`, Effets secondaires, Fonction de nettoyage, Cycle de vie des effets

Jusqu'à présent, vos composants étaient des fonctions "pures" : pour les mêmes entrées (props/state), ils retournaient le même JSX. Ils réagissaient uniquement aux actions de l'utilisateur (clics, frappe clavier).

Mais une application réelle doit interagir avec le monde extérieur :
- Se connecter à un serveur (API fetch).
- Écouter des événements globaux (redimensionnement de fenêtre).
- Démarrer des timers.
- Manipuler le DOM directement (intégration de widgets tiers).

En programmation fonctionnelle, on appelle ces interactions des **Effets Secondaires** (Side Effects). En React, le Hook pour gérer ces effets de bord est `useEffect`.

## Le Hook `useEffect` et les Effets Secondaires {#le-hook-useeffect-et-les-effets-secondaires}

### 1. Quoi
`useEffect` est un Hook qui permet d'exécuter du code arbitraire **après** que React a affiché le composant à l'écran. Il sert à synchroniser votre composant avec un système externe (navigateur, réseau, autre librairie).

### 2. Pourquoi
Le rendu d'un composant (le code principal de la fonction) ne doit jamais contenir d'effets secondaires.
❌ `fetch('/api/data')` dans le corps du composant lancerait une requête à *chaque* rendu (potentiellement des milliers).
✅ `useEffect` isole ce code pour qu'il s'exécute au bon moment, sans bloquer l'affichage de l'interface utilisateur.

### 3. Comment

#### A. Syntaxe de base

```tsx
import { useEffect } from 'react';

function MyComponent() {
  useEffect(() => {
    // Code de l'effet
    // S'exécute APRÈS le rendu et l'affichage dans le DOM
    console.log('Le composant est affiché/mis à jour');
  }); 

  return <div>Mon Composant</div>;
}
```

#### B. Cas concret : Synchroniser le titre du document
React gère le contenu de la page (le `body`), mais pas le `<title>` dans le `<head>`. C'est un système externe.

```tsx
import { useState, useEffect } from 'react';

export function DocumentTitleSync() {
  const [count, setCount] = useState(0);

  // Synchronisation : État React -> Titre du Navigateur
  useEffect(() => {
    document.title = `Compteur : ${count}`;
  });

  return (
    <button onClick={() => setCount(count + 1)}>
      Incrémenter (Voir l'onglet)
    </button>
  );
}
```

---

## Le Tableau de Dépendances {#le-tableau-de-dependances}

### 1. Quoi
Le deuxième argument de `useEffect` est un tableau de dépendances `[]`. Il indique à React **quand** ré-exécuter l'effet.

### 2. Pourquoi
Sans ce tableau, l'effet s'exécute après **chaque** rendu. C'est souvent inutile et coûteux (ex: relancer une requête API alors que les données n'ont pas changé).

### 3. Tableau Comparatif des Dépendances

| Syntaxe | Fréquence d'exécution | Analogie (Cycle de vie classe) |
| :--- | :--- | :--- |
| `useEffect(() => { ... })` | Après **chaque** rendu | `componentDidMount` + `componentDidUpdate` |
| `useEffect(() => { ... }, [])` | Une seule fois, au **montage** | `componentDidMount` |
| `useEffect(() => { ... }, [prop, state])` | Au montage + quand `prop` ou `state` **changent** | `componentDidUpdate` (conditionnel) |

### 4. Zone de Danger

:::danger Dépendances manquantes
Si vous utilisez une variable à l'intérieur de l'effet (ex: `userId`), elle **DOIT** figurer dans le tableau de dépendances.
Si vous mentez à React en omettant une dépendance, votre effet utilisera des valeurs périmées ("stale closures") et créera des bugs difficiles à tracer.

❌ **Incorrect :**
```tsx
useEffect(() => {
  console.log(userId); // Utilise userId
}, []); // 🤥 Mensonge ! userId n'est pas déclaré. L'effet ne se relancera pas si userId change.
```

✅ **Correct :**
```tsx
useEffect(() => {
  console.log(userId);
}, [userId]); // React relancera l'effet si userId change.
```
:::

---

## La Fonction de Nettoyage (Cleanup) {#la-fonction-de-nettoyage}

### 1. Quoi
Un effet peut retourner une fonction. C'est la fonction de nettoyage. React l'appelle avant d'exécuter l'effet une nouvelle fois, ou quand le composant est détruit (démonté).

### 2. Pourquoi
Certains effets doivent être nettoyés pour éviter des fuites de mémoire ou des bugs visuels :
- Arrêter un `setInterval`.
- Supprimer un écouteur d'événement (`removeEventListener`).
- Annuler une requête réseau active.

### 3. Comment

#### Pattern Connexion / Déconnexion

```tsx
import { useEffect, useState } from 'react';

function ChatRoom({ roomId }: { roomId: string }) {
  useEffect(() => {
    // 1. Setup : Connexion
    const connection = createConnection(roomId);
    connection.connect();

    // 2. Cleanup : Déconnexion
    // Cette fonction sera appelée avant que roomId change, ou quand le composant disparaît.
    return () => {
      connection.disconnect();
    };
  }, [roomId]); // Si roomId change, on déconnecte l'ancienne et on connecte la nouvelle

  return <h1>Bienvenue dans la salle {roomId}</h1>;
}
```

### 🚨 Limitations : React Strict Mode
En développement (Strict Mode activé), React **monte, démonte, et remonte** immédiatement votre composant pour vérifier que votre nettoyage fonctionne.
Vous verrez donc votre effet s'exécuter deux fois dans la console.
*   **C'est normal.**
*   Si cela casse votre application, c'est que votre fonction de nettoyage est manquante ou incorrecte.

---

## Cycle de Vie des Effets {#cycle-de-vie-des-effets}

Contrairement aux composants de classe qui pensent en termes de "Montage" et "Mise à jour", `useEffect` vous force à penser en termes de **Synchronisation**.

1.  **Synchronisation démarre** : L'état change, le rendu se fait.
2.  **Effet s'exécute** : React synchronise le système externe avec l'état actuel.
3.  **État change à nouveau** : L'utilisateur clique.
4.  **Nettoyage** : React nettoie la synchronisation précédente (avec les anciennes valeurs).
5.  **Effet s'exécute** : React synchronise avec les nouvelles valeurs.

### 4. Zone de Danger : Fetching de données

Bien que `useEffect` soit souvent utilisé pour fetcher des données, cette approche native a des défauts (Race conditions si deux requêtes partent vite, pas de cache, pas de déduplication).

En 2026, pour des applications de production, on préfère des bibliothèques comme **TanStack Query** ou **SWR**, ou les nouvelles fonctionnalités de React (`use` hook pour les Promises).
Cependant, comprendre le fetching avec `useEffect` reste un fondamental obligatoire.

```tsx
useEffect(() => {
  let ignore = false; // Flag pour éviter les race conditions

  async function startFetching() {
    const json = await fetchTodos(userId);
    if (!ignore) {
      setTodos(json);
    }
  }

  startFetching();

  return () => {
    ignore = true; // Cleanup : on ignore le résultat si le composant est démonté
  };
}, [userId]);
```

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-22}

1.  **Quand s'exécute le code à l'intérieur de `useEffect` ?**
    Il s'exécute **après** que le rendu du composant a été commité à l'écran (après l'affichage DOM).

2.  **Que signifie un tableau de dépendances vide `[]` ?**
    Cela signifie que l'effet ne dépend d'aucune valeur issue des props ou de l'état, et qu'il ne doit s'exécuter qu'une seule fois après le premier montage.

3.  **À quoi sert la fonction retournée par `useEffect` ?**
    C'est la fonction de nettoyage (cleanup). Elle sert à annuler des abonnements, timers ou écouteurs avant que l'effet ne soit relancé ou que le composant ne soit détruit.

4.  **Pourquoi mon `console.log` dans un `useEffect` apparaît-il deux fois en développement ?**
    C'est dû au "Strict Mode" de React qui simule un cycle montage/démontage/montage pour vérifier que votre logique de nettoyage est robuste.

---

## Exercices : {#exercices-22}

### Exercice 1 - Le Titre Dynamique (Dépendances) {#exercice-1---le-titre-dynamique}

🎯 **Objectif** : Comprendre quand l'effet se déclenche via le tableau de dépendances.

💼 **Mise en situation** : Application de messagerie. Vous voulez que l'onglet du navigateur affiche "X nouveaux messages" quand le compteur change.

📝 **Énoncé** :
1. Créez un état `notifications` (nombre, initialisé à 0).
2. Deux boutons : "Recevoir message" (+1) et "Tout lire" (remise à 0).
3. Utilisez `useEffect` pour mettre à jour `document.title` avec le texte "X nouveaux messages".
4. Attention : Si le nombre est 0, le titre doit être "Messagerie" (pas "0 nouveaux messages").

📺 **Résultat attendu** :
Le titre de l'onglet du navigateur change quand on clique.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState, useEffect } from 'react';

export function TitleManager() {
  const [notifications, setNotifications] = useState(0);

  useEffect(() => {
    // Logique de synchronisation
    if (notifications > 0) {
      document.title = `${notifications} nouveaux messages`;
    } else {
      document.title = "Messagerie";
    }
    
    // Pas de cleanup nécessaire ici car écraser document.title est suffisant
  }, [notifications]); // Se déclenche uniquement quand 'notifications' change

  return (
    <div style={{ padding: 20 }}>
      <h3>Notifications : {notifications}</h3>
      <button onClick={() => setNotifications(n => n + 1)}>
        📩 Recevoir message
      </button>
      <button onClick={() => setNotifications(0)} style={{ marginLeft: 10 }}>
        ✅ Tout lire
      </button>
    </div>
  );
}
```
</details>

### Exercice 2 - Le Tracker de Souris (Cleanup) {#exercice-2---le-tracker-de-souris}

🎯 **Objectif** : Gérer les événements globaux et le nettoyage.

💼 **Mise en situation** : Un outil de débogage pour designers qui affiche la position de la souris (X, Y) en pixels. Il est crucial de ne pas laisser traîner l'écouteur d'événement quand l'outil est fermé.

📝 **Énoncé** :
1. Créez un composant `MouseTracker`.
2. Dans ce composant, utilisez `useEffect` pour ajouter un écouteur `mousemove` sur `window`.
3. Stockez la position `{x, y}` dans un state et affichez-la.
4. **IMPORTANT** : Retournez une fonction de nettoyage pour retirer l'écouteur (`removeEventListener`).
5. Dans le composant parent, ajoutez une checkbox pour monter/démonter (afficher/cacher) le `MouseTracker`.
6. Vérifiez avec `console.log` dans l'effet que l'événement s'arrête bien quand on cache le composant.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Texte affichant "X: 150, Y: 300" et une checkbox "Activer le tracker".
> **Annotation** : Montrez les coordonnées qui changent.
> **Alt Text suggéré** : Interface de débogage affichant les coordonnées de la souris en temps réel.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState, useEffect } from 'react';

function MouseTracker() {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  useEffect(() => {
    // Fonction handler
    const handleMove = (e: MouseEvent) => {
      console.log('Mouse move detected'); // Pour vérifier le nettoyage
      setPosition({ x: e.clientX, y: e.clientY });
    };

    // 1. Setup
    window.addEventListener('mousemove', handleMove);

    // 2. Cleanup (OBLIGATOIRE)
    return () => {
      console.log('Cleanup : suppression du listener');
      window.removeEventListener('mousemove', handleMove);
    };
  }, []); // [] = on attache l'événement une seule fois au montage

  return (
    <div style={{ 
      position: 'fixed', 
      bottom: 20, 
      right: 20, 
      background: 'black', 
      color: 'white', 
      padding: 10,
      borderRadius: 5
    }}>
      X: {position.x}, Y: {position.y}
    </div>
  );
}

export function DebugTool() {
  const [enabled, setEnabled] = useState(false);

  return (
    <div>
      <label>
        <input 
          type="checkbox" 
          checked={enabled} 
          onChange={e => setEnabled(e.target.checked)} 
        />
        Activer le tracker de souris
      </label>
      
      {/* Le composant est monté/démonté selon la case */}
      {enabled && <MouseTracker />}
    </div>
  );
}
```
</details>

### Exercice 3 - Le Compte à Rebours (Timer) {#exercice-3---le-compte-a-rebours}

🎯 **Objectif** : Synchroniser avec `setInterval`.

💼 **Mise en situation** : Une page de vente flash ("Offre expire dans...").

📝 **Énoncé** :
1. Créez un composant `Countdown` qui prend une prop `startValue` (ex: 10).
2. Initialisez l'état du compteur avec cette prop.
3. Utilisez `useEffect` pour décrémenter le compteur chaque seconde (`setInterval`).
4. Quand le compteur atteint 0, arrêtez le décompte (ou nettoyez l'intervalle).
5. Assurez-vous que le timer est bien nettoyé si le composant est démonté avant la fin.

📺 **Résultat attendu** :
Un chiffre qui descend chaque seconde.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState, useEffect } from 'react';

export function Countdown({ startValue }: { startValue: number }) {
  const [timeLeft, setTimeLeft] = useState(startValue);

  useEffect(() => {
    // Si on est déjà à 0, on ne fait rien
    if (timeLeft <= 0) return;

    // 1. Setup du timer
    const intervalId = setInterval(() => {
      setTimeLeft((prevTime) => prevTime - 1);
    }, 1000);

    // 2. Cleanup : crucial pour éviter que le timer continue en arrière-plan
    // ou qu'il essaie de mettre à jour l'état d'un composant démonté.
    return () => {
      clearInterval(intervalId);
    };
  }, [timeLeft]); 
  // On dépend de timeLeft pour vérifier la condition d'arrêt <= 0 au début de l'effet.
  // Note : Une autre approche optimisée consisterait à ne pas dépendre de timeLeft 
  // et gérer l'arrêt à l'intérieur du setTimeLeft, mais celle-ci est plus lisible pour débuter.

  return (
    <div style={{ fontSize: '2rem', fontWeight: 'bold', color: timeLeft > 3 ? 'black' : 'red' }}>
      {timeLeft > 0 ? `Il reste ${timeLeft}s` : "OFFRE TERMINÉE !"}
    </div>
  );
}
```
</details>
```