Voici le chapitre **`useRef` : Référencer des Valeurs** pour la formation React 19.2.

```markdown
---
sidebar_label: useRef : Référencer des Valeurs
sidebar_position: 20
---

# Chapitre 20 : `useRef` : Référencer des Valeurs

Hook `useRef`, Références mutables, Accéder aux éléments DOM

Jusqu'à maintenant, nous avons appris que pour changer l'écran, il faut changer l'état (`useState`). Cela déclenche un nouveau rendu.
Mais parfois, vous avez besoin de stocker une information qui **ne doit pas** déclencher de rendu. Ou bien vous avez besoin de "parler" directement à un élément HTML (comme donner le focus à un input).
Pour ces cas d'usage, React propose une "trappe de sortie" : le Hook `useRef`.

## Le Hook `useRef` {#le-hook-useref}

### 1. Quoi
`useRef` est un Hook qui renvoie un objet mutable `{ current: ... }`.
Contrairement à `useState`, modifier la propriété `current` **ne déclenche pas** de nouveau rendu du composant.

### 2. Pourquoi
Imaginez `useRef` comme une "poche" secrète du composant. Vous pouvez y mettre des choses, les modifier, les lire, mais React "ne regarde pas" dedans pour décider de redessiner l'interface. C'est idéal pour :
1.  Stocker des identifiants de timer (`setTimeout`).
2.  Garder des valeurs précédentes pour comparaison.
3.  Stocker des éléments du DOM (voir section suivante).

### 3. Comment

#### A. Syntaxe de base

```tsx
import { useRef } from 'react';

function Component() {
  // Initialisation avec 0
  const ref = useRef(0); 

  function handleClick() {
    // Lecture et Écriture directes
    ref.current = ref.current + 1;
    console.log('Compteur secret :', ref.current);
  }
  
  // Note : ref.current n'apparaît PAS dans le JSX car sa modification ne met pas à jour l'écran
  return <button onClick={handleClick}>Cliquez (Regardez la console)</button>;
}
```

#### B. Cas concret : Le Chronomètre
Si nous utilisions `useState` pour stocker l'ID de l'intervalle, chaque modification provoquerait un rendu inutile.

```tsx
import { useState, useRef } from 'react';

export function Stopwatch() {
  const [startTime, setStartTime] = useState<number | null>(null);
  const [now, setNow] = useState<number | null>(null);
  
  // On utilise useRef pour stocker l'ID de l'intervalle.
  // Cela n'a aucun impact visuel direct, c'est juste de la mécanique interne.
  const intervalRef = useRef<ReturnType<typeof setInterval> | null>(null);

  function handleStart() {
    setStartTime(Date.now());
    setNow(Date.now());

    // On efface l'ancien intervalle s'il existe
    if (intervalRef.current) clearInterval(intervalRef.current);

    // On stocke le nouvel ID
    intervalRef.current = setInterval(() => {
      setNow(Date.now());
    }, 10);
  }

  function handleStop() {
    if (intervalRef.current) {
      clearInterval(intervalRef.current);
      intervalRef.current = null;
    }
  }

  let secondsPassed = 0;
  if (startTime != null && now != null) {
    secondsPassed = (now - startTime) / 1000;
  }

  return (
    <>
      <h1>Temps écoulé : {secondsPassed.toFixed(3)}</h1>
      <button onClick={handleStart}>Démarrer</button>
      <button onClick={handleStop}>Arrêter</button>
    </>
  );
}
```

### 4. Zone de Danger

:::danger Ne lisez/écrivez pas `ref.current` pendant le rendu
Le rendu doit rester pur. Modifier une ref pendant le rendu (dans le corps principal de la fonction composant) rend le comportement imprévisible.

❌ **Interdit :**
```tsx
function BadComponent() {
  const count = useRef(0);
  count.current = count.current + 1; // 😱 Écriture pendant le rendu !
  return <h1>{count.current}</h1>;   // 😱 Lecture pendant le rendu !
}
```

✅ **Permis :**
*   Dans les gestionnaires d'événements (`onClick`, etc.).
*   Dans les `useEffect`.
:::

---

## Références vs État (`useRef` vs `useState`) {#references-vs-etat}

### Tableau Comparatif

| Caractéristique | `useRef` | `useState` |
| :--- | :--- | :--- |
| **Valeur stockée** | Mutable `{ current: ... }` | Immuable (via setter) |
| **Déclenche un rendu ?** | ❌ NON | ✅ OUI |
| **Utilisation** | Mécanique interne, DOM | Données affichées à l'écran |
| **Moment de lecture** | `useEffect`, Event Handlers | Au moment du rendu (JSX) |

Si vous vous demandez "Dois-je utiliser une ref ou un state ?", posez-vous la question : **"Si cette valeur change, l'utilisateur doit-il le voir immédiatement ?"**
*   Oui ➔ `useState`
*   Non ➔ `useRef`

---

## Accéder aux Éléments DOM {#acceder-aux-elements-dom}

### 1. Quoi
C'est l'utilisation la plus courante de `useRef`. React est déclaratif, mais parfois vous devez être impératif : "Mets le focus sur cet input", "Scrolle jusqu'à cette div".
L'attribut JSX `ref` permet d'associer un élément HTML réel à votre objet ref.

### 2. Pourquoi
Pour interagir avec des API du navigateur qui ne sont pas couvertes par React (focus, media playback, scroll position, canvas drawing).

### 3. Comment

#### A. Syntaxe

```tsx
import { useRef } from 'react';

export function AutoFocusInput() {
  // 1. Créer la ref (null au départ)
  const inputRef = useRef<HTMLInputElement>(null);

  function handleClick() {
    // 3. Utiliser l'élément DOM via inputRef.current
    // Note: Le '?' (optional chaining) est recommandé car current est null au premier rendu
    inputRef.current?.focus();
  }

  return (
    <>
      {/* 2. Lier la ref au JSX */}
      <input ref={inputRef} type="text" />
      <button onClick={handleClick}>Focus sur le champ</button>
    </>
  );
}
```

#### B. Gestion des listes de Refs
Comment gérer une ref pour une liste d'éléments générée par `.map()` ? Vous ne pouvez pas appeler `useRef` dans une boucle.
La solution moderne est d'utiliser une **Map** stockée dans une seule ref.

```tsx
import { useRef } from 'react';

export function CatFriends() {
  const itemsRef = useRef<Map<number, HTMLLIElement | null>>(null);

  function getMap() {
    if (!itemsRef.current) {
      itemsRef.current = new Map();
    }
    return itemsRef.current;
  }

  function scrollToId(id: number) {
    const map = getMap();
    const node = map.get(id);
    node?.scrollIntoView({ behavior: 'smooth', block: 'nearest' });
  }

  const catIds = [0, 5, 9];

  return (
    <>
      <nav>
        <button onClick={() => scrollToId(0)}>Chat 0</button>
        <button onClick={() => scrollToId(5)}>Chat 5</button>
        <button onClick={() => scrollToId(9)}>Chat 9</button>
      </nav>
      <ul>
        {catIds.map(id => (
          <li
            key={id}
            // Callback ref : React appelle cette fonction avec le noeud DOM
            ref={(node) => {
              const map = getMap();
              if (node) {
                map.set(id, node);
              } else {
                map.delete(id);
              }
            }}
            style={{ height: 200, border: '1px solid black', margin: 10 }}
          >
            Chat numéro {id}
          </li>
        ))}
      </ul>
    </>
  );
}
```

### 🚨 Limitations
N'abusez pas des Refs pour manipuler le DOM. Évitez de changer les styles, le contenu textuel ou la structure HTML via des refs. Cela entrerait en conflit avec React.
*   ❌ `myRef.current.remove()`
*   ❌ `myRef.current.className = "red"` (Préférez l'état pour les classes CSS)
*   ✅ Focus, Scroll, Mesure de taille (`getBoundingClientRect`).

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-20}

1.  **Quelle est la différence principale entre `useRef` et `useState` ?**
    La modification d'une ref (`ref.current = ...`) ne déclenche **pas** de nouveau rendu, contrairement à `setState`.

2.  **Peut-on lire ou écrire `ref.current` pendant le rendu du composant ?**
    Non, c'est interdit car cela rend le rendu impur et imprévisible. Il faut le faire dans des Event Handlers ou des `useEffect`.

3.  **Comment accéder à un élément DOM (ex: un `<input>`) en React ?**
    On crée une ref avec `useRef(null)`, on la passe à l'attribut `ref` de l'élément JSX (`<input ref={myRef} />`), puis on accède à l'élément via `myRef.current`.

4.  **Que contient `ref.current` avant que le composant ne soit monté dans le DOM ?**
    Il contient `null` (ou la valeur initiale passée à `useRef`).

---

## Exercices : {#exercices-20}

### Exercice 1 - Le Compteur de Clics Silencieux {#exercice-1---le-compteur-de-clics-silencieux}

🎯 **Objectif** : Utiliser une ref pour stocker une valeur mutable sans re-render.

💼 **Mise en situation** : Vous analysez le comportement utilisateur. Vous voulez compter combien de fois l'utilisateur clique sur un bouton "Acheter", mais sans rien changer à l'affichage pour ne pas le perturber. Au bout de 5 clics, affichez une alerte.

📝 **Énoncé** :
1. Créez un composant avec un bouton "Acheter".
2. Utilisez `useRef` pour suivre le nombre de clics.
3. À chaque clic, incrémentez la ref.
4. Si la ref atteint 5, lancez `alert("Vous êtes intéressé !")`.
5. Ajoutez un `console.log` dans le corps du composant pour prouver qu'il ne se re-rend PAS à chaque clic.

📺 **Résultat attendu** :
Le bouton ne change pas visuellement. La console ne montre qu'un seul rendu initial. Au 5ème clic, l'alerte apparaît.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useRef } from 'react';

export function SilentTracker() {
  // Initialisation à 0. Cette valeur persiste entre les rendus.
  const clickCountRef = useRef(0);

  console.log("Rendu du composant (ne doit apparaître qu'une fois)");

  const handleClick = () => {
    // Mutation de la ref : pas de re-render déclenché
    clickCountRef.current += 1;
    
    console.log(`Clics actuels : ${clickCountRef.current}`);

    if (clickCountRef.current === 5) {
      alert("Vous êtes intéressé !");
    }
  };

  return (
    <button onClick={handleClick}>
      Acheter (Tracker silencieux)
    </button>
  );
}
```
</details>

### Exercice 2 - Le Focus Automatique (DOM) {#exercice-2---le-focus-automatique-dom}

🎯 **Objectif** : Manipuler le DOM impérativement avec `useRef`.

💼 **Mise en situation** : Un formulaire de recherche. Quand l'utilisateur clique sur l'icône de loupe, l'input de recherche doit s'ouvrir (CSS) et prendre le focus immédiatement pour que l'utilisateur puisse taper.

📝 **Énoncé** :
1. Un bouton "Ouvrir Recherche".
2. Un input type text.
3. Au clic sur le bouton, utilisez `inputRef.current.focus()`.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Un input avec le curseur clignotant dedans (focus actif).
> **Annotation** : Mettre en évidence que le focus a été donné programmatiquement.
> **Alt Text suggéré** : Champ de saisie ayant reçu le focus automatiquement après un clic bouton.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useRef } from 'react';

export function SearchForm() {
  // Création de la ref typée pour un élément Input HTML
  const inputRef = useRef<HTMLInputElement>(null);

  const handleOpenSearch = () => {
    // On accède directement à l'API du DOM pour mettre le focus
    // Le ?. (optional chaining) protège au cas où l'élément n'existe pas encore
    inputRef.current?.focus();
  };

  return (
    <div style={{ padding: 20, display: 'flex', gap: 10 }}>
      <button onClick={handleOpenSearch}>
        🔍 Ouvrir Recherche
      </button>
      
      <input 
        ref={inputRef} // Liaison JSX -> Ref
        type="text" 
        placeholder="Tapez ici..." 
      />
    </div>
  );
}
```
</details>

### Exercice 3 - Le Lecteur Vidéo Personnalisé {#exercice-3---le-lecteur-video-personnalise}

🎯 **Objectif** : Contrôler un élément média via Ref.

💼 **Mise en situation** : Vous créez un player vidéo custom. Vous voulez vos propres boutons "Play" et "Pause" en dehors de la vidéo.

📝 **Énoncé** :
1. Utilisez une balise `<video>` avec une source MP4 (ex: lien public).
2. Attachez une `ref` à la vidéo.
3. Créez deux boutons : "Lecture" et "Pause".
4. Le bouton Lecture appelle `videoRef.current.play()`.
5. Le bouton Pause appelle `videoRef.current.pause()`.

📺 **Résultat attendu** :
Cliquer sur vos boutons contrôle la vidéo.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useRef } from 'react';

export function VideoPlayer() {
  const videoRef = useRef<HTMLVideoElement>(null);

  const handlePlay = () => {
    videoRef.current?.play();
  };

  const handlePause = () => {
    videoRef.current?.pause();
  };

  return (
    <div style={{ display: 'flex', flexDirection: 'column', gap: 10, maxWidth: 400 }}>
      <video 
        ref={videoRef}
        width="100%"
        style={{ borderRadius: 8, backgroundColor: 'black' }}
        // Source vidéo de démo libre de droits
        src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
      />
      
      <div style={{ display: 'flex', gap: 10, justifyContent: 'center' }}>
        <button onClick={handlePlay}>▶️ Lecture</button>
        <button onClick={handlePause}>⏸ Pause</button>
      </div>
    </div>
  );
}
```
</details>
```