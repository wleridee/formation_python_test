Voici le chapitre **Installation et environnement de développement** pour la formation React 19.2.

```markdown
---
sidebar_label: Installation et environnement de développement
sidebar_position: 2
---

# Chapitre 2 : Installation et environnement de développement

Configuration Node.js,Create React App,Environnement IDE,Hello World React

Ce chapitre couvre l'installation des outils nécessaires pour développer avec React 19.2. Nous définirons un environnement de développement professionnel, robuste et moderne, loin des configurations obsolètes que vous pourriez trouver dans d'anciens tutoriels.

## Configuration Node.js {#configuration-node-js}

### 1. Quoi
**Node.js** est un environnement d'exécution JavaScript hors du navigateur. Bien que React soit une bibliothèque côté client (exécutée dans le navigateur de l'utilisateur), nous avons besoin de Node.js pour faire tourner nos outils de développement (compilateur, serveur local, gestionnaire de paquets).

### 2. Pourquoi
React moderne ne s'utilise pas en important une balise `<script>` dans un fichier HTML. Il nécessite une étape de **build** pour :
- Transformer le JSX en JavaScript standard.
- Compiler TypeScript.
- Optimiser et minifier le code pour la production.
- Gérer les dépendances via **npm** (Node Package Manager) ou ses alternatives (pnpm, yarn).

### 3. Comment
Pour React 19, une version récente de Node.js est impérative.

1.  Rendez-vous sur [nodejs.org](https://nodejs.org/).
2.  Téléchargez la version **LTS** (Long Term Support) actuelle (minimum **v20.x** ou **v22.x** recommandées pour 2026).
3.  Installez-la avec les paramètres par défaut.

Vérifiez l'installation dans votre terminal :
```bash
node -v
# Doit retourner v20.10.0 ou supérieur
npm -v
# Doit retourner 10.x ou supérieur
```

### 4. Zone de Danger

:::danger N'utilisez pas de versions obsolètes
React 19 et ses outils (Vite, Next.js) utilisent des fonctionnalités modernes de Node.js. Utiliser une version 14, 16 ou même 18 risque de provoquer des erreurs cryptiques lors de l'installation des paquets.
:::

---

## Create React App (CRA) : Le Grand Remplacement {#create-react-app}

### 1. Quoi
**Create React App** (`create-react-app` ou CRA) a été pendant des années l'outil officiel recommandé par Facebook pour créer un projet React.
Cependant, en 2026, **CRA est officiellement déprécié** et considéré comme un outil "legacy".

### 2. Pourquoi
CRA repose sur Webpack, un outil puissant mais devenu lent comparé aux standards modernes. De plus, CRA masque trop la configuration, rendant difficile l'adaptation aux nouvelles fonctionnalités de React 19.

Le standard actuel pour une application React "Client-Side" (SPA) est **Vite**.
Le standard pour une application React "Full-Stack" est un framework comme **Next.js** ou **React Router v7**.

### 3. Comment (La méthode moderne : Vite)
Dans ce cours, nous utiliserons **Vite** pour apprendre React pur. Vite est extrêmement rapide (écrit en Go/Rust) et ne nécessite presque aucune configuration.

### 4. Zone de Danger

:::warning Ne tapez pas cette commande
Si vous voyez un tutoriel vous demandant de faire :
`npx create-react-app mon-projet`
❌ **Fuyez.** Ce tutoriel est obsolète. Vous obtiendrez un projet lourd, lent et difficile à mettre à jour.
:::

---

## Environnement IDE {#environnement-ide}

### 1. Quoi
L'éditeur de code standard de l'industrie pour React est **VS Code** (Visual Studio Code).

### 2. Pourquoi
Son support natif de **TypeScript** (le langage que nous utiliserons) et son immense écosystème d'extensions en font l'outil idéal. React 19 étant fortement typé, l'autocomplétion de VS Code vous fera gagner des heures de débogage.

### 3. Comment
Installez VS Code, puis ajoutez impérativement ces extensions :

1.  **ES7+ React/Redux/React-Native snippets** : Raccourcis pour générer des composants (`rafce`...).
2.  **Prettier - Code formatter** : Formate automatiquement votre code à la sauvegarde.
3.  **Tailwind CSS IntelliSense** (si vous utilisez Tailwind).
4.  **Pretty TypeScript Errors** : Rend les erreurs TypeScript lisibles pour les humains.

Activez le "Format on Save" dans les paramètres de VS Code :
`Settings` > cherchez "Format on Save" > Cochez la case.

---

## Hello World React {#hello-world-react}

### 1. Quoi
Nous allons initialiser notre premier projet React 19 avec Vite et TypeScript.

### 2. Pourquoi
Cela permet de vérifier que toute votre chaîne d'outils (Node, npm, IDE) fonctionne correctement avant d'attaquer la théorie.

### 3. Comment

Ouvrez votre terminal dans votre dossier de projets et lancez :

```bash
# 1. Création du projet avec Vite
npm create vite@latest mon-app-react -- --template react-ts

# 2. Entrer dans le dossier
cd mon-app-react

# 3. Installer les dépendances
npm install

# 4. Lancer le serveur de développement
npm run dev
```

> **Explication de la commande :**
> *   `npm create vite@latest` : Télécharge la dernière version de l'outil de création Vite.
> *   `--template react-ts` : Indique qu'on veut un projet React avec **TypeScript** (standard 2026).

Une fois la commande `npm run dev` lancée, Vite vous donnera une URL locale (souvent `http://localhost:5173`). Ouvrez-la dans votre navigateur.

Vous devriez voir le logo React tourner. Félicitations, votre environnement est prêt !

### 4. Zone de Danger

:::danger Ne touchez pas au dossier node_modules
Le dossier `node_modules` contient des milliers de fichiers (les bibliothèques tierces). Ne le modifiez jamais manuellement, ne le versionnez pas avec Git (il doit être dans votre `.gitignore`). Si vous le supprimez, `npm install` le recréera.
:::

### 🚨 Limitations de Vite
Vite crée une "Single Page Application" (SPA).
*   **Avantage** : Parfait pour apprendre React, créer des dashboards ou des outils internes.
*   **Limitation** : Le SEO (référencement Google) est moins performant par défaut qu'avec Next.js, car la page est générée côté client. Pour ce cours d'apprentissage, c'est l'outil idéal car il isole React de la complexité serveur.

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-2}

1.  **Pourquoi ne doit-on plus utiliser `create-react-app` en 2026 ?**
    Car il est obsolète, lent, et non maintenu. Il a été remplacé par des outils modernes comme **Vite** ou des frameworks comme Next.js.

2.  **Quelle est la commande pour lancer le serveur de développement avec Vite ?**
    `npm run dev`.

3.  **Quelle extension de fichier utilise-t-on pour un composant React avec TypeScript ?**
    `.tsx` (TypeScript XML). Les fichiers `.ts` ne peuvent pas contenir de JSX.

4.  **À quoi sert le dossier `node_modules` ?**
    Il contient toutes les dépendances (librairies) du projet téléchargées par npm. Il ne doit jamais être modifié manuellement.
```