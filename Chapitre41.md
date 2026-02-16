Voici le chapitre **Composants React Intégrés (Canary): `<Activity>`, `<ViewTransition>`** pour la formation React 19.2.

```markdown
---
sidebar_label: Composants React Intégrés (Canary): `<Activity>`, `<ViewTransition>`
sidebar_position: 41
---

# Chapitre 41 : Composants React Intégrés (Canary): `<Activity>`, `<ViewTransition>`

Gestion des activités, Transitions visuelles d'UI, API expérimentales

:::warning Fonctionnalités Canary / Expérimentales
Ce chapitre traite de composants qui sont actuellement au stade **Canary** ou **Expérimental** dans React 19.
Leur API peut changer. Ils ne sont pas recommandés pour la production immédiate sans une compréhension approfondie des risques.
:::

React continue d'évoluer pour offrir une expérience utilisateur (UX) comparable aux applications natives. Deux domaines majeurs sont ciblés :
1.  **La persistance d'état** : Garder un écran "en vie" même s'il n'est plus visible (ex: onglets).
2.  **Les transitions fluides** : Animer le passage d'une vue à une autre de manière organique.

C'est ici qu'interviennent `<Activity>` (parfois appelé "Offscreen") et l'intégration des View Transitions.

---

## 1. `<Activity>` (Gestion de visibilité "Offscreen") {#activity}

### 1. Quoi
`<Activity>` est un composant qui permet de masquer une partie de l'arbre des composants (la rendre invisible) **sans la démonter** (unmount).

Contrairement à un rendu conditionnel (`isOpen && <Component />`) qui détruit l'état, ou à du CSS (`display: none`) qui continue de subir les mises à jour de rendu, `<Activity>` "gèle" le composant : il conserve son état (scroll, inputs, state local) mais cesse de se mettre à jour jusqu'à ce qu'il redevienne visible.

### 2. Pourquoi
*   **Préservation de l'état** : L'utilisateur tape dans un formulaire, change d'onglet, et revient : le texte est toujours là.
*   **Performance** : React arrête de calculer les mises à jour pour les composants cachés ("dépriorisation"), libérant des ressources pour l'écran visible.
*   **Navigation instantanée** : Le composant étant déjà monté en mémoire, le réafficher est immédiat.

### 3. Comment

:::info Note d'API
Dans certaines versions expérimentales, ce composant est accessible via `unstable_Activity` ou simplement `Activity` selon le build.
:::

#### A. Syntaxe de base

```tsx
import { unstable_Activity as Activity } from 'react';

function TabContainer({ mode }: { mode: 'visible' | 'hidden' }) {
  return (
    <Activity mode={mode}>
      <HeavyComponent />
      <MyForm />
    </Activity>
  );
}
```

#### B. Cas concret : Système d'Onglets

```tsx
import { useState, unstable_Activity as Activity } from 'react';

function TabContent({ name }: { name: string }) {
  // Cet état sera préservé même quand l'onglet est caché !
  const [count, setCount] = useState(0);
  
  return (
    <div style={{ padding: 20, border: '1px solid #ccc' }}>
      <h3>{name}</h3>
      <p>Compteur : {count}</p>
      <button onClick={() => setCount(c => c + 1)}>Incrémenter</button>
    </div>
  );
}

export function Tabs() {
  const [activeTab, setActiveTab] = useState('A');

  return (
    <div>
      <div style={{ display: 'flex', gap: 10, marginBottom: 10 }}>
        <button onClick={() => setActiveTab('A')}>Onglet A</button>
        <button onClick={() => setActiveTab('B')}>Onglet B</button>
      </div>

      {/* 
         Les deux composants sont montés simultanément.
         Mais un seul est "visible" et actif pour React.
         L'autre est "dormant" mais garde son état.
      */}
      
      <Activity mode={activeTab === 'A' ? 'visible' : 'hidden'}>
        <TabContent name="Contenu A" />
      </Activity>

      <Activity mode={activeTab === 'B' ? 'visible' : 'hidden'}>
        <TabContent name="Contenu B" />
      </Activity>
    </div>
  );
}
```

### 🚨 Limitations de `<Activity>`
1.  **Consommation Mémoire** : Puisque les composants ne sont pas détruits, ils occupent de la RAM. N'utilisez pas cela pour des listes infinies cachées.
2.  **Effets de bord** : Les `useEffect` ne sont pas nettoyés quand le composant passe en `hidden`, mais ils peuvent ne pas s'exécuter non plus. React pourrait introduire un hook `useActivityEffect` pour gérer l'apparition/disparition logique.

---

## 2. `<ViewTransition>` (Transitions de Vue) {#view-transition}

### 1. Quoi
C'est un composant (ou une intégration via `startTransition`) qui connecte le cycle de rendu de React à l'API native du navigateur **View Transitions API**.
Il permet de créer des animations fluides entre deux états du DOM (ex: une image qui s'agrandit pour devenir un header) sans écrire de CSS complexe ou de bibliothèques d'animation JS lourdes.

### 2. Pourquoi
Pour donner une sensation "native" et fluide ("Morphing"). Au lieu qu'un écran saute brutalement à l'autre, les éléments communs glissent à leur nouvelle place.

### 3. Comment

React 19 intègre le support automatique des View Transitions via `startTransition` ou un composant dédié (API en flux). L'idée est de dire à React : "Cette mise à jour d'état doit être animée".

#### A. Syntaxe Conceptuelle (via `startTransition`)

C'est l'approche la plus courante dans React 19 actuellement. On étend `startTransition` pour utiliser l'API navigateur.

```tsx
import { startTransition } from 'react';
// ou import { useTransition } from 'react';

function App() {
  const [isDetail, setIsDetail] = useState(false);

  const toggleView = () => {
    // Si le navigateur supporte l'API
    if (document.startViewTransition) {
      document.startViewTransition(() => {
        // On demande à React de faire la mise à jour (flushSync-like behavior wrapped)
        flushSync(() => {
           setIsDetail(d => !d);
        });
      });
    } else {
      setIsDetail(d => !d);
    }
  };
  
  // ...
}
```

#### B. Avec le composant `<ViewTransition>` (Expérimental)

React explore un composant déclaratif pour isoler les transitions sur une partie de l'arbre.

```tsx
import { unstable_ViewTransition as ViewTransition } from 'react';

function ImageGallery() {
  const [big, setBig] = useState(false);

  return (
    <div>
      <button onClick={() => setBig(!big)}>Toggle</button>
      
      {/* 
        Tout ce qui change à l'intérieur sera capturé par l'API View Transition 
        On utilise la propriété CSS `view-transition-name` pour lier les éléments
      */}
      <ViewTransition>
        <img 
          src="avatar.jpg" 
          style={{ 
            width: big ? 300 : 50,
            // Clé magique pour le morphing
            viewTransitionName: 'avatar-image' 
          }} 
        />
      </ViewTransition>
    </div>
  );
}
```

### 4. Zone de Danger

*   **Support Navigateur** : L'API View Transitions n'est pas supportée par tous les navigateurs (principalement Chromium et Safari récent). Il faut toujours prévoir un "graceful degradation".
*   **Noms Uniques** : La propriété CSS `view-transition-name` doit être **unique** sur la page à un instant T. Sinon, l'animation échoue ou bug.

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-41}

1.  **Quelle est la différence majeure entre `<Activity mode="hidden">` et `style={{ display: 'none' }}` ?**
    Avec `display: none`, React continue de rendre le composant et de mettre à jour le DOM (c'est juste invisible). Avec `<Activity>`, React "gèle" le composant : il arrête les mises à jour de rendu pour économiser le CPU, tout en gardant l'état en mémoire.

2.  **Pourquoi utiliser `<Activity>` pour des onglets ?**
    Pour que l'utilisateur ne perde pas ce qu'il a saisi (scroll, formulaires) en changeant d'onglet, et pour que le retour sur l'onglet soit instantané (pas de re-montage).

3.  **Quel est le prérequis CSS pour qu'une View Transition "morphe" un élément A en élément B ?**
    Les deux éléments (avant et après changement d'état) doivent avoir la même propriété CSS `view-transition-name` unique.

---

## Exercices : {#exercices-41}

:::warning Environnement
Pour ces exercices, assurez-vous d'utiliser une version de React (Canary/Next.js récent) et un navigateur (Chrome/Edge) qui supportent ces fonctionnalités.
:::

### Exercice 1 - Le Formulaire Caché (Activity) {#exercice-1---le-formulaire-cache}

🎯 **Objectif** : Prouver la persistance d'état avec `<Activity>`.

💼 **Mise en situation** : Un utilisateur remplit un long formulaire de contact. Il clique par erreur sur "Infos Légales" (autre vue). Quand il revient sur "Contact", ses données ne doivent pas avoir disparu.

📝 **Énoncé** :
1. Créez deux composants : `ContactForm` (avec un input) et `LegalInfo` (texte statique).
2. Utilisez un state `view` ('contact' | 'legal').
3. Affichez les DEUX composants dans des balises `<Activity>`.
4. Gérez la prop `mode` selon la vue active.
5. Test : Tapez du texte, changez de vue, revenez. Le texte doit être là.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState, unstable_Activity as Activity } from 'react';

function ContactForm() {
  // État local simple
  const [text, setText] = useState('');
  
  return (
    <div>
      <h3>Contactez-nous</h3>
      <textarea 
        value={text} 
        onChange={e => setText(e.target.value)} 
        placeholder="Votre message..."
        rows={5}
      />
    </div>
  );
}

function LegalInfo() {
  return (
    <div>
      <h3>Mentions Légales</h3>
      <p>Lorem ipsum dolor sit amet...</p>
    </div>
  );
}

export function AppRouter() {
  const [view, setView] = useState<'contact' | 'legal'>('contact');

  return (
    <div style={{ padding: 20 }}>
      <nav style={{ marginBottom: 20, display: 'flex', gap: 10 }}>
        <button onClick={() => setView('contact')}>Contact</button>
        <button onClick={() => setView('legal')}>Mentions Légales</button>
      </nav>

      {/* 
         Les deux vues existent tout le temps.
         Activity gère leur priorité CPU et leur visibilité.
      */}
      <Activity mode={view === 'contact' ? 'visible' : 'hidden'}>
        <ContactForm />
      </Activity>

      <Activity mode={view === 'legal' ? 'visible' : 'hidden'}>
        <LegalInfo />
      </Activity>
    </div>
  );
}
```
</details>

### Exercice 2 - La Carte Morphing (ViewTransition) {#exercice-2---la-carte-morphing}

🎯 **Objectif** : Créer une transition fluide entre une liste et un détail.

💼 **Mise en situation** : Une galerie photo. Cliquer sur une miniature l'agrandit en plein écran de manière fluide.

📝 **Énoncé** :
1. Créez une liste de 3 items.
2. Au clic sur un item, mettez à jour un état `selectedId`.
3. Utilisez l'API `document.startViewTransition` (si disponible) lors du `onClick`.
4. Ajoutez dynamiquement `style={{ viewTransitionName: 'hero-image' }}` sur l'image cliquée ET sur l'image agrandie.
5. Observez la magie du navigateur.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Animation en cours (difficile à capturer en image fixe, mais essayez).
> **Annotation** : Montrez l'état "intermédiaire" où l'image est entre sa petite et sa grande taille.
> **Alt Text suggéré** : Transition morphing d'une image React.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState, flushSync } from 'react';

// Données mockées
const IMAGES = [
  { id: 1, color: 'red' },
  { id: 2, color: 'blue' },
  { id: 3, color: 'green' },
];

export function Gallery() {
  const [selectedId, setSelectedId] = useState<number | null>(null);

  const handleSelect = (id: number | null) => {
    // Vérification support navigateur
    if (!document.startViewTransition) {
      setSelectedId(id);
      return;
    }

    // On démarre la transition
    document.startViewTransition(() => {
      // flushSync force React à mettre à jour le DOM MAINTENANT
      // pour que le navigateur puisse capturer le "nouvel" état immédiatement
      flushSync(() => {
        setSelectedId(id);
      });
    });
  };

  // VUE DÉTAIL
  if (selectedId !== null) {
    const item = IMAGES.find(i => i.id === selectedId)!;
    return (
      <div onClick={() => handleSelect(null)} style={{ cursor: 'zoom-out', padding: 20 }}>
        <h1>Vue Détail</h1>
        <div 
          style={{ 
            width: 300, 
            height: 300, 
            background: item.color,
            // Le nom doit correspondre à celui de la miniature
            viewTransitionName: 'active-card' 
          }} 
        />
        <p>Cliquez pour fermer</p>
      </div>
    );
  }

  // VUE LISTE
  return (
    <div style={{ display: 'flex', gap: 20, padding: 20 }}>
      {IMAGES.map(item => (
        <div
          key={item.id}
          onClick={() => handleSelect(item.id)}
          style={{
            width: 100,
            height: 100,
            background: item.color,
            cursor: 'zoom-in',
            // Astuce : on ne donne le nom de transition qu'à l'élément qui va bouger
            // Sinon, il y aurait des doublons de noms, ce qui est interdit
            viewTransitionName: 'active-card' // Simplification pour l'exercice
          }}
        >
          Image {item.id}
        </div>
      ))}
    </div>
  );
}
```
</details>

### Exercice 3 - Le Toggle Lourd (Activity vs Conditional) {#exercice-3---le-toggle-lourd}

🎯 **Objectif** : Comparer la vitesse de ré-affichage.

💼 **Mise en situation** : Un composant `HeavyChart` prend 500ms à s'initialiser (boucle lourde). On veut pouvoir le masquer/afficher instantanément.

📝 **Énoncé** :
1. Créez un composant `Heavy` qui fait une boucle `while` de 200ms au montage (`useEffect`).
2. Créez deux boutons : "Toggle (Conditional)" et "Toggle (Activity)".
3. "Conditional" utilise `isOpen && <Heavy />`.
4. "Activity" utilise `<Activity mode={...}><Heavy /></Activity>`.
5. Constatez le "lag" avec la méthode conditionnelle et la fluidité avec Activity.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState, useEffect, unstable_Activity as Activity } from 'react';

function HeavyComponent() {
  useEffect(() => {
    // Simulation d'un travail lourd au montage
    const start = performance.now();
    while (performance.now() - start < 200) {
      // Bloque le thread principal pendant 200ms
    }
    console.log("HeavyComponent monté !");
  }, []);

  return <div style={{ background: 'orange', padding: 20 }}>Je suis lourd 🐘</div>;
}

export function PerformanceTest() {
  const [showCond, setShowCond] = useState(false);
  const [showAct, setShowAct] = useState(false);

  return (
    <div style={{ display: 'grid', gridTemplateColumns: '1fr 1fr', gap: 20 }}>
      
      {/* CAS 1 : Rendu Conditionnel Classique */}
      <div style={{ border: '1px solid red', padding: 10 }}>
        <h3>Approche Standard</h3>
        <button onClick={() => setShowCond(!showCond)}>
          {showCond ? 'Masquer' : 'Afficher'}
        </button>
        <p>Observez le léger gel lors du clic ci-dessus.</p>
        
        {/* Le composant est détruit et recréé à chaque fois -> Coûteux */}
        {showCond && <HeavyComponent />}
      </div>

      {/* CAS 2 : Activity */}
      <div style={{ border: '1px solid green', padding: 10 }}>
        <h3>Approche Activity</h3>
        <button onClick={() => setShowAct(!showAct)}>
          {showAct ? 'Masquer' : 'Afficher'}
        </button>
        <p>Le clic est instantané car le composant reste en mémoire.</p>
        
        {/* Le composant est monté une seule fois, puis juste caché */}
        <Activity mode={showAct ? 'visible' : 'hidden'}>
          <HeavyComponent />
        </Activity>
      </div>

    </div>
  );
}
```
</details>
```