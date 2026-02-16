Voici le chapitre **`useTransition` et `startTransition`: Mises à Jour non Bloquantes** pour la formation React 19.2.

```markdown
---
sidebar_label: `useTransition` et `startTransition`
sidebar_position: 35
---

# Chapitre 35 : `useTransition` et `startTransition`: Mises à Jour non Bloquantes

Transitions UI, Priorités de rendu, Amélioration de la réactivité

Dans le chapitre précédent, nous avons vu `useDeferredValue` pour gérer des valeurs qui arrivent "trop vite" (comme la frappe au clavier).
Mais que faire quand c'est **vous** (le développeur) qui déclenchez une action lourde (comme changer d'onglet, naviguer vers une page complexe, ou appliquer un filtre global) ?

Par défaut, toutes les mises à jour d'état dans React sont considérées comme **urgentes**. React bloque le navigateur jusqu'à ce que le rendu soit terminé.
Avec `useTransition`, vous pouvez marquer certaines mises à jour comme **non urgentes** (transitions). Cela permet à React d'interrompre le rendu de cette transition si l'utilisateur fait quelque chose de plus important (comme cliquer ailleurs).

## Le Concept de Priorité : Urgent vs Transition {#le-concept-de-priorite}

### 1. Quoi
React distingue désormais deux types de mises à jour :
1.  **Urgentes** : Interactions directes (taper, cliquer, survoler). Elles doivent être instantanées pour que l'interface semble "physique".
2.  **Transitions** : Changements de vue (changer d'onglet, de page, charger des graphiques). L'utilisateur s'attend à un léger délai.

`useTransition` est un Hook qui vous donne le contrôle pour classer une mise à jour dans la catégorie "Transition".

### 2. Pourquoi
Si l'utilisateur clique sur un bouton "Onglet B" qui contient un graphique très lourd :
*   **Sans Transition** : Le bouton reste enfoncé, l'interface gèle pendant 500ms, rien ne bouge, puis tout apparaît d'un coup. L'utilisateur croit que l'app a planté.
*   **Avec Transition** : Le bouton réagit immédiatement (clic). L'ancien contenu reste affiché et interactif le temps que le nouveau soit prêt. React calcule le nouvel onglet en arrière-plan.

### 3. Comment

#### A. Syntaxe de base

```tsx
import { useTransition, useState } from 'react';

function TabContainer() {
  const [isPending, startTransition] = useTransition();
  const [tab, setTab] = useState('home');

  const selectTab = (nextTab: string) => {
    // On enveloppe le changement d'état "lourd"
    startTransition(() => {
      setTab(nextTab);
    });
  };

  return (
    <>
      <button onClick={() => selectTab('settings')}>
        Settings {isPending && '⌛'}
      </button>
      {/* Le contenu de l'onglet changera sans bloquer l'UI */}
      <TabContent tab={tab} />
    </>
  );
}
```

#### B. Nouveauté React 19 : Support Asynchrone
En React 19, `startTransition` supporte désormais les fonctions asynchrones (promesses). C'est la base des **Server Actions**. `isPending` restera `true` tant que la promesse n'est pas résolue.

```tsx
// Exemple conceptuel React 19
const updateProfile = () => {
  startTransition(async () => {
    await saveToDatabase(data); // isPending est true pendant l'attente
    // Une fois fini, React met à jour l'UI
  });
};
```

### 4. Zone de Danger

:::danger Ne pas utiliser pour les Inputs contrôlés
N'utilisez jamais `startTransition` pour mettre à jour un champ texte (`<input value={text} />`).
Taper au clavier est une action **urgente**. Si vous la différez, l'utilisateur verra ses caractères apparaître avec du retard.
Utilisez `useTransition` pour les *conséquences* de la frappe (ex: filtrer une liste), pas pour l'affichage du champ lui-même.
:::

---

## Comparatif : `useTransition` vs `useDeferredValue` {#comparatif-usetransition-vs-usedeferredvalue}

Ces deux Hooks utilisent le même moteur "Concurrent" de React, mais s'utilisent différemment.

| Critère | `useTransition` | `useDeferredValue` |
| :--- | :--- | :--- |
| **Qui a le contrôle ?** | **Vous** (au moment du `setState`) | **Le composant enfant** (qui reçoit des props) |
| **Accès à l'état "Pending"** | **Oui** (`isPending` est fourni) | **Non** (il faut le déduire soi-même) |
| **Cas d'usage typique** | Clics, Navigations, Actions serveur | Inputs texte qui alimentent une liste |
| **Mentalité** | "Exécute cette action en arrière-plan" | "Reçois cette valeur plus tard si nécessaire" |

---

## Pattern : Navigation Optimiste {#pattern-navigation-optimiste}

### 1. Quoi
Lors d'un changement de vue lourd, au lieu d'afficher un gros spinner blanc ("Loading..."), on laisse l'ancienne vue visible jusqu'à ce que la nouvelle soit prête, tout en indiquant un chargement discret dans le menu.

### 2. Pourquoi
Cela évite le "layout shift" (sauts d'interface) et garde l'application utilisable. Si l'utilisateur change d'avis pendant le chargement, il peut cliquer ailleurs car l'interface n'est pas bloquée.

### 3. Comment

```tsx
import { useState, useTransition, memo } from 'react';

// Un composant très lent
const HeavyPage = memo(() => {
  const start = performance.now();
  while (performance.now() - start < 200) {} // Lag 200ms
  return <div className="page">Contenu Lourd Chargé</div>;
});

export function AppRouter() {
  const [page, setPage] = useState('Home');
  const [isPending, startTransition] = useTransition();

  const navigate = (nextPage: string) => {
    startTransition(() => {
      setPage(nextPage); // Cette mise à jour est interruptible
    });
  };

  return (
    <div>
      <nav>
        <button onClick={() => navigate('Home')}>Accueil</button>
        <button onClick={() => navigate('Profile')}>
          Profil {isPending && <span className="spinner">🔄</span>}
        </button>
      </nav>

      <div style={{ opacity: isPending ? 0.5 : 1, transition: 'opacity 0.2s' }}>
        {page === 'Home' ? <h1>Accueil</h1> : <HeavyPage />}
      </div>
    </div>
  );
}
```

### 🚨 Limitations de `useTransition`
1.  **Pas d'animation** : `useTransition` ne fait pas d'animation CSS magique. Il gère juste le timing de la mise à jour React.
2.  **État synchrone** : Si vous avez plusieurs `setState` dans la transition, ils seront tous regroupés (batchés) et traités comme non urgents.

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-35}

1.  **Quelle est la différence fondamentale entre une mise à jour urgente et une transition ?**
    Une mise à jour urgente bloque le thread principal pour être immédiate (réactivité physique). Une transition est calculée en arrière-plan et peut être interrompue ou retardée.

2.  **Que contient la variable `isPending` retournée par `useTransition` ?**
    Un booléen (`true` ou `false`) indiquant si React est actuellement en train de calculer la transition demandée.

3.  **Pourquoi ne pas utiliser `startTransition` sur un `onChange` d'un input texte contrôlé ?**
    Car l'utilisateur doit voir ce qu'il tape instantanément. Retarder l'affichage des caractères crée une sensation de latence insupportable.

---

## Exercices : {#exercices-35}

### Exercice 1 - Le Switch d'Onglets Laggy {#exercice-1---le-switch-d-onglets-laggy}

🎯 **Objectif** : Rendre une interface réactive même avec des composants enfants lents.

💼 **Mise en situation** : Un dashboard RH possède trois onglets : "Employés" (rapide), "Salaires" (lent), "Statistiques" (très lent). Quand on clique sur "Statistiques", le bouton semble cassé car rien ne se passe pendant 300ms.

📝 **Énoncé** :
1. Créez 3 composants (Fast, Slow, VerySlow) simulant des charges CPU différentes.
2. Créez un menu de navigation.
3. Implémentez la navigation **sans** `useTransition` : constatez le gel du bouton.
4. Implémentez la navigation **avec** `useTransition` : le bouton doit réagir instantanément (effet visuel de clic), et l'onglet doit changer après le délai.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Le menu d'onglets.
> **Annotation** : Montrez l'état "Actif" sur l'onglet cliqué alors que le contenu en dessous est encore l'ancien (avec une opacité réduite éventuellement).
> **Alt Text suggéré** : Démonstration d'une transition non bloquante.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState, useTransition, memo } from 'react';

// Composants simulés
const FastTab = () => <div>⚡ Onglet Rapide</div>;

const SlowTab = memo(() => {
  const start = performance.now();
  while (performance.now() - start < 100) {} // 100ms
  return <div>🐢 Onglet Lent</div>;
});

const VerySlowTab = memo(() => {
  const start = performance.now();
  while (performance.now() - start < 500) {} // 500ms !
  return <div>🐌 Onglet Très Lent</div>;
});

export function HRDashboard() {
  const [tab, setTab] = useState('fast');
  const [isPending, startTransition] = useTransition();

  const handleTabChange = (nextTab: string) => {
    // Sans startTransition, le bouton "Statistiques" resterait enfoncé sans effet pendant 500ms
    startTransition(() => {
      setTab(nextTab);
    });
  };

  return (
    <div style={{ padding: 20 }}>
      <div style={{ display: 'flex', gap: 10, marginBottom: 20 }}>
        <button 
          onClick={() => handleTabChange('fast')}
          style={{ fontWeight: tab === 'fast' ? 'bold' : 'normal' }}
        >
          Employés
        </button>
        <button 
          onClick={() => handleTabChange('slow')}
          style={{ fontWeight: tab === 'slow' ? 'bold' : 'normal' }}
        >
          Salaires
        </button>
        <button 
          onClick={() => handleTabChange('veryslow')}
          style={{ fontWeight: tab === 'veryslow' ? 'bold' : 'normal' }}
        >
          Statistiques {isPending && '⌛'}
        </button>
      </div>

      <div style={{ opacity: isPending ? 0.5 : 1 }}>
        {tab === 'fast' && <FastTab />}
        {tab === 'slow' && <SlowTab />}
        {tab === 'veryslow' && <VerySlowTab />}
      </div>
    </div>
  );
}
```
</details>

### Exercice 2 - Filtre de Galerie Photo {#exercice-2---filtre-de-galerie-photo}

🎯 **Objectif** : Différencier l'état urgent (bouton actif) de l'état transitionnel (liste filtrée).

💼 **Mise en situation** : Une galerie de 5000 photos. Des boutons de filtre ("Nature", "Ville", "Personnes") permettent de trier. Le clic sur le filtre doit être immédiat pour que l'utilisateur sache qu'il a cliqué, même si le tri prend du temps.

📝 **Énoncé** :
1. Créez un état `filter` et une liste d'éléments lourde à filtrer.
2. Utilisez `useTransition` sur les boutons de filtre.
3. Affichez un message "Mise à jour de la galerie..." pendant la transition.
4. Constatez que vous pouvez cliquer frénétiquement sur différents filtres sans que l'interface ne se bloque complètement.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState, useTransition, memo } from 'react';

// Génération de fausses données
const PHOTOS = Array.from({ length: 5000 }, (_, i) => ({
  id: i,
  category: ['Nature', 'City', 'People'][i % 3],
  name: `Photo #${i}`
}));

const PhotoList = memo(({ category }: { category: string }) => {
  // Simulation filtre lourd
  const start = performance.now();
  while (performance.now() - start < 50) {} // Petit lag cumulatif
  
  const filtered = PHOTOS.filter(p => category === 'All' || p.category === category);
  
  return (
    <ul style={{ height: 200, overflow: 'auto', border: '1px solid #ccc' }}>
      {filtered.map(p => <li key={p.id}>{p.name} ({p.category})</li>)}
    </ul>
  );
});

export function GalleryFilter() {
  const [category, setCategory] = useState('All');
  const [isPending, startTransition] = useTransition();

  const selectCategory = (cat: string) => {
    startTransition(() => {
      setCategory(cat);
    });
  };

  return (
    <div>
      <h3>Galerie Photo</h3>
      <div style={{ marginBottom: 10 }}>
        {['All', 'Nature', 'City', 'People'].map(cat => (
          <button 
            key={cat} 
            onClick={() => selectCategory(cat)}
            // Le bouton reflète l'état IMMEDIATEMENT (via une prop ou style local), 
            // ou bien on attend que la transition soit finie pour changer le style 'active'.
            // Ici, category n'est mis à jour qu'à la fin de la transition.
            // Pour un feedback immédiat sur le bouton, on pourrait avoir deux états : 
            // 1. visualCategory (immédiat) 2. actualCategory (transition)
            disabled={isPending && category === cat}
            style={{ 
               background: category === cat ? 'blue' : 'gray', 
               color: 'white', margin: 5 
            }}
          >
            {cat}
          </button>
        ))}
      </div>

      {isPending && <p style={{ color: 'orange' }}>Mise à jour du tri...</p>}
      
      <div style={{ opacity: isPending ? 0.3 : 1 }}>
        <PhotoList category={category} />
      </div>
    </div>
  );
}
```
</details>

### Exercice 3 - React 19 Async Transition (Simulation) {#exercice-3---react-19-async-transition}

🎯 **Objectif** : Comprendre comment `startTransition` gère l'asynchrone.

💼 **Mise en situation** : Un bouton "Sauvegarder" déclenche une API. On veut gérer l'état de chargement sans créer manuellement un `useState(isLoading)`.

📝 **Énoncé** :
1. Créez une fonction `fakeApiCall` qui retourne une promesse résolue après 2 secondes.
2. Dans le composant, utilisez `useTransition`.
3. Au clic du bouton, lancez `startTransition(async () => { await fakeApiCall(); })`.
4. Utilisez `isPending` pour désactiver le bouton et changer son texte.
5. Notez comment nous n'avons **pas** besoin de créer manuellement `const [loading, setLoading] = useState(false)`.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState, useTransition } from 'react';

// API simulée
const fakeApiCall = () => new Promise<void>(resolve => setTimeout(resolve, 2000));

export function SaveButton() {
  // Pas besoin de state 'isLoading' manuel !
  const [isPending, startTransition] = useTransition();
  const [message, setMessage] = useState("Prêt");

  const handleSave = () => {
    // React 19 permet de passer une fonction async ici
    startTransition(async () => {
      // isPending passe à true automatiquement
      await fakeApiCall();
      
      // Après 2 secondes, on exécute ceci :
      setMessage("Sauvegarde terminée ✅");
      // isPending repasse à false automatiquement
    });
  };

  return (
    <div style={{ padding: 20, border: '1px solid #ddd' }}>
      <p>État : {message}</p>
      
      <button 
        onClick={handleSave} 
        disabled={isPending}
        style={{ cursor: isPending ? 'wait' : 'pointer' }}
      >
        {isPending ? 'Sauvegarde en cours...' : 'Sauvegarder'}
      </button>
    </div>
  );
}
```
</details>
```