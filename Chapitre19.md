---
sidebar_label: Programmation Orientée Objet : Héritage et Polymorphisme
sidebar_position: 19
---

# Chapitre 19 : Programmation Orientée Objet : Héritage et Polymorphisme

Classe parent, Classe enfant, Surcharge de méthode, Polymorphisme

En biologie, un chat et un chien partagent des caractéristiques communes (mammifères, vertébrés) héritées d'un ancêtre commun, tout en ayant leurs propres spécificités. En programmation orientée objet, c'est la même chose.

L'**héritage** vous permet de créer une nouvelle classe à partir d'une classe existante, en récupérant automatiquement ses attributs et méthodes. Le **polymorphisme** vous permet ensuite de traiter ces objets différents de manière interchangeable (comme traiter un chat et un chien simplement comme des "Animaux"). C'est un pilier fondamental pour écrire du code évolutif et DRY (Don't Repeat Yourself).

---

## 1. L'Héritage Simple {#heritance-simple}

### 1. Quoi
L'héritage permet à une classe (dite **enfant**, dérivée ou sous-classe) de reprendre toutes les fonctionnalités d'une autre classe (dite **parent**, de base ou super-classe). En Python, toutes les classes héritent implicitement de `object`.

### 2. Pourquoi
Pour éviter la duplication de code. Si vous avez une classe `Car` et une classe `Truck` qui ont toutes deux des roues, un moteur et une couleur, il est logique de créer une classe commune `Vehicle` pour ne pas réécrire cette logique deux fois.

### 3. Comment

#### A. Syntaxe de base

```python
# Classe Parent
class Animal:
    def __init__(self, name: str):
        self.name = name

    def breathe(self):
        print(f"{self.name} respire.")

# Classe Enfant (hérite de Animal)
class Cat(Animal):
    def meow(self):
        print(f"{self.name} dit Miaou !")

# Utilisation
minou = Cat("Felix")
minou.breathe() # Hérité de Animal -> "Felix respire."
minou.meow()    # Spécifique à Cat -> "Felix dit Miaou !"
```

#### B. Surcharge (Overriding)
L'enfant peut redéfinir une méthode du parent pour modifier son comportement.

```python
class Dog(Animal):
    # On surcharge breathe
    def breathe(self):
        print(f"{self.name} respire bruyamment avec la langue sortie.")

dog = Dog("Rex")
dog.breathe() # Utilise la version de Dog
```

#### C. Accès au parent avec `super()`
Parfois, on veut étendre le comportement du parent, pas le remplacer totalement. `super()` donne accès aux méthodes de la classe parent.

```python
class Robot:
    def __init__(self, model: str):
        self.model = model

class FlyingRobot(Robot):
    def __init__(self, model: str, wingspan: int):
        # On appelle le constructeur du parent pour gérer 'model'
        super().__init__(model)
        # On gère l'attribut spécifique
        self.wingspan = wingspan

r = FlyingRobot("Drone-X", 120)
print(r.model)    # Drone-X
print(r.wingspan) # 120
```

### 4. Zone de Danger
❌ **Héritage trop profond** :
Évitez les chaînes d'héritage complexes (`A -> B -> C -> D -> E`). Cela rend le code difficile à suivre. Préférez la composition ("a un") à l'héritage ("est un") si la relation n'est pas évidente.

---

## 2. Le Polymorphisme {#polymorphisme}

### 1. Quoi
Le polymorphisme (du grec "plusieurs formes") est la capacité d'objets de types différents à répondre à la **même méthode** de manière spécifique.

### 2. Pourquoi
Pour écrire du code générique qui fonctionne avec n'importe quel objet respectant une certaine interface, sans avoir besoin de connaître son type exact.

### 3. Comment

#### A. Exemple concret
Imaginons un lecteur multimédia qui doit lire différents types de fichiers.

```python
class AudioFile:
    def play(self):
        print("Lecture du fichier audio 🎵")

class VideoFile:
    def play(self):
        print("Lecture de la vidéo 🎬")

# Fonction polymorphique
def start_media(media_object):
    # On ne sait pas si c'est Audio ou Video, mais on sait qu'il a .play()
    media_object.play()

files = [AudioFile(), VideoFile(), AudioFile()]

for f in files:
    start_media(f)
# Sortie :
# Lecture du fichier audio 🎵
# Lecture de la vidéo 🎬
# Lecture du fichier audio 🎵
```

#### B. Duck Typing
En Python, contrairement à Java ou C#, le polymorphisme est souvent implicite grâce au "Duck Typing" : *"Si ça marche comme un canard et que ça cancane comme un canard, c'est un canard"*. Il n'est pas obligatoire d'hériter d'une classe commune tant que la méthode existe.

```python
class Duck:
    def quack(self):
        print("Quack!")

class Person:
    def quack(self):
        print("Je imite le canard !")

def make_it_quack(obj):
    obj.quack()

make_it_quack(Duck())   # Marche
make_it_quack(Person()) # Marche aussi !
```

---

## 3. `isinstance` et `issubclass` {#isinstance-issubclass}

### 1. Quoi
Fonctions natives pour vérifier les types et les relations d'héritage.

### 2. Pourquoi
Parfois, il est nécessaire de valider le type d'un objet avant de l'utiliser (surtout si on ne contrôle pas l'entrée).

### 3. Comment

```python
class Shape: pass
class Circle(Shape): pass

c = Circle()

# Vérification d'instance
print(isinstance(c, Circle)) # True
print(isinstance(c, Shape))  # True (car Circle hérite de Shape)
print(isinstance(c, object)) # True (tout hérite de object)

# Vérification de sous-classe
print(issubclass(Circle, Shape)) # True
print(issubclass(Shape, Circle)) # False
```

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-19}

1.  **Quel est le mot-clé pour appeler une méthode de la classe parente ?**
    `super()`. Par exemple `super().__init__()`.

2.  **Une classe enfant hérite-t-elle du constructeur `__init__` du parent ?**
    Oui, sauf si elle définit son propre `__init__`. Dans ce cas, le parent n'est plus appelé automatiquement, il faut le faire explicitement avec `super()`.

3.  **Qu'est-ce que le polymorphisme en Python ?**
    La capacité d'appeler la même méthode sur des objets de types différents, chaque objet réagissant à sa manière.

4.  **Si `B` hérite de `A`, est-ce que `isinstance(b, A)` est vrai ?**
    Oui, une instance de la classe enfant est aussi considérée comme une instance de la classe parente.

---

## Exercices : {#exercices-19}

### Exercice 1 - Le Zoo Virtuel (Héritage Simple) {#exercice-1---zoo-virtuel}

🎯 **Objectif** : Créer une hiérarchie de classes et surcharger des méthodes.

💼 **Mise en situation** : Vous modélisez les animaux d'un zoo pour le système de nourrissage.

📝 **Énoncé** :
1.  Créez une classe `Animal` avec une méthode `speak()` qui affiche "...".
2.  Créez `Lion` qui hérite d'Animal et surcharge `speak()` pour afficher "ROAR".
3.  Créez `Parrot` qui hérite d'Animal et surcharge `speak()` pour afficher "Coco veut un gâteau".
4.  Créez une liste contenant un Lion et un Perroquet, et faites-les parler via une boucle.

📺 **Résultat attendu** :
```text
ROAR
Coco veut un gâteau
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
class Animal:
    def speak(self):
        print("...")

class Lion(Animal):
    def speak(self):
        print("ROAR")

class Parrot(Animal):
    def speak(self):
        print("Coco veut un gâteau")

zoo = [Lion(), Parrot()]

for animal in zoo:
    animal.speak()
```
</details>

### Exercice 2 - Système de Formes Géométriques (Super) {#exercice-2---formes-geometriques}

🎯 **Objectif** : Utilisation de `super().__init__` et attributs spécifiques.

💼 **Mise en situation** : Un logiciel de dessin vectoriel.

📝 **Énoncé** :
1.  Classe `Shape` avec un attribut `color` et une méthode `describe()` qui affiche "Forme de couleur [color]".
2.  Classe `Rectangle` héritant de `Shape`.
    - Constructeur : prend `color`, `width`, `height`.
    - Appelle le constructeur parent pour la couleur.
    - Surcharge `describe()` pour afficher "Rectangle [color] de [width]x[height]".
3.  Instanciez un rectangle rouge de 10x20 et décrivez-le.

📺 **Résultat attendu** :
```text
Rectangle rouge de 10x20
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
class Shape:
    def __init__(self, color: str):
        self.color = color

    def describe(self):
        print(f"Forme de couleur {self.color}")

class Rectangle(Shape):
    def __init__(self, color: str, width: int, height: int):
        # Initialisation du parent
        super().__init__(color)
        # Initialisation spécifique
        self.width = width
        self.height = height

    def describe(self):
        print(f"Rectangle {self.color} de {self.width}x{self.height}")

r = Rectangle("rouge", 10, 20)
r.describe()
```
</details>

### Exercice 3 - Le Paiement Polymorphe {#exercice-3---paiement-polymorphe}

🎯 **Objectif** : Polymorphisme dans un contexte business.

💼 **Mise en situation** : Votre site e-commerce accepte Paypal et Stripe. Le processus de paiement doit être transparent pour le contrôleur principal.

📝 **Énoncé** :
1.  Classe `PaymentProcessor` (abstraite conceptuellement) avec méthode `pay(amount)`.
2.  Classe `Paypal` : `pay(amount)` affiche "Paiement de X€ via Paypal".
3.  Classe `Stripe` : `pay(amount)` affiche "Paiement de X€ via Carte Bancaire (Stripe)".
4.  Fonction `process_order(processor, amount)` qui appelle la méthode `pay`.
5.  Simulez une commande de 50€ payée par Paypal, puis une de 100€ par Stripe.

📺 **Résultat attendu** :
```text
Traitement de la commande...
Paiement de 50€ via Paypal
Traitement de la commande...
Paiement de 100€ via Carte Bancaire (Stripe)
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
class Paypal:
    def pay(self, amount: float):
        print(f"Paiement de {amount}€ via Paypal")

class Stripe:
    def pay(self, amount: float):
        print(f"Paiement de {amount}€ via Carte Bancaire (Stripe)")

def process_order(processor, amount: float):
    print("Traitement de la commande...")
    # Polymorphisme : on appelle .pay() sans se soucier de la classe exacte
    processor.pay(amount)

# Simulation
process_order(Paypal(), 50)
process_order(Stripe(), 100)
```
</details>