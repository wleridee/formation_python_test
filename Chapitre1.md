Voici le chapitre **Introduction à React** pour la formation React 19.2.

```markdown
---
sidebar_label: Introduction à React
sidebar_position: 1
---

# Chapitre 1 : Introduction à React

Historique,Philosophie,Avantages,Écosystème

Ce chapitre pose les fondations théoriques nécessaires pour comprendre pourquoi React est devenu l'outil standard pour la création d'interfaces modernes et comment la version 19.2 redéfinit son utilisation.

## Historique {#historique}

### 1. Quoi
React est une bibliothèque JavaScript open-source créée par Jordan Walke, ingénieur chez Facebook (aujourd'hui Meta). Initialement déployée sur le fil d'actualité de Facebook en 2011, elle a été rendue publique en 2013.

L'histoire de React se divise en trois grandes éres :
1.  **L'ère des Classes (2013-2018)** : Utilisation de la Programmation Orientée Objet pour définir des composants.
2.  **L'ère des Hooks (2019-2023)** : Introduction de React 16.8. Transition vers la programmation fonctionnelle, simplifiant la réutilisation de la logique.
3.  **L'ère Full-stack (2024+)** : Avec React 19, le framework brouille la frontière entre client et serveur grâce aux **Server Components** et au **React Compiler**.

### 2. Pourquoi
Avant React, les développeurs utilisaient des outils comme jQuery ou des frameworks MVC (Model-View-Controller) comme Angular 1. Ces outils rendaient difficile la gestion d'applications complexes où les données changeaient fréquemment (ex: notifications en temps réel).

React a introduit une idée radicale pour l'époque : **re-rendre toute la vue à chaque changement de données**, plutôt que de tenter de modifier manuellement le DOM (Document Object Model).

### 3. Comment
React a évolué pour résoudre des problèmes de plus en plus complexes :

*   **Virtual DOM** : Pour rendre le re-rendu performant, React garde une copie légère du DOM en mémoire. Lorsqu'une donnée change, il compare cette copie avec l'original (diffing) et ne met à jour que ce qui est nécessaire.
*   **React Compiler (v19)** : Auparavant, les développeurs devaient optimiser manuellement les rendus inutiles. Aujourd'hui, le compilateur analyse le code au moment du build et mémoïse automatiquement les valeurs, rendant le Virtual DOM encore plus efficace sans effort humain.

### 4. Zone de Danger

:::danger Ne vivez pas dans le passé
De nombreux tutoriels en ligne datent de l'ère des classes (ex: `componentDidMount`, `this.setState`).
En 2026 et avec React 19, ces concepts sont considérés comme **legacy**. Concentrez-vous uniquement sur les Composants Fonctionnels et les Hooks.
:::

---

## Philosophie {#philosophie}

### 1. Quoi
La philosophie de React repose sur trois piliers fondamentaux :
1.  **L'approche Déclarative** : Vous décrivez *à quoi* l'interface doit ressembler en fonction de l'état (données), et non *comment* la modifier étape par étape.
2.  **L'architecture à base de Composants** : L'interface est découpée en briques indépendantes, réutilisables et isolées.
3.  **Flux de données unidirectionnel (One-Way Data Flow)** : Les données s'écoulent toujours du parent vers l'enfant.

### 2. Pourquoi
*   **Prédictibilité** : Si vous connaissez l'état de votre application, vous savez exactement à quoi ressemblera l'interface. Le code devient plus facile à lire et à déboguer.
*   **Scalabilité** : En isolant la logique dans des composants, une équipe de 100 développeurs peut travailler sur la même application sans se marcher sur les pieds.

### 3. Comment

#### Comparaison Impératif vs Déclaratif

**Approche Impérative (JavaScript Vanilla / jQuery) :**
Vous donnez des ordres directs au navigateur.
```javascript
// On sélectionne le bouton
const btn = document.getElementById('btn');
// On écoute le clic
btn.addEventListener('click', () => {
  // On modifie manuellement la classe et le texte
  const text = document.getElementById('text');
  text.className = 'visible';
  text.innerText = 'Bonjour';
});
```

**Approche Déclarative (React) :**
Vous décrivez l'état final.
```tsx
// UI = f(state)
function Welcome() {
  const [isVisible, setIsVisible] = useState(false);

  return (
    <div>
      <button onClick={() => setIsVisible(true)}>Afficher</button>
      {/* React gère le DOM pour nous si isVisible est vrai */}
      {isVisible && <p className="visible">Bonjour</p>}
    </div>
  );
}
```

### 4. Zone de Danger

:::warning Mélanger les paradigmes
Une erreur fréquente chez les débutants est d'essayer de manipuler le DOM manuellement (ex: `document.getElementById('mon-input').value = ''`) à l'intérieur d'un composant React. Cela désynchronise React de la réalité et cause des bugs majeurs. Laissez React gérer le DOM.
:::

---

## Avantages {#avantages}

### 1. Quoi
Pourquoi React domine-t-il le marché face à Vue, Angular ou Svelte ?
*   **Universalité (Learn Once, Write Anywhere)** : Le même savoir-faire permet de créer des sites web (React DOM), des applications mobiles natives (React Native), des applications desktop (Electron) et de la VR (React 360).
*   **Typage fort** : React et TypeScript fonctionnent en symbiose parfaite, offrant une autocomplétion et une sécurité inégalées.
*   **Innovation constante** : Avec les Server Components, React permet de faire du rendu côté serveur sans configuration complexe, améliorant le SEO et la performance.

### 2. Pourquoi
Pour une entreprise, choisir React est un pari sur la sécurité et le recrutement.
Pour un développeur, c'est l'assurance d'avoir accès à une documentation massive et à des outils de pointe.

### 3. Comment
React utilise **JSX** (JavaScript XML), une extension syntaxique qui permet d'écrire du balisage directement dans le JavaScript. Bien que cela puisse sembler étrange au début (mettre du HTML dans du JS ?), cela permet d'avoir toute la puissance d'un langage de programmation complet (variables, boucles, conditions) directement dans la vue, contrairement aux templates HTML rigides d'autres frameworks.

### 🚨 Limitations de React
React est une "bibliothèque", pas un "framework" tout-inclus. Par défaut, il ne gère pas :
*   Le routage (navigation entre les pages).
*   Les requêtes HTTP complexes.
*   La gestion de formulaires avancée.
Il faut souvent assembler plusieurs briques pour avoir une application complète, ou utiliser un framework React comme **Next.js** ou **Remix**.

---

## Écosystème {#ecosysteme}

### 1. Quoi
L'écosystème React est le plus vaste du monde JavaScript. Il existe une solution éprouvée pour presque chaque problème.

### 2. Pourquoi
Cette richesse permet de ne pas réinventer la roue. Vous pouvez vous concentrer sur votre logique métier (« business logic ») plutôt que sur l'implémentation de composants d'interface génériques.

### 3. Comment
Voici les catégories d'outils indispensables en 2026 que nous aborderons ou utiliserons :

| Catégorie | Outils Standards (2026) | Rôle |
| :--- | :--- | :--- |
| **Frameworks** | Next.js, React Router v7 | Structure, Routing, SSR/RSC |
| **Build Tools** | Vite, Turbopack | Compilation ultra-rapide |
| **État Global** | Context API, Zustand | Partager des données entre composants éloignés |
| **Data Fetching** | TanStack Query, Server Actions | Récupérer et mettre à jour les données serveur |
| **Styling** | Tailwind CSS, CSS Modules | Gestion du design |
| **Tests** | Vitest, React Testing Library | Assurance qualité |

### 4. Zone de Danger

:::danger Fatigue décisionnelle
L'abondance de choix est le principal piège de l'écosystème. Il est facile de passer des semaines à choisir entre deux bibliothèques de formulaires. Dans ce cours, nous utiliserons une "stack" opinionnée et moderne pour vous éviter cette paralysie.
:::

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-1}

1.  **Quelle est la différence fondamentale entre l'approche impérative et l'approche déclarative ?**
    L'impératif dicte les étapes pour modifier l'interface (comment), tandis que le déclaratif décrit le résultat final attendu en fonction de l'état (quoi).

2.  **Qu'est-ce que le "Flux de données unidirectionnel" ?**
    C'est le principe selon lequel les données dans React ne peuvent circuler que du composant parent vers les composants enfants (via les props), jamais l'inverse directement.

3.  **Pourquoi React 19 marque-t-il un tournant important par rapport aux versions précédentes ?**
    Il introduit nativement les Server Components et le React Compiler, optimisant automatiquement les performances et permettant d'exécuter du code côté serveur sans configuration externe complexe.

4.  **React est-il un framework ou une bibliothèque ?**
    Techniquement, c'est une bibliothèque dédiée à la Vue. Cependant, il est presque toujours utilisé au sein d'un écosystème ou d'un méta-framework (comme Next.js) pour construire des applications complètes.
```