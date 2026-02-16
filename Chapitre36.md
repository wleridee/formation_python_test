Voici le chapitre **`useId`: Générer des ID Uniques** pour la formation React 19.2.

```markdown
---
sidebar_label: `useId`: Générer des ID Uniques
sidebar_position: 36
---

# Chapitre 36 : `useId`: Générer des ID Uniques

Accessibilité, Hydratation, Génération d'ID côté client/serveur

Dans le développement web, relier deux éléments HTML entre eux via des identifiants (attributs `id`) est courant, surtout pour l'accessibilité (lier un `<label>` à un `<input>`).

Cependant, générer des ID uniques dans une application React pose deux problèmes majeurs :
1.  **Conflits** : Si vous utilisez un compteur global ou `Math.random()`, vous risquez des doublons ou des valeurs instables.
2.  **Hydratation (SSR)** : Si le serveur génère l'ID "input-123" et que le client (navigateur) génère "input-456" au premier rendu, React lèvera une erreur d'hydratation ("Hydration Mismatch").

Le Hook `useId` résout ces problèmes en générant des identifiants uniques et **stables** entre le serveur et le client.

## Le Problème de l'Hydratation et des IDs {#le-probleme-de-l-hydratation}

### 1. Quoi
L'hydratation est le processus où React "attache" les écouteurs d'événements au HTML généré par le serveur. Pour que cela fonctionne, le HTML du serveur et le premier rendu du client doivent être **identiques au caractère près**.

### 2. Pourquoi
Si vous faites ceci :
```javascript
const id = Math.random(); // ❌
```
Le serveur générera `0.123` et le client `0.987`. React ne saura pas si c'est le même élément, entraînant une erreur console et potentiellement des bugs visuels ou d'accessibilité.

### 3. Comment fonctionne `useId`
`useId` ne génère pas un nombre aléatoire, mais une chaîne basée sur la **position du composant dans l'arbre React**. Comme la structure de l'arbre est la même sur le serveur et le client, l'ID généré est identique.

---

## Utilisation Basique : Lier Labels et Inputs {#utilisation-basique}

### 1. Quoi
La fonction principale de `useId` est de créer des liens accessibles.

### 2. Pourquoi
Pour les lecteurs d'écran (aveugles ou malvoyants), un `<input>` doit être programmatiquement lié à son `<label>`. Cliquer sur le label doit aussi donner le focus à l'input.

### 3. Comment

#### A. Syntaxe de base

```tsx
import { useId } from 'react';

export function EmailField() {
  // Génère un ID unique et stable (ex: ":r1:")
  const id = useId();

  return (
    <div style={{ display: 'flex', flexDirection: 'column', gap: 5 }}>
      {/* On utilise l'ID pour le htmlFor */}
      <label htmlFor={id}>Adresse Email :</label>
      
      {/* Et pour l'attribut id de l'input */}
      <input id={id} type="email" />
    </div>
  );
}
```

:::info Format de l'ID
L'ID généré ressemble souvent à `:r0:`, `:r1:`, etc. Les deux-points `:` font partie de la spécification interne de React. N'essayez pas de prédire ou de parser ce format, traitez-le comme une chaîne opaque.
:::

---

## Accessibilité Avancée (ARIA) {#accessibilite-avancee-aria}

### 1. Quoi
`useId` est indispensable pour les attributs ARIA comme `aria-describedby` ou `aria-labelledby`, qui référencent d'autres éléments par leur ID.

### 2. Pourquoi
Si un champ de mot de passe a une consigne ("8 caractères min"), un utilisateur voyant le lit juste en dessous. Un utilisateur non-voyant a besoin que l'input lui annonce cette description via `aria-describedby`.

### 3. Comment

#### B. Cas concret : Champ avec description

```tsx
import { useId } from 'react';

export function PasswordField() {
  const passwordId = useId();
  const hintId = useId(); // Génère un second ID unique

  return (
    <div>
      <label htmlFor={passwordId}>Mot de passe</label>
      <input 
        id={passwordId} 
        type="password" 
        // Lie l'input au texte d'aide pour les lecteurs d'écran
        aria-describedby={hintId} 
      />
      
      {/* Le texte d'aide porte l'ID référencé */}
      <p id={hintId} style={{ fontSize: '0.8rem', color: '#666' }}>
        Doit contenir au moins 8 caractères et un symbole.
      </p>
    </div>
  );
}
```

---

## Optimisation : IDs Multiples avec Suffixes {#optimisation-ids-multiples}

### 1. Quoi
Au lieu d'appeler `useId()` 10 fois dans un formulaire complexe, appelez-le une seule fois et ajoutez des suffixes.

### 2. Pourquoi
Cela réduit la surcharge mémoire (un seul Hook vs plusieurs) et garde le code plus propre.

### 3. Comment

```tsx
import { useId } from 'react';

export function UserForm() {
  // Un seul ID de base pour tout le composant
  const baseId = useId();

  return (
    <form>
      <div>
        {/* Construction d'IDs dérivés */}
        <label htmlFor={`${baseId}-firstName`}>Prénom</label>
        <input id={`${baseId}-firstName`} type="text" />
      </div>
      
      <div>
        <label htmlFor={`${baseId}-lastName`}>Nom</label>
        <input id={`${baseId}-lastName`} type="text" />
      </div>
    </form>
  );
}
```

---

## Zone de Danger {#zone-de-danger}

### ❌ Ne PAS utiliser pour les `key` de liste

C'est l'erreur la plus fréquente.

```tsx
// ❌ MAUVAIS : Ne faites jamais ça
const id = useId();
return list.map(item => <li key={`${id}-${item.name}`}>...</li>);
```

**Pourquoi ?** Si l'ordre de la liste change, `useId` restera le même (car il dépend de la position du composant parent, pas des données). React va s'emmêler les pinceaux lors de la réconciliation.
**Solution** : Utilisez les IDs provenant de vos données (`item.id`).

### ❌ Ne PAS utiliser pour les sélecteurs CSS

```css
/* ❌ Ne faites pas ça */
#\:r1\: { color: red; }
```
Les IDs générés par React contiennent des deux-points `:`, qui sont des caractères spéciaux en CSS (pseudo-classes). De plus, l'ID changera si vous déplacez le composant. Utilisez des `className`.

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-36}

1.  **Pourquoi ne doit-on pas utiliser `Math.random()` pour générer des IDs dans un composant React ?**
    Car cela crée une différence entre le rendu serveur (SSR) et le rendu client, provoquant une erreur d'hydratation.

2.  **Peut-on utiliser `useId` pour générer les props `key` dans une boucle `map()` ?**
    **Non**. Les clés doivent être liées aux données, tandis que `useId` est lié à la structure de l'arbre des composants.

3.  **À quoi ressemble généralement la chaîne retournée par `useId` ?**
    Une chaîne contenant des deux-points, comme `:r0:` ou `:r5:`.

---

## Exercices : {#exercices-36}

### Exercice 1 - Réparer le Formulaire Cassé {#exercice-1---reparer-le-formulaire-casse}

🎯 **Objectif** : Remplacer une génération d'ID aléatoire instable par `useId`.

💼 **Mise en situation** : Vous reprenez le code d'un stagiaire. Le formulaire fonctionne, mais la console affiche des avertissements "Hydration failed" et les labels ne fonctionnent pas toujours après un rechargement.

📝 **Énoncé** :
1. Observez le code fourni (qui utilise `Math.random()`).
2. Remplacez la logique par `useId`.
3. Vérifiez que cliquer sur le Label "Newsletter" coche bien la case associée.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState, useId } from 'react';

export function NewsletterSignup() {
  // ❌ AVANT : Instable, change à chaque render, casse le SSR
  // const uniqueId = "input-" + Math.floor(Math.random() * 1000);

  // ✅ APRÈS : Stable et unique
  const uniqueId = useId();

  const [checked, setChecked] = useState(false);

  return (
    <div style={{ padding: 20, border: '1px solid #ddd' }}>
      <h3>Restez informé</h3>
      <div style={{ display: 'flex', alignItems: 'center', gap: 10 }}>
        {/* L'attribut 'for' en HTML s'écrit 'htmlFor' en React */}
        <label htmlFor={uniqueId} style={{ cursor: 'pointer' }}>
          S'abonner à la newsletter
        </label>
        
        <input 
          id={uniqueId} 
          type="checkbox" 
          checked={checked} 
          onChange={e => setChecked(e.target.checked)} 
        />
      </div>
    </div>
  );
}
```
</details>

### Exercice 2 - L'Input Accessible (ARIA) {#exercice-2---l-input-accessible}

🎯 **Objectif** : Lier un input à un message d'erreur conditionnel.

💼 **Mise en situation** : Dans un formulaire de connexion, si l'email est invalide, un message rouge apparaît. Les lecteurs d'écran doivent savoir que ce message est lié à l'input.

📝 **Énoncé** :
1. Créez un champ email.
2. Créez un état d'erreur (booléen).
3. Générez un ID pour le message d'erreur avec `useId`.
4. Si l'erreur est présente, l'input doit avoir `aria-invalid="true"` et `aria-describedby={errorId}`.
5. Affichez le message d'erreur dans une `<p>` ou `<span>` portant cet `errorId`.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Formulaire avec message d'erreur.
> **Annotation** : Mettez en évidence l'inspecteur DOM montrant que `aria-describedby` de l'input correspond à l'`id` du message d'erreur.
> **Alt Text suggéré** : Inspection DOM montrant le lien ARIA entre input et erreur.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState, useId } from 'react';

export function LoginInput() {
  const [email, setEmail] = useState('');
  const isError = email.length > 0 && !email.includes('@');
  
  // On génère deux IDs : un pour l'input, un pour le message d'erreur
  const inputId = useId();
  const errorMsgId = useId();

  return (
    <div>
      <label htmlFor={inputId} style={{ display: 'block', marginBottom: 5 }}>
        Email professionnel
      </label>
      
      <input
        id={inputId}
        type="email"
        value={email}
        onChange={e => setEmail(e.target.value)}
        // Indique aux technologies d'assistance que le champ est invalide
        aria-invalid={isError}
        // Si erreur, on lie l'input à la description de l'erreur
        aria-describedby={isError ? errorMsgId : undefined}
        style={{ borderColor: isError ? 'red' : '#ccc' }}
      />

      {isError && (
        // Ce paragraphe porte l'ID référencé par l'input
        <p id={errorMsgId} style={{ color: 'red', fontSize: '0.9em', marginTop: 5 }}>
          ⚠️ Veuillez entrer une adresse email valide.
        </p>
      )}
    </div>
  );
}
```
</details>

### Exercice 3 - Le Composant Composé (Suffixes) {#exercice-3---le-composant-compose}

🎯 **Objectif** : Créer un composant réutilisable "Adresse" qui utilise un seul `useId` pour 4 champs.

💼 **Mise en situation** : Vous créez un widget "Adresse de Livraison" qui sera utilisé plusieurs fois sur la même page (Adresse Facturation vs Adresse Livraison). Il ne doit pas y avoir de conflit d'IDs entre les deux instances.

📝 **Énoncé** :
1. Créez un composant `AddressGroup`.
2. Générez un `baseId`.
3. Créez 3 champs : Rue, Ville, Code Postal.
4. Les IDs doivent être `${baseId}-street`, `${baseId}-city`, etc.
5. Instanciez ce composant deux fois dans `App` pour prouver qu'il n'y a pas de conflit.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useId } from 'react';

function AddressGroup({ title }: { title: string }) {
  // Un seul hook call pour tout le groupe
  const id = useId();

  return (
    <fieldset style={{ marginBottom: 20, padding: 15 }}>
      <legend>{title}</legend>
      
      <div style={{ marginBottom: 10 }}>
        {/* Suffixe : -street */}
        <label htmlFor={`${id}-street`} style={{ display: 'block' }}>Rue</label>
        <input id={`${id}-street`} type="text" placeholder="123 React Lane" />
      </div>

      <div style={{ marginBottom: 10 }}>
        {/* Suffixe : -city */}
        <label htmlFor={`${id}-city`} style={{ display: 'block' }}>Ville</label>
        <input id={`${id}-city`} type="text" placeholder="San Francisco" />
      </div>
      
      <div>
        {/* Suffixe : -zip */}
        <label htmlFor={`${id}-zip`} style={{ display: 'block' }}>Code Postal</label>
        <input id={`${id}-zip`} type="text" placeholder="94000" />
      </div>
    </fieldset>
  );
}

export function CheckoutPage() {
  return (
    <div>
      <h2>Validation de la commande</h2>
      {/* Deux instances indépendantes -> IDs uniques garantis */}
      <AddressGroup title="Adresse de Livraison" />
      <AddressGroup title="Adresse de Facturation" />
    </div>
  );
}
```
</details>
```