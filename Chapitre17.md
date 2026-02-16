---
sidebar_label: Programmation Orientée Objet : Classes et Objets
sidebar_position: 17
---

# Chapitre 17 : Programmation Orientée Objet : Classes et Objets

Concept de classe, Création d'objets, Attributs, Méthodes

Jusqu'à présent, nous avons écrit du code procédural : des données d'un côté, et des fonctions pour les manipuler de l'autre. Imaginez maintenant un jeu vidéo. Un "Ennemi" a des points de vie (donnée), une position (donnée), mais il peut aussi attaquer (action) ou mourir (action).

La **Programmation Orientée Objet (POO)** permet de regrouper ces données (attributs) et ces actions (méthodes) dans une seule entité cohérente : l'**Objet**. Ce paradigme est dominant dans l'industrie pour structurer des applications complexes.

---

## 1. Concept de Classe et d'Objet {#concept-classe-objet}

### 1. Quoi
*   **Classe (`class`)** : C'est le **plan de construction** (le moule). Elle définit à quoi *ressemble* un objet et ce qu'il peut *faire*. Exemple : Le plan d'architecte d'une maison.
*   **Objet (Instance)** : C'est la **réalisation concrète** de ce plan. On peut créer une infinité d'objets à partir d'une seule classe. Exemple : La maison construite au 12 rue des Lilas.

### 2. Pourquoi
*   **Organisation** : Regrouper logiquement tout ce qui concerne une entité métier (Utilisateur, Commande, Produit).
*   **Réutilisabilité** : Créer des centaines d'objets `User` sans réécrire le code de gestion des utilisateurs.

### 3. Comment

#### A. Syntaxe de base

```python
# Définition de la classe (Le Plan)
# Convention : CamelCase pour les noms de classes
class Robot:
    pass # Classe vide pour l'instant

# Création d'objets (L'Instanciation)
r1 = Robot()
r2 = Robot()

print(r1) # <__main__.Robot object at 0x...>
print(r1 == r2) # False (ce sont deux objets distincts en mémoire)
```

---

## 2. Attributs (Données) et `self` {#attributs-et-self}

### 1. Quoi
Les **attributs** sont les variables attachées à un objet. Ils représentent l'**état** de l'objet.
Le mot-clé **`self`** est crucial : il représente **l'objet actuel** en train d'être manipulé. C'est l'équivalent de "moi-même".

### 2. Pourquoi
Chaque objet doit avoir ses propres données. Si je change la couleur de la voiture A, la voiture B ne doit pas changer de couleur.

### 3. Comment

#### A. Initialisation avec `__init__`
C'est la méthode "constructeur" appelée automatiquement à la création de l'objet.

```python
class User:
    def __init__(self, name: str, email: str):
        # On attache les valeurs à l'objet "self"
        self.name = name
        self.email = email
        self.is_active = True # Valeur par défaut

# Création
alice = User("Alice", "alice@corp.com")
bob = User("Bob", "bob@corp.com")

# Accès aux attributs
print(alice.name) # Alice
print(bob.name)   # Bob

# Modification
alice.is_active = False
```

#### B. Attributs de classe vs d'instance
*   **Instance (`self.x`)** : Propre à chaque objet (ex: le nom de l'utilisateur).
*   **Classe (déclaré hors méthodes)** : Partagé par TOUS les objets (ex: le nom de la table en base de données).

```python
class Server:
    # Attribut de classe (partagé)
    platform = "Linux"

    def __init__(self, ip: str):
        # Attribut d'instance (unique)
        self.ip = ip

s1 = Server("10.0.0.1")
s2 = Server("10.0.0.2")

print(s1.platform) # Linux
print(s2.platform) # Linux
```

### 4. Zone de Danger
❌ **Oublier `self`** :
Dans une méthode, si vous écrivez `name = "Alice"` au lieu de `self.name = "Alice"`, vous créez une variable locale temporaire qui disparaîtra à la fin de la fonction, au lieu de modifier l'objet.

---

## 3. Méthodes (Comportements) {#methodes}

### 1. Quoi
Les **méthodes** sont simplement des fonctions définies à l'intérieur d'une classe. Elles définissent ce que l'objet peut **faire**. Leur premier paramètre est toujours `self`.

### 2. Pourquoi
Pour agir sur les données de l'objet (les attributs) sans avoir à les passer en argument à chaque fois.

### 3. Comment

#### A. Définition et Appel

```python
class BankAccount:
    def __init__(self, owner: str, balance: float):
        self.owner = owner
        self.balance = balance

    # Méthode d'instance
    def deposit(self, amount: float):
        if amount > 0:
            self.balance += amount
            print(f"Dépôt de {amount}€. Nouveau solde : {self.balance}€")

    def withdraw(self, amount: float) -> bool:
        if self.balance >= amount:
            self.balance -= amount
            return True
        print("Fonds insuffisants")
        return False

account = BankAccount("Alice", 100.0)
account.deposit(50.0) # self est passé automatiquement !
# Affiche : Dépôt de 50.0€. Nouveau solde : 150.0€
```

#### B. La méthode `__str__` (Représentation)
Par défaut, `print(obj)` affiche une adresse mémoire illisible. `__str__` permet de définir un affichage humain.

```python
class Product:
    def __init__(self, name: str, price: float):
        self.name = name
        self.price = price

    def __str__(self) -> str:
        return f"Produit: {self.name} ({self.price}€)"

p = Product("Laptop", 999.99)
print(p) # Produit: Laptop (999.99€)
```

### 🚨 Limitations
En Python, contrairement à Java ou C++, il n'y a pas de véritables méthodes ou attributs **privés** (inaccessibles de l'extérieur). Tout est public par défaut.
*   Convention : `_variable` (un seul underscore) signale "Ne touchez pas à ça, c'est interne".
*   Pour une protection plus forte, voir Chapitre 20 (Encapsulation).

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-17}

1.  **Quelle est la différence entre une classe et un objet ?**
    La classe est le plan (le type), l'objet est l'instance concrète créée à partir de ce plan.

2.  **À quoi sert le paramètre `self` ?**
    Il représente l'instance courante de l'objet sur laquelle on travaille. Il permet d'accéder aux attributs et autres méthodes de cet objet spécifique.

3.  **Quand la méthode `__init__` est-elle appelée ?**
    Automatiquement dès qu'on crée une nouvelle instance de la classe (ex: `MyClass()`).

4.  **Que se passe-t-il si on modifie un attribut de classe ?**
    Si on le modifie via la classe (`MyClass.attr = 2`), cela impacte toutes les instances qui n'ont pas surchargé cet attribut.

---

## Exercices : {#exercices-17}

### Exercice 1 - Le Gestionnaire de Tâches (Todo) {#exercice-1---gestionnaire-taches}

🎯 **Objectif** : Création de classe, attributs et méthodes de base.

💼 **Mise en situation** : Vous développez le backend d'une application de productivité. Vous devez représenter une "Tâche".

📝 **Énoncé** :
1.  Créez une classe `TodoItem`.
2.  Attributs : `title` (str), `is_done` (bool, par défaut False).
3.  Méthode `mark_as_done()` : passe `is_done` à True.
4.  Méthode `__str__` : retourne `[x] Titre` si fait, `[ ] Titre` sinon.
5.  Créez une tâche "Apprendre Python", affichez-la, marquez-la comme faite, ré-affichez.

📺 **Résultat attendu** :
```text
[ ] Apprendre Python
[x] Apprendre Python
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
class TodoItem:
    def __init__(self, title: str):
        self.title = title
        self.is_done = False # Par défaut, une tâche n'est pas finie

    def mark_as_done(self):
        # Modification de l'état de l'objet
        self.is_done = True

    def __str__(self) -> str:
        # Affichage conditionnel selon l'état
        status = "[x]" if self.is_done else "[ ]"
        return f"{status} {self.title}"

# Test
task = TodoItem("Apprendre Python")
print(task)

task.mark_as_done()
print(task)
```
</details>

### Exercice 2 - Le Panier E-commerce {#exercice-2---panier-ecommerce}

🎯 **Objectif** : Interaction entre objets et méthodes complexes.

💼 **Mise en situation** : Gestion d'un panier d'achat qui calcule son propre total.

📝 **Énoncé** :
1.  Créez une classe `ShoppingCart`.
2.  Attribut : `items` (liste de dictionnaires `{'name': str, 'price': float}`).
3.  Méthode `add_item(name, price)` : ajoute un article.
4.  Méthode `remove_item(name)` : retire le premier article correspondant.
5.  Méthode `total()` : retourne la somme des prix.
6.  Simulez un parcours client : ajout de 2 articles, retrait de 1, affichage du total.

📺 **Résultat attendu** :
```text
Ajout de Mouse (25€)
Ajout de Screen (150€)
Retrait de Mouse
Total panier : 150€
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
class ShoppingCart:
    def __init__(self):
        # Liste vide à la création
        self.items: list[dict] = []

    def add_item(self, name: str, price: float):
        item = {"name": name, "price": price}
        self.items.append(item)
        print(f"Ajout de {name} ({price}€)")

    def remove_item(self, name: str):
        # On cherche l'article à retirer
        for item in self.items:
            if item["name"] == name:
                self.items.remove(item)
                print(f"Retrait de {name}")
                return # On arrête après le premier retrait
        print(f"{name} non trouvé dans le panier")

    def total(self) -> float:
        # Somme des prix avec un générateur
        return sum(item["price"] for item in self.items)

# Simulation
cart = ShoppingCart()
cart.add_item("Mouse", 25)
cart.add_item("Screen", 150)
cart.remove_item("Mouse")
print(f"Total panier : {cart.total()}€")
```
</details>

### Exercice 3 - Le Système de RPG (Héros) {#exercice-3---systeme-rpg}

🎯 **Objectif** : Logique métier et gestion d'état (Points de vie).

💼 **Mise en situation** : Logique de combat simple pour un jeu.

📝 **Énoncé** :
1.  Classe `Hero` avec `name`, `hp` (vie, défaut 100), `attack_power` (défaut 10).
2.  Méthode `is_alive()` : retourne True si hp > 0.
3.  Méthode `take_damage(amount)` : réduit les hp (minimum 0).
4.  Méthode `attack(target)` : Inflige des dégâts à un autre objet `Hero` (appelle `target.take_damage`).
5.  Duel : Héros A attaque Héros B. Affichez les vies restantes.

📺 **Résultat attendu** :
```text
Arthur attaque Lancelot pour 10 dégâts !
Lancelot a 90 PV restants.
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
class Hero:
    def __init__(self, name: str, hp: int = 100, attack_power: int = 10):
        self.name = name
        self.hp = hp
        self.attack_power = attack_power

    def is_alive(self) -> bool:
        return self.hp > 0

    def take_damage(self, amount: int):
        self.hp -= amount
        if self.hp < 0:
            self.hp = 0

    def attack(self, target: 'Hero'):
        # On interagit avec une autre instance de la même classe
        print(f"{self.name} attaque {target.name} pour {self.attack_power} dégâts !")
        target.take_damage(self.attack_power)
        print(f"{target.name} a {target.hp} PV restants.")

# Duel
arthur = Hero("Arthur")
lancelot = Hero("Lancelot")

arthur.attack(lancelot)
```
</details>