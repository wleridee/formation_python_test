Voici le chapitre **Manipuler le DOM avec les Refs** pour la formation React 19.2.

```markdown
---
sidebar_label: Manipuler le DOM avec les Refs
sidebar_position: 21
---

# Chapitre 21 : Manipuler le DOM avec les Refs

Focus, sélection de texte, Animations impératives, Intégration de bibliothèques tierces

React est déclaratif : vous décrivez *ce que* vous voulez voir, et React met à jour le DOM pour vous. Cependant, il y a des situations où vous devez contourner React pour parler directement au navigateur.
Mettre le focus sur un champ, scroller vers une section précise, ou dessiner dans un Canvas sont des tâches impératives qui nécessitent un accès direct aux nœuds du DOM.
Nous utilisons pour cela le Hook `useRef` couplé à l'attribut JSX `ref`.

## Focus et Sélection de Texte {#focus-et-selection-de-texte}

### 1. Quoi
Le "Focus" est l'état d'un élément interactif (champ de saisie, bouton) prêt à recevoir des entrées clavier. La "Sélection" est la mise en surbrillance du texte à l'intérieur d'un input.

### 2. Pourquoi
L'accessibilité et l'UX (Expérience Utilisateur) exigent souvent une gestion fine du focus :
*   Ouvrir une modale doit déplacer le focus à l'intérieur.
*   Valider un formulaire avec une erreur doit placer le focus sur le champ erroné.
*   Cliquer sur un bouton "Copier" peut sélectionner tout le texte pour faciliter le `Ctrl+C`.

### 3. Comment

#### A. Syntaxe de base (Focus)
```tsx
import { useRef } from 'react';

function Form() {
  const inputRef = useRef<HTMLInputElement>(null);

  function handleClick() {
    // 🎯 Impératif : "Mets le focus maintenant !"
    inputRef.current?.focus();
  }

  return (
    <>
      <input ref={inputRef} />
      <button onClick={handleClick}>Focus</button>
    </>
  );
}
```

#### B. Cas concret : Sélection automatique
Imaginez un champ de partage de lien. Au clic, on veut tout sélectionner.

```tsx
export function ShareLink({ url }: { url: string }) {
  const inputRef = useRef<HTMLInputElement>(null);

  const handleCopyClick = () => {
    if (inputRef.current) {
      inputRef.current.select(); // Sélectionne tout le texte
      navigator.clipboard.writeText(url); // Copie dans le presse-papier
      alert("Lien copié !");
    }
  };

  return (
    <div style={{ display: 'flex', gap: 10 }}>
      <input 
        ref={inputRef}
        type="text" 
        value={url} 
        readOnly 
        style={{ width: 300 }}
      />
      <button onClick={handleCopyClick}>Copier</button>
    </div>
  );
}
```

---

## Animations Impératives (Scroll) {#animations-imperatives}

### 1. Quoi
Bien que les animations CSS soient préférables, certaines interactions nécessitent de calculer des positions ou de forcer un défilement (scroll) à un endroit précis de la page.

### 2. Pourquoi
Dans une application de chat, par exemple, vous voulez scroller automatiquement vers le bas quand un nouveau message arrive. Ou scroller vers une section spécifique d'une Landing Page lors du clic sur le menu.

### 3. Comment

#### A. Scroll simple vers un élément
La méthode native `element.scrollIntoView()` est très utile.

```tsx
import { useRef } from 'react';

function LandingPage() {
  const pricingRef = useRef<HTMLDivElement>(null);

  const scrollToPricing = () => {
    pricingRef.current?.scrollIntoView({ 
      behavior: 'smooth', // Animation douce
      block: 'start'      // Aligner en haut de la fenêtre
    });
  };

  return (
    <>
      <nav>
        <button onClick={scrollToPricing}>Voir les Prix</button>
      </nav>
      <div style={{ height: '100vh' }}>Hero Section...</div>
      
      {/* Cible du scroll */}
      <div ref={pricingRef} style={{ height: '100vh', background: '#eee' }}>
        <h2>Nos Tarifs</h2>
      </div>
    </>
  );
}
```

### 4. Zone de Danger : Ne pas abuser
❌ N'utilisez pas de Refs pour animer des éléments si vous pouvez le faire avec CSS ou une librairie d'animation déclarative (comme Framer Motion).
✅ Utilisez les Refs pour le scroll, la lecture vidéo/audio, ou les Canvas.

---

## Intégration de Bibliothèques Tierces {#integration-de-bibliotheques-tierces}

### 1. Quoi
Parfois, vous devez utiliser une librairie JavaScript qui ne connaît pas React (ex: Google Maps vanilla, Chart.js, D3.js, ou un widget jQuery legacy). Ces librairies ont besoin de s'attacher à un élément DOM réel.

### 2. Pourquoi
React travaille sur un DOM Virtuel. Pour laisser une librairie tierce manipuler une partie de l'écran, vous devez lui donner une "ancre" dans le DOM réel via une Ref, et l'empêcher d'être écrasée par React.

### 3. Comment

Pattern standard :
1.  Créer une `div` avec une `ref`.
2.  Utiliser `useEffect` pour initialiser la librairie *après* que le composant soit monté (quand la `div` existe).
3.  Utiliser la fonction de nettoyage de `useEffect` pour détruire l'instance de la librairie.

#### Exemple : Une fausse carte interactive

```tsx
import { useRef, useEffect } from 'react';

// Imaginons une classe externe "MyMapLib"
class MyMapLib {
  constructor(domNode: HTMLElement) {
    domNode.innerText = "🗺 CARTE CHARGÉE";
    domNode.style.backgroundColor = "#e0f7fa";
  }
  destroy() {
    console.log("Carte nettoyée");
  }
}

export function MapComponent() {
  const mapContainerRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    // La div est prête dans le DOM
    if (mapContainerRef.current) {
      const mapInstance = new MyMapLib(mapContainerRef.current);
      
      // Nettoyage impératif lors du démontage
      return () => {
        mapInstance.destroy();
      };
    }
  }, []); // [] = Seulement au montage

  // React rend une div vide et ne s'en occupe plus
  return <div ref={mapContainerRef} style={{ width: 300, height: 200 }} />;
}
```

### 🚨 Limitations
Si la librairie tierce modifie le DOM, React ne le saura pas. Si React essaie ensuite de mettre à jour ce même DOM, il y aura des conflits (erreurs "NotFoundError").
**Règle** : Si une Ref est gérée par une lib tierce, React ne doit jamais toucher à ses enfants (`<div ref={...}></div>` doit rester vide dans le JSX).

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-21}

1.  **Pourquoi utiliser `ref.current.focus()` au lieu d'une prop `autoFocus` ?**
    `autoFocus` ne fonctionne qu'au chargement initial. Pour déplacer le focus suite à une action utilisateur (clic bouton, erreur validation) sans recharger la page, il faut une commande impérative via Ref.

2.  **Quelle méthode DOM permet de scroller vers un élément ?**
    `element.scrollIntoView({ behavior: 'smooth' })`.

3.  **Pourquoi a-t-on besoin de `useEffect` pour intégrer une librairie tierce avec des Refs ?**
    Parce que les Refs vers le DOM (`ref.current`) sont vides (`null`) au moment du rendu initial. `useEffect` s'exécute *après* que React a créé les éléments DOM, garantissant que la librairie tierce trouve bien sa cible.

4.  **Si je modifie le style d'un élément via `ref.current.style.color = 'red'`, est-ce que le composant se re-rend ?**
    Non. C'est une modification directe du DOM qui contourne le cycle de vie de React. C'est déconseillé pour la gestion d'état visuel principale, mais acceptable pour des cas très spécifiques (performance, animation complexe).

---

## Exercices : {#exercices-21}

### Exercice 1 - Le Formulaire Intelligent {#exercice-1---le-formulaire-intelligent}

🎯 **Objectif** : Gérer le focus de manière ergonomique.

💼 **Mise en situation** : Un formulaire de login à 3 champs. Quand l'utilisateur appuie sur "Entrée" dans le champ Login, le focus doit passer au champ Password. Quand il appuie sur "Entrée" dans Password, le formulaire doit se soumettre.

📝 **Énoncé** :
1. Deux inputs : Login, Password.
2. Un bouton "Se connecter".
3. Sur l'input Login, gérez l'événement `onKeyDown`. Si `e.key === 'Enter'`, déplacez le focus sur l'input Password via une Ref.
4. Sur l'input Password, si `Enter`, lancez une `alert("Connexion...")`.

📺 **Résultat attendu** :
L'utilisateur peut naviguer et valider sans jamais toucher sa souris.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useRef, KeyboardEvent } from 'react';

export function SmartLoginForm() {
  // On a besoin d'une référence vers le champ mot de passe
  const passwordRef = useRef<HTMLInputElement>(null);

  const handleLoginKeyDown = (e: KeyboardEvent<HTMLInputElement>) => {
    if (e.key === 'Enter') {
      e.preventDefault(); // Évite le submit par défaut si dans un form
      // 🎯 Focus impératif sur le champ suivant
      passwordRef.current?.focus();
    }
  };

  const handlePasswordKeyDown = (e: KeyboardEvent<HTMLInputElement>) => {
    if (e.key === 'Enter') {
      alert("Tentative de connexion...");
    }
  };

  return (
    <div style={{ display: 'flex', flexDirection: 'column', gap: 10, maxWidth: 300 }}>
      <input 
        type="text" 
        placeholder="Login" 
        onKeyDown={handleLoginKeyDown}
      />
      <input 
        ref={passwordRef} // Liaison de la ref
        type="password" 
        placeholder="Mot de passe" 
        onKeyDown={handlePasswordKeyDown}
      />
      <button onClick={() => alert("Connexion...")}>Se connecter</button>
    </div>
  );
}
```
</details>

### Exercice 2 - Le Carrousel Scrollable {#exercice-2---le-carrousel-scrollable}

🎯 **Objectif** : Utiliser `scrollIntoView` pour naviguer dans une liste horizontale.

💼 **Mise en situation** : Une galerie d'images type Netflix. Une liste horizontale d'images qui dépasse de l'écran. Des boutons "Image 1", "Image 5", "Image 10" permettent de sauter directement à l'image correspondante.

📝 **Énoncé** :
1. Créez une liste de 10 images (ou div colorées) dans un conteneur `overflow-x: scroll`.
2. Utilisez une Ref de type `Map` (comme vu au Chapitre 20) pour stocker les références de chaque élément de la liste.
3. Créez 3 boutons en haut. Au clic, scrollez l'élément correspondant au centre (`block: 'center', inline: 'center'`).

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Une rangée de carrés colorés numérotés avec une barre de défilement horizontale.
> **Annotation** : Montrez le scroll centré sur le carré numéro 5.
> **Alt Text suggéré** : Interface de carrousel avec défilement horizontal piloté par boutons.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useRef } from 'react';

export function Carousel() {
  // Stockage dynamique des refs
  const itemsRef = useRef<Map<number, HTMLDivElement>>(null);

  function getMap() {
    if (!itemsRef.current) {
      itemsRef.current = new Map();
    }
    return itemsRef.current;
  }

  const scrollToId = (id: number) => {
    const map = getMap();
    const node = map.get(id);
    node?.scrollIntoView({ 
      behavior: 'smooth', 
      inline: 'center', // Centre horizontalement
      block: 'nearest' 
    });
  };

  // Génération de 10 items
  const items = Array.from({ length: 10 }, (_, i) => i);

  return (
    <div>
      <div style={{ marginBottom: 10 }}>
        <button onClick={() => scrollToId(0)}>Début</button>
        <button onClick={() => scrollToId(5)}>Milieu (5)</button>
        <button onClick={() => scrollToId(9)}>Fin</button>
      </div>

      <div style={{ 
        display: 'flex', 
        gap: 20, 
        overflowX: 'auto', 
        padding: 20, 
        border: '1px solid #ccc' 
      }}>
        {items.map(id => (
          <div
            key={id}
            ref={(node) => {
              const map = getMap();
              if (node) map.set(id, node);
              else map.delete(id);
            }}
            style={{
              minWidth: 150,
              height: 150,
              backgroundColor: `hsl(${id * 30}, 70%, 50%)`,
              display: 'flex',
              alignItems: 'center',
              justifyContent: 'center',
              fontSize: '2rem',
              color: 'white',
              borderRadius: 10
            }}
          >
            {id}
          </div>
        ))}
      </div>
    </div>
  );
}
```
</details>

### Exercice 3 - Le Lecteur Vidéo Autoplay (Visible) {#exercice-3---le-lecteur-video-autoplay}

🎯 **Objectif** : Détecter la visibilité pour déclencher une action impérative.

💼 **Mise en situation** : Un fil d'actualité type réseau social. Les vidéos doivent se lancer (`play()`) uniquement quand elles sont visibles à l'écran, et se mettre en pause (`pause()`) quand elles sortent.

*Note : Pour simplifier, nous utiliserons un bouton "Simuler Visibilité" au lieu de l'Intersection Observer complexe, mais la logique de Ref reste la même.*

📝 **Énoncé** :
1. Un composant `AutoVideo` qui prend une prop `isPlaying`.
2. Un `useEffect` dans ce composant : si `isPlaying` est true -> `ref.play()`, sinon `ref.pause()`.
3. Le Parent affiche la vidéo et un bouton "Toggle Play/Pause" qui passe le booléen.

📺 **Résultat attendu** :
La vidéo réagit aux changements de props en appelant ses méthodes impératives internes.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useRef, useEffect, useState } from 'react';

// Composant Enfant : Encapsule la logique impérative
function AutoVideo({ isPlaying }: { isPlaying: boolean }) {
  const videoRef = useRef<HTMLVideoElement>(null);

  // Synchronisation : Prop déclarative -> Action impérative
  useEffect(() => {
    if (isPlaying) {
      videoRef.current?.play().catch(e => console.log("Autoplay bloqué par le navigateur"));
    } else {
      videoRef.current?.pause();
    }
  }, [isPlaying]); // Dépendance : s'exécute quand isPlaying change

  return (
    <video 
      ref={videoRef}
      src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
      width="300"
      muted // Muted est souvent requis pour l'autoplay
      loop
      style={{ borderRadius: 10 }}
    />
  );
}

// Composant Parent
export function Feed() {
  const [playing, setPlaying] = useState(false);

  return (
    <div>
      <h3>Fil d'actualité</h3>
      <div style={{ border: '1px solid #ddd', padding: 10, borderRadius: 8 }}>
        <AutoVideo isPlaying={playing} />
        <div style={{ marginTop: 10 }}>
          <label>
            <input 
              type="checkbox" 
              checked={playing} 
              onChange={e => setPlaying(e.target.checked)} 
            />
            {playing ? " Lecture en cours" : " En pause"}
          </label>
        </div>
      </div>
    </div>
  );
}
```
</details>
```