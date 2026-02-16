---
sidebar_label: Compréhensions de Listes, Dictionnaires et Ensembles
sidebar_position: 39
---

# Chapitre 39 : Compréhensions de Listes, Dictionnaires et Ensembles

List comprehensions, Dict comprehensions, Set comprehensions, Expressions génératrices

Python est célèbre pour sa syntaxe concise et lisible. Parmi ses fonctionnalités les plus appréciées figurent les **compréhensions**. C'est une manière élégante (dite "Pythonique") de créer de nouvelles collections à partir d'itérables existants, en une seule ligne de code expressive, remplaçant souvent les boucles `for` classiques.

Ce n'est pas seulement du sucre syntaxique : c'est souvent plus performant.

---

## 1. Compréhension de Liste (List Comprehension) {#comprehension-de-liste}

### 1. Quoi
Une syntaxe compacte pour construire une liste en appliquant une expression à chaque élément d'une séquence, optionnellement filtrée par une condition.
Le format est : `[expression for item in iterable if condition]`

### 2. Pourquoi
Pour transformer ou filtrer des données sans écrire 4 lignes de boucle `for` et `append()`. Le code est plus "déclaratif" : on décrit *ce qu'on veut obtenir* plutôt que *comment l'obtenir*.

### 3. Comment

#### A. Syntaxe de base vs Boucle classique

**❌ Avant (Classique) :**
```python
numbers = [1, 2, 3, 4, 5]
squares = []
for n in numbers:
    squares.append(n ** 2)
# squares vaut [1, 4, 9, 16, 25]
```

**✅ Après (Compréhension) :**
```python
numbers = [1, 2, 3, 4, 5]
# "Le carré de n POUR chaque n DANS numbers"
squares = [n ** 2 for n in numbers]
```

#### B. Avec Condition (Filtrage)

```python
# Garder uniquement les nombres pairs
evens = [n for n in numbers if n % 2 == 0]
# evens vaut [2, 4]
```

#### C. Cas pratiques

**Exemple 1 : Nettoyage de données (String strip)**
```python
raw_users = ["  alice ", "bob", "  charlie  "]
clean_users = [u.strip().title() for u in raw_users]
# ['Alice', 'Bob', 'Charlie']
```

**Exemple 2 : Extraction d'attributs d'objets**
```python
from dataclasses import dataclass

@dataclass
class Product:
    id: int
    name: str
    price: float

products = [
    Product(1, "Laptop", 1000.0),
    Product(2, "Mouse", 25.0),
    Product(3, "Keyboard", 50.0)
]

# Liste des noms de produits chers (> 40€)
expensive_names = [p.name for p in products if p.price > 40]
# ['Laptop', 'Keyboard']
```

### 4. Zone de Danger
❌ **Trop de logique** : N'essayez pas de tout faire en une ligne. Si la compréhension dépasse la largeur de l'écran ou implique plusieurs conditions imbriquées, utilisez une boucle `for` normale pour la lisibilité.

```python
# Illisible ❌
data = [x if x > 10 else x*2 for x in range(20) if x % 2 == 0 or x % 3 == 0]
```

---

## 2. Compréhension de Dictionnaire (Dict Comprehension) {#comprehension-de-dictionnaire}

### 1. Quoi
Le même principe, mais pour créer un dictionnaire.
Syntaxe : `{key_expr: value_expr for item in iterable if condition}`

### 2. Pourquoi
Idéal pour indexer des données (créer un lookup par ID), inverser un dictionnaire (valeur -> clé), ou transformer les valeurs d'un mapping existant.

### 3. Comment

#### A. Syntaxe de base

```python
users = [("alice", 25), ("bob", 30), ("charlie", 35)]

# Créer un dictionnaire {nom: age}
user_ages = {name: age for name, age in users}
# {'alice': 25, 'bob': 30, 'charlie': 35}
```

#### B. Indexation d'objets (Cas très fréquent)

```python
# Reprenons notre classe Product de l'exemple précédent
products = [
    Product(1, "Laptop", 1000.0),
    Product(2, "Mouse", 25.0)
]

# On veut accéder rapidement à un produit par son ID
product_by_id = {p.id: p for p in products}

# Accès immédiat O(1)
print(product_by_id[1]) # Product(id=1, name='Laptop', ...)
```

---

## 3. Compréhension d'Ensemble (Set Comprehension) {#comprehension-d-ensemble}

### 1. Quoi
Créer un `set` (valeurs uniques) à partir d'un itérable.
Syntaxe : `{expression for item in iterable}` (comme le dict, mais sans `:`)

### 2. Pourquoi
Pour dédoublonner et transformer une liste en une seule opération.

### 3. Comment

```python
tags = ["python", "JAVA", "Python", "javascript", "Java"]

# Dédoublonner sans tenir compte de la casse
unique_tags = {t.lower() for t in tags}

print(unique_tags)
# {'python', 'java', 'javascript'} (ordre non garanti)
```

---

## 4. Expressions Génératrices (Generator Expressions) {#expressions-generatrices}

### 1. Quoi
Une syntaxe similaire aux list comprehensions, mais avec des parenthèses `()` au lieu des crochets `[]`. Cela ne crée **pas** de liste en mémoire. Cela retourne un **générateur** qui produit les valeurs une par une à la demande.

### 2. Pourquoi
Pour économiser la mémoire (RAM). Si vous traitez 1 million de lignes pour en calculer la somme, inutile de stocker le million de nombres transformés dans une liste intermédiaire.

### 3. Comment

```python
# Liste (RAM utilisée = proportionnelle à 1 million)
squares_list = [x**2 for x in range(1_000_000)]

# Générateur (RAM utilisée = quasi nulle, quelques octets)
squares_gen = (x**2 for x in range(1_000_000))

print(squares_gen) 
# <generator object ... at 0x...>

# Consommation
total = sum(squares_gen) # Le générateur calcule les carrés à la volée
```

#### D. Tableau Comparatif

| Type | Syntaxe | Résultat | Mémoire | Vitesse d'accès |
| :--- | :--- | :--- | :--- | :--- |
| **Liste** | `[x for x in data]` | `list` | Haute (Tout stocké) | Accès indexé rapide |
| **Générateur** | `(x for x in data)` | `generator` | Basse (Un par un) | Uniquement itération |

### 4. Zone de Danger
❌ **Usage unique** : Un générateur ne peut être parcouru qu'une seule fois. Une fois "consommé", il est vide.

```python
gen = (x for x in range(3))
list(gen) # [0, 1, 2]
list(gen) # [] -> Vide !
```

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-39}

1.  **Quelle est la différence syntaxique entre une compréhension de liste et un générateur ?**
    La liste utilise des crochets `[]`, le générateur utilise des parenthèses `()`.

2.  **Pourquoi préférer une expression génératrice à une liste compréhension pour calculer une somme ?**
    Pour éviter de stocker tous les éléments intermédiaires en mémoire. Le générateur produit les valeurs une à une pour la fonction `sum()`.

3.  **Peut-on mettre une boucle `for` imbriquée dans une compréhension ?**
    Oui, par exemple `[x*y for x in range(3) for y in range(3)]`. Mais attention à la lisibilité.

4.  **Comment créer un dictionnaire qui associe un mot à sa longueur pour une liste de mots ?**
    `{mot: len(mot) for mot in liste_mots}`

---

## Exercices : {#exercices-39}

### Exercice 1 - Filtrage de Logs {#exercice-1-filtrage-logs}

🎯 **Objectif** : Utiliser une list comprehension avec condition.

💼 **Mise en situation** : Vous analysez des logs serveur. Vous avez une liste de chaînes brutes et vous voulez extraire uniquement les messages d'erreur (contenant "ERROR") en les mettant en majuscules.

📝 **Énoncé** :
1.  Liste brute : `["INFO: Started", "ERROR: DB Crash", "WARNING: Slow", "error: file not found"]`.
2.  Créez une nouvelle liste contenant uniquement les chaînes ayant "error" (insensible à la casse).
3.  La sortie doit être en MAJUSCULES et nettoyée des espaces inutiles.

📺 **Résultat attendu** :
```text
['ERROR: DB CRASH', 'ERROR: FILE NOT FOUND']
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
logs = ["INFO: Started", "ERROR: DB Crash", "WARNING: Slow", "error: file not found"]

# Compréhension avec filtrage et transformation
# 1. log.upper() : Transformation
# 2. if "ERROR" in log.upper() : Condition (après transformation virtuelle ou sur l'original si on veut)
# Ici on vérifie "ERROR" dans la version majuscule pour être insensible à la casse
error_logs = [log.upper() for log in logs if "ERROR" in log.upper()]

print(error_logs)
```
</details>

### Exercice 2 - Indexation de Clients (Dict Comp) {#exercice-2-indexation-clients}

🎯 **Objectif** : Transformer une liste de tuples en dictionnaire indexé.

💼 **Mise en situation** : Vous recevez un export CSV sous forme de liste de tuples `(id, nom, email)`. Pour optimiser les recherches, vous voulez un dictionnaire où la clé est l'ID.

📝 **Énoncé** :
1.  Données : `[(101, "Alice", "a@corp.com"), (102, "Bob", "b@corp.com")]`.
2.  Créez un dictionnaire `clients_db`.
3.  Clé = ID, Valeur = dictionnaire `{"name": nom, "email": email}`.
4.  Affichez les infos du client 102.

📺 **Résultat attendu** :
```text
Info client 102 : {'name': 'Bob', 'email': 'b@corp.com'}
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
raw_data = [
    (101, "Alice", "a@corp.com"), 
    (102, "Bob", "b@corp.com")
]

# Dict comprehension
# key : client_id
# value : un petit dictionnaire structuré
clients_db = {
    client_id: {"name": name, "email": email} 
    for client_id, name, email in raw_data
}

print(f"Info client 102 : {clients_db[102]}")
```
</details>

### Exercice 3 - La Matrice (Nested Comp) {#exercice-3-matrice-flatten}

🎯 **Objectif** : Aplatir une liste de listes (Flatten) ou créer une matrice.

💼 **Mise en situation** : Vous avez une matrice (liste de listes) représentant une grille de pixels. Vous voulez une liste plate de tous les pixels > 0 (pixels allumés).

📝 **Énoncé** :
1.  Matrice : `[[0, 255, 0], [128, 0, 64], [0, 0, 255]]`.
2.  Utilisez une double boucle `for` DANS une compréhension de liste pour extraire les valeurs non-nulles.
3.  L'ordre de lecture est ligne par ligne.

📺 **Résultat attendu** :
```text
Pixels allumés : [255, 128, 64, 255]
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
matrix = [
    [0, 255, 0], 
    [128, 0, 64], 
    [0, 0, 255]
]

# Double boucle dans la compréhension
# L'ordre se lit : 
# 1. for row in matrix (boucle externe)
# 2. for pixel in row (boucle interne)
# 3. if pixel > 0 (condition)
active_pixels = [pixel for row in matrix for pixel in row if pixel > 0]

print(f"Pixels allumés : {active_pixels}")
```
</details>