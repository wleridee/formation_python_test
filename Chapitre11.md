---
sidebar_label: Structures de Données : Tuples
sidebar_position: 11
---

# Chapitre 11 : Structures de Données : Tuples

Création, Immutabilité, Dépaquetage de tuples, Utilisations courantes

Si la liste est un sac à dos souple où l'on peut ajouter et retirer des objets à volonté, le **tuple** est un coffre-fort scellé. Une fois créé, son contenu est gravé dans le marbre.

En Python, le tuple est une séquence ordonnée, semblable à une liste, mais **immuable**. Cette propriété fondamentale en fait l'outil de choix pour garantir l'intégrité des données critiques et structurer des ensembles de valeurs fixes.

---

## 1. Création et Syntaxe {#creation-et-syntaxe}

### 1. Quoi
Un tuple est une collection d'objets ordonnée définie par des parenthèses `()` (contrairement aux crochets `[]` des listes).

### 2. Pourquoi
Pour regrouper des données hétérogènes qui forment un tout logique (ex: une coordonnée GPS, une date, une ligne de base de données) sans risque de modification accidentelle.

### 3. Comment

#### A. Syntaxe de base

```python
# Tuple vide
empty_tuple: tuple = ()

# Tuple standard
point: tuple[int, int] = (10, 20)

# Tuple hétérogène (str, int, float)
user_record: tuple[str, int, float] = ("Alice", 25, 1.75)

# Tuple implicite (sans parenthèses - "Packing")
coordinates = 48.85, 2.35 
```

#### B. Le piège du tuple à un seul élément
Pour créer un tuple contenant un seul élément, la virgule est **obligatoire**. Sinon, Python voit juste une parenthèse mathématique.

```python
# Ceci est un entier, pas un tuple
not_a_tuple = (5) 
print(type(not_a_tuple)) # <class 'int'>

# Ceci est un tuple
is_a_tuple = (5,) 
print(type(is_a_tuple)) # <class 'tuple'>
```

#### C. Fonction `tuple()`
Conversion d'autres séquences.

```python
numbers_list: list[int] = [1, 2, 3]
numbers_tuple: tuple[int, ...] = tuple(numbers_list)
```

### 4. Zone de Danger
❌ **Ne pas abuser des parenthèses** : Si le contexte est clair, les parenthèses sont souvent optionnelles en Python (style "Pythonic"), mais il est recommandé de les garder pour la lisibilité, sauf lors des `return` multiples.

---

## 2. Immutabilité {#immutabilite}

### 1. Quoi
Une fois défini, on ne peut **ni ajouter, ni supprimer, ni modifier** les éléments d'un tuple.

### 2. Pourquoi
*   **Sécurité** : Garantit que les constantes (configuration, coordonnées) restent intègres tout au long du programme.
*   **Performance** : Légèrement plus rapide et moins gourmand en mémoire que les listes.
*   **Hashabilité** : Peut être utilisé comme clé de dictionnaire (contrairement aux listes).

### 3. Comment

#### A. Tentative de modification (Erreur)

```python
colors: tuple[str, str, str] = ("Rouge", "Vert", "Bleu")

# Lecture OK
print(colors[0]) # "Rouge"

# Écriture INTERDITE
# colors[1] = "Jaune" 
# TypeError: 'tuple' object does not support item assignment
```

#### B. Cas concret : Configuration serveur

```python
# Ces paramètres ne doivent jamais changer en cours d'exécution
SERVER_CONFIG: tuple[str, int] = ("192.168.1.1", 8080)

def connect(config: tuple[str, int]):
    ip, port = config
    print(f"Connexion à {ip}:{port}")
```

### 🚨 Limitations
Si un tuple contient un objet mutable (comme une liste), **le contenu de cette liste peut changer**. L'immutabilité ne concerne que la "coquille" du tuple (les références qu'il contient).

```python
# Tuple contenant une liste
risky_tuple: tuple[int, list[int]] = (1, [2, 3])

# On ne peut pas changer l'élément 1 : risky_tuple[1] = [4, 5] (ERREUR)

# MAIS on peut modifier la liste interne !
risky_tuple[1].append(4) 
print(risky_tuple) # (1, [2, 3, 4])
```

---

## 3. Dépaquetage (Unpacking) {#depaquetage-unpacking}

### 1. Quoi
L'extraction des valeurs d'un tuple vers des variables individuelles en une seule ligne.

### 2. Pourquoi
Rend le code extrêmement lisible et évite les accès verbeux par index (`t[0]`, `t[1]`). C'est l'une des fonctionnalités les plus élégantes de Python.

### 3. Comment

#### A. Assignation multiple

```python
coords: tuple[int, int, int] = (10, 20, 30)

# Dépaquetage
x, y, z = coords

print(f"X={x}, Y={y}, Z={z}")
```

#### B. Swap de variables (Échange)
Grâce au packing/unpacking implicite, on peut échanger deux variables sans variable temporaire.

```python
a: int = 5
b: int = 10

a, b = b, a
# a vaut 10, b vaut 5
```

#### C. L'opérateur `*` (Asterisk)
Pour récupérer "le reste" des éléments.

```python
scores: tuple[int, ...] = (100, 95, 80, 70, 60)

winner, runner_up, *others = scores

print(winner)    # 100
print(others)    # [80, 70, 60] (C'est une liste !)
```

#### D. Ignorer des valeurs (`_`)
Convention pour dire "je me fiche de cette valeur".

```python
user_data: tuple[str, int, str] = ("Alice", 25, "Engineer")

name, _, job = user_data
# On a ignoré l'âge
```

### 4. Zone de Danger
❌ **ValueError** : Le nombre de variables à gauche doit correspondre **exactement** au nombre d'éléments dans le tuple (sauf si utilisation de `*`).

```python
t = (1, 2, 3)
# a, b = t # ValueError: too many values to unpack (expected 2)
```

---

## 4. Utilisations Courantes {#utilisations-courantes}

### 1. Quoi
Les patterns où les tuples brillent par rapport aux listes ou aux classes.

### 2. Comment

#### A. Retour multiple de fonctions
Une fonction ne peut renvoyer qu'un seul objet. Pour renvoyer plusieurs valeurs, Python les emballe automatiquement dans un tuple.

```python
def get_min_max(numbers: list[int]) -> tuple[int, int]:
    return min(numbers), max(numbers)

low, high = get_min_max([10, 5, 20])
```

#### B. Itération sur dictionnaires (`.items()`)
La méthode `.items()` renvoie des tuples `(clé, valeur)`.

```python
prices: dict[str, float] = {"Pomme": 1.2, "Poire": 1.5}

for fruit, price in prices.items():
    print(f"{fruit}: {price}€")
```

#### C. Clés de dictionnaires composées
Comme les listes sont mutables, elles ne peuvent pas servir de clés. Les tuples, oui.

```python
# Dictionnaire représentant une grille de bataille navale
# Clé : (x, y), Valeur : État
grid: dict[tuple[int, int], str] = {}

grid[(1, 2)] = "Touché"
grid[(3, 4)] = "À l'eau"
```

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-11}

1.  **Quelle est la différence principale entre une liste et un tuple ?**
    La liste est mutable (modifiable), le tuple est immuable (fixe). Les listes utilisent `[]`, les tuples `()`.

2.  **Comment créer un tuple contenant un seul élément ?**
    En ajoutant une virgule après l'élément : `(value,)`. Sans virgule, c'est juste une expression entre parenthèses.

3.  **Que permet l'opérateur `*` lors du dépaquetage ?**
    Il permet de capturer tous les éléments restants dans une liste. Ex: `a, *b = (1, 2, 3)` -> `a=1`, `b=[2, 3]`.

4.  **Pourquoi préférer un tuple à une liste pour des coordonnées GPS ?**
    Pour garantir l'intégrité des données : une latitude/longitude ne doit pas changer accidentellement. De plus, cela permet d'utiliser ces coordonnées comme clés dans un dictionnaire.

---

## Exercices : {#exercices-11}

### Exercice 1 - Le Système de Géolocalisation {#exercice-1---geo-system}

🎯 **Objectif** : Création et accès basique aux tuples.

💼 **Mise en situation** : Vous développez une app de livraison. Chaque point d'intérêt est stocké sous forme de tuple `(Nom, Latitude, Longitude)`.

📝 **Énoncé** :
1.  Créez un tuple `warehouse` contenant "Entrepôt Central", 40.7128, -74.0060.
2.  Affichez : "Lieu : [Nom]".
3.  Affichez : "GPS : [Lat] / [Long]".
4.  Tentez de modifier le nom pour "Nouvel Entrepôt" et observez l'erreur (mettez le code en commentaire).

📺 **Résultat attendu** :
```text
Lieu : Entrepôt Central
GPS : 40.7128 / -74.006
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
# Création du tuple
warehouse: tuple[str, float, float] = ("Entrepôt Central", 40.7128, -74.0060)

# Accès par index
print(f"Lieu : {warehouse[0]}")
print(f"GPS : {warehouse[1]} / {warehouse[2]}")

# Tentative de modification (provoquerait une TypeError)
# warehouse[0] = "Nouvel Entrepôt"
```
</details>

### Exercice 2 - Analyse de Ventes (Unpacking) {#exercice-2---analyse-ventes}

🎯 **Objectif** : Maîtriser le dépaquetage et l'opérateur `*`.

💼 **Mise en situation** : Un export CSV vous donne des lignes sous forme : `(ID_Produit, Nom, Prix, Vente_J1, Vente_J2, Vente_J3...)`. Le nombre de jours de vente est variable.

📝 **Énoncé** :
1.  Soit le tuple `row = (101, "Laptop Pro", 1200.00, 5, 2, 8, 10, 3)`.
2.  Utilisez l'unpacking pour extraire : `id`, `name`, `price`, et `daily_sales` (tous les chiffres de vente dans une liste).
3.  Calculez le total des ventes.
4.  Affichez "Produit : [Name], Total Ventes : [Total]".

📺 **Résultat attendu** :
```text
Produit : Laptop Pro, Total Ventes : 28
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
row: tuple = (101, "Laptop Pro", 1200.00, 5, 2, 8, 10, 3)

# Unpacking avec * pour capturer le reste
prod_id, name, price, *daily_sales = row

# daily_sales est maintenant une liste : [5, 2, 8, 10, 3]
total_sales: int = sum(daily_sales)

print(f"Produit : {name}, Total Ventes : {total_sales}")
```
</details>

### Exercice 3 - Retour Multiple et Échange {#exercice-3---retour-multiple}

🎯 **Objectif** : Utilisation des tuples dans les fonctions.

💼 **Mise en situation** : Calculer le périmètre et l'aire d'un rectangle en une seule passe.

📝 **Énoncé** :
1.  Écrivez une fonction `calculate_geometry(width, height)` qui retourne un tuple `(perimeter, area)`.
2.  Appelez la fonction avec largeur=10, hauteur=5.
3.  Récupérez les résultats dans deux variables distinctes.
4.  Affichez les résultats.

📺 **Résultat attendu** :
```text
Périmètre : 30
Aire : 50
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
def calculate_geometry(width: float, height: float) -> tuple[float, float]:
    perimeter = (width + height) * 2
    area = width * height
    # Le retour est implicitement un tuple
    return perimeter, area

w, h = 10, 5

# Récupération via unpacking
p, a = calculate_geometry(w, h)

print(f"Périmètre : {p}")
print(f"Aire : {a}")
```
</details>