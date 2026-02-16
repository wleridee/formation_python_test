Voici le chapitre **API ReactDOM Statique: Génération de Sites Statiques (SSG)** pour la formation React 19.2.

```markdown
---
sidebar_label: API ReactDOM Statique: Génération de Sites Statiques (SSG)
sidebar_position: 47
---

# Chapitre 47 : API ReactDOM Statique: Génération de Sites Statiques (SSG)

`prerender`, `prerenderToNodeStream`, `resumeAndPrerender`, Performances SSG

:::warning APIs Expérimentales et Spécialisées
Les APIs présentées dans ce chapitre (`react-dom/static`) sont des nouveautés de React 19 conçues principalement pour les **générateurs de sites statiques** (comme Gatsby, Next.js en mode export, ou vos scripts de build maison). Elles remplacent les anciennes méthodes pour garantir la compatibilité avec **Suspense** et les données asynchrones lors de la génération.
:::

Le **SSG** (Static Site Generation) consiste à générer des fichiers HTML complets au moment de la compilation (Build Time), et non au moment de la requête utilisateur. C'est le Graal de la performance web pour le contenu public.

---

## 1. `prerender` {#prerender}

### 1. Quoi
`prerender` est la nouvelle API standard pour générer du HTML statique. Contrairement à l'ancien `renderToString`, `prerender` est **asynchrone** : elle attend que toutes les limites `<Suspense>` soient résolues et que toutes les données soient chargées avant de produire le HTML final.

Signature simplifiée :
```ts
const { prelude } = await prerender(<App />);
```

### 2. Pourquoi
Avec `renderToString` (l'ancienne méthode), si vous aviez un composant qui chargeait des données (fetch), React générait le HTML du "loading state" (le fallback) et s'arrêtait là. Le contenu réel n'était pas dans le HTML statique, ce qui est catastrophique pour le SEO d'une page statique.
`prerender` a la patience d'attendre que tout le travail soit fini pour prendre une "photo" parfaite et complète de l'application.

### 3. Comment

#### A. Script de Build (Node.js)

Voici comment on écrirait un script pour générer un fichier `index.html`.

```tsx
import { prerender } from 'react-dom/static';
import { createWriteStream } from 'node:fs';
import App from './App';

async function buildPage() {
  // 1. On lance le pré-rendu. React exécute l'app et attend les Suspenses.
  const { prelude } = await prerender(<App />);

  // 2. 'prelude' est un Web Stream. On le convertit pour l'écrire dans un fichier.
  const writeStream = createWriteStream('./dist/index.html');
  
  // Note : Il faut un adaptateur pour piper un Web Stream vers un Node Stream
  // ou attendre la fin de la lecture.
  // Pour l'exemple simple, supposons que nous convertissons le stream en string/buffer.
  // (Voir section suivante pour l'API native Node).
}
```

#### B. Cas concret : Génération d'un Article de Blog

Imaginons un composant qui fetch le contenu d'un article depuis un CMS Headless.

```tsx
import { prerender } from 'react-dom/static';
// Supposons une fonction helper pour écrire le fichier
import { writeOutput } from './build-utils'; 

// Composant avec Fetch asynchrone (React Server Component ou similaire)
async function BlogPost({ slug }: { slug: string }) {
  // Simulation d'attente BDD
  await new Promise(r => setTimeout(r, 100)); 
  return <article><h1>Article {slug}</h1><p>Contenu incroyable...</p></article>;
}

async function generateStaticFile(slug: string) {
  try {
    // prerender attend la fin du setTimeout ci-dessus !
    const { prelude } = await prerender(<BlogPost slug={slug} />);
    
    // On écrit le résultat final complet
    await writeOutput(`dist/${slug}.html`, prelude);
    console.log(`✅ Page ${slug} générée.`);
  } catch (e) {
    console.error(`❌ Échec pour ${slug}`, e);
  }
}

generateStaticFile('react-19-features');
```

### 🚨 Limitations de `prerender`
*   **Temps de Build** : Comme `prerender` attend tout le monde, si une seule requête API est lente, la génération de la page est bloquée. Sur 10 000 pages, cela peut exploser le temps de build.
*   **Pas de streaming utilisateur** : Cette API est faite pour écrire des fichiers, pas pour répondre à une requête HTTP en temps réel (pour cela, utilisez `renderToPipeableStream` du chapitre précédent).

---

## 2. `prerenderToNodeStream` {#prerender-to-node-stream}

### 1. Quoi
C'est la version spécifique à Node.js de `prerender`. Elle retourne une Promise qui résout vers un objet contenant un `prelude` qui est un **Node.js Readable Stream**.

### 2. Pourquoi
Pour des raisons de gestion mémoire. Si vous générez une page énorme (ex: une liste de 5000 produits), charger toute la chaîne de caractères en mémoire (String) peut faire planter le processus de build (`Out of Memory`). Les Streams permettent d'écrire le fichier morceau par morceau sur le disque dur.

### 3. Comment

```tsx
import { prerenderToNodeStream } from 'react-dom/static';
import { createWriteStream } from 'node:fs';
import App from './App';

async function generate() {
  const fileStream = createWriteStream('./public/index.html');

  const { prelude } = await prerenderToNodeStream(<App />, {
    // On peut injecter des données pour l'hydratation future
    bootstrapScripts: ['/assets/main.js'],
  });

  // Pipe direct vers le disque dur : Efficacité maximale
  prelude.pipe(fileStream);
  
  console.log("Génération en cours d'écriture...");
}
```

---

## 3. `resumeAndPrerender` (Avancé) {#resume-and-prerender}

### 1. Quoi
Une API très spécialisée qui permet de reprendre ("resume") un rendu qui avait commencé ailleurs (ou qui a été mis en pause) et de le terminer pour en faire un rendu statique.

### 2. Pourquoi
C'est un concept lié au **Partial Prerendering (PPR)** ou aux architectures complexes de build.
Imaginez un scénario où une partie de l'arbre a été calculée, mais une autre attendait des données. `resumeAndPrerender` permet de "finir le travail" pour obtenir un HTML statique, tout en réutilisant l'état existant. C'est principalement utilisé par les auteurs de frameworks (Next.js, Remix, Waku).

### 3. Comment
L'utilisation directe est rare pour un développeur d'application standard, mais conceptuellement :

```tsx
// Pseudocode conceptuel
import { resumeAndPrerender } from 'react-dom/static';

const stream = await resumeAndPrerender(
  <App />,
  postponedState // État capturé d'un rendu précédent interrompu
);
// Résultat : un HTML statique combinant le vieux state et le nouveau rendu
```

---

## 4. Performances SSG et Bonnes Pratiques {#performances-ssg}

### 1. TTFB (Time To First Byte) instantané
Le SSG offre la meilleure performance possible pour l'utilisateur final. Le serveur web (Nginx, Apache, Vercel Edge) ne fait que servir un fichier physique. Temps de réponse < 50ms.

### 2. Hydratation Sélective
Même si la page est statique, React doit "s'éveiller" côté client (`hydrateRoot`).
*   **Piège** : Générer une page HTML de 5Mo. L'utilisateur la voit vite, mais le navigateur gèle pendant 2 secondes pour l'hydrater (Main Thread bloqué).
*   **Solution** : Utiliser le Code Splitting et le Lazy Loading même sur des sites statiques pour réduire le JS envoyé.

### 4. Zone de Danger : Données Dynamiques

❌ **À ne pas faire** : Essayer d'utiliser `prerender` pour du contenu qui dépend de l'utilisateur connecté (ex: "Bonjour Alice").
*   Au build time, il n'y a pas d'utilisateur "Alice". Le HTML sera généré avec "Bonjour Invité" ou vide.
*   Si vous hydratez ensuite avec Alice connecté, vous aurez une erreur de "Hydration Mismatch".

✅ **Bonne Pratique** : Pour les parties dynamiques dans un site statique, render un "placeholder" vide ou un squelette au build time, et ne charger les données utilisateur qu'au montage côté client (`useEffect`).

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-47}

1.  **Quelle est la différence majeure entre `prerender` et l'ancien `renderToString` ?**
    `prerender` est asynchrone et attend la résolution de tous les composants `<Suspense>` pour fournir un HTML complet, alors que `renderToString` n'attend pas et génère les états de chargement (fallbacks).

2.  **Dans quel cas utiliser `prerenderToNodeStream` plutôt que `prerender` ?**
    Lorsque vous travaillez dans un environnement Node.js et que vous devez générer des pages lourdes. L'utilisation de streams évite de saturer la mémoire RAM en écrivant le fichier au fur et à mesure sur le disque.

3.  **Le SSG est-il adapté pour un Tableau de Bord utilisateur personnalisé ?**
    Non. Le SSG génère le même fichier HTML pour tout le monde au moment du déploiement. Un tableau de bord nécessite des données fraîches et spécifiques à l'utilisateur (SSR ou CSR est préférable).

---

## Exercices : {#exercices-47}

### Exercice 1 - Le Script de Build Manuel {#exercice-1---le-script-de-build-manuel}

🎯 **Objectif** : Créer un petit script Node.js qui génère un fichier HTML statique.

💼 **Mise en situation** : Vous ne voulez pas utiliser Next.js pour une simple landing page, mais vous voulez quand même utiliser React et avoir du HTML statique pour le SEO. Vous créez votre propre générateur.

📝 **Énoncé** :
1. Créez un fichier `builder.js`.
2. Importez `prerenderToNodeStream`.
3. Créez un composant simple `<LandingPage />`.
4. Utilisez `fs.createWriteStream` pour écrire le résultat dans `output.html`.
5. Exécutez le script avec `node builder.js` (nécessite une config supportant JSX, ex: `tsx` ou `babel-node`).

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
// builder.tsx (à exécuter avec tsx ou esrun)
import React from 'react';
import { prerenderToNodeStream } from 'react-dom/static';
import { createWriteStream } from 'node:fs';
import { mkdir } from 'node:fs/promises';
import { dirname } from 'node:path';

const OUTPUT_PATH = './dist/landing.html';

function LandingPage() {
  return (
    <html>
      <head><title>Ma Super Landing Page</title></head>
      <body>
        <header><h1>Bienvenue sur mon produit SaaS 🚀</h1></header>
        <main>
          <p>La performance statique est incroyable.</p>
        </main>
      </body>
    </html>
  );
}

async function build() {
  // Création du dossier si inexistant
  await mkdir(dirname(OUTPUT_PATH), { recursive: true });
  
  const fileStream = createWriteStream(OUTPUT_PATH);

  try {
    const { prelude } = await prerenderToNodeStream(<LandingPage />);
    
    // Écriture sur le disque
    prelude.pipe(fileStream);
    
    fileStream.on('finish', () => {
      console.log(`✅ Fichier généré avec succès : ${OUTPUT_PATH}`);
    });
  } catch (err) {
    console.error("❌ Erreur de build :", err);
    process.exit(1);
  }
}

build();
```
</details>

### Exercice 2 - La Preuve du Suspense {#exercice-2---la-preuve-du-suspense}

🎯 **Objectif** : Vérifier que `prerender` attend bien les données.

💼 **Mise en situation** : Vous générez des fiches produits. Les données viennent d'une base lente (simulée). Vous voulez être sûr que le HTML final contient le prix, et pas un spinner.

📝 **Énoncé** :
1. Créez un composant `ProductPrice` qui `await` une promesse de 1 seconde avant d'afficher "99.99 €".
2. Enveloppez-le dans un `Suspense fallback="Calcul..."`.
3. Utilisez `prerender` (ou `prerenderToNodeStream`).
4. Vérifiez le fichier de sortie. Il doit contenir "99.99 €".
5. Si vous aviez utilisé `renderToString` (exercice mental), il aurait contenu "Calcul...".

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import React, { Suspense } from 'react';
import { prerenderToNodeStream } from 'react-dom/static';

// Simulation composant asynchrone (Server Component pattern)
async function ProductPrice() {
  // Le build "gèle" ici pendant 1 seconde
  await new Promise(resolve => setTimeout(resolve, 1000));
  return <span>99.99 €</span>;
}

function ProductPage() {
  return (
    <div>
      <h1>Super Produit</h1>
      <div>
        Prix : 
        <Suspense fallback="Calcul...">
          <ProductPrice />
        </Suspense>
      </div>
    </div>
  );
}

// Dans le script de build...
async function testBuild() {
  console.time("Build Duration");
  const { prelude } = await prerenderToNodeStream(<ProductPage />);
  // ... pipe vers stdout pour voir le résultat
  prelude.pipe(process.stdout);
  
  prelude.on('end', () => {
    console.log("\n");
    console.timeEnd("Build Duration"); 
    // Devrait afficher > 1000ms, prouvant qu'il a attendu.
    // Le HTML affiché contiendra "99.99 €".
  });
}

testBuild();
```
</details>

### Exercice 3 - Génération de Sitemap XML {#exercice-3---generation-de-sitemap}

🎯 **Objectif** : Utiliser React pour générer autre chose que du HTML.

💼 **Mise en situation** : Pour le SEO, vous avez besoin d'un fichier `sitemap.xml`. Plutôt que de concaténer des chaînes, utilisez la puissance déclarative de React.

📝 **Énoncé** :
1. Créez un composant `Sitemap` qui retourne des balises `<urlset>`, `<url>`, `<loc>`.
2. Utilisez `prerender` pour générer le XML.
3. Attention : React ajoute `<!DOCTYPE html>` par défaut ou des commentaires. `renderToStaticMarkup` (vu au chapitre précédent) est parfois mieux pour le XML pur, mais essayons avec les streams pour la performance si la liste est longue. *Note: React peut ne pas aimer les balises XML non-standard sans configuration, utilisez des composants minuscules ou des primitives.*

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import React from 'react';
import { prerenderToNodeStream } from 'react-dom/static';
import { createWriteStream } from 'node:fs';

const pages = ['/', '/about', '/contact', '/products'];

function SitemapXML() {
  return (
    // Astuce: utiliser des balises personnalisées
    <urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
      {pages.map(path => (
        <url key={path}>
          <loc>{`https://monsite.com${path}`}</loc>
          <changefreq>daily</changefreq>
          <priority>0.8</priority>
        </url>
      ))}
    </urlset>
  );
}

async function buildSitemap() {
  const stream = createWriteStream('./dist/sitemap.xml');
  
  // Écriture manuelle du header XML car React ne le gère pas
  stream.write('<?xml version="1.0" encoding="UTF-8"?>\n');

  const { prelude } = await prerenderToNodeStream(<SitemapXML />);
  prelude.pipe(stream);
  
  console.log('Sitemap généré.');
}

buildSitemap();
```
</details>
```