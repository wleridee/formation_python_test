Voici le chapitre **API React Avancées (Expérimentales/Canary)** pour la formation React 19.2.

```markdown
---
sidebar_label: API React Avancées (Expérimentales/Canary)
sidebar_position: 43
---

# Chapitre 43 : API React Avancées (Expérimentales/Canary)

`addTransitionType`, `cache`, `cacheSignal`, `captureOwnerStack`, `experimental_taintObjectReference`, `experimental_taintUniqueValue`

:::warning Fonctionnalités Avancées et Instables
Les APIs présentées dans ce chapitre font partie des versions **Canary** ou **Expérimentales** de React 19. Elles sont principalement destinées aux auteurs de frameworks (Next.js, Remix) ou de bibliothèques. Leur syntaxe et leur comportement peuvent changer. Utilisez-les avec précaution en production.
:::

Ce chapitre explore les entrailles de React, là où la magie de la performance, de la sécurité et du débogage opère.

---

## 1. `cache` et `cacheSignal` {#cache-et-cachesignal}

### 1. Quoi
*   **`cache`** : Une fonction qui permet de mémoïser le résultat d'une fonction asynchrone (souvent un fetch de données) pour la durée d'une requête serveur.
*   **`cacheSignal`** : Un signal (similaire à `AbortSignal`) qui permet de contrôler l'invalidation ou l'annulation de ces caches.

### 2. Pourquoi
Dans les Server Components, on a souvent besoin de la même donnée à plusieurs endroits (ex: l'utilisateur courant dans le Header, dans la Sidebar et dans le Main).
Sans `cache`, on ferait 3 requêtes à la base de données.
Avec `cache`, React dé-duplique automatiquement ces appels. C'est la version serveur de `useMemo` mais pour les requêtes.

### 3. Comment

#### A. Syntaxe de base

```tsx
import { cache } from 'react';
import db from './db';

// Création d'une version mémoïsée de la fonction
export const getUser = cache(async (id: string) => {
  console.log('Fetching user from DB...'); // Ne s'affichera qu'une fois par requête
  return await db.user.findUnique({ where: { id } });
});
```

#### B. Utilisation dans les composants

```tsx
import { getUser } from './data';

async function Header({ userId }: { userId: string }) {
  const user = await getUser(userId); // Premier appel : Fetch réel
  return <header>Bienvenue {user.name}</header>;
}

async function Sidebar({ userId }: { userId: string }) {
  const user = await getUser(userId); // Second appel : Retourne le résultat en cache
  return <nav>Profil de {user.name}</nav>;
}
```

### 🚨 Limitations de `cache`
*   Fonctionne uniquement dans les **Server Components**.
*   Le cache ne vit que le temps d'une requête HTTP (Request Scope). Il ne persiste pas entre deux visiteurs.

---

## 2. Taint APIs : `experimental_taintObjectReference` et `experimental_taintUniqueValue` {#taint-apis}

### 1. Quoi
Les APIs de "Tainting" (marquage) permettent de **sécuriser** les données sensibles (clés API, mots de passe, tokens) pour empêcher qu'elles ne soient accidentellement envoyées au client (le navigateur).

*   `taintObjectReference` : Empêche un objet entier d'être sérialisé vers le client.
*   `taintUniqueValue` : Empêche une valeur spécifique (string, number) d'être passée.

### 2. Pourquoi
Avec les Server Actions et Server Components, il est facile de passer un objet `User` complet au client : `<ClientComponent user={user} />`.
Si cet objet contient `user.hashedPassword` ou `user.stripeToken`, c'est une faille de sécurité critique. Le Tainting agit comme un pare-feu : si React détecte une donnée "teintée" qui part vers le client, il lève une erreur bloquante.

### 3. Comment

#### A. Protection d'un objet (ex: User)

```tsx
import { experimental_taintObjectReference } from 'react';

export async function getUser(id: string) {
  const user = await db.user.findUnique({ where: { id } });
  
  // Si on essaie de passer `user` à un Client Component, React lancera une erreur
  experimental_taintObjectReference(
    "Ne pas passer l'objet User complet au client : il contient des données privées.",
    user
  );
  
  return user;
}
```

#### B. Protection d'un token (ex: API Key)

```tsx
import { experimental_taintUniqueValue } from 'react';

const API_KEY = process.env.SECRET_API_KEY;

experimental_taintUniqueValue(
  "Ne pas exposer la clé API secrète au client !",
  process.env, // L'objet où vit la valeur (lifetime)
  API_KEY      // La valeur à protéger
);
```

### 4. Zone de Danger
❌ **À ne pas faire** : Penser que le tainting chiffre les données.
✅ **Bonne Pratique** : Utilisez le tainting comme une ceinture de sécurité ("Defense in Depth"). Ne comptez pas uniquement dessus, filtrez vos données (DTOs) manuellement aussi.

---

## 3. `captureOwnerStack` (DevTools) {#captureownerstack}

### 1. Quoi
Une API permettant de capturer la pile d'appels (stack trace) du composant "propriétaire" (celui qui a créé l'élément actuel).

### 2. Pourquoi
Pour le débogage avancé. Quand une erreur survient dans un composant enfant générique (ex: un `<Button>`), on veut savoir quel composant parent l'a rendu. `captureOwnerStack` permet aux outils (comme React DevTools ou Sentry) de reconstruire l'arbre des composants au moment de l'erreur.

### 3. Comment (Usage interne/avancé)

```tsx
// Principalement utilisé par les outils de monitoring d'erreurs
if (process.env.NODE_ENV === 'development') {
  const stack = React.captureOwnerStack ? React.captureOwnerStack() : '';
  console.log("Composant parent :", stack);
}
```

---

## 4. `addTransitionType` (Transitions nommées) {#addtransitiontype}

### 1. Quoi
Permet d'ajouter des métadonnées (comme un type ou un nom) à une transition React initiée par `startTransition`.

### 2. Pourquoi
Pour que les outils de performance ou de logging puissent distinguer différentes transitions concurrentes.
"Est-ce que l'utilisateur attend la transition 'Navigation' ou la transition 'Like' ?"

### 3. Comment

```tsx
import { startTransition, unstable_addTransitionType } from 'react';

function handleNavigation() {
  startTransition(() => {
    unstable_addTransitionType('navigation'); // Tag la transition
    navigateTo('/profile');
  });
}
```

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-43}

1.  **Quelle est la différence majeure entre `cache` de React et un cache HTTP classique ?**
    `cache` de React est limité à la durée de vie d'une seule requête serveur (Request Scope) pour dé-dupliquer les appels de fonction, tandis que le cache HTTP peut durer des jours et est partagé entre utilisateurs.

2.  **Que se passe-t-il si j'essaie d'afficher une variable protégée par `experimental_taintUniqueValue` dans un `<div>` côté client ?**
    React intercepte la tentative de sérialisation, bloque le rendu et lance une erreur critique avec le message de sécurité que vous avez défini, empêchant la fuite de donnée.

3.  **Pourquoi `captureOwnerStack` est-il utile pour les outils de monitoring ?**
    Il permet de fournir un contexte riche (quel composant a instancié celui qui a planté ?) au lieu d'une simple stack trace JavaScript qui ne reflète pas la hiérarchie des composants React.

---

## Exercices : {#exercices-43}

### Exercice 1 - Le Cache Intelligent (Serveur) {#exercice-1---le-cache-intelligent}

🎯 **Objectif** : Optimiser les appels API simulés dans un environnement Server Component.

💼 **Mise en situation** : Vous construisez une page "Profil" qui affiche les infos utilisateur, ses dernières commandes et ses favoris. Ces 3 sections ont besoin de récupérer l'objet `User`.

📝 **Énoncé** :
1. Créez une fonction `fetchUser(id)` qui loggue "Appel API..." et retourne un objet utilisateur après 500ms.
2. Enveloppez-la avec `cache`.
3. Appelez cette fonction 3 fois de suite dans une simulation de rendu (fonction `Page`).
4. Vérifiez que "Appel API..." n'apparaît qu'une seule fois dans la console.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { cache } from 'react';

// Simulation BDD
const db = {
  user: { id: '1', name: 'Alice' }
};

// 1. & 2. Création de la fonction cachée
const getUser = cache(async (id: string) => {
  console.log(`🔥 APPEL API RÉEL pour ${id}`); // Preuve de l'appel
  await new Promise(r => setTimeout(r, 100));
  return db.user;
});

// 3. Simulation de composants
async function UserInfo({ id }: { id: string }) {
  const user = await getUser(id);
  return <div>Info: {user.name}</div>;
}

async function UserOrders({ id }: { id: string }) {
  const user = await getUser(id);
  return <div>Commandes de {user.name}</div>;
}

// Composant Page (Racine)
export async function Page() {
  const userId = '1';
  console.log("--- Début du Rendu ---");
  
  // Les composants sont des promesses en Server Components
  const p1 = UserInfo({ id: userId });
  const p2 = UserOrders({ id: userId });
  
  // On attend tout
  await Promise.all([p1, p2]);
  
  console.log("--- Fin du Rendu ---");
  return "Rendu terminé (voir console pour les logs)";
}

// Pour tester cet exercice, il faudrait un environnement RSC (Next.js/Waku).
// En JS pur, `cache` ne fonctionne pas comme attendu sans le contexte React Server.
```
</details>

### Exercice 2 - La Ceinture de Sécurité (Tainting) {#exercice-2---la-ceinture-de-securite}

🎯 **Objectif** : Protéger un "Secret" contre l'affichage accidentel.

💼 **Mise en situation** : Vous manipulez des objets `PaymentMethod` qui contiennent le numéro complet de carte de crédit (PAN). Ce champ ne doit JAMAIS arriver au client.

📝 **Énoncé** :
1. Créez un objet `creditCard = { pan: "1234-5678", last4: "5678" }`.
2. Utilisez `experimental_taintUniqueValue` sur le champ `pan`.
3. Essayez de passer cet objet à un composant Client simulé (ou affichez-le).
4. Observez l'erreur (si vous êtes dans un environnement compatible React Canary).

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { experimental_taintUniqueValue } from 'react';

function getPaymentMethod() {
  const creditCard = {
    pan: "4242-4242-4242-4242", // Secret !
    last4: "4242",
    brand: "Visa"
  };

  // Protection du PAN
  experimental_taintUniqueValue(
    "🚨 ALERTE SÉCURITÉ : Tentative d'envoi du numéro de carte complet au client !",
    creditCard, // Lifetime object
    creditCard.pan // Valeur à protéger
  );

  return creditCard;
}

// Simulation d'un composant Client qui essaie d'afficher le PAN
export function ClientCardDisplay({ card }: { card: any }) {
  return (
    <div>
      <p>Marque : {card.brand}</p>
      {/* 
        Ceci provoquerait une ERREUR BLOQUANTE côté serveur lors de la sérialisation
        React refuserait d'envoyer le HTML ou le JSON contenant le PAN.
      */}
      <p>Numéro : {card.pan}</p> 
    </div>
  );
}
```
</details>

### Exercice 3 - Stack Trace Améliorée (`captureOwnerStack`) {#exercice-3---stack-trace-amelioree}

🎯 **Objectif** : Comprendre l'utilité de la capture de stack.

💼 **Mise en situation** : Vous créez un logger d'erreurs maison pour votre Design System.

📝 **Énoncé** :
1. Créez un composant `Button` qui lance une erreur volontaire (`throw new Error`).
2. Dans un `ErrorBoundary` (ou un `try/catch` simulé), utilisez `React.captureOwnerStack` pour voir qui a appelé ce bouton.
3. Comparez avec `error.stack` standard.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import React from 'react';

function BuggyButton() {
  // Capture AVANT l'erreur pour l'exemple, normalement fait dans un outil externe
  const ownerStack = (React as any).captureOwnerStack ? (React as any).captureOwnerStack() : 'Non supporté';
  
  console.log("📸 Stack React (Owner) :", ownerStack);
  console.log("📄 Stack JS standard :", new Error().stack);

  return <button>Click me</button>;
}

export function App() {
  return (
    <div className="layout">
      <header>
        <BuggyButton />
      </header>
    </div>
  );
}
```
</details>
```