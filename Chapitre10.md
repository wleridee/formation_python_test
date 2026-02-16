Voici le chapitre **Rendu Conditionnel** pour la formation React 19.2.

```markdown
---
sidebar_label: Rendu Conditionnel
sidebar_position: 10
---

# Chapitre 10 : Rendu Conditionnel

`if...else`, Opérateur ternaire `? :`, Opérateur logique `&&`

En React, il n'y a pas de directives spéciales comme `v-if` (Vue) ou `*ngIf` (Angular). React reste fidèle au JavaScript pur. Si vous savez écrire une condition en JavaScript, vous savez faire du rendu conditionnel en React.
Ce chapitre explore les différentes manières d'afficher (ou de masquer) des éléments selon l'état de votre application.

## `if...else` {#if-else}

### 1. Quoi
L'instruction classique JavaScript `if` et `else`. Elle permet d'exécuter des blocs de code différents selon une condition. En React, on l'utilise souvent pour **sortir prématurément** d'un composant (Early Return) ou pour préparer des variables avant le `return`.

### 2. Pourquoi
C'est la méthode la plus lisible lorsque vous devez afficher des blocs complètement différents (ex: une page de chargement vs le contenu réel) ou lorsque la logique est complexe.

### 3. Comment

#### A. Syntaxe : Le "Early Return"
Si une condition critique n'est pas remplie (données pas prêtes, erreur, utilisateur non connecté), on retourne un autre morceau de JSX immédiatement, ce qui stoppe l'exécution de la fonction.

```tsx
type UserProfileProps = {
  isLoading: boolean;
  username?: string;
};

export function UserProfile({ isLoading, username }: UserProfileProps) {
  // 1. Condition d'arrêt
  if (isLoading) {
    return <div>Chargement du profil...</div>;
  }

  // 2. Le rendu principal (si isLoading est faux)
  return (
    <div className="profile">
      <h1>Bienvenue, {username}</h1>
    </div>
  );
}
```

#### B. Cas concret : Assignation de variable
Parfois, on ne veut pas retourner tout de suite, mais juste varier une partie du contenu.

```tsx
export function LoginStatus({ isLoggedIn }: { isLoggedIn: boolean }) {
  let button;

  if (isLoggedIn) {
    button = <button>Se déconnecter</button>;
  } else {
    button = <button>Se connecter</button>;
  }

  return (
    <nav>
      <Logo />
      {button}
    </nav>
  );
}
```

### 4. Zone de Danger

:::danger Pas de `if` dans le JSX
Rappel du Chapitre 5 : Vous ne pouvez pas utiliser d'instructions (`if`, `for`, `switch`) **à l'intérieur** des accolades `{ ... }` du JSX.
JSX n'accepte que des **expressions** (qui retournent une valeur).

❌ **Interdit :**
```tsx
return <div>{ if (isLoggedIn) { ... } }</div>
```

✅ **Alternative :** Préparez le résultat du `if` dans une variable avant le `return`, ou utilisez l'opérateur ternaire.
:::

---

## Opérateur ternaire `? :` {#operateur-ternaire}

### 1. Quoi
L'opérateur ternaire est la version "expression" du `if...else`. Il prend trois opérandes : `condition ? valeurSiVrai : valeurSiFaux`.

### 2. Pourquoi
C'est l'outil standard pour gérer des conditions **binaires** (A ou B) directement à l'intérieur du JSX. Il est concis et maintient le flux de lecture du template.

### 3. Comment

#### A. Syntaxe de base

```tsx
return (
  <div className="container">
    {isEditMode ? (
      <input type="text" />  // Si Vrai
    ) : (
      <span>Lecture seule</span> // Si Faux
    )}
  </div>
);
```

#### B. Cas concret : Affichage dynamique de classe
Le ternaire est excellent pour le rendu conditionnel, mais aussi pour les valeurs d'attributs.

```tsx
type AlertProps = {
  isError: boolean;
  message: string;
};

export function Alert({ isError, message }: AlertProps) {
  return (
    <div 
      className={isError ? "bg-red-500 text-white" : "bg-blue-500 text-white"}
      role="alert"
    >
      {/* Imbrication possible mais à limiter pour la lisibilité */}
      <strong>{isError ? "Erreur :" : "Info :"}</strong> {message}
    </div>
  );
}
```

### 🚨 Limitations de l'Opérateur Ternaire
Évitez d'imbriquer des ternaires dans des ternaires (Nested Ternaries). Cela rend le code illisible très rapidement.
Si vous avez besoin de `condition1 ? A : (condition2 ? B : C)`, il est temps d'extraire un sous-composant ou d'utiliser un `if...else` classique.

---

## Opérateur logique `&&` {#operateur-logique-et}

### 1. Quoi
L'opérateur logique "ET" (`&&`). En JavaScript, `A && B` retourne `A` si `A` est faux, sinon il retourne `B`.
En React, on l'utilise pour le motif : **"Si condition vraie, alors afficher ceci, sinon rien"**.

### 2. Pourquoi
Souvent, nous n'avons pas besoin de `else`. Nous voulons juste afficher un badge, une erreur ou une modal si une condition est remplie. Le ternaire `condition ? <Comp /> : null` fonctionne, mais le `&&` est plus idiomatique.

### 3. Comment

#### A. Syntaxe de base
```tsx
{hasUnreadMessages && <Badge />}
```
Si `hasUnreadMessages` est `true`, React rend `<Badge />`.
Si `hasUnreadMessages` est `false`, React ignore la ligne (rend `false`, donc rien).

#### B. Cas concret : Liste de fonctionnalités
```tsx
type PlanCardProps = {
  name: string;
  isPremium: boolean;
};

export function PlanCard({ name, isPremium }: PlanCardProps) {
  return (
    <div className="card">
      <h2>Plan {name}</h2>
      <ul>
        <li>Utilisateurs illimités</li>
        <li>Support par email</li>
        
        {/* S'affiche UNIQUEMENT si isPremium est true */}
        {isPremium && <li>Support téléphonique 24/7</li>}
        {isPremium && <li>Analytics avancés</li>}
      </ul>
    </div>
  );
}
```

### 4. Zone de Danger

:::danger Le piège du Zéro
C'est le bug de rendu conditionnel le plus fréquent en React.
JavaScript considère le nombre `0` comme "falsy" (faux).
Cependant, React **affiche** le nombre `0` dans le DOM.

❌ **Bug :**
```tsx
const count = 0;
// Affiche "0" à l'écran car count est 0 (falsy), donc JS retourne count (0)
return <div>{count && <h1>Messages ({count})</h1>}</div>;
```

✅ **Correction (Forcer un booléen) :**
```tsx
// count > 0 est un booléen (false). React n'affiche rien pour false.
return <div>{count > 0 && <h1>Messages ({count})</h1>}</div>;
```
:::

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-10}

1.  **Pourquoi ne peut-on pas utiliser `if...else` directement dans le JSX ?**
    Parce que JSX n'accepte que des expressions (valeurs), et `if` est une instruction (contrôle de flux).

2.  **Quelle est la différence entre `condition && <Comp />` et `condition ? <Comp /> : null` ?**
    Fonctionnellement, c'est identique pour l'affichage. Le `&&` est plus court (sucre syntaxique) quand il n'y a pas de cas "sinon".

3.  **Que se passe-t-il si j'écris `{messages.length && <List />}` et que `messages` est vide ?**
    React affichera le chiffre `0` à l'écran. Il faut écrire `messages.length > 0 && ...`.

4.  **Comment masquer un composant pour qu'il ne rende rien du tout ?**
    Le composant doit retourner `null` (ou `false` ou `undefined`).

---

## Exercices : {#exercices-10}

### Exercice 1 - Le Gardien (Early Return) {#exercice-1---le-gardien}

🎯 **Objectif** : Utiliser le "Early Return" pour gérer les états de chargement et d'erreur.

💼 **Mise en situation** : Vous affichez les détails d'un produit e-commerce. Les données arrivent de manière asynchrone (simulé).

📝 **Énoncé** :
1. Créez un composant `ProductDetails`.
2. Props : `isLoading` (boolean), `error` (string | null), `product` ({ name: string, price: number } | null).
3. Si `isLoading` est vrai, retournez une `div` "Chargement en cours...".
4. Si `error` existe (n'est pas null), retournez une `div` rouge avec le message d'erreur.
5. Sinon, affichez le nom du produit et son prix.

📺 **Résultat attendu** :
Testez en changeant les props : vous devez voir soit le loader, soit l'erreur, soit le produit. Jamais deux en même temps.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
type Product = {
  name: string;
  price: number;
};

type ProductDetailsProps = {
  isLoading: boolean;
  error: string | null;
  product: Product | null;
};

export function ProductDetails({ isLoading, error, product }: ProductDetailsProps) {
  // 1. Priorité haute : Chargement
  if (isLoading) {
    return <div style={{ color: 'gray' }}>Chargement en cours...</div>;
  }

  // 2. Priorité moyenne : Gestion d'erreur
  if (error) {
    return <div style={{ color: 'red', border: '1px solid red', padding: '10px' }}>Erreur : {error}</div>;
  }

  // 3. Sécurité : Si pas de loading, pas d'erreur, mais pas de produit non plus
  if (!product) {
    return <div>Aucun produit trouvé.</div>;
  }

  // 4. Cas nominal : Affichage
  return (
    <div className="card">
      <h1>{product.name}</h1>
      <p>Prix : {product.price} €</p>
    </div>
  );
}
```
</details>

### Exercice 2 - La Cloche de Notification (&& Operator) {#exercice-2---la-cloche-de-notification}

🎯 **Objectif** : Éviter le piège du zéro avec l'opérateur `&&`.

💼 **Mise en situation** : Dans la barre de navigation, vous avez une icône de cloche. Si l'utilisateur a des notifications, un petit badge rouge avec le nombre doit apparaître.

📝 **Énoncé** :
1. Créez un composant `NotificationBell`.
2. Prop : `count` (number).
3. Affichez toujours l'icône "🔔".
4. Utilisez l'opérateur `&&` pour afficher une `span` rouge avec le `count` **seulement si** `count` est strictement supérieur à 0.
5. Si `count` vaut 0, rien ne doit s'afficher à côté de la cloche (surtout pas le chiffre 0).

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Une cloche avec un badge "5" rouge à côté.
> **Annotation** : Montrez que le badge est conditionnel.
> **Alt Text suggéré** : Icône de cloche avec badge de notification actif.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
export function NotificationBell({ count }: { count: number }) {
  return (
    <div style={{ position: 'relative', display: 'inline-block', fontSize: '24px' }}>
      {/* L'icône est toujours là */}
      <span role="img" aria-label="notifications">🔔</span>
      
      {/* 
         ⚠️ ATTENTION : count && ... afficherait "0" si count=0
         ✅ CORRECTION : count > 0 && ... assure un booléen
      */}
      {count > 0 && (
        <span style={{
          position: 'absolute',
          top: -5,
          right: -5,
          backgroundColor: 'red',
          color: 'white',
          fontSize: '12px',
          borderRadius: '50%',
          padding: '2px 6px',
          fontWeight: 'bold'
        }}>
          {count}
        </span>
      )}
    </div>
  );
}
```
</details>

### Exercice 3 - Le Bouton de Connexion (Ternaire) {#exercice-3---le-bouton-de-connexion}

🎯 **Objectif** : Utiliser l'opérateur ternaire pour une UI qui change d'état.

💼 **Mise en situation** : Un bouton dans le header change d'apparence et de texte selon que l'utilisateur est connecté ou non.

📝 **Énoncé** :
1. Créez un composant `AuthButton`.
2. Prop : `isLoggedIn` (boolean).
3. Le bouton doit avoir :
   - Un texte : "Se déconnecter" (si true) ou "Se connecter" (si false).
   - Une couleur de fond (style) : Rouge (si true) ou Vert (si false).
   - Une action `onClick` (simulée avec un `alert`) : "Bye!" (si true) ou "Bienvenue!" (si false).
4. Utilisez des ternaires pour gérer ces 3 variations.

📺 **Résultat attendu** :
Un seul élément `<button>` dans le code, mais qui change totalement de comportement selon la prop.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
type AuthButtonProps = {
  isLoggedIn: boolean;
};

export function AuthButton({ isLoggedIn }: AuthButtonProps) {
  return (
    <button
      // Style conditionnel avec ternaire
      style={{
        backgroundColor: isLoggedIn ? '#e53e3e' : '#38a169', // Rouge vs Vert
        color: 'white',
        padding: '8px 16px',
        border: 'none',
        borderRadius: '4px',
        cursor: 'pointer'
      }}
      // Gestionnaire d'événement conditionnel (ou logique interne)
      onClick={() => alert(isLoggedIn ? "Bye!" : "Bienvenue!")}
    >
      {/* Contenu textuel conditionnel */}
      {isLoggedIn ? "Se déconnecter" : "Se connecter"}
    </button>
  );
}
```
</details>
```