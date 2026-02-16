Voici le chapitre **`useOptimistic` (Canary): Mises à Jour Optimistes** pour la formation React 19.2.

```markdown
---
sidebar_label: `useOptimistic` (Canary)
sidebar_position: 37
---

# Chapitre 37 : `useOptimistic` (Canary): Mises à Jour Optimistes

Feedback immédiat, Gestion des états asynchrones, Expérience utilisateur améliorée

Dans les chapitres précédents, nous avons appris à gérer les états de chargement (`isPending`) avec `useTransition` ou `useDeferredValue`. C'est bien, mais on peut faire mieux.

Imaginez cliquer sur un bouton "J'aime". Avez-vous vraiment envie de voir un petit spinner tourner pendant 200ms avant que le cœur ne devienne rouge ? Non. Vous voulez que ce soit **instantané**.
C'est ce qu'on appelle une **Interface Optimiste** (Optimistic UI).

Le Hook `useOptimistic` (nouveauté React 19) permet d'afficher un état "futur probable" pendant qu'une action asynchrone (souvent une Server Action) est en cours d'exécution. Si l'action réussit, l'état réel prend le relais. Si elle échoue, l'état revient automatiquement en arrière.

:::info Disponibilité
Ce hook fait partie des API stables de React 19, bien qu'il ait longtemps été en Canary. Il est conçu pour fonctionner main dans la main avec les **Server Actions**.
:::

## Le Concept d'UI Optimiste {#le-concept-d-ui-optimiste}

### 1. Quoi
C'est une stratégie d'interface où l'on part du principe que la requête serveur va réussir.
1.  L'utilisateur déclenche une action (ex: envoyer un message).
2.  L'interface affiche **immédiatement** le message comme "envoyé".
3.  La requête part au serveur en arrière-plan.
4.  Quand le serveur répond, l'interface se synchronise avec la vraie donnée.

### 2. Pourquoi
Pour donner une sensation de **fluidité absolue**. L'application semble répondre à la vitesse de la pensée, sans attendre le réseau.

### 3. Comment fonctionne `useOptimistic`

Il permet de "surveiller" un état existant et d'y superposer une modification temporaire.

Signature :
```tsx
const [optimisticState, addOptimistic] = useOptimistic(state, updateFn);
```

*   `state` : La donnée source de vérité (venant souvent des props ou du serveur).
*   `updateFn` : Une fonction pure (reducer) qui calcule le nouvel état optimiste.
*   `optimisticState` : L'état à utiliser dans votre JSX (il vaut `state` initialement, ou la version modifiée pendant une transition).
*   `addOptimistic` : La fonction à appeler pour déclencher la modification temporaire.

#### A. Syntaxe de base

```tsx
import { useOptimistic, useState } from 'react';

function LikeButton({ likeCount }: { likeCount: number }) {
  // 1. Définition de l'état optimiste
  // "Si on ajoute un like, l'état temporaire est 'state actuel + 1'"
  const [optimisticLikes, addOptimisticLike] = useOptimistic(
    likeCount,
    (currentState, optimisticValue: number) => currentState + optimisticValue
  );

  const handleLike = async () => {
    // 2. Mise à jour immédiate de l'UI (avant le réseau)
    addOptimisticLike(1);
    
    // 3. Appel réel au serveur (simulation)
    await fetch('/api/like', { method: 'POST' });
    // Une fois cette promesse finie, React re-rendra le composant 
    // avec la nouvelle valeur de 'likeCount' reçue du parent ou revalidée.
  };

  return (
    <button onClick={handleLike}>
      ❤️ {optimisticLikes}
    </button>
  );
}
```

---

## Intégration avec les Formulaires (Server Actions) {#integration-avec-les-formulaires}

### 1. Quoi
Le cas d'usage royal de `useOptimistic` est dans les formulaires utilisant les Server Actions de React 19.

### 2. Pourquoi
React gère automatiquement le cycle de vie de l'action. L'état optimiste ne vit que le temps de l'exécution de l'action (`async`). Dès que l'action est terminée, `optimisticState` redevient égal au `state` réel (qui devrait avoir été mis à jour entre-temps).

### 3. Comment

#### B. Cas concret : Liste de Messages

```tsx
import { useOptimistic, useState, useRef } from 'react';

type Message = { id: number; text: string; sending?: boolean };

// Action simulée (normalement importée d'un fichier serveur)
async function sendMessageToServer(formData: FormData): Promise<Message> {
  await new Promise(r => setTimeout(r, 1000)); // Latence réseau
  return { 
    id: Date.now(), 
    text: formData.get('message') as string 
  };
}

export function ChatRoom() {
  const [messages, setMessages] = useState<Message[]>([
    { id: 1, text: "Bienvenue !" }
  ]);

  // Hook Optimiste
  // state : liste actuelle des messages
  // newMessage : le message temporaire qu'on veut ajouter
  const [optimisticMessages, addOptimisticMessage] = useOptimistic(
    messages,
    (state, newMessage: Message) => [...state, newMessage]
  );

  const formAction = async (formData: FormData) => {
    const text = formData.get('message') as string;
    
    // 1. UI Optimiste IMMÉDIATE
    addOptimisticMessage({ 
      id: Math.random(), // ID temporaire
      text: text, 
      sending: true // Flag pour le style visuel
    });

    // 2. Action Serveur
    const savedMessage = await sendMessageToServer(formData);
    
    // 3. Mise à jour de l'état réel (Source de vérité)
    // Cela effacera automatiquement l'état optimiste car l'action est finie
    setMessages(prev => [...prev, savedMessage]);
  };

  return (
    <div>
      <ul>
        {optimisticMessages.map((msg) => (
          <li key={msg.id} style={{ opacity: msg.sending ? 0.5 : 1 }}>
            {msg.text} {msg.sending && '(Envoi...)'}
          </li>
        ))}
      </ul>
      
      <form action={formAction}>
        <input name="message" required placeholder="Votre message..." />
        <button type="submit">Envoyer</button>
      </form>
    </div>
  );
}
```

### 4. Zone de Danger

:::danger Contexte d'exécution
`addOptimistic` ne fonctionne que si React est dans une phase de transition ou une action.
Si vous l'appelez dans un gestionnaire d'événement classique (`onClick`), vous devez envelopper l'appel dans `startTransition` :
```tsx
const handleClick = () => {
  startTransition(async () => {
    addOptimistic(newItem); // ✅ OK
    await serverAction();
  });
};
```
Dans un attribut `<form action={fn}>`, React le fait automatiquement pour vous.
:::

### 🚨 Limitations de `useOptimistic`
1.  **Gestion des erreurs** : Si l'action serveur échoue, l'état optimiste disparaît et l'interface revient à l'état initial. L'utilisateur verra son message disparaître. Vous **devez** gérer l'affichage d'une notification d'erreur (`toast`) pour prévenir l'utilisateur.
2.  **Complexité du Reducer** : Pour des structures de données complexes (arbres, graphes), la fonction de mise à jour optimiste peut devenir difficile à écrire et à maintenir.

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-37}

1.  **Que se passe-t-il avec l'état optimiste quand l'action asynchrone se termine ?**
    L'état optimiste est automatiquement "jeté" par React. Le composant se re-rend avec l'état réel (le `state` passé en 1er argument de `useOptimistic`), qui doit normalement avoir été mis à jour par le retour du serveur.

2.  **Pourquoi utiliser `useOptimistic` plutôt qu'un simple `setState` local ?**
    Car `useOptimistic` gère automatiquement le nettoyage. Vous n'avez pas besoin de faire un `setLoading(false)` ou de supprimer manuellement l'item temporaire quand la requête finit. C'est déclaratif et lié au cycle de vie de l'action.

3.  **Peut-on utiliser `useOptimistic` pour autre chose que des ajouts (ex: suppression, modification) ?**
    **Oui**. Le second argument est un "reducer". Vous pouvez filtrer un tableau (suppression optimiste) ou mapper dessus pour changer une propriété (édition optimiste).

---

## Exercices : {#exercices-37}

### Exercice 1 - Le Compteur de Likes Ultime {#exercice-1---le-compteur-de-likes-ultime}

🎯 **Objectif** : Implémenter un bouton like qui réagit instantanément, même avec une API lente.

💼 **Mise en situation** : Sur votre réseau social, les utilisateurs cliquent frénétiquement sur "Like". L'API met 1 seconde à répondre. Le compteur doit s'incrémenter tout de suite.

📝 **Énoncé** :
1. Créez un composant `Post` avec un prop `initialLikes`.
2. Utilisez `useOptimistic` pour gérer le compteur affiché.
3. Le formulaire doit avoir un bouton type `submit`.
4. Dans l'action du formulaire : ajoutez le like optimiste, attendez 1 seconde, puis (optionnel) mettez à jour un état local réel.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useOptimistic, useState } from 'react';

// Fausse API
const updateLikesApi = async (current: number) => {
  await new Promise(r => setTimeout(r, 1000)); // Latence
  return current + 1;
};

export function LikeCounter({ initialLikes = 0 }: { initialLikes?: number }) {
  const [likes, setLikes] = useState(initialLikes);

  // Hook Optimiste
  // On stocke le nombre (number)
  const [optimisticLikes, addOptimistic] = useOptimistic(
    likes,
    (state, amount: number) => state + amount
  );

  const handleLike = async () => {
    // 1. Feedback immédiat
    addOptimistic(1);
    
    // 2. Appel serveur
    const newCount = await updateLikesApi(likes);
    
    // 3. Sync source de vérité
    setLikes(newCount);
  };

  return (
    <div style={{ padding: 20 }}>
      <h3>Post Populaire</h3>
      <p>Ce post a des super likes.</p>
      
      {/* Note : Pour simplifier, ici on utilise un onClick avec une forme de transition implicite 
          si on utilisait startTransition, ou un <form action> pour le support natif React 19.
          L'exemple form est plus robuste pour React 19. */}
      
      <form action={handleLike}>
        <button type="submit">
           👍 {optimisticLikes} J'aime
        </button>
      </form>
      
      <small style={{ color: 'gray', marginLeft: 10 }}>
        (Réel : {likes})
      </small>
    </div>
  );
}
```
</details>

### Exercice 2 - La Todo List Instantanée {#exercice-2---la-todo-list-instantanee}

🎯 **Objectif** : Ajouter des éléments à une liste visuellement avant la confirmation serveur.

💼 **Mise en situation** : Une application de tâches partagées. Quand j'ajoute "Acheter du lait", je veux le voir apparaître tout de suite dans la liste, peut-être légèrement grisé pour dire "en cours d'enregistrement".

📝 **Énoncé** :
1. État initial : `['Tâche 1']`.
2. `useOptimistic` gère un tableau de strings.
3. L'action du formulaire ajoute la tâche optimiste, attend 2s, puis met à jour l'état réel.
4. Appliquez un style `font-style: italic` ou `opacity: 0.7` aux tâches optimistes (Astuce : votre reducer optimiste peut transformer les strings en objets `{text, pending}`).

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Liste de tâches.
> **Annotation** : Montrez une tâche "Acheter du pain" qui est grisée (optimiste) tandis que les autres sont normales.
> **Alt Text suggéré** : Démonstration visuelle d'un ajout optimiste dans une liste.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useOptimistic, useState } from 'react';

type Todo = { id: number; text: string; pending?: boolean };

export function TodoList() {
  const [todos, setTodos] = useState<Todo[]>([
    { id: 1, text: "Apprendre React" }
  ]);

  // Le reducer optimiste doit gérer le cas où on ajoute un item
  const [optimisticTodos, addOptimisticTodo] = useOptimistic(
    todos,
    (state, newTodo: Todo) => [...state, newTodo]
  );

  const formAction = async (formData: FormData) => {
    const text = formData.get('todo') as string;
    if (!text) return;

    // 1. Ajout Optimiste
    addOptimisticTodo({
      id: Math.random(), // ID temporaire
      text: text,
      pending: true // Marqueur pour le style
    });

    // 2. Simulation serveur
    await new Promise(r => setTimeout(r, 2000));

    // 3. Mise à jour réelle (normalement via revalidation de données serveur)
    setTodos(prev => [...prev, { id: Date.now(), text }]);
  };

  return (
    <div>
      <ul>
        {optimisticTodos.map((todo) => (
          <li 
            key={todo.id} 
            style={{ 
              opacity: todo.pending ? 0.5 : 1,
              transition: 'opacity 0.2s'
            }}
          >
            {todo.text} {todo.pending && '⏳'}
          </li>
        ))}
      </ul>

      <form action={formAction}>
        <input name="todo" placeholder="Nouvelle tâche..." />
        <button type="submit">Ajouter</button>
      </form>
    </div>
  );
}
```
</details>

### Exercice 3 - Édition de Profil (Modification) {#exercice-3---edition-de-profil}

🎯 **Objectif** : Utiliser `useOptimistic` pour une modification (update) et non un ajout.

💼 **Mise en situation** : L'utilisateur change son pseudo. Il tape "SuperDev", valide. Le titre de la page doit changer instantanément de "Dev" à "SuperDev", sans attendre.

📝 **Énoncé** :
1. Affichez un titre `<h1>Bonjour, {name}</h1>`.
2. Un input permet de changer le nom.
3. Utilisez `useOptimistic` sur la variable `name`. Le reducer remplacera simplement l'ancien nom par le nouveau.
4. L'action doit simuler un délai réseau.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useOptimistic, useState } from 'react';

export function ProfileEditor() {
  const [name, setName] = useState("Invité");

  // Ici le reducer remplace complètement l'état au lieu d'ajouter
  const [optimisticName, setOptimisticName] = useOptimistic(
    name,
    (_currentState, newName: string) => newName
  );

  const updateNameAction = async (formData: FormData) => {
    const newName = formData.get('username') as string;
    
    // 1. Update Optimiste
    setOptimisticName(newName);

    // 2. Latence réseau
    await new Promise(r => setTimeout(r, 1500));

    // 3. Confirmation
    setName(newName);
  };

  return (
    <div style={{ border: '1px solid #ccc', padding: 20, borderRadius: 8 }}>
      {/* Le titre reflète immédiatement la saisie après validation */}
      <h1>Bonjour, <span style={{ color: 'blue' }}>{optimisticName}</span></h1>
      
      <form action={updateNameAction} style={{ display: 'flex', gap: 10 }}>
        <input 
          name="username" 
          defaultValue={name} 
          placeholder="Nouveau pseudo" 
        />
        <button type="submit">Mettre à jour</button>
      </form>
      
      <p style={{ fontSize: '0.8em', color: '#666' }}>
        (Le serveur mettra 1.5s à confirmer, mais le titre change tout de suite)
      </p>
    </div>
  );
}
```
</details>
```