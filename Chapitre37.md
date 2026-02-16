---
sidebar_label: Module `abc` : Classes Abstraites (Abstract Base Classes)
sidebar_position: 37
---

# Chapitre 37 : Module `abc` : Classes Abstraites (Abstract Base Classes)

Décorateur @abstractmethod, Classes abstraites, Mise en œuvre d'interfaces, Subclassing

En programmation orientée objet, il est crucial de définir des **contrats**. Imaginez une prise électrique murale : peu importe ce que vous branchez (lampe, ordinateur, aspirateur), l'appareil *doit* avoir une fiche compatible pour fonctionner.

En Python, le module `abc` (Abstract Base Classes) permet de définir ces "prises". Il permet de créer des classes qui ne peuvent pas être instanciées elles-mêmes (les plans) mais qui forcent leurs enfants à implémenter certaines méthodes (le contrat). C'est la base de l'architecture logicielle robuste et du polymorphisme.

---

## 1. Classes Abstraites et le Décorateur `@abstractmethod` {#classes-abstraites-et-abstractmethod}

### 1. Quoi
Une **classe abstraite** est une classe qui hérite de `ABC` (fournie par le module `abc`) et qui contient au moins une méthode décorée par `@abstractmethod`.
Ce décorateur signale à Python : "Cette méthode n'a pas d'implémentation concrète ici, mais toute classe fille **DOIT** fournir la sienne".

### 2. Pourquoi
Pour interdire la création d'objets incomplets. Si vous créez un jeu vidéo, instancier un "Monstre" générique n'a pas de sens. Vous voulez instancier un "Gobelin" ou un "Dragon". Mais tous deux *doivent* avoir une méthode `attaquer()`. `abc` transforme cette obligation de documentation en obligation technique.

### 3. Comment

#### A. Syntaxe de base

```python
from abc import ABC, abstractmethod

# 1. On hérite de ABC pour activer la mécanique
class Animal(ABC):
    
    @abstractmethod
    def cri(self) -> str:
        """Méthode que les enfants DOIVENT écraser."""
        pass

# ❌ Impossible d'instancier la classe abstraite
# a = Animal() -> TypeError: Can't instantiate abstract class Animal with abstract method cri

class Chien(Animal):
    def cri(self) -> str:
        return "Wouf !"

class Chat(Animal):
    # Si on oublie d'implémenter cri(), Chat sera aussi considérée abstraite !
    pass

# ✅ Instanciation OK
medor = Chien()
print(medor.cri()) 

# ❌ Felix = Chat() -> TypeError car cri() manque
```

#### B. Cas concret : Système de Notification

```python
from abc import ABC, abstractmethod
from typing import Any

class Notifier(ABC):
    """Contrat pour tout système d'envoi de message."""
    
    def __init__(self, user_config: dict[str, Any]):
        self.config = user_config

    @abstractmethod
    def send(self, message: str) -> bool:
        """Envoie un message. Doit retourner True si succès."""
        pass

class EmailNotifier(Notifier):
    def send(self, message: str) -> bool:
        print(f"📧 Email envoyé à {self.config.get('email')} : {message}")
        return True

class SMSNotifier(Notifier):
    def send(self, message: str) -> bool:
        print(f"📱 SMS envoyé au {self.config.get('phone')} : {message}")
        return True

# Utilisation polymorphique
services: list[Notifier] = [
    EmailNotifier({"email": "admin@corp.com"}),
    SMSNotifier({"phone": "+33612345678"})
]

for service in services:
    service.send("Alerte système !")
```

### 4. Zone de Danger
❌ **Ne pas appeler `super().__init__`** : Si votre classe abstraite a un constructeur `__init__` (pour stocker une config commune par exemple), n'oubliez pas de l'appeler dans les classes filles.

✅ **Code dans une méthode abstraite** : Une méthode abstraite *peut* avoir du code. On peut l'appeler via `super().methode()` pour partager une logique commune obligatoire.

---

## 2. Interfaces et Propriétés Abstraites {#interfaces-et-proprietes}

### 1. Quoi
Une "Interface" en Python est souvent une classe abstraite ne contenant *que* des méthodes abstraites (aucune logique métier).
On peut aussi forcer la présence d'attributs (propriétés) via `@property` combiné à `@abstractmethod`.

### 2. Pourquoi
Pour s'assurer qu'un objet possède certaines données avant de l'utiliser. Par exemple, forcer tout produit d'un e-commerce à avoir un `price` et un `sku`.

### 3. Comment

#### A. Propriétés abstraites

```python
from abc import ABC, abstractmethod

class Product(ABC):
    
    @property
    @abstractmethod
    def price(self) -> float:
        """Le prix est obligatoire."""
        pass
        
    @property
    @abstractmethod
    def name(self) -> str:
        pass

class PhysicalBook(Product):
    def __init__(self, title: str, cost: float):
        self._title = title
        self._cost = cost
        
    @property
    def name(self) -> str:
        return self._title
        
    @property
    def price(self) -> float:
        return self._cost * 1.20 # Avec TVA

# book = PhysicalBook("Python 101", 50.0) -> OK
```

### 🚨 Limitations de `abc`
Python est un langage à **typage dynamique**. Le module `abc` vérifie la conformité **au moment de l'instanciation**, pas à la définition de la classe, ni à la compilation.
Si vous définissez une classe `Chat(Animal)` sans implémenter `cri`, Python ne dira rien tant que vous n'essayez pas de faire `c = Chat()`.

---

## 3. Mixins et Héritage Multiple avec ABC {#mixins-et-heritage}

### 1. Quoi
On peut utiliser des ABC pour créer des **Mixins** : des classes destinées à ajouter une fonctionnalité spécifique à d'autres classes via l'héritage multiple.

### 2. Pourquoi
Pour composer des comportements. Par exemple, rendre un objet "JSONifiable" sans l'enfermer dans une hiérarchie stricte.

### 3. Comment

```python
import json
from abc import ABC, abstractmethod

class JsonSerializable(ABC):
    @abstractmethod
    def to_dict(self) -> dict:
        pass
        
    def to_json(self) -> str:
        """Méthode concrète offerte par le Mixin qui utilise le contrat abstrait."""
        return json.dumps(self.to_dict())

class User(JsonSerializable):
    def __init__(self, name: str, age: int):
        self.name = name
        self.age = age
        
    def to_dict(self) -> dict:
        # Implémentation requise par le parent
        return {"name": self.name, "age": self.age, "role": "user"}

u = User("Alice", 30)
print(u.to_json()) # {"name": "Alice", "age": 30, "role": "user"}
```

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-37}

1.  **Peut-on instancier une classe qui hérite de `ABC` mais qui n'a aucune méthode abstraite ?**
    Oui. Si aucune méthode n'est marquée `@abstractmethod`, la classe se comporte comme une classe normale. `ABC` sert juste de marqueur.

2.  **Quand l'erreur est-elle levée si une sous-classe n'implémente pas une méthode abstraite ?**
    Au moment de l'**instanciation** de la sous-classe (runtime), pas au moment de sa définition.

3.  **Une méthode abstraite peut-elle contenir du code ?**
    Oui. Les sous-classes peuvent l'appeler avec `super().ma_methode()`. Cela sert souvent à définir un comportement par défaut ou une étape obligatoire de validation.

4.  **Quel est l'ordre des décorateurs pour une propriété abstraite ?**
    `@property` doit être à l'extérieur (au-dessus), et `@abstractmethod` à l'intérieur (en-dessous).
    ```python
    @property
    @abstractmethod
    def ma_prop(self): ...
    ```

---

## Exercices : {#exercices-37}

### Exercice 1 - Formes Géométriques {#exercice-1-formes-geometriques}

🎯 **Objectif** : Créer une hiérarchie stricte pour le calcul d'aires.

💼 **Mise en situation** : Vous développez un logiciel de CAO (Dessin Assisté par Ordinateur). Chaque forme dessinée doit pouvoir calculer sa propre surface et son périmètre, sinon le moteur de rendu plante.

📝 **Énoncé** :
1.  Créez une classe abstraite `Shape` héritant de `ABC`.
2.  Définissez deux méthodes abstraites : `area()` et `perimeter()`.
3.  Créez deux classes concrètes :
    - `Rectangle` (longueur, largeur).
    - `Circle` (rayon).
4.  Implémentez les formules mathématiques.
5.  Tentez d'instancier une classe `Triangle` vide héritant de `Shape` pour voir l'erreur.

📺 **Résultat attendu** :
```text
Rectangle Area: 50
Circle Perimeter: 31.415...
Erreur attendue : Can't instantiate abstract class Triangle...
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
from abc import ABC, abstractmethod
import math

# 1. Le contrat
class Shape(ABC):
    @abstractmethod
    def area(self) -> float:
        pass
        
    @abstractmethod
    def perimeter(self) -> float:
        pass

# 2. Implémentation 1
class Rectangle(Shape):
    def __init__(self, length: float, width: float):
        self.length = length
        self.width = width
        
    def area(self) -> float:
        return self.length * self.width
        
    def perimeter(self) -> float:
        return 2 * (self.length + self.width)

# 3. Implémentation 2
class Circle(Shape):
    def __init__(self, radius: float):
        self.radius = radius
        
    def area(self) -> float:
        return math.pi * (self.radius ** 2)
        
    def perimeter(self) -> float:
        return 2 * math.pi * self.radius

# 4. Tests
rect = Rectangle(10, 5)
circ = Circle(5)

print(f"Rectangle Area: {rect.area()}")
print(f"Circle Perimeter: {circ.perimeter()}")

# Test d'erreur
class Triangle(Shape):
    pass

try:
    t = Triangle()
except TypeError as e:
    print(f"Erreur attendue : {e}")
```
</details>

### Exercice 2 - Plugin de Paiement {#exercice-2-plugin-paiement}

🎯 **Objectif** : Utiliser le polymorphisme pour interchanger des services.

💼 **Mise en situation** : Votre site e-commerce accepte PayPal et Stripe. Vous voulez pouvoir changer de processeur de paiement sans réécrire tout le code du panier d'achat.

📝 **Énoncé** :
1.  Créez l'interface `PaymentProcessor(ABC)` avec la méthode abstraite `pay(amount: float)`.
2.  Créez `StripeProcessor` et `PayPalProcessor`. Ils affichent juste un message simulant le paiement ("Paiement de X€ via Stripe").
3.  Créez une fonction `checkout(processor: PaymentProcessor, total: float)`.
4.  Passez une instance de chaque processeur à la fonction `checkout`.

📺 **Résultat attendu** :
```text
Connecting to Stripe API... Paiement de 100.0€ accepté.
Redirecting to PayPal... Paiement de 100.0€ accepté.
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
from abc import ABC, abstractmethod

# L'interface
class PaymentProcessor(ABC):
    @abstractmethod
    def pay(self, amount: float) -> None:
        pass

# Implémentation A
class StripeProcessor(PaymentProcessor):
    def pay(self, amount: float) -> None:
        print(f"💳 [Stripe] Paiement de {amount}€ traité.")

# Implémentation B
class PayPalProcessor(PaymentProcessor):
    def pay(self, amount: float) -> None:
        print(f"🅿️ [PayPal] Redirection... Paiement de {amount}€ validé.")

# Le code métier (agnostique du processeur réel)
def checkout(processor: PaymentProcessor, total: float):
    # Typage fort : on sait que processor a forcément une méthode pay
    processor.pay(total)

# Exécution
checkout(StripeProcessor(), 99.99)
checkout(PayPalProcessor(), 99.99)
```
</details>

### Exercice 3 - Le "Template Method" Pattern {#exercice-3-template-method}

🎯 **Objectif** : Combiner logique concrète et abstraite dans la classe mère.

💼 **Mise en situation** : Un outil de génération de rapport. La structure du rapport (En-tête, Contenu, Pied de page) est toujours la même, mais le format (PDF, HTML) change le contenu.

📝 **Énoncé** :
1.  Créez `ReportGenerator(ABC)`.
2.  Ajoutez une méthode concrète `generate()` qui appelle séquentiellement 3 méthodes abstraites : `header()`, `body()`, `footer()`.
3.  Créez `HtmlReport` qui implémente ces méthodes avec des balises HTML (`<h1>`, etc.).
4.  Créez `TextReport` avec du texte brut (`===`, etc.).
5.  Appelez `generate()` sur les deux.

📺 **Résultat attendu** :
```text
=== Rapport HTML ===
<header>Titre</header>
<body>Données</body>
<footer>Fin</footer>

=== Rapport Texte ===
--- TITRE ---
Données brutes
--- FIN ---
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
from abc import ABC, abstractmethod

class ReportGenerator(ABC):
    def generate(self):
        """Le squelette de l'algorithme (Template Method)."""
        print(f"\n=== Rapport {self.__class__.__name__} ===")
        self.header()
        self.body()
        self.footer()
    
    @abstractmethod
    def header(self): pass
    
    @abstractmethod
    def body(self): pass
    
    @abstractmethod
    def footer(self): pass

class HtmlReport(ReportGenerator):
    def header(self):
        print("<header>Rapport Annuel</header>")
    def body(self):
        print("<body><p>Tout va bien</p></body>")
    def footer(self):
        print("<footer>Copyright 2026</footer>")

class TextReport(ReportGenerator):
    def header(self):
        print("--- RAPPORT ANNUEL ---")
    def body(self):
        print("Status: OK")
    def footer(self):
        print("--- FIN ---")

# Utilisation
HtmlReport().generate()
TextReport().generate()
```
</details>