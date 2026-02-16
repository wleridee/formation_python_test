---
sidebar_label: Module `dataclasses` : Classes de Données Simplifiées
sidebar_position: 35
---

# Chapitre 35 : Module `dataclasses` : Classes de Données Simplifiées

Décorateur @dataclass, Champs par défaut, Méthodes générées, Immutabilité

En Python, nous passons une grande partie de notre temps à écrire des classes dont le seul but est de stocker des données (comme une structure C ou un DTO en Java).

Avant Python 3.7, cela impliquait d'écrire manuellement une méthode `__init__` fastidieuse, une méthode `__repr__` pour le débogage, et `__eq__` pour comparer les objets. Le module `dataclasses` automatise toute cette "plomberie". Il génère du code robuste à votre place, rendant vos classes plus lisibles et moins sujettes aux erreurs.

---

## 1. Le Décorateur `@dataclass` : La Fin du Code Répétitif {#le-decorateur-dataclass}

### 1. Quoi
Le décorateur `@dataclass` analyse les annotations de type de votre classe et génère automatiquement les méthodes spéciales "magiques" (`__init__`, `__repr__`, `__eq__`, etc.).

### 2. Pourquoi
Pour éliminer le code "boilerplate" (code répétitif sans valeur ajoutée). Cela permet de se concentrer sur la définition des données plutôt que sur la mécanique de la classe.

### 3. Comment

#### A. Comparaison Avant / Après

**❌ Avant (Style "Old School") :**
```python
class Product:
    def __init__(self, name: str, price: float, stock: int):
        self.name = name
        self.price = price
        self.stock = stock

    def __repr__(self):
        return f"Product(name='{self.name}', price={self.price}, stock={self.stock})"

    def __eq__(self, other):
        if not isinstance(other, Product):
            return NotImplemented
        return (self.name, self.price, self.stock) == (other.name, other.price, other.stock)
```

**✅ Avec Dataclasses :**
```python
from dataclasses import dataclass

@dataclass
class Product:
    name: str
    price: float
    stock: int

# C'est tout. __init__, __repr__ et __eq__ sont générés automatiquement.
```

#### B. Utilisation concrète

```python
from dataclasses import dataclass

@dataclass
class User:
    id: int
    username: str
    email: str

# 1. __init__ automatique
u1 = User(1, "alice", "alice@corp.com")
u2 = User(1, "alice", "alice@corp.com")
u3 = User(2, "bob", "bob@corp.com")

# 2. __repr__ automatique (très utile pour les logs)
print(u1) 
# Sortie : User(id=1, username='alice', email='alice@corp.com')

# 3. __eq__ automatique (comparaison par valeur, pas par adresse mémoire)
print(f"u1 est égal à u2 ? {u1 == u2}") # True
print(f"u1 est égal à u3 ? {u1 == u3}") # False
```

### 4. Zone de Danger
❌ **Typage requis** : Les dataclasses **dépendent** des annotations de type. Si vous écrivez `name = "Bob"` sans le type (`name: str = "Bob"`), le champ sera considéré comme un attribut de classe classique et ignoré par dataclass.

---

## 2. Valeurs par Défaut et `field` {#valeurs-par-defaut-et-field}

### 1. Quoi
On peut définir des valeurs par défaut simples directement (`x: int = 0`). Mais pour les types mutables (listes, dictionnaires) ou des configurations complexes, il faut utiliser la fonction `field()`.

### 2. Pourquoi
En Python, utiliser une liste vide `[]` comme valeur par défaut dans une fonction ou une classe est un piège classique (la liste est partagée entre toutes les instances). `dataclasses` interdit cela et oblige à utiliser une "factory" pour créer une nouvelle liste pour chaque instance.

### 3. Comment

#### A. Syntaxe de base et `default_factory`

```python
from dataclasses import dataclass, field
from typing import list

@dataclass
class Team:
    name: str
    # ❌ Interdit : members: list[str] = []  <- Le module lèvera une ValueError
    
    # ✅ Correct : Création d'une nouvelle liste par instance
    members: list[str] = field(default_factory=list)
    
    # Valeur par défaut simple
    score: int = 0
    
    # Champ exclu de __repr__ (ex: mot de passe ou donnée binaire lourde)
    metadata: dict = field(default_factory=dict, repr=False)

t1 = Team("Alpha")
t1.members.append("Joueur 1")

t2 = Team("Beta")
# t2.members est vide, pas d'effet de bord partagé avec t1
print(t2.members) # []
```

---

## 3. Immutabilité avec `frozen=True` {#immutabilite-frozen}

### 1. Quoi
L'argument `frozen=True` rend les instances de la dataclass **immuables** (en lecture seule). Une fois créé, l'objet ne peut plus être modifié.

### 2. Pourquoi
*   **Sécurité** : Garantit que les données ne sont pas altérées accidentellement.
*   **Hachage** : Les objets immuables peuvent être hachés (`__hash__`), ce qui permet de les utiliser comme **clés de dictionnaire** ou éléments d'un **set**.

### 3. Comment

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class Config:
    host: str
    port: int

conf = Config("localhost", 8080)

# Lecture OK
print(conf.host)

# Écriture INTERDITE
try:
    conf.port = 9000 # Lève une exception FrozenInstanceError
except Exception as e:
    print(f"Erreur : {e}")

# Utilisation dans un set (impossible sans frozen=True)
configs = {conf} 
print("Ajout dans un set réussi !")
```

---

## 4. Initialisation Avancée : `__post_init__` {#initialisation-avancee-post-init}

### 1. Quoi
`__post_init__` est une méthode spéciale appelée automatiquement par la dataclass *juste après* le `__init__` généré.

### 2. Pourquoi
Puisque vous n'écrivez pas le `__init__` vous-même, vous ne pouvez pas y placer de logique (validation, calcul de champs dérivés). `__post_init__` sert à ça.

### 3. Comment

```python
from dataclasses import dataclass, field

@dataclass
class OrderLine:
    unit_price: float
    quantity: int
    # init=False signifie que ce champ n'est pas passé en argument lors de la création
    total_price: float = field(init=False)

    def __post_init__(self):
        # Calcul automatique après l'initialisation
        self.total_price = self.unit_price * self.quantity
        
        # Validation
        if self.quantity <= 0:
            raise ValueError("La quantité doit être positive")

line = OrderLine(10.5, 2)
print(f"Total : {line.total_price}€") # Total : 21.0€
```

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-35}

1.  **Quelles sont les 3 principales méthodes générées automatiquement par `@dataclass` ?**
    `__init__` (constructeur), `__repr__` (affichage lisible), et `__eq__` (comparaison d'égalité).

2.  **Pourquoi ne peut-on pas écrire `tags: list = []` dans une dataclass ?**
    Car `[]` est un objet mutable qui serait partagé par toutes les instances (effet de bord dangereux). Dataclasses lève une `ValueError` pour empêcher cela. Il faut utiliser `field(default_factory=list)`.

3.  **Comment rendre une dataclass utilisable comme clé de dictionnaire ?**
    En ajoutant le paramètre `@dataclass(frozen=True)`. Cela rend l'objet immuable et génère une méthode `__hash__`.

4.  **À quoi sert `field(init=False)` ?**
    À déclarer un champ qui existe dans l'objet mais qui ne doit pas être demandé comme argument dans le constructeur (souvent calculé dans `__post_init__`).

---

## Exercices : {#exercices-35}

### Exercice 1 - Système d'Inventaire RPG {#exercice-1-inventaire-rpg}

🎯 **Objectif** : Créer une dataclass simple avec comparaison.

💼 **Mise en situation** : Vous codez un jeu vidéo. Vous devez gérer des items (épées, potions) qui ont un nom, un poids et une valeur.

📝 **Énoncé** :
1.  Créez une dataclass `Item` avec : `name` (str), `weight` (float), `value` (int).
2.  Créez deux instances représentant une "Potion de Soin" (0.5kg, 10 pièces d'or).
3.  Vérifiez que `potion1 == potion2` renvoie `True`.
4.  Affichez `potion1` pour voir le formatage automatique.

📺 **Résultat attendu** :
```text
Items identiques : True
Item(name='Potion de Soin', weight=0.5, value=10)
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
from dataclasses import dataclass

@dataclass
class Item:
    name: str
    weight: float
    value: int

# Création des instances
potion1 = Item("Potion de Soin", 0.5, 10)
potion2 = Item("Potion de Soin", 0.5, 10)
epee = Item("Épée", 5.0, 100)

# Test d'égalité
print(f"Items identiques : {potion1 == potion2}")

# Test d'affichage (__repr__)
print(potion1)
```
</details>

### Exercice 2 - Configuration Serveur Sécurisée {#exercice-2-config-frozen}

🎯 **Objectif** : Utiliser `frozen` et des valeurs par défaut.

💼 **Mise en situation** : Vous développez un module réseau. La configuration du serveur (IP, Port, Protocol) ne doit jamais changer une fois chargée.

📝 **Énoncé** :
1.  Créez une dataclass `ServerConfig` avec `frozen=True`.
2.  Champs : 
    - `host` (str, défaut "127.0.0.1")
    - `port` (int, défaut 80)
    - `protocol` (str, défaut "http")
3.  Instanciez une config par défaut.
4.  Essayez de changer le port de cette instance dans un bloc `try/except` et affichez l'erreur capturée.

📺 **Résultat attendu** :
```text
Config chargée : ServerConfig(host='127.0.0.1', port=80, protocol='http')
Tentative de modification...
Erreur : cannot assign to field 'port'
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class ServerConfig:
    host: str = "127.0.0.1"
    port: int = 80
    protocol: str = "http"

config = ServerConfig()
print(f"Config chargée : {config}")

print("Tentative de modification...")
try:
    config.port = 443 # Interdit !
except Exception as e:
    print(f"Erreur : {e}")
```
</details>

### Exercice 3 - Facture E-commerce (Post-init) {#exercice-3-facture-post-init}

🎯 **Objectif** : Utiliser `field(default_factory)` et `__post_init__`.

💼 **Mise en situation** : Vous gérez des factures. Une facture a un ID, une liste de montants (prix des articles), et doit calculer automatiquement le total TTC (Taxe 20%).

📝 **Énoncé** :
1.  Créez une dataclass `Invoice`.
2.  Champs :
    - `id` (str)
    - `amounts` (liste de float, par défaut liste vide).
    - `total_ttc` (float, non passé à l'init).
3.  Utilisez `__post_init__` pour calculer `total_ttc` comme la somme des `amounts` * 1.20.
4.  Créez une facture avec les montants `[100.0, 50.0]`.

📺 **Résultat attendu** :
```text
Facture F-2023 :
Montants HT : [100.0, 50.0]
Total TTC (calculé) : 180.0
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
from dataclasses import dataclass, field

@dataclass
class Invoice:
    id: str
    # Utilisation de default_factory pour la liste mutable
    amounts: list[float] = field(default_factory=list)
    # init=False car on le calcule, on ne le renseigne pas manuellement
    total_ttc: float = field(init=False)

    def __post_init__(self):
        # Somme HT
        total_ht = sum(self.amounts)
        # Application TVA 20%
        self.total_ttc = total_ht * 1.20

# Test
facture = Invoice("F-2023", [100.0, 50.0])

print(f"Facture {facture.id} :")
print(f"Montants HT : {facture.amounts}")
print(f"Total TTC (calculé) : {facture.total_ttc}")
```
</details>