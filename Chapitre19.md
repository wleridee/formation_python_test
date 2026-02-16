Voici le chapitre **Partager l'État entre Composants (Lifting State Up)** pour la formation React 19.2.

```markdown
---
sidebar_label: Partager l'État entre Composants (Lifting State Up)
sidebar_position: 19
---

# Chapitre 19 : Partager l'État entre Composants (Lifting State Up)

État local vs partagé, Remonter l'état, Flux de données unidirectionnel

Jusqu'à présent, chaque composant que nous avons créé gérait ses propres affaires : il avait son propre `useState`, ses propres données, et vivait sa vie de manière isolée.
Mais dans une vraie application, les composants doivent communiquer.
Imaginez une barre de recherche (Composant A) qui doit filtrer une liste de produits (Composant B). Comment A transmet-il sa valeur à B ?

En React, deux composants enfants ne peuvent pas se parler directement. Ils doivent passer par leur parent commun. C'est le principe fondamental de la **Remontée d'État** (Lifting State Up).

## État Local vs État Partagé {#etat-local-vs-etat-partage}

### 1. Quoi
*   **État Local** : Une donnée qui n'est utilisée QUE par le composant lui-même (ex: un champ de saisie, l'état ouvert/fermé d'un menu déroulant isolé).
*   **État Partagé** : Une donnée qui affecte l'affichage de plusieurs composants simultanément.

### 2. Pourquoi
Si deux composants doivent toujours être synchronisés (afficher la même chose), ou si le changement de l'un doit affecter l'autre, ils ne peuvent pas garder une copie locale de l'état. Ils doivent partager une **Source de Vérité Unique** (Single Source of Truth).

### 3. Comment
Le principe est de retirer l'état des enfants et de le déplacer (le "remonter") vers leur parent commun le plus proche.

#### A. Situation Problématique (États désynchronisés)
Ici, chaque compteur est indépendant. Cliquer sur l'un ne change pas l'autre.

```tsx
function Counter() {
  const [count, setCount] = useState(0); // État local
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}

function Parent() {
  return (
    <>
      <Counter /> {/* Affiche 0 */}
      <Counter /> {/* Affiche 0 */}
      {/* Si je clique sur le premier, il passe à 1, le deuxième reste à 0 */}
    </>
  );
}
```

#### B. Solution (État Partagé)
Pour qu'ils avancent ensemble, l'état doit vivre dans le `Parent`.

```tsx
function Counter({ count, onIncrement }: { count: number, onIncrement: () => void }) {
  // Le composant ne "possède" plus l'état. Il le reçoit (Lecture) et demande le changement (Écriture).
  return <button onClick={onIncrement}>{count}</button>;
}

function Parent() {
  const [count, setCount] = useState(0); // État remonté ici

  return (
    <>
      <Counter count={count} onIncrement={() => setCount(count + 1)} />
      <Counter count={count} onIncrement={() => setCount(count + 1)} />
    </>
  );
}
```

---

## Le Processus de Remontée d'État {#processus-de-remontee-d-etat}

### 1. Quoi
C'est un refactoring en 3 étapes standard pour transformer un composant "non contrôlé" (qui gère son état) en composant "contrôlé" (piloté par les props).

### 2. Pourquoi
Pour coordonner l'UI. Par exemple, dans un accordéon, un seul panneau peut être ouvert à la fois. Si chaque panneau gère son propre booléen `isOpen`, ils peuvent tous être ouverts. Si le parent gère `activeIndex`, il peut forcer la fermeture des autres quand l'un s'ouvre.

### 3. Comment (Les 3 Étapes)

Prenons l'exemple de deux champs de saisie qui doivent rester synchronisés (ex: convertisseur Euros / Dollars).

**Étape 1 : Retirer l'état de l'enfant**
Supprimez `useState` de l'enfant.

**Étape 2 : Ajouter l'état au parent commun**
Déclarez `useState` dans le parent qui contient les deux enfants.

**Étape 3 : Passer les données et les fonctions de contrôle**
Le parent passe deux choses à l'enfant via les props :
1.  **La donnée** (ex: `value`) : Pour l'affichage.
2.  **Le callback** (ex: `onChange`) : Pour demander une modification.

#### Exemple Complet : Accordéon Exclusif

```tsx
import { useState } from 'react';

// 1. L'enfant est "Contrôlé" : il ne décide rien, il affiche ce qu'on lui dit.
interface PanelProps {
  title: string;
  children: React.ReactNode;
  isActive: boolean; // Reçoit l'état
  onShow: () => void; // Reçoit la commande d'activation
}

function Panel({ title, children, isActive, onShow }: PanelProps) {
  return (
    <section style={{ border: '1px solid #ccc', padding: 10, margin: 5 }}>
      <h3>{title}</h3>
      {isActive ? (
        <p>{children}</p>
      ) : (
        <button onClick={onShow}>Afficher</button>
      )}
    </section>
  );
}

// 2. Le Parent détient la Source de Vérité
export function Accordion() {
  // Au lieu de plusieurs booléens, on stocke l'ID du panneau actif
  const [activeIndex, setActiveIndex] = useState(0);

  return (
    <>
      <Panel 
        title="À propos" 
        isActive={activeIndex === 0} 
        onShow={() => setActiveIndex(0)} // Le parent décide qui s'active
      >
        Nous sommes une super entreprise.
      </Panel>
      
      <Panel 
        title="Contact" 
        isActive={activeIndex === 1} 
        onShow={() => setActiveIndex(1)}
      >
        Envoyez-nous un email.
      </Panel>
    </>
  );
}
```

---

## Flux de Données Unidirectionnel (One-Way Data Flow) {#flux-de-donnees-unidirectionnel}

### 1. Quoi
En React, les données "coulent" vers le bas (Down), et les événements "remontent" vers le haut (Up).

*   **Parent ➔ Enfant** : Props (Données).
*   **Enfant ➔ Parent** : Events (Appels de fonction).

### 2. Pourquoi
Cela rend le débogage beaucoup plus simple. Si une donnée est fausse à l'écran, vous savez qu'il faut remonter l'arbre des composants pour trouver quel parent a envoyé la mauvaise prop ou détient le mauvais état.

### 3. Zone de Danger : Prop Drilling

:::warning Attention à la profondeur
Si vous devez remonter l'état très haut (par exemple jusqu'à `App.js`) et le faire redescendre à travers 5 niveaux de composants qui ne s'en servent pas, on appelle cela le **Prop Drilling**.
Si cela devient ingérable, la solution sera d'utiliser le **Context API** (voir Chapitre 30), mais ne vous précipitez pas dessus. Le "Lifting State Up" suffit dans 90% des cas.
:::

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-19}

1.  **Qu'est-ce que "Remonter l'État" (Lifting State Up) ?**
    C'est le processus de déplacer une variable d'état d'un composant enfant vers un composant parent commun pour partager cette donnée entre plusieurs enfants.

2.  **Qu'est-ce qu'un composant "contrôlé" ?**
    C'est un composant qui ne possède pas son propre état local pour ses données principales, mais qui les reçoit entièrement via ses `props` (et notifie les changements via des callbacks).

3.  **Dans quelle direction circulent les données en React ?**
    Uniquement vers le bas (du parent vers l'enfant via les props). Les enfants communiquent avec les parents en exécutant des fonctions passées en props.

---

## Exercices : {#exercices-19}

### Exercice 1 - Le Miroir de Texte {#exercice-1---le-miroir-de-texte}

🎯 **Objectif** : Synchroniser deux inputs simples.

💼 **Mise en situation** : Vous créez une interface d'édition où l'utilisateur peut taper un titre en haut de page ou en bas de page, et les deux doivent rester identiques en temps réel.

📝 **Énoncé** :
1. Créez un composant `SyncedInput` qui prend `value` (string) et `onChange` (function) en props.
2. Dans le composant parent `PageEditor`, créez un état `text`.
3. Affichez deux instances de `SyncedInput` qui pointent vers le même état.
4. Vérifiez que taper dans l'un met à jour l'autre instantanément.

📺 **Résultat attendu** :
Deux champs de texte. Quand je tape "Hello" dans le premier, le deuxième affiche "Hello" en même temps.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState } from 'react';

// Composant Enfant (Contrôlé)
// Il ne sait pas ce qu'il contient, il ne fait qu'afficher 'value' et signaler les changements.
interface SyncedInputProps {
  label: string;
  value: string;
  onChange: (newText: string) => void;
}

function SyncedInput({ label, value, onChange }: SyncedInputProps) {
  return (
    <div style={{ margin: 10 }}>
      <label>
        {label} : 
        <input 
          value={value} 
          onChange={(e) => onChange(e.target.value)} 
          style={{ marginLeft: 5 }}
        />
      </label>
    </div>
  );
}

// Composant Parent
export function PageEditor() {
  // L'état vit ici
  const [text, setText] = useState('');

  return (
    <div>
      <h3>Éditeur Synchronisé</h3>
      {/* Les deux enfants reçoivent la MÊME valeur et le MÊME setter */}
      <SyncedInput 
        label="Input A" 
        value={text} 
        onChange={setText} 
      />
      <SyncedInput 
        label="Input B (Miroir)" 
        value={text} 
        onChange={setText} 
      />
    </div>
  );
}
```
</details>

### Exercice 2 - La Liste Filtrable {#exercice-2---la-liste-filtrable}

🎯 **Objectif** : Communication entre Frères (Searchbar -> List).

💼 **Mise en situation** : Un tableau de bord RH. Vous avez une barre de recherche en haut, et la liste des employés en dessous. La barre de recherche ne doit pas filtrer elle-même, elle doit juste dire au parent "l'utilisateur cherche X". Le parent filtrera la liste envoyée au composant d'affichage.

📝 **Énoncé** :
1. Données : `const employees = ["Alice", "Bob", "Charlie", "David"]`.
2. Composant `SearchBar` : contient un input. Il ne stocke pas la recherche, il reçoit `query` et `onQueryChange` du parent.
3. Composant `EmployeeList` : reçoit un tableau de strings (déjà filtré) et les affiche.
4. Composant `Dashboard` (Parent) : détient l'état `query`. Il calcule la liste filtrée et la passe à `EmployeeList`.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Input avec le texte "ali" et liste dessous affichant seulement "Alice".
> **Annotation** : Montrez que le filtrage est dynamique.
> **Alt Text suggéré** : Interface de recherche filtrant une liste de noms en temps réel.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState } from 'react';

// Enfant 1 : La barre de recherche
function SearchBar({ query, onQueryChange }: { query: string, onQueryChange: (v: string) => void }) {
  return (
    <div style={{ marginBottom: 20 }}>
      Rechercher : 
      <input 
        value={query} 
        onChange={(e) => onQueryChange(e.target.value)} 
        placeholder="Nom..."
      />
    </div>
  );
}

// Enfant 2 : La liste d'affichage (bête et méchante, elle affiche ce qu'on lui donne)
function EmployeeList({ items }: { items: string[] }) {
  if (items.length === 0) return <p>Aucun résultat.</p>;
  
  return (
    <ul>
      {items.map(item => <li key={item}>{item}</li>)}
    </ul>
  );
}

// Parent : Le cerveau
export function HRDashboard() {
  const [query, setQuery] = useState('');
  
  const allEmployees = ["Alice", "Bob", "Charlie", "David", "Eve", "Frank"];

  // Logique de filtrage (effectuée à chaque rendu du parent)
  // On passe le résultat filtré à l'enfant, pas besoin de state pour la liste filtrée
  const filteredEmployees = allEmployees.filter(employee => 
    employee.toLowerCase().includes(query.toLowerCase())
  );

  return (
    <div style={{ border: '1px solid #ddd', padding: 20, maxWidth: 300 }}>
      <h2>Annuaire</h2>
      {/* On passe l'état et le setter à la SearchBar */}
      <SearchBar query={query} onQueryChange={setQuery} />
      
      {/* On passe la donnée calculée (Derived State) à la List */}
      <EmployeeList items={filteredEmployees} />
    </div>
  );
}
```
</details>

### Exercice 3 - Le Selecteur de Prix (Master-Detail) {#exercice-3---le-selecteur-de-prix}

🎯 **Objectif** : Sélection exclusive (Radio-button logic avec composants personnalisés).

💼 **Mise en situation** : Page de Pricing d'un SaaS. Trois cartes "Basic", "Pro", "Enterprise". Cliquer sur une carte la met en surbrillance (bordure épaisse bleue) et affiche son prix en bas de page.

📝 **Énoncé** :
1. Créez un composant `PricingCard` qui prend `title`, `price`, `isSelected` (bool), et `onSelect` (func).
2. Si `isSelected` est true, la div a un style `border: 2px solid blue` et `backgroundColor: #eef`. Sinon bordure grise.
3. Le Parent contient un tableau d'offres. Il gère l'état `selectedId`.
4. En bas du parent, affichez : "Prix mensuel sélectionné : X €".

📺 **Résultat attendu** :
3 cartes côte à côte. Impossible d'en sélectionner deux. Le total en bas change au clic.

<details>
<summary>💡 Voir le code complet commenté</summary>

```tsx
import { useState } from 'react';

interface Plan {
  id: string;
  title: string;
  price: number;
}

const PLANS: Plan[] = [
  { id: 'basic', title: 'Basic', price: 10 },
  { id: 'pro', title: 'Pro', price: 29 },
  { id: 'ent', title: 'Enterprise', price: 99 },
];

function PricingCard({ 
  plan, 
  isSelected, 
  onSelect 
}: { 
  plan: Plan; 
  isSelected: boolean; 
  onSelect: () => void; 
}) {
  return (
    <div 
      onClick={onSelect}
      style={{
        border: isSelected ? '2px solid blue' : '1px solid #ccc',
        backgroundColor: isSelected ? '#f0f8ff' : 'white',
        padding: 20,
        cursor: 'pointer',
        borderRadius: 8,
        width: 100,
        textAlign: 'center'
      }}
    >
      <h4>{plan.title}</h4>
      <p>{plan.price} €</p>
    </div>
  );
}

export function PricingPage() {
  const [selectedPlanId, setSelectedPlanId] = useState<string>('pro'); // 'pro' par défaut

  // On trouve le plan complet basé sur l'ID sélectionné pour afficher le total
  const selectedPlan = PLANS.find(p => p.id === selectedPlanId);

  return (
    <div>
      <div style={{ display: 'flex', gap: 10, marginBottom: 20 }}>
        {PLANS.map(plan => (
          <PricingCard
            key={plan.id}
            plan={plan}
            // La magie est ici : on compare l'ID du plan avec l'ID stocké dans l'état
            isSelected={selectedPlanId === plan.id}
            // Au clic, on demande au parent de changer l'ID sélectionné
            onSelect={() => setSelectedPlanId(plan.id)}
          />
        ))}
      </div>
      
      <div style={{ fontSize: '1.2em', fontWeight: 'bold' }}>
        Total mensuel : {selectedPlan?.price} €
      </div>
    </div>
  );
}
```
</details>
```