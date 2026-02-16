Voici le chapitre **`useEffectEvent`: Optimiser les Effets (Canary)** pour la formation React 19.2.

```markdown
---
sidebar_label: `useEffectEvent`: Optimiser les Effets (Canary)
sidebar_position: 24
---

# Chapitre 24 : `useEffectEvent`: Optimiser les Effets (Canary)

Séparer les événements des effets, Performances des effets, Patterns d'effets avancés

:::warning Fonctionnalité Canary (Expérimentale)
Ce chapitre traite d'une API expérimentale (`experimental_useEffectEvent`) disponible dans les versions Canary et React 19.
Elle résout un problème fondamental de React : comment lire une valeur dans un effet sans que cette valeur ne déclenche la ré-exécution de l'effet ?
:::

Dans le chapitre précédent, nous avons appris que **toute valeur réactive** utilisée dans un effet doit figurer dans son tableau de dépendances. C'est la règle d'or.
Cependant, cette règle peut parfois causer des problèmes de performance ou de comportement : vous voulez utiliser une variable, mais vous ne voulez pas qu'elle *déclenche* l'effet.

Jusqu'à récemment, nous utilisions des astuces complexes avec `useRef`. React introduit désormais une solution élégante : les **Événements d'Effet**.

## Séparer le "Déclencheur" de la "Donnée" {#separer-le-declencheur-de-la-donnee}

### 1. Quoi
`useEffectEvent` est un Hook qui permet d'extraire une partie de la logique de votre effet dans une fonction spéciale.
Cette fonction peut lire les props et le state les plus récents, **mais elle n'est pas réactive**. Elle ne doit pas être ajoutée au tableau de dépendances de l'effet.

### 2. Pourquoi
Imaginez une salle de chat. Vous devez vous connecter quand `roomId` change. Lors de la connexion, vous voulez envoyer un log analytique incluant le `theme` actuel (clair/sombre).
*   **Problème** : Si vous mettez `theme` dans les dépendances, le chat se déconnecte et reconnecte à chaque fois que l'utilisateur change de thème. C'est mauvais.
*   **Solution** : `roomId` est le déclencheur (réactif). `theme` est une donnée (non-réactive pour cet effet).

### 3. Comment

#### A. Le Problème (Re-connexion inutile)

```tsx
function ChatRoom({ roomId, theme }) {
  useEffect(() => {
    const connection = createConnection(roomId);
    connection.connect();
    
    // 😱 PROBLÈME : Utiliser 'theme' ici oblige à l'ajouter aux dépendances.
    // Résultat : changer de thème déconnecte le chat !
    logAnalytics(`Connected to ${roomId} with theme ${theme}`);

    return () => connection.disconnect();
  }, [roomId, theme]); 
}
```

#### B. La Solution avec `useEffectEvent`

L'import est souvent préfixé car expérimental :
`import { experimental_useEffectEvent as useEffectEvent } from 'react';`

```tsx
import { useState, useEffect, experimental_useEffectEvent as useEffectEvent } from 'react';

function ChatRoom({ roomId, theme }: { roomId: string, theme: string }) {
  // 1. On encapsule la logique qui lit les données "fraîches" mais non-déclencheuses
  const onConnected = useEffectEvent(() => {
    // Ici, on a accès à la dernière version de 'theme' et 'roomId'
    console.log(`✅ Connexion à ${roomId} (Thème: ${theme})`);
  });

  useEffect(() => {
    const connection = createConnection(roomId);
    connection.connect();
    
    // 2. On appelle l'événement d'effet
    onConnected();

    return () => connection.disconnect();
    // 3. ✨ MAGIE : 'onConnected' n'a PAS besoin d'être dans les dépendances
    // L'effet ne dépend QUE de 'roomId'.
  }, [roomId]); 

  return <h1>Salon {roomId}</h1>;
}
```

---

## Règles d'Utilisation {#regles-d-utilisation}

### 1. Quoi
`useEffectEvent` retourne une fonction qui est "stable" (son identité ne change jamais), mais qui a accès à la portée (scope) du dernier rendu.

### 2. Pourquoi
React impose des restrictions strictes pour garantir que ce mécanisme ne brise pas la pureté des composants.

### 3. Zone de Danger : ❌ À ne pas faire

:::danger Restrictions Strictes
1.  **Uniquement dans les Effets** : Vous ne pouvez appeler la fonction retournée par `useEffectEvent` QUE depuis un `useEffect`. Ne l'appelez pas pendant le rendu.
2.  **Pas de passage en Props** : Ne passez jamais cette fonction à un composant enfant ou à un autre Hook personnalisé. Elle est "liée" à l'effet local.

**Pourquoi ?** Parce que React peut décider de ne pas mettre à jour cette fonction tant que l'effet ne s'exécute pas. La passer ailleurs créerait des bugs imprévisibles.
:::

#### Exemple Incorrect
```tsx
function BadComponent() {
  const onEvent = useEffectEvent(() => console.log('Hello'));

  // ❌ INTERDIT : Passer à un enfant
  return <Child onSomething={onEvent} />;
}
```

---

## Pattern : Logger sans Ré-exécuter {#pattern-logger-sans-re-executer}

### 1. Quoi
C'est le cas d'utilisation le plus courant : l'analytique et le logging. Vous voulez tracker "Quand X change, loggue aussi la valeur de Y".

### 2. Comment

```tsx
import { useEffect, experimental_useEffectEvent as useEffectEvent } from 'react';

function PageTracker({ path, userId }: { path: string, userId: string }) {
  // On veut déclencher l'effet quand 'path' change (navigation).
  // Mais on a besoin de 'userId' pour le log.
  // Si 'userId' change (ex: update du profil), on ne veut PAS relogger la page.

  const logPageView = useEffectEvent((currentPath) => {
    console.log(`User ${userId} visited ${currentPath}`);
  });

  useEffect(() => {
    // Seul 'path' déclenche l'effet
    logPageView(path);
  }, [path]); 
  
  return null;
}
```

### 3. Tableau Comparatif

| Approche | Code | Conséquence |
| :--- | :--- | :--- |
| **Naïve** | `useEffect(() => log(user), [path, user])` | Log en double si `user` change ❌ |
| **Triche (Linter disable)** | `useEffect(() => log(user), [path])` | Logue un vieil `user` (stale closure) ❌ |
| **Ref Hack** | `useRef(user)` + `useEffect` | Fonctionne mais verbeux et complexe ⚠️ |
| **`useEffectEvent`** | `const log = useEffectEvent(...)` | Code propre, données fraîches, exécution minimale ✅ |

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-24}

1.  **Quel problème `useEffectEvent` résout-il principalement ?**
    Il permet d'utiliser des valeurs réactives (props/state) à l'intérieur d'un effet sans avoir à les ajouter au tableau de dépendances, évitant ainsi des ré-exécutions inutiles.

2.  **Peut-on passer une fonction créée par `useEffectEvent` en prop à un enfant ?**
    **Non**, jamais. Elle doit être utilisée uniquement à l'intérieur des `useEffect` du même composant.

3.  **Pourquoi ne peut-on pas simplement supprimer la dépendance du tableau `[]` sans utiliser ce Hook ?**
    Parce que les "fermetures lexicales" (closures) en JavaScript captureraient l'ancienne valeur de la variable. L'effet utiliserait des données périmées. `useEffectEvent` garantit l'accès à la valeur la plus récente.

---

## Exercices : {#exercices-24}

> Note : Pour ces exercices, assurez-vous d'utiliser une version de React compatible (Canary ou Next.js récent) ou simulez le comportement mentalement si l'environnement n'est pas prêt.

### Exercice 1 - Le Mouchard de Panier {#exercice-1---le-mouchard-de-panier}

🎯 **Objectif** : Logger le contenu du panier seulement quand l'utilisateur change de page.

💼 **Mise en situation** : Site E-commerce. Vous voulez savoir ce que les utilisateurs ont dans leur panier (`cartTotal`) au moment exact où ils changent de catégorie de produits (`categoryId`). Si le total du panier change, cela ne doit pas déclencher de log.

📝 **Énoncé** :
1. Props : `categoryId` (string), `cartTotal` (number).
2. Utilisez `useEffect` pour détecter le changement de `categoryId`.
3. Utilisez `useEffectEvent` pour lire `cartTotal` et faire un `console.log`.
4. Testez : Changez le `cartTotal` -> Rien ne se passe. Changez `categoryId` -> Le log affiche le nouveau total.

📺 **Résultat attendu** :
Le console log n'apparaît que lors du changement de catégorie, mais affiche bien le prix à jour.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useEffect, experimental_useEffectEvent as useEffectEvent } from 'react';

function AnalyticsTracker({ categoryId, cartTotal }: { categoryId: string, cartTotal: number }) {
  
  // Cette fonction "voit" toujours le dernier cartTotal
  // mais elle est stable et ne déclenchera jamais rien.
  const logCategoryVisit = useEffectEvent((cat: string) => {
    console.log(`Visite de ${cat}. Panier actuel : ${cartTotal}€`);
  });

  useEffect(() => {
    // Seul categoryId est une dépendance réactive ici
    logCategoryVisit(categoryId);
  }, [categoryId]);

  return <div>Tracker actif pour {categoryId}</div>;
}
```
</details>

### Exercice 2 - Notification Silencieuse {#exercice-2---notification-silencieuse}

🎯 **Objectif** : Gérer une notification qui dépend du mode "Ne pas déranger".

💼 **Mise en situation** : Une application de messagerie. Quand un nouveau message arrive (`lastMessage`), on doit jouer un son. MAIS, si le mode `isDoNotDisturb` est activé, on ne doit pas jouer de son.
*Piège* : Activer ou désactiver le mode "Ne pas déranger" ne doit pas rejouer le son du dernier message reçu.

📝 **Énoncé** :
1. Props : `lastMessage` (objet), `isDoNotDisturb` (boolean).
2. Effet déclenché par `lastMessage`.
3. Logique conditionnelle utilisant `isDoNotDisturb` encapsulée dans un `useEffectEvent`.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useEffect, experimental_useEffectEvent as useEffectEvent } from 'react';

interface Message { id: number; text: string }

function MessageToaster({ lastMessage, isDoNotDisturb }: { lastMessage: Message, isDoNotDisturb: boolean }) {
  
  const playSoundIfNeeded = useEffectEvent(() => {
    if (isDoNotDisturb) {
      console.log('🔕 Mode silence : chut !');
    } else {
      console.log('🔔 DING ! Nouveau message');
    }
  });

  useEffect(() => {
    // On ne joue le son que si le message change.
    // Le changement du mode DND n'a aucun effet ici.
    playSoundIfNeeded();
  }, [lastMessage]);

  return (
    <div style={{ border: '1px solid gray', padding: 10, margin: 10 }}>
      Dernier message : {lastMessage.text}
      <br/>
      Mode Silence : {isDoNotDisturb ? 'ON' : 'OFF'}
    </div>
  );
}
```
</details>

### Exercice 3 - Le Timer de Session Sécurisé {#exercice-3---le-timer-de-session-securise}

🎯 **Objectif** : Corriger un bug de timer qui redémarre tout le temps.

💼 **Mise en situation** : Un timer de sécurité déconnecte l'utilisateur après 30 secondes d'inactivité. Si l'utilisateur bouge la souris, on reset le timer.
Lorsque le timer arrive à zéro, on affiche une alerte avec le nom de l'utilisateur (`userName`).
*Problème actuel* : Si `userName` change (ex: correction d'une typo), le timer se reset alors qu'il ne devrait pas.

📝 **Énoncé** :
1. Créez un composant `SessionTimer` avec prop `userName`.
2. Utilisez `useEffect` avec `setTimeout` pour déclencher une action après 5 secondes (pour tester vite).
3. L'action doit loguer : "Session expirée pour [userName]".
4. Faites en sorte que changer `userName` ne redémarre pas le compte à rebours.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState, useEffect, experimental_useEffectEvent as useEffectEvent } from 'react';

export function SessionTimer({ userName }: { userName: string }) {
  const [timeLeft, setTimeLeft] = useState(5);

  // Événement d'effet : Lit userName sans être une dépendance
  const onExpire = useEffectEvent(() => {
    console.log(`🛑 Session expirée pour l'utilisateur : ${userName}`);
    alert(`Bye bye ${userName} !`);
  });

  useEffect(() => {
    // Le timer décrémente chaque seconde
    const id = setInterval(() => {
      setTimeLeft((t) => {
        if (t <= 1) {
          clearInterval(id);
          onExpire(); // Appel de la fonction stable
          return 0;
        }
        return t - 1;
      });
    }, 1000);

    return () => clearInterval(id);
    // Le tableau de dépendances est VIDE [].
    // Le timer ne redémarre jamais, même si userName change.
  }, []); 

  return <h3>Temps restant : {timeLeft}s (Utilisateur: {userName})</h3>;
}
```
</details>
```