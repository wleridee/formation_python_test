Voici le chapitre **Balises de Métadonnées et de Ressources en React** pour la formation React 19.2.

```markdown
---
sidebar_label: Balises de Métadonnées et de Ressources en React
sidebar_position: 51
---

# Chapitre 51 : Balises de Métadonnées et de Ressources en React

Balises `<link>`, `<meta>`, `<script>`, `<style>`, `<title>`, Gestion du `<head>` du document, SEO

Jusqu'à récemment (React 18 et avant), gérer le contenu de la balise `<head>` (comme le titre de la page, la description SEO ou les feuilles de style) nécessitait des bibliothèques externes comme `react-helmet`.

Avec **React 19**, le support des balises de métadonnées est **natif**. React "hisse" (hoist) automatiquement ces balises dans le `<head>` du document, même si vous les déclarez au fin fond de votre arborescence de composants. C'est une révolution pour le SEO et la gestion des ressources.

---

## 1. La balise `<title>` : Le Titre de la Page {#la-balise-title}

### 1. Quoi
C'est le texte affiché dans l'onglet du navigateur et le titre principal dans les résultats de recherche Google.

### 2. Pourquoi
Pour le SEO et l'expérience utilisateur. Chaque "page" de votre application (ou état significatif) doit avoir un titre unique.

### 3. Comment

Il suffit de rendre une balise `<title>` n'importe où dans votre JSX. React s'occupe de la déplacer dans le `<head>`.

#### A. Syntaxe de base

```tsx
function HomePage() {
  return (
    <>
      <title>Accueil - Mon Super Site</title>
      <h1>Bienvenue !</h1>
    </>
  );
}
```

#### B. Cas concret : Titre Dynamique

```tsx
function ProductPage({ product }: { product: { name: string } }) {
  return (
    <article>
      {/* Le titre change selon les props */}
      <title>{product.name} | Boutique</title>
      <h2>{product.name}</h2>
    </article>
  );
}
```

### 🚨 Limitations de `<title>`
Si plusieurs composants affichent `<title>` en même temps, React essaiera de gérer cela, mais la dernière balise rendue gagne souvent. Assurez-vous d'avoir une logique claire pour éviter les conflits (ex: un titre par page/route).

---

## 2. La balise `<meta>` : SEO et Réseaux Sociaux {#la-balise-meta}

### 1. Quoi
Les balises `<meta>` définissent la description, les mots-clés, l'auteur, et les propriétés Open Graph (pour les partages Facebook/Twitter/LinkedIn).

### 2. Pourquoi
Indispensable pour :
*   **SEO** : `description` est le résumé sous le lien Google.
*   **SMO (Social Media Optimization)** : `og:image` définit l'image de prévisualisation quand on partage votre lien.

### 3. Comment

React 19 dé-duplique automatiquement les balises `<meta>` si elles ont des clés uniques (comme `name` ou `property`).

#### A. Syntaxe de base

```tsx
function AboutPage() {
  return (
    <>
      <title>À Propos</title>
      <meta name="description" content="Découvrez notre équipe incroyable." />
      <meta name="keywords" content="react, formation, équipe" />
      <div>Contenu de la page...</div>
    </>
  );
}
```

#### B. Cas concret : Open Graph pour le partage

```tsx
function Article({ title, summary, imageUrl }: ArticleProps) {
  return (
    <>
      <title>{title}</title>
      <meta name="description" content={summary} />
      
      {/* Métadonnées Open Graph */}
      <meta property="og:title" content={title} />
      <meta property="og:description" content={summary} />
      <meta property="og:image" content={imageUrl} />
      <meta property="og:type" content="article" />
      
      <h1>{title}</h1>
    </>
  );
}
```

### 4. Zone de Danger
❌ **Ne pas dupliquer manuellement** : Si vous avez un `<meta name="description">` dans votre layout principal ET dans votre page, React 19 est assez intelligent pour remplacer l'ancien si les attributs correspondent, mais soyez vigilants sur les clés utilisées pour la déduplication.

---

## 3. Les balises `<link>` : Styles et Ressources Externes {#les-balises-link}

### 1. Quoi
Utilisé pour charger des feuilles de style CSS, des icônes (favicon), ou des versions canoniques de l'URL.

### 2. Pourquoi
Charger un CSS spécifique uniquement pour une route donnée (Code Splitting CSS), ou changer le favicon selon le thème (clair/sombre).

### 3. Comment

React gère la **précédence** des feuilles de style pour éviter les conflits d'ordre.

#### A. Chargement d'un CSS spécifique

```tsx
function Dashboard() {
  return (
    <>
      {/* React va charger ce CSS et l'insérer dans le head */}
      {/* precedence="default" aide React à gérer l'ordre d'insertion */}
      <link rel="stylesheet" href="/dashboard-theme.css" precedence="default" />
      
      <div className="dashboard-container">
        {/* ... */}
      </div>
    </>
  );
}
```

---

## 4. Les balises `<script>` et `<style>` : Code Inline {#les-balises-script-et-style}

### 1. Quoi
*   `<script>` : Pour charger ou exécuter du JS (analytics, widgets).
*   `<style>` : Pour injecter du CSS inline critique.

### 2. Pourquoi
*   **Analytics** : Insérer le tag Google Analytics uniquement si l'utilisateur a accepté les cookies.
*   **JSON-LD** : Injecter des données structurées pour Google (Schema.org).

### 3. Comment

#### A. Script Externe (Async)

```tsx
function PaymentPage() {
  return (
    <>
      {/* Script PayPal chargé uniquement sur cette page */}
      <script src="https://www.paypal.com/sdk/js?client-id=MY_ID" async />
      <form>...</form>
    </>
  );
}
```

#### B. Données Structurées (JSON-LD)

Pour le SEO riche (recettes, produits, événements).

```tsx
function ProductSchema({ product }: { product: any }) {
  const schema = {
    "@context": "https://schema.org/",
    "@type": "Product",
    "name": product.name,
    "image": product.image,
    "description": product.description
  };

  return (
    <script type="application/ld+json">
      {JSON.stringify(schema)}
    </script>
  );
}
```

### 🚨 Limitations
Pour les scripts complexes nécessitant des callbacks (`onload`), préférez utiliser l'API impérative `preinit` (voir Chapitre 48) ou les balises avec gestion d'état, car insérer une balise `<script>` brute ne donne pas facilement accès à son cycle de vie.

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-51}

1.  **A-t-on encore besoin de `react-helmet` avec React 19 ?**
    Non, pour la grande majorité des cas, le support natif des balises `<title>`, `<meta>` et `<link>` dans n'importe quel composant rend `react-helmet` obsolète.

2.  **Où React place-t-il physiquement les balises `<meta>` déclarées dans un composant enfant ?**
    Il les "hisse" (hoist) automatiquement dans la section `<head>` du document HTML final, quel que soit l'endroit où le composant est rendu dans l'arbre.

3.  **À quoi sert l'attribut `precedence` sur une balise `<link rel="stylesheet">` ?**
    Il indique à React comment ordonner l'insertion de cette feuille de style par rapport aux autres, ce qui est crucial pour garantir que vos règles CSS s'appliquent dans le bon ordre (cascading).

---

## Exercices : {#exercices-51}

### Exercice 1 - SEO de Base pour un Blog {#exercice-1---seo-blog}

🎯 **Objectif** : Configurer les métadonnées essentielles pour une page d'article.

💼 **Mise en situation** : Vous créez le template d'article de blog. Vous devez définir le titre, la description et l'auteur.

📝 **Énoncé** :
1. Créez un composant `BlogPost` prenant `title`, `description` et `author`.
2. Affichez ces informations visibles dans le `<body>`.
3. Injectez les bonnes balises `<title>` et `<meta>` pour le SEO.
4. Vérifiez (mentalement ou via inspecteur) qu'elles atterrissent dans le `<head>`.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
interface BlogPostProps {
  title: string;
  description: string;
  author: string;
}

export function BlogPost({ title, description, author }: BlogPostProps) {
  return (
    <article>
      {/* --- Métadonnées (Hissées dans le Head) --- */}
      <title>{title} | Mon Blog Tech</title>
      <meta name="description" content={description} />
      <meta name="author" content={author} />
      
      {/* --- Contenu Visible --- */}
      <header>
        <h1>{title}</h1>
        <p>Par {author}</p>
      </header>
      <p>{description}</p>
      {/* ... reste de l'article */}
    </article>
  );
}
```
</details>

### Exercice 2 - Carte Twitter / Open Graph {#exercice-2---carte-twitter-og}

🎯 **Objectif** : Rendre votre page partageable sur les réseaux sociaux.

💼 **Mise en situation** : Le marketing veut que les liens partagés sur Twitter affichent une "Grande Image".

📝 **Énoncé** :
1. Créez un composant `SocialMeta`.
2. Il doit générer les balises pour `twitter:card` (valeur "summary_large_image"), `twitter:title`, et `og:image`.
3. Intégrez-le dans une page fictive.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
function SocialMeta({ title, imageUrl }: { title: string, imageUrl: string }) {
  return (
    <>
      {/* Configuration Open Graph standard */}
      <meta property="og:title" content={title} />
      <meta property="og:image" content={imageUrl} />
      
      {/* Configuration spécifique Twitter */}
      <meta name="twitter:card" content="summary_large_image" />
      <meta name="twitter:title" content={title} />
      <meta name="twitter:image" content={imageUrl} />
    </>
  );
}

// Utilisation
export function LandingPage() {
  return (
    <div>
      <SocialMeta 
        title="Lancement Produit V2" 
        imageUrl="https://mon-site.com/og-launch.jpg" 
      />
      <h1>Bienvenue sur la V2 !</h1>
    </div>
  );
}
```
</details>

### Exercice 3 - Injection de CSS Conditionnelle {#exercice-3---injection-css-conditionnelle}

🎯 **Objectif** : Charger un thème "Sombre" via CSS uniquement si nécessaire.

💼 **Mise en situation** : Votre application a un thème sombre optionnel. Le fichier `dark-theme.css` est lourd, on ne veut le charger que si l'utilisateur active le mode sombre.

📝 **Énoncé** :
1. Utilisez un état `isDark`.
2. Si `isDark` est true, rendez une balise `<link rel="stylesheet">`.
3. Utilisez `precedence="high"` pour s'assurer qu'il surcharge le style de base.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState } from 'react';

export function ThemeManager() {
  const [isDark, setIsDark] = useState(false);

  return (
    <div className={isDark ? "dark-mode-container" : ""}>
      
      {/* Injection conditionnelle du CSS */}
      {isDark && (
        <link 
          rel="stylesheet" 
          href="/themes/dark-theme.css" 
          // @ts-expect-error : precedence est une nouveauté React 19 pas toujours typée
          precedence="high" 
        />
      )}

      <h1>Gestion du Thème</h1>
      <button onClick={() => setIsDark(!isDark)}>
        Basculer en mode {isDark ? "Clair" : "Sombre"}
      </button>
      
      <p>Le fond changera de couleur si le fichier CSS est chargé.</p>
    </div>
  );
}
```
</details>
```