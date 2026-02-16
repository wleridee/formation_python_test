Voici le chapitre **API ReactDOM Serveur: Rendu Côté Serveur (SSR)** pour la formation React 19.2.

```markdown
---
sidebar_label: API ReactDOM Serveur: Rendu Côté Serveur (SSR)
sidebar_position: 46
---

# Chapitre 46 : API ReactDOM Serveur: Rendu Côté Serveur (SSR)

`renderToPipeableStream`, `renderToReadableStream`, `renderToStaticMarkup`, `renderToString`, Streaming SSR

Le Rendu Côté Serveur (SSR - Server-Side Rendering) consiste à générer le HTML de vos composants React sur le serveur, avant de l'envoyer au navigateur. Cela permet à l'utilisateur de voir le contenu plus vite (meilleur *Time to First Paint*) et améliore le référencement (SEO).

Depuis React 18 et confirmé en 19, le SSR a évolué vers le **Streaming**. Au lieu d'attendre que *toute* la page soit prête, React envoie le HTML morceau par morceau.

Ce chapitre couvre les APIs du package `react-dom/server`.

---

## 1. `renderToPipeableStream` (Node.js) {#render-to-pipeable-stream}

### 1. Quoi
C'est l'API moderne recommandée pour faire du SSR dans un environnement **Node.js**. Elle convertit l'arbre React en un flux (Stream) Node.js qui peut être "pipé" (envoyé) directement dans la réponse HTTP.

Elle supporte le **Streaming**, **Suspense**, et le chargement paresseux (`lazy`).

### 2. Pourquoi
Les anciennes méthodes (`renderToString`) étaient bloquantes : le serveur devait tout calculer avant d'envoyer le moindre octet.
Avec `renderToPipeableStream` :
1.  **Réactivité** : Le serveur envoie le "coquillage" (header, layout) immédiatement.
2.  **Performance** : Les parties lentes (ex: commentaires, suggestions) arrivent plus tard via le même flux HTTP, sans bloquer l'affichage initial.
3.  **Expérience Utilisateur** : L'hydratation peut commencer sur les parties déjà chargées même si le reste de la page charge encore.

### 3. Comment

#### A. Syntaxe de base (Express.js)

```tsx
import { renderToPipeableStream } from 'react-dom/server';
import App from './App';

// Dans votre handler de requête serveur (ex: Express)
app.get('/', (req, res) => {
  const { pipe } = renderToPipeableStream(<App />, {
    // Scripts JS pour l'hydratation côté client
    bootstrapScripts: ['/main.js'],
    
    // Callback : Le "coquillage" HTML minimal est prêt
    onShellReady() {
      res.setHeader('content-type', 'text/html');
      pipe(res);
    },
    
    // Gestion des erreurs
    onShellError(error) {
      res.statusCode = 500;
      res.send('<!doctype html><p>Erreur critique</p>');
    }
  });
});
```

#### B. Cas concret : Gestion des Bots vs Utilisateurs

Pour les robots d'indexation (Googlebot), on préfère parfois attendre que *tout* soit chargé avant d'envoyer la réponse, pour garantir l'indexation complète.

```tsx
import { renderToPipeableStream } from 'react-dom/server';

function handleRequest(req, res) {
  let didError = false;
  const isBot = req.headers['user-agent']?.includes('Googlebot');

  const { pipe } = renderToPipeableStream(<App />, {
    bootstrapScripts: ['/client.js'],
    
    // Appelé quand le shell est prêt (rapide)
    onShellReady() {
      if (!isBot) {
        res.statusCode = didError ? 500 : 200;
        res.setHeader('Content-type', 'text/html');
        pipe(res);
      }
    },

    // Appelé quand TOUT est fini (plus lent)
    onAllReady() {
      if (isBot) {
        res.statusCode = didError ? 500 : 200;
        res.setHeader('Content-type', 'text/html');
        pipe(res);
      }
    },

    onError(x) {
      didError = true;
      console.error(x);
    }
  });
}
```

### 🚨 Limitations
Cette API est spécifique à **Node.js**. Elle ne fonctionne pas dans les environnements "Edge" (Cloudflare Workers, Deno) qui utilisent les Web Streams.

---

## 2. `renderToReadableStream` (Edge / Web Streams) {#render-to-readable-stream}

### 1. Quoi
C'est l'équivalent de `renderToPipeableStream`, mais conçu pour les environnements basés sur les standards du Web (Web Streams API), comme **Cloudflare Workers**, **Vercel Edge**, **Deno**, ou **Bun**.

### 2. Pourquoi
Les runtimes modernes ("Edge") n'utilisent pas les streams Node.js classiques. Cette API retourne une `Promise` qui résout vers un `ReadableStream`, standard du web.

### 3. Comment

```tsx
import { renderToReadableStream } from 'react-dom/server';
import App from './App';

// Exemple pour un Worker (Cloudflare/Web standard)
async function handleRequest(request: Request) {
  const stream = await renderToReadableStream(<App />, {
    bootstrapScripts: ['/main.js'],
    onError(error) {
      console.error(error);
      // On peut logger l'erreur mais on continue le streaming
    }
  });

  return new Response(stream, {
    headers: { 'Content-Type': 'text/html' },
  });
}
```

---

## 3. `renderToString` (Legacy / Synchrone) {#render-to-string}

### 1. Quoi
Convertit un arbre React en une chaîne de caractères HTML complète, de manière synchrone.

### 2. Pourquoi
C'était la méthode standard avant React 18.
Aujourd'hui, elle est utile uniquement si :
*   Votre environnement serveur ne supporte pas les streams.
*   Vous avez besoin du HTML complet sous forme de `string` pour un traitement spécifique (ex: pré-traitement lourd avant envoi).

### 3. Comment

```tsx
import { renderToString } from 'react-dom/server';
import App from './App';

const html = renderToString(<App />);
// Renvoie : "<div data-reactroot><h1>Hello</h1></div>"

res.send(`<!DOCTYPE html><body>${html}</body>`);
```

### 4. Zone de Danger
❌ **Bloquant** : `renderToString` bloque le thread principal (Event Loop) du serveur tant que le rendu n'est pas fini. Sur une page lourde avec beaucoup d'utilisateurs simultanés, cela dégrade sévèrement les performances du serveur.
❌ **Pas de Streaming** : Pas de support pour `<Suspense>` côté serveur (React attendra que toutes les promesses Suspense soient résolues avant de retourner la chaîne, ou lèvera une erreur selon la version).

---

## 4. `renderToStaticMarkup` (Email / Statique) {#render-to-static-markup}

### 1. Quoi
Similaire à `renderToString`, mais génère un HTML "propre", sans les attributs spécifiques à React (comme `data-reactroot`) nécessaires à l'hydratation.

### 2. Pourquoi
Idéal pour générer du HTML qui ne sera **jamais** hydraté (pas interactif) :
*   **Emails transactionnels** (React Email).
*   Génération de fichiers statiques simples (ex: flux RSS, sitemap).
*   Articles de blog statiques sans JS.

### 3. Comment

```tsx
import { renderToStaticMarkup } from 'react-dom/server';

function EmailTemplate({ name }: { name: string }) {
  return (
    <div style={{ fontFamily: 'Arial' }}>
      <h1>Bienvenue, {name} !</h1>
      <p>Merci de votre inscription.</p>
    </div>
  );
}

const emailHtml = renderToStaticMarkup(<EmailTemplate name="Alice" />);
// Résultat : <div style="font-family:Arial"><h1>Bienvenue, Alice !</h1>...</div>
// Pas d'attributs React inutiles.
```

---

## 5. Le Streaming SSR et Suspense {#streaming-ssr-et-suspense}

### 1. Quoi
Le Streaming SSR permet d'envoyer le HTML en plusieurs morceaux (chunks). React travaille main dans la main avec `<Suspense>`.

### 2. Comment ça marche
1.  React rend le composant racine.
2.  S'il rencontre un `<Suspense>`, il envoie immédiatement le HTML du `fallback` (ex: Spinner) au navigateur.
3.  Le serveur continue de calculer le contenu du Suspense (fetch de données, etc.).
4.  Une fois prêt, React envoie le HTML final du contenu, accompagné d'un petit script `<script>` inline.
5.  Ce script échange le `fallback` (Spinner) par le vrai contenu dans le DOM.

### 3. Exemple visuel (Code)

```tsx
// Server
<Layout>
  <NavBar />
  <Suspense fallback={<Spinner />}>
    <HeavyComments />
  </Suspense>
</Layout>

// 1. Le navigateur reçoit immédiatement :
// <Layout><NavBar /><Spinner /></Layout>

// 2. Quelques secondes plus tard (via le même stream HTTP) :
// <div id="hidden-comments">...vrais commentaires...</div>
// <script>remplacerSpinnerPar('hidden-comments')</script>
```

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-46}

1.  **Pourquoi `renderToPipeableStream` est-elle préférée à `renderToString` en React 19 ?**
    Car elle supporte le Streaming et Suspense, permettant d'envoyer le contenu plus vite (meilleur TTFB) et de ne pas bloquer le thread du serveur, contrairement à `renderToString` qui est synchrone et bloquant.

2.  **Quelle méthode utiliser pour générer le HTML d'un email transactionnel ?**
    `renderToStaticMarkup`, car elle produit un HTML plus léger sans les attributs de données internes de React nécessaires à l'hydratation (qui est inutile dans un email).

3.  **Quelle est la différence d'environnement entre `renderToPipeableStream` et `renderToReadableStream` ?**
    `renderToPipeableStream` utilise les Streams Node.js (pour les serveurs Node.js/Express), tandis que `renderToReadableStream` utilise les Web Streams (pour les environnements Edge comme Cloudflare Workers, Deno ou le navigateur).

---

## Exercices : {#exercices-46}

### Exercice 1 - Serveur Node Simple {#exercice-1---serveur-node-simple}

🎯 **Objectif** : Mettre en place un rendu SSR basique avec Node.js.

💼 **Mise en situation** : Vous devez créer un petit serveur Express qui sert votre application React.

📝 **Énoncé** :
1. Créez un composant `App` simple (`<h1>Hello SSR</h1>`).
2. Écrivez une fonction (handler) qui utilise `renderToPipeableStream`.
3. Configurez le `pipe` pour envoyer la réponse.
4. N'oubliez pas le squelette HTML (`<html><body>...`) qui doit être géré soit par React, soit écrit manuellement dans le stream.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
// server.ts (simulation)
import express from 'express'; // imaginaire
import { renderToPipeableStream } from 'react-dom/server';
// import App from './App'; 

const app = express();

app.get('/', (req, res) => {
  // Définir le composant inline pour l'exercice
  const App = () => (
    <html>
      <head><title>Mon SSR</title></head>
      <body>
        <div id="root"><h1>Hello SSR World! 🌍</h1></div>
      </body>
    </html>
  );

  const { pipe } = renderToPipeableStream(<App />, {
    bootstrapScripts: ['/client-bundle.js'],
    
    onShellReady() {
      // Dès que la structure est prête, on envoie les headers et on pipe
      res.setHeader('content-type', 'text/html');
      pipe(res);
    },
    
    onShellError(err) {
      console.error(err);
      res.status(500).send("Erreur critique serveur");
    }
  });
});
```
</details>

### Exercice 2 - Générateur de Facture (Static Markup) {#exercice-2---generateur-de-facture}

🎯 **Objectif** : Générer du HTML statique pour un PDF ou un Email.

💼 **Mise en situation** : Votre application SaaS doit envoyer une facture par email. Vous réutilisez vos composants React pour le design, mais vous voulez du HTML pur.

📝 **Énoncé** :
1. Créez un composant `Invoice` prenant `amount` et `user`.
2. Utilisez `renderToStaticMarkup` pour obtenir la chaîne HTML.
3. Affichez le résultat dans la console. Vérifiez l'absence de `data-reactroot`.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { renderToStaticMarkup } from 'react-dom/server';

function Invoice({ amount, user }: { amount: number, user: string }) {
  return (
    <div style={{ border: '1px solid #ccc', padding: 20 }}>
      <h1>Facture pour {user}</h1>
      <p>Montant dû : <strong>{amount} €</strong></p>
      <small>Merci de votre confiance.</small>
    </div>
  );
}

// Simulation d'exécution (ex: dans une Cloud Function)
function generateEmailHTML() {
  const html = renderToStaticMarkup(<Invoice amount={150} user="Entreprise XYZ" />);
  
  console.log("HTML généré pour l'email :");
  console.log(html);
  // Sortie : <div style="..."><h1>Facture pour Entreprise XYZ</h1>...</div>
  // Notez l'absence d'attributs React bizarres.
}

generateEmailHTML();
```
</details>

### Exercice 3 - Simulation de Streaming avec Suspense {#exercice-3---simulation-de-streaming}

🎯 **Objectif** : Visualiser l'ordre d'arrivée du HTML.

💼 **Mise en situation** : Une page dashboard avec un widget "Revenus" lent.

📝 **Énoncé** :
1. Imaginez un composant `SlowWidget` (dans un vrai serveur, il ferait un `await db.getData()`).
2. Enveloppez-le dans un `<Suspense fallback="Chargement...">`.
3. Utilisez `renderToPipeableStream`.
4. Observez (théoriquement) que le texte "Chargement..." est envoyé dans le premier chunk, et que le contenu de `SlowWidget` arrive dans un second chunk plus tard.

*Note : Cet exercice est conceptuel sans serveur Node réel sous la main, mais le code montre la structure.*

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { renderToPipeableStream } from 'react-dom/server';
import { Suspense } from 'react';

// Simulation d'un composant asynchrone (Server Component)
async function SlowWidget() {
  // Simule une attente de 2 secondes
  await new Promise(resolve => setTimeout(resolve, 2000));
  return <div style={{ color: 'green' }}>💰 Revenus : 50,000 € (Chargé !)</div>;
}

function Dashboard() {
  return (
    <html>
      <body>
        <h1>Mon Dashboard</h1>
        <section>
          <h2>Statistiques Rapides</h2>
          <p>Visites : 1200</p>
        </section>
        
        <section style={{ border: '2px dashed red', padding: 10 }}>
          {/* Le serveur enverra le fallback immédiatement */}
          <Suspense fallback={<span>⏳ Calcul des revenus en cours...</span>}>
            {/* Puis streaméra le résultat de SlowWidget 2s plus tard */}
            <SlowWidget />
          </Suspense>
        </section>
      </body>
    </html>
  );
}

// Code serveur express...
// app.get('/', (req, res) => {
//    const { pipe } = renderToPipeableStream(<Dashboard />, { onShellReady() { pipe(res); } });
// });
```
</details>
```