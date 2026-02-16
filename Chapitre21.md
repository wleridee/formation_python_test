---
sidebar_label: Modules et Packages
sidebar_position: 21
---

# Chapitre 21 : Modules et Packages

Importer des modules, Créer des packages, __init__.py, Réutilisation de code

Jusqu'à présent, nous avons écrit tout notre code dans un seul fichier. Pour des scripts de 50 lignes, c'est parfait. Pour une application de 10 000 lignes, c'est ingérable.

Imaginez une bibliothèque où tous les livres seraient empilés en vrac au milieu de la pièce. Impossible de s'y retrouver. Les **modules** sont comme des livres individuels (fichiers), et les **packages** sont comme des rayonnages ou des sections (dossiers) qui organisent ces livres par thématique.

Structurer son code en modules et packages est essentiel pour le rendre maintenable, lisible et réutilisable.

---

## 1. Les Modules : Diviser pour régner {#les-modules}

### 1. Quoi
Un **module** en Python est simplement un fichier contenant du code Python (fonctions, classes, variables) avec l'extension `.py`.

### 2. Pourquoi
*   **Organisation** : Séparer la logique métier (calculs) de l'interface (affichage).
*   **Réutilisation** : Écrire une fonction une fois dans un module et l'importer dans 10 scripts différents.
*   **Namespacing** : Éviter les conflits de noms. `math.pi` est différent de `my_config.pi`.

### 3. Comment

Supposons un fichier `geometry.py` (le module) :

```python
# geometry.py
PI: float = 3.14159

def area_circle(radius: float) -> float:
    return PI * (radius ** 2)
```

#### A. Importation simple (`import`)
On importe tout le module. On doit préfixer les appels.

```python
# main.py
import geometry

# Utilisation avec le préfixe
print(geometry.PI)
result = geometry.area_circle(5)
```

#### B. Importation spécifique (`from ... import`)
On importe seulement ce dont on a besoin. Pas de préfixe nécessaire.

```python
# main.py
from geometry import area_circle

# Utilisation directe
result = area_circle(5)
# print(PI) # ❌ NameError: name 'PI' is not defined (car pas importé)
```

#### C. Alias (`as`)
Pour raccourcir les noms longs ou résoudre des conflits.

```python
import geometry as geo
from geometry import area_circle as calc_area

print(geo.PI)
print(calc_area(10))
```

### 4. Zone de Danger
❌ **L'import étoile (`from module import *`)** :
Ceci importe *tous* les noms du module dans votre espace de noms actuel.
*   **Risque** : Vous ne savez pas ce que vous importez. Si `geometry.py` contient une variable `print`, elle écrasera la fonction `print()` native de Python !
*   **✅ Bonne pratique** : Soyez toujours explicite (`import module` ou `from module import specific_item`).

---

## 2. Les Packages : Structurer les dossiers {#les-packages}

### 1. Quoi
Un **package** est un dossier contenant des modules et (généralement) un fichier spécial nommé `__init__.py`. Les packages peuvent contenir d'autres packages (sous-packages).

### 2. Pourquoi
Pour organiser hiérarchiquement de grands projets. Exemple : une librairie de traitement d'image pourrait avoir des sous-packages `filters`, `io`, `colors`.

### 3. Comment

Structure de fichiers :
```text
my_project/
    main.py
    ecommerce/          <-- Package (Dossier)
        __init__.py     <-- Marqueur de package
        database.py     <-- Module
        payments.py     <-- Module
```

Utilisation dans `main.py` :
```python
# Importation via le chemin du package (dot notation)
import ecommerce.database
from ecommerce.payments import process_payment

ecommerce.database.connect()
process_payment(100.0)
```

---

## 3. Le rôle de `__init__.py` {#init-py}

### 1. Quoi
C'est un fichier exécuté automatiquement la première fois qu'un package est importé.

### 2. Pourquoi
*   **Marqueur** : Indique à Python que ce dossier est un package (historiquement obligatoire, optionnel mais recommandé depuis Python 3.3).
*   **Façade** : Il permet de simplifier les imports pour l'utilisateur final en exposant les fonctions importantes à la racine du package.

### 3. Comment

Sans `__init__.py` configuré, l'utilisateur doit faire :
`from ecommerce.payments import process_payment`

Dans `ecommerce/__init__.py`, on peut écrire :
```python
# On expose process_payment directement au niveau du package
from .payments import process_payment
from .database import connect
```

Désormais, l'utilisateur peut faire plus simplement :
```python
# main.py
from ecommerce import process_payment, connect
```

---

## 4. Le bloc `if __name__ == "__main__":` {#name-main}

### 1. Quoi
Une condition qui vérifie si le fichier est exécuté directement (comme script principal) ou s'il est importé par un autre fichier.

### 2. Pourquoi
Pour empêcher l'exécution de code (tests, prints de debug) lorsqu'on importe simplement un module pour utiliser ses fonctions.

### 3. Comment

```python
# calculator.py

def add(a: int, b: int) -> int:
    return a + b

# Ce bloc s'exécute SEULEMENT si on lance "python calculator.py"
# Il est IGNORÉ si on fait "import calculator"
if __name__ == "__main__":
    print("Test manuel : 2 + 3 =", add(2, 3))
```

### 🚨 Limitations et Pièges

1.  **Imports Circulaires** : Si le module A importe B, et que le module B importe A, Python peut planter ou lever une erreur d'import car l'un des deux n'est pas encore complètement chargé.
    *   *Solution* : Repenser l'architecture ou importer à l'intérieur d'une fonction (import différé).
2.  **Shadowing de la Stdlib** : Ne nommez JAMAIS vos fichiers avec des noms de modules standards (ex: `email.py`, `random.py`, `math.py`). Si vous faites cela, `import random` importera votre fichier au lieu de la librairie standard, cassant tout votre environnement.

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-21}

1.  **Quelle est la différence physique entre un module et un package ?**
    Un module est un fichier (`.py`), un package est un dossier (contenant généralement un `__init__.py`).

2.  **À quoi sert le fichier `__init__.py` ?**
    Il signale qu'un dossier est un package, initialise le package, et permet de définir ce qui est exposé à l'extérieur (API publique).

3.  **Pourquoi l'instruction `from module import *` est-elle déconseillée ?**
    Elle pollue l'espace de noms actuel et peut écraser des variables ou fonctions existantes sans avertissement, rendant le code difficile à déboguer.

4.  **Que signifie `if __name__ == "__main__":` ?**
    C'est un bloc de code qui ne s'exécute que si le script est lancé directement, et non importé comme un module.

---

## Exercices : {#exercices-21}

### Exercice 1 - La Boîte à Outils de Conversion {#exercice-1-outils-conversion}

🎯 **Objectif** : Créer et importer un module simple avec des alias.

💼 **Mise en situation** : Vous travaillez sur un logiciel international et avez besoin de convertir régulièrement des unités (Miles vers Km, Celsius vers Fahrenheit).

📝 **Énoncé** :
1.  Créez un fichier `converters.py`.
2.  Ajoutez deux fonctions : `miles_to_km(miles: float)` et `celsius_to_fahrenheit(celsius: float)`. (1 mile = 1.609 km, F = C * 9/5 + 32).
3.  Créez un fichier `app.py` dans le même dossier.
4.  Dans `app.py`, importez le module sous l'alias `cv`.
5.  Utilisez les fonctions pour convertir 10 miles et 20°C.

📺 **Résultat attendu** :
```text
10 miles font 16.09 km
20°C font 68.0°F
```

<details>
<summary>💡 Voir le code complet commenté</summary>

**Fichier `converters.py`** :
```python
def miles_to_km(miles: float) -> float:
    return miles * 1.609

def celsius_to_fahrenheit(celsius: float) -> float:
    return (celsius * 9/5) + 32
```

**Fichier `app.py`** :
```python
import converters as cv  # Import avec alias

dist = 10
temp = 20

km = cv.miles_to_km(dist)
fahr = cv.celsius_to_fahrenheit(temp)

print(f"{dist} miles font {km} km")
print(f"{temp}°C font {fahr}°F")
```
</details>

### Exercice 2 - Structure d'une Boutique (Package) {#exercice-2-package-boutique}

🎯 **Objectif** : Créer un package structuré avec `__init__.py`.

💼 **Mise en situation** : Vous structurez le backend d'un site e-commerce.

📝 **Énoncé** :
1.  Créez une structure de dossiers suivante :
    ```text
    shop/
        __init__.py
        cart.py
        user.py
    main.py
    ```
2.  Dans `cart.py`, créez une fonction `get_cart_total()` qui retourne 50.0.
3.  Dans `user.py`, créez une fonction `get_user_name()` qui retourne "Alice".
4.  Dans `shop/__init__.py`, importez ces deux fonctions pour les exposer directement.
5.  Dans `main.py`, importez-les depuis `shop` (et non `shop.cart`) et affichez : "Le panier de Alice vaut 50.0€".

📺 **Résultat attendu** :
```text
Le panier de Alice vaut 50.0€
```

<details>
<summary>💡 Voir le code complet commenté</summary>

**Fichier `shop/cart.py`** :
```python
def get_cart_total() -> float:
    return 50.0
```

**Fichier `shop/user.py`** :
```python
def get_user_name() -> str:
    return "Alice"
```

**Fichier `shop/__init__.py`** :
```python
# L'import relatif '.' fait référence au dossier courant
from .cart import get_cart_total
from .user import get_user_name
```

**Fichier `main.py`** :
```python
# Grâce au __init__.py, on importe directement depuis le package 'shop'
from shop import get_cart_total, get_user_name

print(f"Le panier de {get_user_name()} vaut {get_cart_total()}€")
```
</details>

### Exercice 3 - Le Script Hybride (`__main__`) {#exercice-3-script-hybride}

🎯 **Objectif** : Comprendre le double usage script/module.

💼 **Mise en situation** : Vous écrivez un module de génération de mot de passe. Il doit pouvoir être utilisé par l'application web, mais aussi testé rapidement en ligne de commande.

📝 **Énoncé** :
1.  Créez `security.py`.
2.  Importez `random` et `string`.
3.  Créez `generate_password(length: int)` qui retourne une chaîne aléatoire.
4.  Ajoutez un bloc `if __name__ == "__main__":` qui génère et affiche un mot de passe de 8 caractères.
5.  Lancez le script directement : il doit afficher un mot de passe.
6.  Créez un autre fichier `test_import.py` qui fait `import security` et affiche "Import réussi". Il ne DOIT PAS afficher de mot de passe généré.

📺 **Résultat attendu** :
*En lançant `python security.py` :* `Mot de passe généré : aB9#kL2m`
*En lançant `python test_import.py` :* `Import réussi` (et rien d'autre)

<details>
<summary>💡 Voir le code complet commenté</summary>

**Fichier `security.py`** :
```python
import random
import string

def generate_password(length: int = 8) -> str:
    # Combine lettres et chiffres
    chars = string.ascii_letters + string.digits
    # Sélectionne 'length' caractères au hasard
    return ''.join(random.choice(chars) for _ in range(length))

# Ce bloc ne s'exécute QUE si le fichier est lancé directement
if __name__ == "__main__":
    pwd = generate_password(8)
    print(f"Mot de passe généré : {pwd}")
```

**Fichier `test_import.py`** :
```python
import security 

# Comme on importe seulement, le print du bloc 'main' de security.py ne se lance pas
print("Import réussi")
```
</details>