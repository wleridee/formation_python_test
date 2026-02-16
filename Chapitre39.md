Voici le chapitre **`useActionState` (Canary): Actions d'État Asynchrones** pour la formation React 19.2.

```markdown
---
sidebar_label: `useActionState` (Canary)
sidebar_position: 39
---

# Chapitre 39 : `useActionState` (Canary): Actions d'État Asynchrones

Gestion des formulaires, Actions asynchrones, Erreurs et états de chargement

:::info Changement de Nom
Ce Hook était précédemment nommé `useFormState` dans les versions Canary antérieures de React. Dans React 19, il a été renommé `useActionState` pour refléter sa capacité à gérer n'importe quelle action, pas seulement les formulaires.
:::

Jusqu'à présent, gérer la soumission d'un formulaire en React nécessitait beaucoup de "boilerplate" (code répétitif) :
1.  Créer un état pour les données (`useState`).
2.  Créer un état pour le chargement (`isLoading`).
3.  Créer un état pour les erreurs (`error`).
4.  Gérer le `try/catch` dans le gestionnaire `onSubmit`.

React 19 introduit `useActionState` pour automatiser tout ce cycle de vie. Il connecte une action asynchrone (souvent une Server Action) à l'état de votre composant.

## Le Cycle de Vie des Actions {#le-cycle-de-vie-des-actions}

### 1. Quoi
`useActionState` est un Hook qui accepte une fonction (l'action) et un état initial. Il retourne l'état actuel de l'action, une fonction pour la déclencher, et un indicateur de chargement.

Signature :
```tsx
const [state, formAction, isPending] = useActionState(fn, initialState, permalink?);
```

*   **`state`** : La valeur retournée par la dernière exécution de l'action (ou `initialState` au début).
*   **`formAction`** : La fonction à passer à la prop `action` d'un formulaire (ou à appeler manuellement).
*   **`isPending`** : Un booléen indiquant si l'action est en cours d'exécution.
*   **`fn`** : La fonction à exécuter. Elle reçoit deux arguments : `(previousState, formData)`.

### 2. Pourquoi
Pour simplifier radicalement la gestion des formulaires et des mutations de données. Plus besoin de gérer manuellement les `setLoading(true)` et `setLoading(false)`.

### 3. Comment

#### A. Syntaxe de base

```tsx
import { useActionState } from 'react';

// L'action (peut être définie hors du composant ou importée)
async function updateName(previousState: string, formData: FormData) {
  const newName = formData.get("name") as string;
  await new Promise(r => setTimeout(r, 1000)); // Simulation API
  return `Bonjour, ${newName} !`;
}

export function NameForm() {
  // state contient le résultat de updateName
  const [state, formAction, isPending] = useActionState(updateName, "Invité");

  return (
    <form action={formAction}>
      <input name="name" required />
      <button type="submit" disabled={isPending}>
        {isPending ? "Mise à jour..." : "Mettre à jour"}
      </button>
      <p>Résultat : {state}</p>
    </form>
  );
}
```

---

## Gestion des Erreurs et Validation {#gestion-des-erreurs-et-validation}

### 1. Quoi
Le cas d'usage le plus puissant est la validation de formulaire côté serveur (ou via une fonction async). L'action retourne un objet contenant les erreurs ou le succès, et `useActionState` met à jour l'UI automatiquement.

### 2. Pourquoi
Cela permet de garder la logique de validation pure et découplée de l'interface utilisateur. L'action reçoit l'état précédent, ce qui est utile pour incrémenter des valeurs ou garder l'historique des erreurs.

### 3. Comment

#### B. Cas concret : Formulaire d'Inscription

```tsx
import { useActionState } from 'react';

// Définition du type de l'état
type FormState = {
  message: string;
  errors?: {
    email?: string[];
    password?: string[];
  };
};

// Action simulée
async function signupAction(prevState: FormState, formData: FormData): Promise<FormState> {
  const email = formData.get('email') as string;
  
  // Validation simple
  if (!email.includes('@')) {
    return {
      message: 'Échec de la validation',
      errors: { email: ['Email invalide'] }
    };
  }

  // Simulation succès
  await new Promise(resolve => setTimeout(resolve, 800));
  return { message: 'Inscription réussie ! ✅' };
}

const initialState: FormState = { message: '' };

export function SignupForm() {
  const [state, formAction, isPending] = useActionState(signupAction, initialState);

  return (
    <form action={formAction} style={{ display: 'flex', flexDirection: 'column', gap: 10 }}>
      <div>
        <label htmlFor="email">Email</label>
        <input id="email" name="email" type="email" />
        {/* Affichage des erreurs spécifiques au champ */}
        {state.errors?.email && (
          <span style={{ color: 'red', fontSize: '0.8em' }}>
            {state.errors.email[0]}
          </span>
        )}
      </div>

      <button type="submit" disabled={isPending}>
        {isPending ? 'Envoi...' : "S'inscrire"}
      </button>

      {/* Message global */}
      {state.message && <p>{state.message}</p>}
    </form>
  );
}
```

---

## Pattern : Interaction avec `useOptimistic` {#interaction-avec-useoptimistic}

`useActionState` fonctionne main dans la main avec `useOptimistic` (vu au chapitre 37).
*   `useActionState` gère la source de vérité (le retour du serveur).
*   `useOptimistic` gère le feedback visuel immédiat.

```tsx
const [state, formAction] = useActionState(updateAction, null);
const [optimisticState, setOptimistic] = useOptimistic(state);

// formAction déclenchera d'abord l'optimiste, puis l'action réelle
```

### 🚨 Limitations de `useActionState`

1.  **Exécution Client vs Serveur** : Si vous utilisez `useActionState` avec une Server Action (directive `'use server'`), l'action s'exécute sur le serveur. Si vous passez une fonction async normale, elle s'exécute sur le client.
2.  **Reset de formulaire** : `useActionState` ne reset pas automatiquement les champs du formulaire après un succès. Vous devez souvent utiliser `key` ou `useRef` pour le faire, ou utiliser les nouvelles API de React DOM `requestFormReset` (encore expérimentales).

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-39}

1.  **Quels sont les arguments reçus par la fonction d'action passée à `useActionState` ?**
    Elle reçoit `(previousState, formData)`. Le `previousState` est la valeur retournée par l'exécution précédente de l'action.

2.  **Quelle est la différence entre `useActionState` et `useFormStatus` ?**
    `useActionState` est utilisé au niveau du formulaire parent pour récupérer les données (résultats, erreurs). `useFormStatus` est utilisé dans les composants **enfants** du formulaire pour savoir s'il est en cours de soumission (pending), sans avoir accès aux données.

3.  **Pourquoi `isPending` est-il utile dans `useActionState` ?**
    Il permet de désactiver le bouton de soumission ou d'afficher un indicateur de chargement pendant que l'action asynchrone s'exécute, sans avoir à gérer un booléen `isLoading` manuellement.

---

## Exercices : {#exercices-39}

### Exercice 1 - Le Compteur Persistant (Côté Client) {#exercice-1---le-compteur-persistant}

🎯 **Objectif** : Comprendre le mécanisme de `previousState`.

💼 **Mise en situation** : Vous voulez un compteur simple, mais implémenté via une action de formulaire pour préparer une future migration serveur.

📝 **Énoncé** :
1. Créez une fonction `incrementAction` qui prend un nombre (état précédent) et retourne ce nombre + 1 après 500ms de délai.
2. Utilisez `useActionState` avec un état initial de `0`.
3. Affichez le compteur et un bouton "Incrémenter".
4. Observez que le bouton se désactive automatiquement grâce à `isPending`.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useActionState } from 'react';

// L'action reçoit l'état précédent.
// formData est ignoré ici car on n'a pas d'inputs.
async function increment(previousCount: number, formData: FormData): Promise<number> {
  await new Promise(resolve => setTimeout(resolve, 500)); // Délai artificiel
  return previousCount + 1;
}

export function AsyncCounter() {
  // Initialisé à 0
  const [count, formAction, isPending] = useActionState(increment, 0);

  return (
    <div style={{ padding: 20, border: '1px solid #ccc' }}>
      <h3>Compteur Asynchrone</h3>
      <p style={{ fontSize: '2rem', fontWeight: 'bold' }}>{count}</p>
      
      <form action={formAction}>
        <button type="submit" disabled={isPending}>
          {isPending ? "Calcul..." : "+1"}
        </button>
      </form>
    </div>
  );
}
```
</details>

### Exercice 2 - Ajouter au Panier avec Feedback {#exercice-2---ajouter-au-panier}

🎯 **Objectif** : Gérer un objet complexe en retour d'action.

💼 **Mise en situation** : Une fiche produit e-commerce. L'utilisateur choisit une quantité et clique sur "Ajouter". L'action retourne le nouveau total du panier ou une erreur si le stock est insuffisant.

📝 **Énoncé** :
1. Créez une action `addToCart` qui accepte `{ success: boolean, message: string }`.
2. Si la quantité (récupérée via `formData`) est > 5, retournez une erreur "Stock insuffisant".
3. Sinon, retournez "Produit ajouté".
4. Affichez le message en vert (succès) ou rouge (erreur).

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Formulaire produit avec message de retour.
> **Annotation** : Montrez le message "Stock insuffisant" en rouge après avoir tenté d'ajouter 10 articles.
> **Alt Text suggéré** : Gestion d'erreur avec useActionState.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useActionState } from 'react';

type CartState = {
  success: boolean;
  message: string | null;
};

async function addToCartAction(prevState: CartState, formData: FormData): Promise<CartState> {
  const quantity = Number(formData.get('quantity'));
  
  // Simulation délai réseau
  await new Promise(r => setTimeout(r, 600));

  if (quantity > 5) {
    return { success: false, message: "❌ Stock insuffisant (Max 5)" };
  }

  return { success: true, message: `✅ ${quantity} article(s) ajouté(s)` };
}

export function ProductPage() {
  const [state, formAction, isPending] = useActionState(addToCartAction, {
    success: false,
    message: null
  });

  return (
    <div style={{ padding: 20 }}>
      <h2>Super Gadget</h2>
      <form action={formAction}>
        <label>
          Quantité :
          <input type="number" name="quantity" defaultValue="1" min="1" />
        </label>
        
        <br /><br />
        
        <button type="submit" disabled={isPending}>
          {isPending ? "Ajout..." : "Ajouter au panier"}
        </button>
      </form>

      {state.message && (
        <p style={{ 
          color: state.success ? 'green' : 'red', 
          fontWeight: 'bold',
          marginTop: 10
        }}>
          {state.message}
        </p>
      )}
    </div>
  );
}
```
</details>

### Exercice 3 - Le Formulaire de Login Robuste {#exercice-3---login-robuste}

🎯 **Objectif** : Combiner validation, état précédent et accessibilité.

💼 **Mise en situation** : Un formulaire de login classique. Si l'utilisateur se trompe 3 fois, le compte est bloqué temporairement.

📝 **Énoncé** :
1. L'état doit contenir `attempts` (nombre d'essais) et `error` (message).
2. L'action incrémente `attempts` à chaque échec.
3. Si `attempts >= 3`, retournez une erreur "Compte bloqué".
4. Sinon, vérifiez si password === "secret".

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useActionState } from 'react';

type LoginState = {
  attempts: number;
  message: string;
  isLocked: boolean;
};

async function loginAction(prevState: LoginState, formData: FormData): Promise<LoginState> {
  // Si déjà bloqué, on reste bloqué (logique simple)
  if (prevState.isLocked) return prevState;

  const password = formData.get('password') as string;
  await new Promise(r => setTimeout(r, 500));

  if (password === 'secret') {
    return { attempts: 0, message: "Bienvenue ! 🎉", isLocked: false };
  }

  const newAttempts = prevState.attempts + 1;
  const isLocked = newAttempts >= 3;

  return {
    attempts: newAttempts,
    message: isLocked ? "Trop d'essais. Compte bloqué. 🔒" : "Mot de passe incorrect ❌",
    isLocked
  };
}

export function LoginForm() {
  const [state, formAction, isPending] = useActionState(loginAction, {
    attempts: 0,
    message: "",
    isLocked: false
  });

  return (
    <div style={{ border: '1px solid #ddd', padding: 20, maxWidth: 300 }}>
      <h3>Connexion Admin</h3>
      
      <form action={formAction}>
        <div style={{ marginBottom: 10 }}>
          <input 
            type="password" 
            name="password" 
            placeholder="Mot de passe..." 
            disabled={state.isLocked}
          />
        </div>

        <button type="submit" disabled={isPending || state.isLocked}>
          {state.isLocked ? "Bloqué" : "Se connecter"}
        </button>
      </form>

      {state.message && (
        <div style={{ 
          marginTop: 15, 
          padding: 8, 
          background: state.attempts === 0 ? '#e6fffa' : '#fff5f5',
          color: state.attempts === 0 ? '#2c7a7b' : '#c53030'
        }}>
          {state.message}
          {state.attempts > 0 && !state.isLocked && (
            <small style={{ display: 'block' }}>
              Essai {state.attempts}/3
            </small>
          )}
        </div>
      )}
    </div>
  );
}
```
</details>
```