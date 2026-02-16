Voici le chapitre **`useInsertionEffect`: Injection de Styles (pour CSS-in-JS)** pour la formation React 19.2.

```markdown
---
sidebar_label: `useInsertionEffect`: Injection de Styles (pour CSS-in-JS)
sidebar_position: 27
---

# Chapitre 27 : `useInsertionEffect`: Injection de Styles (pour CSS-in-JS)

Styles dynamiques, Intégration de librairies, Performance CSS-in-JS

:::warning Public Cible
Ce Hook est conçu principalement pour les **auteurs de bibliothèques CSS-in-JS** (comme *Styled Components*, *Emotion*, ou votre propre système de design).
Dans 99% des cas, vous utiliserez `useEffect` ou `useLayoutEffect` pour votre logique d'application.
Cependant, comprendre ce Hook est essentiel pour maîtriser le cycle de rendu de React et la performance des styles dynamiques.
:::

Dans les architectures React modernes, nous définissons souvent le CSS directement dans le JavaScript (CSS-in-JS). Cela implique que le JavaScript doit créer des balises `<style>` et les injecter dans le `<head>` du document à la volée.
Si cette injection se fait au mauvais moment, elle force le navigateur à recalculer la mise en page (Layout Thrashing), ce qui ralentit l'affichage. `useInsertionEffect` est la solution de React à ce problème précis.

## Le Problème du "Runtime CSS-in-JS" {#le-probleme-du-runtime-css-in-js}

### 1. Quoi
Le "Runtime CSS-in-JS" signifie que les classes CSS sont générées et injectées pendant que l'application tourne, souvent en réponse à un changement de props ou d'état (ex: `color="red"`).

### 2. Pourquoi
Si vous injectez des styles dans un `useLayoutEffect` ou un `useEffect` :
1.  React calcule l'arbre DOM.
2.  React applique les changements au DOM.
3.  Le navigateur commence à calculer la mise en page (Layout).
4.  Votre effet se lance et injecte une nouvelle balise `<style>`.
5.  🛑 Le navigateur doit **invalider** ses calculs précédents et recommencer le Layout.

C'est un problème de performance majeur pour les animations ou les mises à jour fréquentes.

### 3. Comment fonctionne `useInsertionEffect`
`useInsertionEffect` s'exécute de manière synchrone **avant** que React ne fasse la moindre mutation dans le DOM. C'est le tout premier Hook à s'exécuter après le rendu.

**L'ordre d'exécution est strict :**
1.  **Rendu (Render)** : React appelle vos composants.
2.  **`useInsertionEffect`** : Injection des styles `<style>`.
3.  **Mutation DOM** : React met à jour le DOM réel.
4.  **`useLayoutEffect`** : Lectures de layout (mesures).
5.  **Paint** : Le navigateur affiche l'image.
6.  **`useEffect`** : Logique asynchrone.

---

## Utilisation de `useInsertionEffect` {#utilisation-de-useinsertioneffect}

### 1. Quoi
C'est un Hook qui permet d'insérer des règles CSS globales avant que le layout ne soit calculé. Il n'a pas accès aux `refs` du DOM (car le DOM n'est pas encore à jour).

### 2. Pourquoi
Pour garantir que lorsque le navigateur commencera à calculer la géométrie des éléments (taille, position), toutes les règles CSS pertinentes seront déjà présentes dans le `<head>`.

### 3. Comment

#### A. Syntaxe de base

```tsx
import { useInsertionEffect } from 'react';

function useThemeColor(color: string) {
  useInsertionEffect(() => {
    // Création et injection de style
    const style = document.createElement('style');
    style.innerHTML = `.dynamic-theme { color: ${color}; }`;
    document.head.appendChild(style);

    // Nettoyage (suppression de la balise)
    return () => {
      document.head.removeChild(style);
    };
  }, [color]);
}
```

#### B. Cas concret : Un mini moteur CSS-in-JS
Imaginons que vous créiez votre propre librairie de styles pour une application SaaS.

```tsx
import { useInsertionEffect } from 'react';

// Une fonction simple pour hasher une chaîne (pour l'exemple)
const hashString = (str: string) => 'css-' + str.length + str.charCodeAt(0);

// Set global pour ne pas injecter 2 fois la même règle
const injectedStyles = new Set<string>();

export function useCSS(rule: string) {
  // On génère un nom de classe unique basé sur la règle CSS
  const className = hashString(rule);

  useInsertionEffect(() => {
    if (!injectedStyles.has(className)) {
      const styleTag = document.createElement('style');
      styleTag.innerHTML = `.${className} { ${rule} }`;
      styleTag.setAttribute('data-injected', 'true');
      document.head.appendChild(styleTag);
      
      injectedStyles.add(className);
    }
  }, [rule]); // Se relance si la règle change

  return className;
}

// Utilisation dans un composant
function Button({ primary }: { primary?: boolean }) {
  const className = useCSS(`
    padding: 10px 20px;
    background: ${primary ? 'blue' : 'gray'};
    color: white;
    border-radius: 5px;
  `);

  return <button className={className}>Click me</button>;
}
```

### 4. Zone de Danger

:::danger Interdictions Strictes
À l'intérieur de `useInsertionEffect` :
1.  ❌ **Ne jamais accéder aux `refs`** : Les nœuds DOM correspondant au composant actuel ne sont pas encore créés/mis à jour.
2.  ❌ **Ne pas mettre à jour l'état (`setState`)** : Cela déclencherait un nouveau rendu avant même que le premier ne soit fini.
3.  ❌ **Ne pas lire de mesures de layout** (`offsetWidth`, etc.) : Le layout n'est pas prêt.
:::

---

## Limitations et Bonnes Pratiques {#limitations-et-bonnes-pratiques}

### 1. Server-Side Rendering (SSR)
Comme `useLayoutEffect`, `useInsertionEffect` ne s'exécute **pas** sur le serveur.
Pour le SSR, vous devez collecter les styles pendant le rendu serveur et les injecter dans le HTML initial (souvent via `useServerInsertedHTML` dans les architectures React Server Components).

### 2. Tableau Comparatif des Hooks d'Effets

| Hook | Moment d'exécution | Accès au DOM ? | Usage principal |
| :--- | :--- | :--- | :--- |
| **`useInsertionEffect`** | Avant mutation DOM | ❌ Non | Injection de balises `<style>` |
| **`useLayoutEffect`** | Après mutation, avant Paint | ✅ Oui | Mesures de géométrie, synchronisation visuelle |
| **`useEffect`** | Après Paint (Asynchrone) | ✅ Oui | Fetching, abonnements, logs |

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-27}

1.  **Quel est le but unique de `useInsertionEffect` ?**
    Injecter des styles CSS dans le DOM avant que React ne calcule la mise en page, pour éviter les recalculs de style coûteux (Layout Thrashing).

2.  **Peut-on utiliser une `ref` pour accéder à un élément (`inputRef.current`) dans `useInsertionEffect` ?**
    **Non**. À ce stade du cycle de vie, les modifications du DOM pour le composant courant n'ont pas encore été appliquées.

3.  **Pourquoi ne pas simplement utiliser `useLayoutEffect` pour injecter du CSS ?**
    Parce que `useLayoutEffect` s'exécute *après* la mutation du DOM. Injecter du CSS à ce moment forcerait le navigateur à invalider ses calculs de layout et à recommencer, ce qui est mauvais pour les performances.

---

## Exercices : {#exercices-27}

### Exercice 1 - L'Injecteur de Variables CSS {#exercice-1---l-injecteur-de-variables-css}

🎯 **Objectif** : Modifier des variables CSS globales (`:root`) dynamiquement sans flash visuel.

💼 **Mise en situation** : Vous créez un panneau d'administration pour un CRM. L'utilisateur peut choisir la "Couleur de la marque" via un color picker. Cette couleur doit s'appliquer instantanément à tous les boutons et titres.

📝 **Énoncé** :
1. Créez un composant `BrandColorProvider` qui prend une prop `color`.
2. Utilisez `useInsertionEffect` pour injecter une balise `<style>` définissant `:root { --brand-color: ... }`.
3. Assurez-vous de nettoyer l'ancienne balise quand la couleur change.
4. Utilisez cette variable dans un bouton enfant.

📺 **Résultat attendu** :
Le bouton change de couleur instantanément sans scintillement lors du changement de la prop.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useInsertionEffect, useState } from 'react';

function BrandColorProvider({ color, children }: { color: string, children: React.ReactNode }) {
  useInsertionEffect(() => {
    // 1. Création de la balise style
    const style = document.createElement('style');
    // 2. Définition de la variable CSS globale
    style.innerHTML = `:root { --brand-color: ${color}; }`;
    style.setAttribute('data-theme', 'dynamic');
    
    // 3. Injection AVANT le calcul du layout
    document.head.appendChild(style);

    // 4. Cleanup obligatoire pour éviter l'accumulation
    return () => {
      document.head.removeChild(style);
    };
  }, [color]);

  return <div style={{ padding: 20 }}>{children}</div>;
}

// Composant consommateur
function ThemedButton() {
  // Utilise la variable native CSS
  return (
    <button style={{ 
      backgroundColor: 'var(--brand-color)', 
      color: 'white', 
      padding: '10px 20px', 
      border: 'none',
      borderRadius: '4px'
    }}>
      Bouton de Marque
    </button>
  );
}

export function App() {
  const [color, setColor] = useState('#3498db');
  
  return (
    <div>
      <input type="color" value={color} onChange={e => setColor(e.target.value)} />
      <BrandColorProvider color={color}>
        <ThemedButton />
      </BrandColorProvider>
    </div>
  );
}
```
</details>

### Exercice 2 - Le Débugger de Cycle de Vie {#exercice-2---le-debugger-de-cycle-de-vie}

🎯 **Objectif** : Prouver l'ordre d'exécution des Hooks.

💼 **Mise en situation** : Vous formez des juniors et voulez leur montrer la preuve concrète que `useInsertionEffect` passe avant tout le monde.

📝 **Énoncé** :
1. Créez un composant `LifecycleLogger`.
2. Ajoutez `useEffect`, `useLayoutEffect`, et `useInsertionEffect`.
3. Dans chacun, faites un `console.log('Nom du Hook')`.
4. Ajoutez aussi un log directement dans le corps du composant ("Render").
5. Observez l'ordre dans la console.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Console du navigateur affichant les logs dans l'ordre.
> **Annotation** : Surlignez l'ordre : Render -> Insertion -> Layout -> Effect.
> **Alt Text suggéré** : Capture de la console montrant la séquence d'exécution des hooks React.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useEffect, useLayoutEffect, useInsertionEffect, useState } from 'react';

export function LifecycleDemo() {
  console.log('1. 🎨 Render (Corps du composant)');

  useEffect(() => {
    console.log('4. 🐢 useEffect (Post-Paint)');
  }, []);

  useLayoutEffect(() => {
    console.log('3. 📏 useLayoutEffect (Post-Mutation, Pré-Paint)');
  }, []);

  useInsertionEffect(() => {
    console.log('2. 💉 useInsertionEffect (Pré-Mutation)');
    // C'est ici qu'on injecterait les styles
  }, []);

  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(c => c + 1)}>
      Forcer un re-render ({count})
    </button>
  );
}
```
</details>

### Exercice 3 - Grille Dynamique (Grid System) {#exercice-3---grille-dynamique}

🎯 **Objectif** : Générer des classes utilitaires à la volée.

💼 **Mise en situation** : Vous construisez un outil de dashboarding où l'utilisateur peut configurer la largeur des colonnes (ex: "300px"). Vous devez générer une classe CSS spécifique pour cette largeur.

📝 **Énoncé** :
1. Créez un hook `useColumnWidth(width: string)` qui génère une classe CSS `.col-[hash] { width: [width] }`.
2. Utilisez `useInsertionEffect` pour injecter cette classe si elle n'existe pas.
3. Le composant `Column` utilise ce hook pour s'appliquer la classe.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useInsertionEffect } from 'react';

// Stockage simple des classes déjà créées
const styleCache = new Set<string>();

function useColumnWidth(width: string) {
  // Hash simple pour le nom de classe
  // Dans une vraie lib, utilisez un vrai algo de hash (murmurhash, etc.)
  const className = `col-${width.replace(/[^a-z0-9]/gi, '')}`;

  useInsertionEffect(() => {
    if (!styleCache.has(className)) {
      const style = document.createElement('style');
      style.innerHTML = `.${className} { flex: 0 0 ${width}; border: 1px solid gray; padding: 10px; }`;
      document.head.appendChild(style);
      styleCache.add(className);
      console.log(`Injecté classe: ${className}`);
    }
  }, [width, className]);

  return className;
}

export function DashboardColumn({ width, title }: { width: string, title: string }) {
  const colClass = useColumnWidth(width);
  
  return (
    <div className={colClass}>
      <h3>{title}</h3>
      <p>Largeur: {width}</p>
    </div>
  );
}

export function Dashboard() {
  return (
    <div style={{ display: 'flex', gap: 10 }}>
      <DashboardColumn width="200px" title="Sidebar" />
      <DashboardColumn width="50%" title="Main Content" />
      <DashboardColumn width="300px" title="Widgets" />
    </div>
  );
}
```
</details>
```