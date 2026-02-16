Voici le chapitre **Gestion des Erreurs: Error Boundaries** pour la formation **React 19.2**.

```markdown
---
sidebar_label: Gestion des Erreurs: Error Boundaries
sidebar_position: 52
---

# Chapitre 52 : Gestion des Erreurs: Error Boundaries

Capture des erreurs UI, Composants de classe, Méthodes de cycle de vie `getDerivedStateFromError`, `componentDidCatch`

En JavaScript standard, une erreur non gérée peut casser l'exécution de tout le script. Dans une application React, cela se traduit souvent par un **écran blanc** complet (White Screen of Death) : si un seul composant plante pendant le rendu, React démonte tout l'arbre de composants par sécurité.

Les **Error Boundaries** (Périmètres d'Erreur) sont la solution. Ce sont des composants React qui agissent comme des blocs `try...catch`, mais pour l'interface utilisateur.

---

## 1. Le Concept d'Error Boundary {#le-concept-d-error-boundary}

### 1. Quoi
Un Error Boundary est un composant React spécial qui **capture les erreurs JavaScript** survenant n'importe où dans son arbre de composants enfants, logue ces erreurs, et affiche une **interface de repli** (fallback UI) au lieu de l'arbre de composants planté.

### 2. Pourquoi
Pour la résilience de l'application.
Imaginez un Dashboard E-commerce avec trois widgets : "Dernières Ventes", "Utilisateurs Actifs" et "Graphique de Revenus". Si le composant "Graphique" plante à cause d'une donnée mal formatée :
*   **Sans Error Boundary** : Toute la page devient blanche. L'utilisateur ne voit rien.
*   **Avec Error Boundary** : Le graphique est remplacé par un message "Une erreur est survenue", mais le reste du tableau de bord reste fonctionnel et interactif.

### 3. Comment

Pour créer un Error Boundary, vous devez **impérativement** utiliser un **Composant de Classe**. C'est l'un des rares cas d'usage où les Hooks ne suffisent pas encore (bien que des RFC existent).

Un composant de classe devient un Error Boundary s'il définit au moins une des deux méthodes de cycle de vie suivantes :
1.  `static getDerivedStateFromError()` : Pour mettre à jour l'état et afficher l'UI de repli.
2.  `componentDidCatch()` : Pour logger l'erreur (vers Sentry, Datadog, etc.).

#### A. Syntaxe de base (TypeScript)

```tsx
import React, { Component, ReactNode } from 'react';

interface Props {
  children: ReactNode;
  fallback: ReactNode; // Ce qu'on affiche en cas d'erreur
}

interface State {
  hasError: boolean;
}

class ErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    // Initialisation : pas d'erreur par défaut
    this.state = { hasError: false };
  }

  // 1. Mise à jour de l'état suite à une erreur
  static getDerivedStateFromError(_: Error): State {
    // On retourne le nouvel état pour déclencher le rendu du fallback
    return { hasError: true };
  }

  // 2. Effet de bord (Logging)
  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    console.error("Uncaught error:", error, errorInfo);
    // Ici, envoyez l'erreur à votre service de monitoring
    // logToSentry(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      // Si une erreur a été capturée, on rend le fallback
      return this.props.fallback;
    }

    // Sinon, on rend les enfants normalement
    return this.props.children;
  }
}

export default ErrorBoundary;
```

#### B. Utilisation

Vous enveloppez simplement les parties "dangereuses" ou critiques de votre application.

```tsx
<ErrorBoundary fallback={<h1>Oups, le widget météo a planté.</h1>}>
  <WeatherWidget />
</ErrorBoundary>
```

---

## 2. Granularité et Stratégie de Capture {#granularite-et-strategie}

### 1. Quoi
Vous pouvez placer des Error Boundaries à différents niveaux de votre application. Le choix de l'emplacement détermine l'expérience utilisateur en cas de crash.

### 2. Pourquoi
*   **Niveau Racine** : Empêche l'écran blanc total, mais l'utilisateur perd toute l'application.
*   **Niveau Fonctionnalité** : Isole les pannes. Si le Chat plante, le reste de l'app fonctionne.

### 3. Comment

#### A. Cas concret : Protection Granulaire

```tsx
function AppDashboard() {
  return (
    <div className="dashboard-grid">
      {/* Si la Sidebar plante, toute l'app est affectée ? Non, si on l'isole. */}
      <ErrorBoundary fallback={<div className="sidebar-error">Menu indisponible</div>}>
        <Sidebar />
      </ErrorBoundary>

      <main>
        <Header />
        
        {/* Protection individuelle des widgets */}
        <div className="widgets">
          <ErrorBoundary fallback={<ErrorCard title="Ventes" />}>
            <SalesWidget />
          </ErrorBoundary>
          
          <ErrorBoundary fallback={<ErrorCard title="Trafic" />}>
            <TrafficWidget />
          </ErrorBoundary>
        </div>
      </main>
    </div>
  );
}
```

### 4. Zone de Danger : Ce que les Error Boundaries NE capturent PAS

C'est une confusion fréquente. Les Error Boundaries capturent les erreurs lors du **rendu**, dans les méthodes de cycle de vie, et dans les constructeurs de l'arbre enfant.

Elles **NE CAPTURENT PAS** les erreurs dans :
1.  ❌ Les gestionnaires d'événements (ex: `onClick`). Pour cela, utilisez `try/catch` classique.
2.  ❌ Le code asynchrone (ex: `setTimeout` ou `requestAnimationFrame`).
3.  ❌ Le rendu côté serveur (SSR).
4.  ❌ Les erreurs survenues dans l'Error Boundary lui-même (pas dans ses enfants).

#### Exemple : Erreur non capturée (Event Handler)

```tsx
function BuggyButton() {
  const handleClick = () => {
    // Cette erreur NE SERA PAS capturée par l'ErrorBoundary parent !
    // Elle remontera dans la console du navigateur.
    throw new Error('Boom cliqué'); 
  };

  return <button onClick={handleClick}>Ne pas cliquer</button>;
}
```

Pour capturer une erreur dans un `onClick` et déclencher l'Error Boundary, vous devez "lancer" l'erreur dans le cycle de rendu via un `setState` (astuce avancée) ou utiliser une bibliothèque comme `react-error-boundary`.

---

## 3. Récupération et Reset (Try Again) {#recuperation-et-reset}

### 1. Quoi
Une bonne UX ne se contente pas de dire "Erreur". Elle propose un bouton "Réessayer".

### 2. Pourquoi
Parfois, l'erreur est transitoire (problème réseau momentané, glitch d'état). Permettre à l'utilisateur de réinitialiser le composant peut résoudre le problème sans recharger toute la page.

### 3. Comment

Il faut ajouter une méthode pour reset l'état de l'Error Boundary.

```tsx
// Dans la classe ErrorBoundary :
class ErrorBoundary extends Component<Props, State> {
  // ... (code précédent)

  resetErrorBoundary = () => {
    // On remet hasError à false pour tenter de re-rendre les enfants
    this.setState({ hasError: false });
  }

  render() {
    if (this.state.hasError) {
      // On passe la fonction de reset au composant de Fallback si c'est un composant React
      // Ou on l'utilise directement
      return (
        <div role="alert">
          <p>Une erreur est survenue.</p>
          <button onClick={this.resetErrorBoundary}>Réessayer</button>
        </div>
      );
    }
    return this.props.children;
  }
}
```

### 💡 Utiliser `react-error-boundary` (Best Practice 2026)

Bien qu'il soit important de savoir écrire un Error Boundary, la communauté utilise massivement la bibliothèque `react-error-boundary`. Elle offre une API plus simple et gère le reset automatiquement.

```tsx
import { ErrorBoundary } from "react-error-boundary";

function Fallback({ error, resetErrorBoundary }) {
  return (
    <div role="alert">
      <p>Erreur : {error.message}</p>
      <button onClick={resetErrorBoundary}>Réessayer</button>
    </div>
  );
}

<ErrorBoundary FallbackComponent={Fallback} onReset={() => setKey(k => k + 1)}>
  <MyComponent />
</ErrorBoundary>
```

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-52}

1.  **Peut-on écrire un Error Boundary avec un composant fonctionnel et des Hooks ?**
    Non (pas en React 19.2). Seuls les composants de classe peuvent implémenter `getDerivedStateFromError` et `componentDidCatch`. Cependant, on peut utiliser des bibliothèques qui enveloppent cette logique.

2.  **Un Error Boundary capture-t-il une erreur survenue dans un `fetch` ou un `onClick` ?**
    Non. Il ne capture que les erreurs survenant pendant la phase de **rendu** (render phase), le cycle de vie, ou le constructeur des enfants. Les erreurs asynchrones ou d'événements doivent être gérées par des `try/catch`.

3.  **Quelle est la différence entre `getDerivedStateFromError` et `componentDidCatch` ?**
    `getDerivedStateFromError` est utilisé pour **mettre à jour l'état** et afficher l'UI de repli (doit être pur, pas d'effets de bord). `componentDidCatch` est utilisé pour les **effets de bord** comme l'envoi de logs d'erreurs à un service externe.

---

## Exercices : {#exercices-52}

### Exercice 1 - La Bombe à Retardement {#exercice-1---la-bombe-a-retardement}

🎯 **Objectif** : Implémenter un Error Boundary basique et provoquer une erreur de rendu.

💼 **Mise en situation** : Vous développez un composant instable pour tester votre infrastructure de gestion d'erreurs.

📝 **Énoncé** :
1. Créez un composant de classe `SimpleBoundary` qui affiche "Oups !" si une erreur survient.
2. Créez un composant `Bomb` qui accepte une prop `shouldExplode`.
3. Si `shouldExplode` est true, le composant `Bomb` doit lancer une erreur (`throw new Error('Boom')`) directement dans le corps de la fonction (pendant le rendu).
4. Enveloppez `Bomb` dans `SimpleBoundary` et ajoutez un bouton pour déclencher l'explosion.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import React, { Component, useState, ReactNode } from 'react';

// 1. Le composant Error Boundary
class SimpleBoundary extends Component<{ children: ReactNode }, { hasError: boolean }> {
  state = { hasError: false };

  static getDerivedStateFromError() {
    // Met à jour l'état pour afficher le fallback
    return { hasError: true };
  }

  render() {
    if (this.state.hasError) {
      return <h2 style={{ color: 'red' }}>Oups ! Explosion contenue. 🛡️</h2>;
    }
    return this.props.children;
  }
}

// 2. Le composant instable
function Bomb({ shouldExplode }: { shouldExplode: boolean }) {
  if (shouldExplode) {
    // Lancer une erreur pendant le rendu déclenche l'Error Boundary
    throw new Error('KABOOM');
  }
  return <div className="p-4 border">Tout va bien... pour l'instant. 💣</div>;
}

// 3. L'application de test
export function AppExercice() {
  const [explode, setExplode] = useState(false);

  return (
    <div>
      <button onClick={() => setExplode(true)}>Activer la bombe</button>
      <hr />
      <SimpleBoundary>
        <Bomb shouldExplode={explode} />
      </SimpleBoundary>
    </div>
  );
}
```
</details>

### Exercice 2 - Isolation de Widgets (SaaS Dashboard) {#exercice-2---isolation-widgets}

🎯 **Objectif** : Comprendre l'importance de la granularité.

💼 **Mise en situation** : Un tableau de bord SaaS affiche "Profil Utilisateur" et "Facturation". Si le service de facturation est en panne (crash JS), l'utilisateur doit quand même pouvoir voir son profil.

📝 **Énoncé** :
1. Créez un `GenericErrorBoundary` qui accepte une prop `moduleName` pour personnaliser le message d'erreur.
2. Créez deux composants : `UserProfile` (stable) et `BillingInfo` (qui plante toujours).
3. Affichez-les côte à côte sans Error Boundary (observez l'écran blanc).
4. Enveloppez-les individuellement pour voir que `UserProfile` survit au crash de `BillingInfo`.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Un écran divisé en deux.
> **Annotation** : À gauche, un profil utilisateur affiché correctement. À droite, un cadre rouge indiquant "Erreur dans le module Facturation".
> **Alt Text suggéré** : Isolation d'erreur React : le profil reste visible malgré le crash de la facturation.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import React, { Component, ReactNode } from 'react';

// Composant Boundary Réutilisable
class GenericErrorBoundary extends Component<
  { children: ReactNode; moduleName: string },
  { hasError: boolean }
> {
  state = { hasError: false };

  static getDerivedStateFromError() {
    return { hasError: true };
  }

  render() {
    if (this.state.hasError) {
      return (
        <div style={{ border: '2px solid red', padding: '1rem', background: '#fff0f0' }}>
          <h3>⚠️ Module {this.props.moduleName} indisponible</h3>
          <p>Nos ingénieurs sont sur le coup.</p>
        </div>
      );
    }
    return this.props.children;
  }
}

function UserProfile() {
  return (
    <div style={{ border: '1px solid green', padding: '1rem' }}>
      <h2>Profil de Alice</h2>
      <p>Statut : Actif</p>
    </div>
  );
}

function BillingInfo() {
  // Simulation d'un crash inévitable
  throw new Error("Service Facturation HS");
  return <div>Factures...</div>;
}

export function Dashboard() {
  return (
    <div style={{ display: 'grid', gridTemplateColumns: '1fr 1fr', gap: '20px' }}>
      {/* Ce module survit car l'autre est isolé */}
      <GenericErrorBoundary moduleName="Profil">
        <UserProfile />
      </GenericErrorBoundary>

      {/* Ce module plante, mais proprement */}
      <GenericErrorBoundary moduleName="Facturation">
        <BillingInfo />
      </GenericErrorBoundary>
    </div>
  );
}
```
</details>

### Exercice 3 - Le Bouton "Réessayer" {#exercice-3---le-bouton-reessayer}

🎯 **Objectif** : Permettre à l'utilisateur de récupérer l'application sans recharger la page.

💼 **Mise en situation** : Une erreur réseau aléatoire a fait planter une liste. L'utilisateur doit pouvoir cliquer sur "Réessayer" pour relancer le rendu.

📝 **Énoncé** :
1. Reprenez le `GenericErrorBoundary` de l'exercice précédent.
2. Ajoutez une méthode `handleRetry` qui remet `hasError` à `false`.
3. Passez cette méthode au rendu de l'erreur pour afficher un bouton.
4. Utilisez un composant `RandomFail` qui a 50% de chance de planter.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import React, { Component, ReactNode } from 'react';

class RecoverableBoundary extends Component<{ children: ReactNode }, { hasError: boolean }> {
  state = { hasError: false };

  static getDerivedStateFromError() {
    return { hasError: true };
  }

  handleRetry = () => {
    // Reset de l'état d'erreur pour forcer React à re-rendre les enfants
    this.setState({ hasError: false });
  };

  render() {
    if (this.state.hasError) {
      return (
        <div className="error-alert">
          <p>Une erreur transitoire est survenue.</p>
          <button onClick={this.handleRetry}>🔄 Réessayer</button>
        </div>
      );
    }
    return this.props.children;
  }
}

function RandomFail() {
  // 50% de chance d'échouer
  if (Math.random() > 0.5) {
    throw new Error("Pas de chance !");
  }
  return <div className="success">✅ Chargement réussi ! (Chanceux)</div>;
}

export function AppRetry() {
  return (
    <RecoverableBoundary>
      <RandomFail />
    </RecoverableBoundary>
  );
}
```
</details>
```