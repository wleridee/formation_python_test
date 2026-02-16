Voici le chapitre **`useImperativeHandle`: Personnaliser les Refs** pour la formation React 19.2.

```markdown
---
sidebar_label: `useImperativeHandle`: Personnaliser les Refs
sidebar_position: 26
---

# Chapitre 26 : `useImperativeHandle`: Personnaliser les Refs

Exposer des méthodes via ref, Refs aux composants, Abstraction de l'implémentation

React privilégie le flux de données unidirectionnel (props descendantes). Cependant, il arrive parfois qu'un parent doive impérativement déclencher une action chez un enfant, comme "Scroller en haut", "Mettre le focus" ou "Réinitialiser le formulaire".
Bien que l'on puisse utiliser des Refs pour accéder directement au DOM, cela brise souvent l'encapsulation.
`useImperativeHandle` permet de définir précisément *quelles* méthodes un composant expose à son parent via une Ref, gardant ainsi le contrôle sur son API publique.

## Exposer des Méthodes via Ref {#exposer-des-methodes-via-ref}

### 1. Quoi
`useImperativeHandle` est un Hook qui permet de personnaliser l'objet qui est renvoyé lorsqu'un parent utilise une `ref` sur votre composant.
Au lieu de renvoyer le nœud DOM brut (ex: `<input>`), vous renvoyez un objet JavaScript avec des méthodes spécifiques (ex: `{ focus: () => ... }`).

### 2. Pourquoi
L'encapsulation est un principe clé. Si vous exposez tout le nœud DOM d'un composant enfant :
1.  Le parent peut faire n'importe quoi (changer les styles, supprimer des attributs).
2.  Si vous changez l'implémentation interne (ex: remplacer `<input>` par une lib tierce), vous cassez le code du parent qui s'attendait à trouver un `<input>`.
`useImperativeHandle` crée une **interface stable** entre le parent et l'enfant.

### 3. Comment

#### A. Syntaxe de base (React 19+)

> **Note React 19** : `forwardRef` n'est plus nécessaire pour passer des refs aux composants fonctionnels. La ref est maintenant reçue directement comme une prop.

```tsx
import { useImperativeHandle, useRef } from 'react';

// 1. Définir le type de l'objet exposé (le "Handle")
export interface MyInputHandle {
  focus: () => void;
  shake: () => void;
}

function MyInput({ ref }: { ref: React.Ref<MyInputHandle> }) {
  const internalInputRef = useRef<HTMLInputElement>(null);

  // 2. Définir ce que la ref renverra
  useImperativeHandle(ref, () => {
    return {
      focus() {
        internalInputRef.current?.focus();
      },
      shake() {
        // Logique d'animation complexe encapsulée ici
        console.log("Shaking input!");
      }
    };
  });

  return <input ref={internalInputRef} />;
}
```

#### B. Utilisation par le Parent

```tsx
function ParentForm() {
  const inputRef = useRef<MyInputHandle>(null);

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={() => inputRef.current?.focus()}>Focus</button>
      <button onClick={() => inputRef.current?.shake()}>Erreur</button>
      {/* ❌ inputRef.current.style.color = 'red' -> Impossible car non exposé ! */}
    </>
  );
}
```

### 4. Zone de Danger

:::danger N'en abusez pas
Les méthodes impératives ne doivent pas remplacer les props et l'état.
❌ **Mauvais :** Une méthode `openModal()` exposée via ref.
✅ **Bon :** Une prop `isOpen={true}` passée au composant Modal.
Utilisez `useImperativeHandle` uniquement pour des comportements que les props ne peuvent pas bien modéliser (scroll, focus, animations, lecture média).
:::

---

## Abstraction de l'Implémentation {#abstraction-de-l-implementation}

### 1. Quoi
Cacher la complexité interne d'un composant derrière une API simple.

### 2. Pourquoi
Imaginez un composant `VideoPlayer` complexe qui contient des contrôles, des sous-titres, et une balise `<video>`.
Si le parent veut faire "Play", il ne devrait pas avoir à chercher `videoRef.current.children[0].play()`.
Il doit juste appeler `playerRef.current.play()`.

### 3. Comment

```tsx
import { useImperativeHandle, useRef, useState } from 'react';

export interface VideoHandle {
  play: () => void;
  pause: () => void;
  getStatus: () => string;
}

export function SmartVideoPlayer({ src, ref }: { src: string, ref: React.Ref<VideoHandle> }) {
  const videoRef = useRef<HTMLVideoElement>(null);
  const [isPlaying, setIsPlaying] = useState(false);

  useImperativeHandle(ref, () => ({
    play() {
      videoRef.current?.play();
      setIsPlaying(true);
    },
    pause() {
      videoRef.current?.pause();
      setIsPlaying(false);
    },
    getStatus() {
      return isPlaying ? "Lecture en cours" : "En pause";
    }
  }), [isPlaying]); // Dépendances si nécessaire

  return (
    <div className="video-wrapper">
      <video ref={videoRef} src={src} width="300" />
      <div className="status-badge">{isPlaying ? '▶️' : '⏸️'}</div>
    </div>
  );
}
```

---

## Refs Multiples et Composition {#refs-multiples-et-composition}

### 1. Quoi
Parfois, un composant a besoin de sa propre ref interne pour fonctionner, tout en devant aussi exposer une ref à son parent. `useImperativeHandle` gère cela naturellement.

### 2. Pourquoi
Sans ce hook, vous devriez manuellement synchroniser deux refs. Avec ce hook, la ref interne reste privée, et la ref externe est construite sur mesure.

### 3. Cas Concret : Liste virtuelle avec Scroll

```tsx
import { useImperativeHandle, useRef } from 'react';

export interface ListHandle {
  scrollToRow: (index: number) => void;
}

export function VirtualList({ items, ref }: { items: string[], ref: React.Ref<ListHandle> }) {
  const listRef = useRef<HTMLUListElement>(null);

  useImperativeHandle(ref, () => ({
    scrollToRow(index) {
      const node = listRef.current;
      if (node) {
        const row = node.children[index];
        row?.scrollIntoView({ behavior: 'smooth' });
      }
    }
  }));

  return (
    <ul ref={listRef} style={{ height: 200, overflow: 'auto' }}>
      {items.map((item, i) => <li key={i}>{item}</li>)}
    </ul>
  );
}
```

### 🚨 Limitations
- **React 18 et antérieur** : Nécessite d'envelopper le composant avec `forwardRef(Component)`.
- **Complexité** : Rend le flux de données moins évident. À utiliser avec parcimonie.
- **Testabilité** : Tester des méthodes impératives peut être plus complexe que tester des changements de props.

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-26}

1.  **À quoi sert `useImperativeHandle` ?**
    À personnaliser l'objet (instance) exposé au composant parent via une `ref`, permettant de restreindre l'accès ou de créer des méthodes utilitaires.

2.  **Pourquoi ne pas simplement exposer la ref DOM directement ?**
    Pour l'encapsulation. Cela permet de changer l'implémentation interne sans casser le code du parent, et d'empêcher le parent de modifier le DOM de manière non contrôlée.

3.  **Quelle est la différence syntaxique majeure en React 19 pour les refs ?**
    On n'a plus besoin d'utiliser `React.forwardRef`. La `ref` est maintenant passée comme une prop standard aux composants fonctionnels.

4.  **Citez deux cas d'usage valides pour ce Hook.**
    - Gérer le focus, le scroll ou la sélection de texte.
    - Contrôler des médias (audio/vidéo) ou des animations impératives.

---

## Exercices : {#exercices-26}

### Exercice 1 - Le Formulaire Super-Validateur {#exercice-1---le-formulaire-super-validateur}

🎯 **Objectif** : Exposer une méthode de validation complexe au parent.

💼 **Mise en situation** : Un formulaire d'inscription divisé en plusieurs sous-composants (`IdentitySection`, `AddressSection`). Le composant parent `RegisterPage` a un bouton "Valider tout". Il doit pouvoir déclencher la validation de chaque section et récupérer les erreurs, sans connaître la logique interne des champs.

📝 **Énoncé** :
1. Créez un composant `IdentitySection` avec deux champs (Nom, Email).
2. Utilisez `useImperativeHandle` pour exposer une méthode `validate()`.
3. Cette méthode doit vérifier si les champs sont remplis. Si oui, retourne `true`. Sinon, met le focus sur le champ vide et retourne `false`.
4. Le parent appelle `sectionRef.current.validate()` au clic sur le bouton final.

📺 **Résultat attendu** :
Au clic sur "Valider tout", si le champ "Nom" est vide, le focus s'y place automatiquement.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState, useRef, useImperativeHandle } from 'react';

// Interface pour TypeScript
interface SectionHandle {
  validate: () => boolean;
}

function IdentitySection({ ref }: { ref: React.Ref<SectionHandle> }) {
  const [name, setName] = useState("");
  const [email, setEmail] = useState("");
  
  const nameInputRef = useRef<HTMLInputElement>(null);
  const emailInputRef = useRef<HTMLInputElement>(null);

  useImperativeHandle(ref, () => ({
    validate() {
      if (!name.trim()) {
        nameInputRef.current?.focus();
        alert("Nom requis !");
        return false;
      }
      if (!email.includes("@")) {
        emailInputRef.current?.focus();
        alert("Email invalide !");
        return false;
      }
      return true; // Tout est bon
    }
  }));

  return (
    <div style={{ border: '1px solid #ccc', padding: 10, marginBottom: 10 }}>
      <h3>Identité</h3>
      <input 
        ref={nameInputRef}
        placeholder="Nom" 
        value={name} 
        onChange={e => setName(e.target.value)} 
      />
      <br/><br/>
      <input 
        ref={emailInputRef}
        placeholder="Email" 
        value={email} 
        onChange={e => setEmail(e.target.value)} 
      />
    </div>
  );
}

export function RegisterPage() {
  const identityRef = useRef<SectionHandle>(null);

  const handleSubmit = () => {
    // Appel impératif de la validation enfant
    const isValid = identityRef.current?.validate();
    
    if (isValid) {
      console.log("Formulaire envoyé !");
    }
  };

  return (
    <div>
      <h1>Inscription</h1>
      <IdentitySection ref={identityRef} />
      <button onClick={handleSubmit}>Valider tout</button>
    </div>
  );
}
```
</details>

### Exercice 2 - Le Timer Contrôlable {#exercice-2---le-timer-controllable}

🎯 **Objectif** : Contrôler un timer interne depuis l'extérieur.

💼 **Mise en situation** : Un composant `Stopwatch` affiche un chrono. Le parent possède les boutons "Start", "Stop", et "Reset". Le parent ne doit pas gérer l'état `time` (qui change chaque seconde), pour éviter de se re-rendre inutilement.

📝 **Énoncé** :
1. `Stopwatch` gère son propre état `seconds`.
2. Il expose `{ start(), stop(), reset() }` via Ref.
3. Le Parent affiche les 3 boutons et pilote le chrono.
4. Notez que le Parent ne se re-rend **pas** quand les secondes défilent (optimisation).

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Un compteur "00:15" et trois boutons dessous.
> **Annotation** : Montrez que le composant parent reste statique pendant que le compteur tourne.
> **Alt Text suggéré** : Chronomètre piloté par des boutons externes via des refs impératives.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState, useRef, useImperativeHandle } from 'react';

export interface TimerHandle {
  start: () => void;
  stop: () => void;
  reset: () => void;
}

function Stopwatch({ ref }: { ref: React.Ref<TimerHandle> }) {
  const [seconds, setSeconds] = useState(0);
  const intervalRef = useRef<number | null>(null);

  useImperativeHandle(ref, () => ({
    start() {
      if (intervalRef.current) return; // Déjà lancé
      intervalRef.current = window.setInterval(() => {
        setSeconds(s => s + 1);
      }, 1000);
    },
    stop() {
      if (intervalRef.current) {
        clearInterval(intervalRef.current);
        intervalRef.current = null;
      }
    },
    reset() {
      if (intervalRef.current) {
        clearInterval(intervalRef.current);
        intervalRef.current = null;
      }
      setSeconds(0);
    }
  }));

  return (
    <div style={{ fontSize: '2rem', fontFamily: 'monospace' }}>
      ⏱️ {seconds}s
    </div>
  );
}

export function TimerController() {
  const timerRef = useRef<TimerHandle>(null);
  console.log("Render Parent (ne doit pas se répéter)");

  return (
    <div style={{ padding: 20 }}>
      <Stopwatch ref={timerRef} />
      <div style={{ marginTop: 10, display: 'flex', gap: 10 }}>
        <button onClick={() => timerRef.current?.start()}>Start</button>
        <button onClick={() => timerRef.current?.stop()}>Stop</button>
        <button onClick={() => timerRef.current?.reset()}>Reset</button>
      </div>
    </div>
  );
}
```
</details>

### Exercice 3 - L'API Vidéo Abstraite {#exercice-3---l-api-video-abstraite}

🎯 **Objectif** : Masquer l'élément DOM réel.

💼 **Mise en situation** : Vous créez un composant `SafeVideo`. Vous voulez que le parent puisse faire `play()` et `pause()`, mais vous voulez lui **interdire** d'accéder à la propriété `src` ou de changer le volume directement via la ref.

📝 **Énoncé** :
1. Créez un composant `SafeVideo` avec une vidéo.
2. Exposez uniquement `play` et `pause`.
3. Dans le parent, essayez d'accéder à `ref.current.volume` (TypeScript devrait bloquer, et au runtime ce sera undefined).

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useRef, useImperativeHandle } from 'react';

interface SafeVideoHandle {
  play: () => void;
  pause: () => void;
}

function SafeVideo({ ref }: { ref: React.Ref<SafeVideoHandle> }) {
  const vidRef = useRef<HTMLVideoElement>(null);

  useImperativeHandle(ref, () => ({
    play: () => vidRef.current?.play(),
    pause: () => vidRef.current?.pause(),
    // On n'expose RIEN d'autre. L'élément DOM est complètement privé.
  }));

  return (
    <video 
      ref={vidRef} 
      src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4" 
      width="300" 
    />
  );
}
```
</details>
```