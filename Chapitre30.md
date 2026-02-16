Voici le chapitre **`useContext` : Passer des Données en Profondeur** pour la formation React 19.2.

```markdown
---
sidebar_label: `useContext` : Passer des Données en Profondeur (API Context)
sidebar_position: 30
---

# Chapitre 30 : `useContext` : Passer des Données en Profondeur (API Context)

Créer un Context, Fournir des valeurs, Consommer des valeurs, Éviter le prop drilling

En React, les données circulent généralement de haut en bas (du parent vers l'enfant) via les **props**. C'est explicite et robuste.
Cependant, cela devient pénible lorsque vous devez passer une prop à travers 10 niveaux de composants intermédiaires qui ne l'utilisent pas eux-mêmes, juste pour l'amener au composant tout en bas de l'arbre. C'est ce qu'on appelle le **Prop Drilling**.

L'API Context et le Hook `useContext` permettent de "téléporter" des données d'un composant parent vers n'importe quel composant descendant, peu importe sa profondeur, sans passer par les props intermédiaires.

## Le Problème du Prop Drilling {#le-probleme-du-prop-drilling}

### 1. Quoi
Le "Prop Drilling" (perçage de props) survient quand un composant A doit envoyer une donnée à un composant Z, mais que A et Z sont séparés par B, C, D et E. Vous êtes obligé de passer la prop à B, qui la passe à C, etc.

### 2. Pourquoi
Bien que le flux explicite soit une bonne chose, le prop drilling rend les composants intermédiaires (B, C, D) dépendants de données qui ne les concernent pas. Cela complexifie le refactoring et alourdit le code.

### 3. Comment (L'approche "Avant Context")

Imaginez que vous vouliez passer l'utilisateur connecté (`user`) jusqu'au bouton de profil (`Avatar`) situé tout en bas.

```tsx
// ❌ AVANT : Enfer du Prop Drilling
function App() {
  const user = { name: "Alice", role: "Admin" };
  return <Layout user={user} />; // Layout n'a pas besoin de user
}

function Layout({ user }) {
  return (
    <header>
      <Navigation user={user} /> {/* Navigation n'a pas besoin de user */}
    </header>
  );
}

function Navigation({ user }) {
  return (
    <nav>
      <UserMenu user={user} /> {/* UserMenu n'a pas besoin de user */}
    </nav>
  );
}

function UserMenu({ user }) {
  return <Avatar user={user} />; // ENFIN ! Avatar utilise user
}
```

---

## L'API Context : Créer, Fournir, Consommer {#l-api-context-creer-fournir-consommer}

### 1. Quoi
Context fonctionne comme un système de diffusion (broadcast).
1.  **Créer** : On définit un canal de communication.
2.  **Fournir (Provider)** : Un composant parent émet des données sur ce canal.
3.  **Consommer (Consumer/Hook)** : N'importe quel enfant se branche sur le canal pour lire les données.

### 2. Pourquoi
Pour partager des données "globales" à une partie de l'arbre (Thème UI, Utilisateur authentifié, Langue, Panier d'achat) sans polluer les props.

### 3. Comment

#### A. Syntaxe de base

Il y a 3 étapes distinctes.

**Étape 1 : Créer le Context**
Généralement dans un fichier séparé.

```tsx
import { createContext } from 'react';

// 1. Définir le type des données
interface ThemeContextType {
  theme: 'light' | 'dark';
  toggleTheme: () => void;
}

// 2. Créer le contexte avec une valeur par défaut (souvent null ou un stub)
export const ThemeContext = createContext<ThemeContextType | null>(null);
```

**Étape 2 : Fournir la valeur (Provider)**
Dans un composant parent (souvent à la racine).

```tsx
import { useState } from 'react';
import { ThemeContext } from './ThemeContext';

export function App() {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');

  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };

  // La valeur passée ici sera accessible par TOUS les descendants
  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      <Layout />
    </ThemeContext.Provider>
  );
}
```

**Étape 3 : Consommer la valeur (`useContext`)**
Dans n'importe quel composant enfant, même très profond.

```tsx
import { useContext } from 'react';
import { ThemeContext } from './ThemeContext';

export function ThemeToggler() {
  // On récupère l'objet passé dans le Provider
  const context = useContext(ThemeContext);

  // Sécurité TypeScript : context peut être null si utilisé hors du Provider
  if (!context) {
    throw new Error("ThemeToggler doit être utilisé dans un ThemeContext.Provider");
  }

  const { theme, toggleTheme } = context;

  return (
    <button onClick={toggleTheme} style={{ background: theme === 'dark' ? '#333' : '#fff' }}>
      Mode actuel : {theme}
    </button>
  );
}
```

### 4. Zone de Danger

:::danger Performance et Re-rendus
Chaque fois que la valeur du `Provider` change (même une petite partie), **TOUS** les composants qui consomment ce contexte (`useContext`) se re-rendent.
Si vous mettez tout votre état d'application dans un seul Context géant, vous aurez des problèmes de performance.
**Solution** : Séparez les contextes (ex: `UserContext`, `ThemeContext`, `NotificationContext`) ou utilisez des gestionnaires d'état comme Redux/Zustand pour les cas très complexes.
:::

---

## Pattern : Le Hook Personnalisé de Context {#pattern-le-hook-personnalise-de-context}

### 1. Quoi
Au lieu d'importer le contexte brut et de faire `useContext(MyContext)` dans chaque composant, on crée un Hook dédié (ex: `useAuth`, `useTheme`).

### 2. Pourquoi
1.  **Abstraction** : Les composants ne savent pas qu'il s'agit d'un Context.
2.  **Sécurité** : On peut gérer l'erreur "Context is undefined" (oubli du Provider) une seule fois.
3.  **Clean Code** : Import plus propre.

### 3. Comment

```tsx
// AuthContext.tsx
import { createContext, useContext, useState } from 'react';

// Types
interface User { id: string; name: string }
interface AuthContextType { user: User | null; login: (name: string) => void; }

const AuthContext = createContext<AuthContextType | null>(null);

// Composant Provider encapsulé (souvent exporté aussi)
export function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<User | null>(null);

  const login = (name: string) => setUser({ id: '123', name });

  return (
    <AuthContext.Provider value={{ user, login }}>
      {children}
    </AuthContext.Provider>
  );
}

// ✅ Le Hook Personnalisé (seul export nécessaire pour les consommateurs)
export function useAuth() {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error("useAuth doit être utilisé à l'intérieur d'un <AuthProvider />");
  }
  return context;
}
```

**Utilisation :**
```tsx
import { useAuth } from './AuthContext';

function Profile() {
  const { user, login } = useAuth(); // Simple, typé et sécurisé
  
  if (!user) return <button onClick={() => login("Bob")}>Se connecter</button>;
  return <div>Bienvenue, {user.name}</div>;
}
```

### 🚨 Limitations de l'API Context
Context n'est **PAS** conçu pour des mises à jour à haute fréquence (comme un jeu vidéo, ou un champ input qui change à chaque frappe 60 fois par seconde) si beaucoup de composants le consomment.
Pour des données qui changent très vite, préférez l'état local ou une librairie de gestion d'état atomique (Recoil, Jotai, Zustand) ou `useSyncExternalStore`.

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-30}

1.  **Quel problème principal le Context résout-il ?**
    Le Prop Drilling (passer des props à travers des composants intermédiaires inutiles).

2.  **Quels sont les trois éléments nécessaires pour faire fonctionner un Context ?**
    `createContext` (la définition), `<Context.Provider>` (l'émetteur), et `useContext` (le récepteur).

3.  **Que se passe-t-il si j'appelle `useContext` dans un composant qui n'est pas enveloppé par le Provider correspondant ?**
    `useContext` retournera la valeur par défaut définie lors du `createContext` (souvent `null` ou `undefined`).

4.  **Pourquoi est-il recommandé d'envelopper `useContext` dans un Hook personnalisé ?**
    Pour centraliser la gestion des erreurs (vérifier si le provider existe) et simplifier les imports dans les composants consommateurs.

---

## Exercices : {#exercices-30}

### Exercice 1 - Le Thème Jour/Nuit {#exercice-1---le-theme-jour-nuit}

🎯 **Objectif** : Implémenter un contexte basique pour changer une classe CSS globale.

💼 **Mise en situation** : Votre application SaaS doit supporter le Dark Mode. Ce choix doit impacter toute l'application (Header, Cards, Footer).

📝 **Énoncé** :
1. Créez `ThemeContext` et `ThemeProvider`.
2. Le Provider gère un state `mode` ('light' | 'dark').
3. Créez un hook `useTheme()`.
4. Dans `App`, enveloppez le contenu avec le Provider.
5. Créez un composant `Card` profond qui change de couleur de fond selon le thème.
6. Créez un bouton dans le Header pour basculer le thème.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Deux captures côte à côte : une en mode clair, une en mode sombre.
> **Annotation** : Montrez que le bouton est dans le Header mais que la Card (enfant profond) change de couleur.
> **Alt Text suggéré** : Démonstration du basculement de thème via React Context.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { createContext, useContext, useState } from 'react';

// 1. Définition et Hook
type Theme = 'light' | 'dark';
type ThemeContextType = { theme: Theme; toggle: () => void };

const ThemeContext = createContext<ThemeContextType | null>(null);

function useTheme() {
  const ctx = useContext(ThemeContext);
  if (!ctx) throw new Error("Missing ThemeProvider");
  return ctx;
}

// 2. Provider
function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<Theme>('light');
  const toggle = () => setTheme(t => t === 'light' ? 'dark' : 'light');

  return (
    <ThemeContext.Provider value={{ theme, toggle }}>
      <div style={{ 
        minHeight: '100vh', 
        background: theme === 'light' ? '#fff' : '#222',
        color: theme === 'light' ? '#000' : '#fff',
        transition: 'all 0.3s'
      }}>
        {children}
      </div>
    </ThemeContext.Provider>
  );
}

// 3. Consommateurs
function Header() {
  const { toggle } = useTheme();
  return (
    <header style={{ borderBottom: '1px solid gray', padding: 10 }}>
      <h1>Mon SaaS</h1>
      <button onClick={toggle}>Basculer le thème</button>
    </header>
  );
}

function DeepCard() {
  const { theme } = useTheme();
  return (
    <div style={{ 
      margin: 20, padding: 20, 
      border: '1px solid currentColor',
      borderRadius: 8,
      boxShadow: theme === 'light' ? '0 2px 5px rgba(0,0,0,0.1)' : '0 2px 5px rgba(255,255,255,0.1)'
    }}>
      Je suis un composant profond qui réagit au contexte !
    </div>
  );
}

export function App() {
  return (
    <ThemeProvider>
      <Header />
      <main>
        <DeepCard />
      </main>
    </ThemeProvider>
  );
}
```
</details>

### Exercice 2 - Gestion Utilisateur (AuthContext) {#exercice-2---gestion-utilisateur-authcontext}

🎯 **Objectif** : Stocker un objet complexe et des méthodes dans le Contexte.

💼 **Mise en situation** : Un tableau de bord administrateur. Certaines sections ne sont visibles que si l'utilisateur est connecté et a le rôle "ADMIN".

📝 **Énoncé** :
1. Créez un `AuthProvider` qui gère un `user` (`{ name: string, role: 'USER' | 'ADMIN' }` ou `null`).
2. Exposez `login(name, role)` et `logout()`.
3. Créez un composant `AdminPanel` qui utilise `useAuth`.
4. Si l'utilisateur n'est pas "ADMIN", affichez "Accès Interdit". Sinon, affichez les contrôles secrets.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { createContext, useContext, useState } from 'react';

// Types
type Role = 'USER' | 'ADMIN';
type User = { name: string; role: Role };

interface AuthContextType {
  user: User | null;
  login: (name: string, role: Role) => void;
  logout: () => void;
}

const AuthContext = createContext<AuthContextType | null>(null);

export function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<User | null>(null);

  const login = (name: string, role: Role) => setUser({ name, role });
  const logout = () => setUser(null);

  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

// Hook Custom
const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) throw new Error("useAuth must be used within AuthProvider");
  return context;
};

// Composants
function AdminPanel() {
  const { user } = useAuth();

  if (!user) return <p>Veuillez vous connecter.</p>;
  if (user.role !== 'ADMIN') return <p style={{ color: 'red' }}>⛔ Accès Interdit (Réservé aux Admins)</p>;

  return (
    <div style={{ background: '#f0f9ff', padding: 20, border: '1px solid #0984e3' }}>
      <h3>🛠️ Panneau d'Administration</h3>
      <p>Bienvenue Chef {user.name} ! Vous avez le contrôle total.</p>
      <button>Supprimer la base de données (Fake)</button>
    </div>
  );
}

function LoginForm() {
  const { login, logout, user } = useAuth();

  if (user) {
    return <button onClick={logout}>Se déconnecter ({user.name})</button>;
  }

  return (
    <div style={{ gap: 10, display: 'flex' }}>
      <button onClick={() => login("Alice", "USER")}>Login Alice (User)</button>
      <button onClick={() => login("Bob", "ADMIN")}>Login Bob (Admin)</button>
    </div>
  );
}

export function App() {
  return (
    <AuthProvider>
      <div style={{ padding: 20 }}>
        <LoginForm />
        <hr />
        <AdminPanel />
      </div>
    </AuthProvider>
  );
}
```
</details>

### Exercice 3 - Le Sélecteur de Langue (i18n) {#exercice-3---le-selecteur-de-langue-i18n}

🎯 **Objectif** : Utiliser le contexte pour distribuer un dictionnaire de traduction.

💼 **Mise en situation** : Une application e-commerce internationale. L'utilisateur peut changer la langue (FR/EN) et tous les textes doivent se mettre à jour instantanément.

📝 **Énoncé** :
1. Créez un dictionnaire simple :
   ```js
   const translations = {
     en: { welcome: "Welcome", cart: "Cart" },
     fr: { welcome: "Bienvenue", cart: "Panier" }
   };
   ```
2. Le Context fournit la langue actuelle (`lang`), la fonction pour changer (`setLang`), et surtout la fonction de traduction `t(key)`.
3. Le composant `App` affiche un titre traduit et un bouton pour changer de langue.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { createContext, useContext, useState } from 'react';

// Dictionnaire simple
const dictionary = {
  en: { welcome: "Welcome to our Shop", items: "Items in cart", checkout: "Checkout" },
  fr: { welcome: "Bienvenue dans notre boutique", items: "Articles au panier", checkout: "Payer" }
};

type Lang = 'en' | 'fr';
type I18nContextType = {
  lang: Lang;
  setLang: (l: Lang) => void;
  t: (key: keyof typeof dictionary['en']) => string;
};

const I18nContext = createContext<I18nContextType | null>(null);

// Provider
function I18nProvider({ children }: { children: React.ReactNode }) {
  const [lang, setLang] = useState<Lang>('fr');

  // Fonction helper de traduction
  const t = (key: keyof typeof dictionary['en']) => {
    return dictionary[lang][key];
  };

  return (
    <I18nContext.Provider value={{ lang, setLang, t }}>
      {children}
    </I18nContext.Provider>
  );
}

// Hook
const useTranslation = () => {
  const ctx = useContext(I18nContext);
  if (!ctx) throw new Error("No I18nProvider");
  return ctx;
};

// Composant UI
function ShopUI() {
  const { t, lang, setLang } = useTranslation();

  return (
    <div>
      <div style={{ display: 'flex', justifyContent: 'space-between', alignItems: 'center' }}>
        <h1>{t('welcome')} 🌍</h1>
        <select value={lang} onChange={(e) => setLang(e.target.value as Lang)}>
          <option value="fr">Français</option>
          <option value="en">English</option>
        </select>
      </div>
      <p>5 {t('items')}</p>
      <button>{t('checkout')}</button>
    </div>
  );
}

export function App() {
  return (
    <I18nProvider>
      <ShopUI />
    </I18nProvider>
  );
}
```
</details>
```