---
sidebar_label: Programmation Orientée Objet : Encapsulation et Propriétés
sidebar_position: 20
---

# Chapitre 20 : Programmation Orientée Objet : Encapsulation et Propriétés

Attributs privés (convention), Getters et setters, Décorateur @property, Attributs de classe

Imaginez un smartphone. Vous pouvez appuyer sur des boutons, toucher l'écran, le charger. Mais pouvez-vous directement modifier le voltage du processeur ou réécrire un secteur de la mémoire RAM en appuyant sur l'écran ? Heureusement, non. Ces détails internes sont **encapsulés**.

En Programmation Orientée Objet, l'encapsulation est le principe de cacher les détails d'implémentation et de contrôler l'accès aux données. Python a une approche unique : il fait confiance aux développeurs ("We are all consenting adults here") plutôt que d'imposer des barrières strictes comme Java ou C++.

---

## 1. Encapsulation et Conventions de Nommage {#encapsulation-et-conventions}

### 1. Quoi
En Python, il n'existe pas de mot-clé `private` ou `protected` qui rend une variable techniquement inaccessible. Tout est public par défaut. L'encapsulation repose sur des **conventions de nommage** strictes respectées par la communauté.

*   **Public** (`name`) : Accessible de partout.
*   **Protégé** (`_name`) : Usage interne à la classe et ses enfants (convention).
*   **Privé** (`__name`) : Usage strictement interne à la classe (mécanisme de *Name Mangling*).

### 2. Pourquoi
Pour signaler aux autres développeurs : "Attention, cette variable est gérée en interne, ne la modifiez pas directement sinon vous risquez de casser l'objet."

### 3. Comment

#### A. Syntaxe de base

```python
class BankAccount:
    def __init__(self, owner: str, balance: float):
        self.owner = owner       # Public : tout le monde peut lire/écrire
        self._currency = "EUR"   # Protected : usage interne suggéré
        self.__pin = "1234"      # Private : accès difficile depuis l'extérieur

account = BankAccount("Alice", 1000)

print(account.owner)      # Alice
print(account._currency)  # EUR (Possible, mais déconseillé par convention)

# print(account.__pin)    # ❌ AttributeError: 'BankAccount' object has no attribute '__pin'
```

#### B. Le Name Mangling (Pour le "Privé")
Python renomme secrètement les attributs commençant par `__` en `_ClassName__AttributeName`. C'est pour éviter les conflits de noms dans l'héritage, pas pour la sécurité pure.

```python
# Accès "forceur" (à éviter absolument en production)
print(account._BankAccount__pin) # 1234
```

### 4. Zone de Danger
❌ **Abus de `__double_underscore`** :
N'utilisez pas `__` partout pour rendre vos variables "privées" comme en Java. Cela rend le code difficile à tester et à déboguer.
✅ **Préférez `_single_underscore`** :
C'est la convention standard Python pour dire "touche pas à ça". Les IDE et linters vous avertiront si vous y touchez de l'extérieur.

---

## 2. Getters et Setters (Accesseurs et Mutateurs) {#getters-et-setters}

### 1. Quoi
Des méthodes dédiées pour lire (get) ou modifier (set) la valeur d'un attribut privé/protégé.

### 2. Pourquoi
*   **Validation** : Vérifier une donnée avant de l'assigner (ex: âge négatif interdit).
*   **Transformation** : Formater une donnée avant de la renvoyer.
*   **Abstraction** : Changer la manière dont la donnée est stockée sans changer le code qui l'utilise.

### 3. Comment

#### A. Approche Classique (Style Java - Déconseillé en Python moderne)

```python
class User:
    def __init__(self, age: int):
        self._age = age

    def get_age(self) -> int:
        return self._age

    def set_age(self, new_age: int):
        if new_age < 0:
            raise ValueError("L'âge ne peut pas être négatif")
        self._age = new_age

u = User(25)
u.set_age(30)
print(u.get_age())
```

### 🚨 Limitations
Cette syntaxe est verbeuse et peu "Pythonique". Si vous commencez avec un attribut public `u.age = 25` et que vous voulez ajouter une validation plus tard, vous devez casser tout le code existant pour le remplacer par `u.set_age(25)`. C'est là qu'intervient `@property`.

---

## 3. Le Décorateur `@property` {#decorateur-property}

### 1. Quoi
La manière **Pythonique** de faire de l'encapsulation. `@property` permet d'accéder à une méthode comme si c'était un attribut. Cela permet de transformer un attribut public en attribut géré par des getters/setters sans changer la syntaxe d'appel.

### 2. Pourquoi
Pour garder une API propre (`objet.valeur`) tout en ayant la puissance de la logique de validation (`objet.set_valeur()`) en arrière-plan.

### 3. Comment

#### A. Syntaxe complète

```python
class Temperature:
    def __init__(self, celsius: float):
        self._celsius = celsius # Attribut protégé interne

    # Getter
    @property
    def celsius(self) -> float:
        print("Lecture de la température...")
        return self._celsius

    # Setter
    @celsius.setter
    def celsius(self, value: float):
        print("Modification de la température...")
        if value < -273.15:
            raise ValueError("Impossible de descendre sous le zéro absolu !")
        self._celsius = value

t = Temperature(20)

# Utilisation comme une variable normale !
t.celsius = 25  # Appelle automatiquement le Setter
print(t.celsius) # Appelle automatiquement le Getter
# Sortie:
# Modification de la température...
# Lecture de la température...
# 25
```

#### B. Propriétés Calculées (Read-Only)
Vous pouvez définir une propriété sans setter. Elle devient alors accessible en lecture seule et peut être calculée à la volée.

```python
class Rectangle:
    def __init__(self, width: float, height: float):
        self.width = width
        self.height = height

    @property
    def area(self) -> float:
        return self.width * self.height

r = Rectangle(10, 5)
print(r.area) # 50
# r.area = 60 # ❌ AttributeError: can't set attribute
```

---

## 4. Attributs de Classe vs Instance {#attributs-classe-instance}

### 1. Quoi
*   **Attribut d'Instance (`self.x`)** : Appartient à un objet spécifique.
*   **Attribut de Classe (défini hors méthode)** : Partagé par **toutes** les instances de la classe.

### 2. Pourquoi
Pour définir des constantes, des compteurs globaux ou des configurations par défaut communes à tous les objets.

### 3. Comment

```python
class Employee:
    # Attribut de classe
    minimum_wage = 1500 
    company_name = "TechCorp"

    def __init__(self, name: str, salary: float):
        self.name = name
        # Attribut d'instance
        self.salary = max(salary, Employee.minimum_wage)

e1 = Employee("Alice", 1400) # Sera forcé à 1500
e2 = Employee("Bob", 2000)

print(e1.salary) # 1500
print(e2.company_name) # TechCorp

# Modification globale
Employee.minimum_wage = 1600 # Impacte les futures créations ou accès via classe
```

### 4. Zone de Danger
❌ **Modifier un attribut de classe via une instance** :
Cela crée un nouvel attribut d'instance qui "masque" l'attribut de classe pour cet objet seulement.

```python
e1.company_name = "NewCorp" # Crée e1.company_name, ne change pas Employee.company_name
print(e2.company_name) # Affiche toujours "TechCorp"
```

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-20}

1.  **Quelle est la différence entre `_variable` et `__variable` ?**
    `_variable` est une convention (protected) signalant un usage interne. `__variable` déclenche le *name mangling* (renommage) pour éviter les conflits, rendant l'accès direct plus difficile (simili-privé).

2.  **Pourquoi utiliser `@property` plutôt que `get_x()` et `set_x()` ?**
    Pour conserver une syntaxe d'accès aux attributs naturelle (`obj.x`) tout en permettant d'ajouter de la logique (validation, calcul) de manière transparente.

3.  **Peut-on modifier un attribut "privé" en Python depuis l'extérieur de la classe ?**
    Oui. Rien n'est techniquement verrouillé en Python. On peut accéder à `_var` directement ou `_Class__var` pour les doubles underscores. C'est cependant une très mauvaise pratique.

4.  **Comment créer un attribut en lecture seule (read-only) ?**
    En définissant une méthode avec `@property` mais en ne définissant **pas** de méthode correspondante avec `@nom.setter`.

---

## Exercices : {#exercices-20}

### Exercice 1 - Le Compte Bancaire Sécurisé {#exercice-1---compte-bancaire}

🎯 **Objectif** : Utiliser `@property` pour valider des données.

💼 **Mise en situation** : Dans une néo-banque, on ne peut pas avoir un solde négatif sans autorisation, et on ne peut pas changer le titulaire du compte une fois créé.

📝 **Énoncé** :
1.  Créez une classe `Account`.
2.  `__init__` prend `owner` et `balance`.
3.  L'attribut `owner` doit être accessible en lecture seule (pas de setter).
4.  L'attribut `balance` doit être accessible en lecture et écriture via `@property`.
5.  Le setter de `balance` doit lever une `ValueError` si le nouveau solde est négatif.
6.  Testez les accès et les erreurs.

📺 **Résultat attendu** :
```text
Compte de Alice créé avec 100€
Nouveau solde : 150€
Erreur : Solde négatif interdit
Erreur : Impossible de modifier owner
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
class Account:
    def __init__(self, owner: str, balance: float):
        self._owner = owner
        self._balance = balance

    # Getter pour owner (Read-Only car pas de setter)
    @property
    def owner(self) -> str:
        return self._owner

    # Getter pour balance
    @property
    def balance(self) -> float:
        return self._balance

    # Setter pour balance avec validation
    @balance.setter
    def balance(self, value: float):
        if value < 0:
            raise ValueError("Solde négatif interdit")
        self._balance = value

# Tests
acc = Account("Alice", 100)
print(f"Compte de {acc.owner} créé avec {acc.balance}€")

acc.balance = 150
print(f"Nouveau solde : {acc.balance}€")

try:
    acc.balance = -50
except ValueError as e:
    print(f"Erreur : {e}")

try:
    acc.owner = "Bob"
except AttributeError:
    print("Erreur : Impossible de modifier owner")
```
</details>

### Exercice 2 - Le Produit E-commerce (Propriété Calculée) {#exercice-2---produit-calcule}

🎯 **Objectif** : Créer des propriétés dynamiques dépendantes d'autres attributs.

💼 **Mise en situation** : Un produit a un prix HT et un taux de TVA. Le prix TTC n'a pas besoin d'être stocké, il peut être calculé.

📝 **Énoncé** :
1.  Créez une classe `Product`.
2.  Attributs : `name`, `price_ht` (hors taxe), `vat_rate` (taux TVA, ex: 0.20 pour 20%).
3.  Propriété `price_ttc` (Toutes Taxes Comprises) qui retourne le prix calculé.
4.  Propriété `price_ttc` avec un **setter** : si on définit le prix TTC, cela recalcule et met à jour le `price_ht` automatiquement (formule : `ht = ttc / (1 + vat)`).

📺 **Résultat attendu** :
```text
Laptop: HT=1000€, TTC=1200.0€
Modification du TTC à 600€...
Nouveau HT=500.0€
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
class Product:
    def __init__(self, name: str, price_ht: float, vat_rate: float = 0.20):
        self.name = name
        self.price_ht = price_ht
        self.vat_rate = vat_rate

    @property
    def price_ttc(self) -> float:
        # Calcul à la volée
        return self.price_ht * (1 + self.vat_rate)

    @price_ttc.setter
    def price_ttc(self, ttc_value: float):
        # Logique inverse : on déduit le HT à partir du TTC souhaité
        self.price_ht = ttc_value / (1 + self.vat_rate)

p = Product("Laptop", 1000)
print(f"{p.name}: HT={p.price_ht}€, TTC={p.price_ttc}€")

print("Modification du TTC à 600€...")
p.price_ttc = 600
print(f"Nouveau HT={p.price_ht}€")
```
</details>

### Exercice 3 - Le Système de Configuration (Singleton partagé) {#exercice-3---systeme-config}

🎯 **Objectif** : Manipuler les attributs de classe pour partager des données.

💼 **Mise en situation** : Une application possède un "Mode Debug" qui, s'il est activé, affecte le comportement de tous les modules (objets) de l'application.

📝 **Énoncé** :
1.  Créez une classe `Logger`.
2.  Attribut de classe `debug_mode = False`.
3.  Méthode d'instance `log(message)` :
    - Affiche le message seulement si `Logger.debug_mode` est True.
    - Sinon, ne fait rien.
4.  Créez deux loggers distincts. Tentez de logger (rien ne se passe).
5.  Activez le debug mode au niveau de la classe (`Logger.debug_mode = True`).
6.  Réessayez : les deux loggers doivent maintenant afficher les messages.

📺 **Résultat attendu** :
```text
(Rien ne s'affiche)
--- Activation Debug ---
[LOG] Connexion DB
[LOG] Envoi email
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
class Logger:
    # Attribut partagé par toutes les instances
    debug_mode = False

    def log(self, message: str):
        # On vérifie la configuration globale de la classe
        if Logger.debug_mode:
            print(f"[LOG] {message}")

l1 = Logger()
l2 = Logger()

# Par défaut, debug est False
l1.log("Connexion DB")
l2.log("Envoi email")

print("--- Activation Debug ---")
# On change la config pour tout le monde
Logger.debug_mode = True

# Les instances existantes voient le changement
l1.log("Connexion DB")
l2.log("Envoi email")
```
</details>