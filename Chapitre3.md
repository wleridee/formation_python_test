---
sidebar_label: Bases de la Syntaxe : Variables et Types
sidebar_position: 3
---

# Chapitre 3 : Bases de la Syntaxe : Variables et Types

Variables, Assignation, Types de données fondamentaux, Commentaires

Dans n'importe quel langage de programmation, la première étape consiste à savoir comment stocker et manipuler des données. En Python, cette simplicité apparente cache une mécanique puissante et flexible.

Ce chapitre pose les fondations absolues : comment nommer les choses, comment leur donner une valeur, et comment Python comprend ce que sont ces données. Nous adopterons dès maintenant les bonnes pratiques modernes (Python 3.14+), notamment l'annotation de type, pour prendre de bonnes habitudes.

---

## 1. Variables et Assignation {#variables-et-assignation}

### 1. Quoi
Une **variable** est une étiquette (un nom) collée sur une valeur stockée en mémoire. L'**assignation** est l'action de lier ce nom à cette valeur à l'aide de l'opérateur `=`.

Contrairement à des langages comme C ou Java, une variable en Python n'est pas une "boîte" qui contient la valeur, mais une **référence** (un pointeur) vers un objet en mémoire.

### 2. Pourquoi
Sans variables, nous devrions répéter les mêmes valeurs partout dans le code. Si le taux de TVA change, il faudrait le modifier à 50 endroits. Avec une variable `tva`, on ne change la valeur qu'à un seul endroit.

### 3. Comment

#### A. Syntaxe de base

```python
# Syntaxe : nom_variable = valeur
score = 10
nom = "Alice"
```

#### B. Cas concret avec Typage (Type Hinting)

En Python moderne, on explicite souvent le type attendu pour aider les outils de développement (IDE) et les collègues, même si Python reste dynamiquement typé à l'exécution.

```python
# Définition avec annotation de type
user_age: int = 25
user_name: str = "Sophie"
is_active: bool = True

# Assignation multiple (Unpacking)
x, y, z = 1, 2, 3

# Échange de valeurs (Swap) sans variable temporaire
a: int = 5
b: int = 10
a, b = b, a  # a vaut 10, b vaut 5
```

#### C. Exemples pratiques

**Cas 1 : Configuration d'une application**
```python
max_retries: int = 3
timeout_seconds: float = 5.5
api_url: str = "https://api.example.com/v1"
```

**Cas 2 : Calculs financiers simples**
```python
price_ht: float = 100.00
tax_rate: float = 0.20
price_ttc: float = price_ht * (1 + tax_rate)
```

**Cas 3 : État d'un système**
```python
server_name: str = "Ubuntu-01"
is_online: bool = True
current_load: float = 0.45
```

### 4. Zone de Danger

❌ **À ne pas faire** :
*   Utiliser des noms cryptiques (`a`, `x`, `temp`).
*   Utiliser des mots réservés (`if`, `for`, `class`, `def`).
*   Commencer par un chiffre (`1er_joueur` est invalide).

✅ **Bonne Pratique (PEP 8)** :
*   Utiliser le **snake_case** pour les variables : `user_first_name`, `total_count`.
*   Les constantes (valeurs qui ne changent jamais) s'écrivent en **MAJUSCULES** : `MAX_CONNECTIONS = 100`.

---

## 2. Types de Données Fondamentaux {#types-de-donnees-fondamentaux}

### 1. Quoi
En Python, **tout est objet**. Cependant, il existe des types "primitifs" ou fondamentaux intégrés au langage :
*   `int` : Entiers (positifs ou négatifs, taille illimitée).
*   `float` : Nombres à virgule flottante (réels).
*   `str` : Chaînes de caractères (texte).
*   `bool` : Booléens (Vrai ou Faux).
*   `NoneType` : L'absence de valeur (`None`).

### 2. Pourquoi
Python doit savoir comment traiter les données. On n'additionne pas du texte comme on additionne des nombres. Connaître le type permet de prédire le comportement des opérations (ex: `+` additionne des nombres mais concatène des chaînes).

### 3. Comment

#### A. Identification du type

On utilise la fonction `type()` pour inspecter une variable.

```python
count: int = 42
pi: float = 3.14159
message: str = "Hello"
is_valid: bool = False
nothing: None = None

print(type(count))   # <class 'int'>
print(type(pi))      # <class 'float'>
```

#### B. Typage dynamique mais fort

Python déduit le type automatiquement (dynamique), mais il refuse les opérations absurdes entre types incompatibles (fort).

```python
a: int = 10
b: str = "20"

# Ceci provoquera une TypeError
# total = a + b 

# Il faut convertir explicitement (Casting)
total: int = a + int(b) # 30
text_total: str = str(a) + b # "1020"
```

#### D. Tableau Comparatif des Types Scalaires

| Type | Exemple | Description | Utilisation typique |
| :--- | :--- | :--- | :--- |
| `int` | `42`, `-1` | Entier de précision arbitraire | Compteurs, index, ID |
| `float` | `3.14`, `1.0` | Nombre à virgule (approximation) | Mesures physiques, prix, stats |
| `bool` | `True`, `False` | Logique binaire | Conditions, drapeaux (flags) |
| `str` | `"Python"`, `'A'` | Séquence de caractères Unicode | Texte, noms, données brutes |
| `None` | `None` | Objet unique (Singleton) | Absence de résultat, valeur par défaut |

### 4. Zone de Danger

❌ **À ne pas faire** :
*   Comparer des `floats` avec `==` (problèmes de précision).
*   Utiliser `None` comme un zéro ou une chaîne vide. `None` est un type à part.

### 🚨 Limitations des Types
Python 3.14 est très souple, mais le typage dynamique a un coût en performance et en sécurité (bugs découverts à l'exécution). C'est pourquoi l'utilisation des annotations de type (`: int`, `: str`) est fortement recommandée pour les projets sérieux, bien qu'elles soient ignorées par l'interpréteur au moment de l'exécution (elles servent aux outils externes comme `mypy`).

---

## 3. Commentaires et Documentation {#commentaires-et-documentation}

### 1. Quoi
Les commentaires sont des lignes de texte ignorées par l'interpréteur Python. Ils servent à expliquer le code aux humains.

### 2. Pourquoi
"Le code explique *comment*, les commentaires expliquent *pourquoi*". Un code sans commentaire est une dette technique immédiate.

### 3. Comment

#### A. Commentaire simple (`#`)

```python
# Calcule le prix final avec la réduction membre
final_price = base_price * 0.90 
```

#### B. Docstrings (`"""`)
Pour documenter des modules, des classes ou des fonctions, on utilise les **Docstrings** (chaînes littérales sur plusieurs lignes) placées juste après la définition.

```python
def calculate_area(radius: float) -> float:
    """
    Calcule l'aire d'un cercle à partir de son rayon.
    
    Args:
        radius (float): Le rayon du cercle en mètres.
        
    Returns:
        float: L'aire en mètres carrés.
    """
    return 3.14159 * (radius ** 2)
```

#### C. Exemples Pratiques

**Bon commentaire :**
```python
# Nous utilisons 300s car l'API externe est lente le week-end
timeout = 300 
```

**Mauvais commentaire (Redondant) :**
```python
timeout = 300 # Assigne 300 à timeout
```

### 4. Zone de Danger
❌ **Code mort** : Ne laissez pas de vieux code commenté "au cas où". Utilisez Git pour l'historique. Le code commenté pollue la lecture.

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-3}

1.  **Quelle est la différence entre `valeur = 10` et `valeur == 10` ?**
    `=` est l'opérateur d'assignation (stocke une valeur), tandis que `==` est l'opérateur de comparaison (vérifie l'égalité).

2.  **Python est-il un langage à typage statique ou dynamique ?**
    Dynamique. Le type de la variable est déterminé à l'exécution selon la valeur qu'elle contient. Cependant, on peut utiliser des *Type Hints* pour simuler un typage statique lors du développement.

3.  **Pourquoi ne peut-on pas faire `"Score: " + 10` en Python ?**
    Parce que Python est fortement typé. Il ne convertit pas implicitement l'entier `10` en chaîne de caractères. Il faut faire `"Score: " + str(10)`.

4.  **Quelle convention de nommage utilise-t-on pour les variables ?**
    Le **snake_case** (tout en minuscules avec des underscores), par exemple : `nombre_de_clients`.

---

## Exercices : {#exercices-3}

### Exercice 1 - La Facture du Restaurant {#exercice-1---la-facture-du-restaurant}

🎯 **Objectif** : Manipuler les variables, les types numériques (`int`, `float`) et l'assignation.

💼 **Mise en situation** : Vous développez le logiciel de caisse d'un food-truck. Vous devez calculer le total d'une commande.

📝 **Énoncé** :
1.  Déclarez une variable `burger_price` (float) à 12.50.
2.  Déclarez une variable `fries_price` (float) à 4.00.
3.  Déclarez le nombre de burgers `burger_count` (int) à 2.
4.  Déclarez le nombre de frites `fries_count` (int) à 3.
5.  Calculez le total dans une variable `total`.
6.  Affichez le type de la variable `total`.

📺 **Résultat attendu** :
```text
37.0
<class 'float'>
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
# Déclaration des prix unitaires
burger_price: float = 12.50
fries_price: float = 4.00

# Déclaration des quantités
burger_count: int = 2
fries_count: int = 3

# Calcul du total
# Python respecte la priorité des opérations (* avant +)
total: float = (burger_price * burger_count) + (fries_price * fries_count)

print(total)
# On vérifie le type final
print(type(total)) 
```
</details>

### Exercice 2 - Le Formulaire d'Inscription {#exercice-2---le-formulaire-d-inscription}

🎯 **Objectif** : Comprendre les types `str`, `bool` et le casting (conversion de type).

💼 **Mise en situation** : Un utilisateur remplit un formulaire Web. Les données arrivent toujours sous forme de texte (String), même l'âge. Vous devez les traiter.

📝 **Énoncé** :
1.  Déclarez une variable `input_age` contenant la chaîne `"25"`.
2.  Convertissez cette chaîne en entier dans une variable `age`.
3.  Déclarez une variable `is_adult` qui est `True` (hardcodé pour l'instant).
4.  Affichez une phrase en concaténant : "Age : [age], Adulte : [is_adult]". Attention aux types !

📺 **Résultat attendu** :
```text
Age : 25, Adulte : True
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
# Donnée brute reçue du formulaire (string)
input_age: str = "25"

# Conversion (Casting) en entier pour pouvoir faire des maths plus tard si besoin
age: int = int(input_age)

# Statut booléen
is_adult: bool = True

# Concaténation
# Il faut convertir les non-strings en strings avec str()
result: str = "Age : " + str(age) + ", Adulte : " + str(is_adult)

print(result)
```
</details>

### Exercice 3 - Swap de variables (Pythonic Way) {#exercice-3---swap-de-variables}

🎯 **Objectif** : Utiliser l'assignation multiple, une fonctionnalité élégante de Python.

💼 **Mise en situation** : Dans un algorithme de tri, vous devez inverser la position de deux éléments.

📝 **Énoncé** :
1.  Créez une variable `rouge` avec la valeur `"Bleu"`.
2.  Créez une variable `bleu` avec la valeur `"Rouge"`.
3.  Les étiquettes sont inversées ! Corrigez cela en une seule ligne de code sans créer de troisième variable.
4.  Affichez les valeurs finales.

📺 **Résultat attendu** :
```text
Rouge contient : Rouge
Bleu contient : Bleu
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
# Situation initiale erronée
rouge: str = "Bleu"
bleu: str = "Rouge"

# Échange des valeurs (Tuple unpacking)
# Python évalue d'abord la partie droite (le tuple temporaire), puis assigne à gauche
rouge, bleu = bleu, rouge

print("Rouge contient :", rouge)
print("Bleu contient :", bleu)
```
</details>