---
sidebar_label: "Fonctions Lambda: Fonctions Anonymes"
sidebar_position: 19
difficulty: "confirmé"
---

# Fonctions Lambda: Fonctions Anonymes {#fonctions-lambda-anonymes-19}

Parfois, vous avez besoin d'une petite fonction pour une tâche très spécifique et à usage unique, par exemple comme argument pour une autre fonction. Déclarer une fonction complète avec `def` peut sembler lourd et verbeux. Python offre une solution élégante pour cela : les **fonctions lambda**.

Une fonction lambda est une petite fonction anonyme (sans nom) qui peut avoir un nombre quelconque d'arguments, mais ne peut contenir qu'une seule expression.

## 1. La Syntaxe Lambda {#syntaxe-lambda-19}

### Quoi
La syntaxe d'une fonction lambda est conçue pour être concise. Elle est définie avec le mot-clé `lambda`, suivi des arguments, de deux-points `:`, et enfin de l'expression à évaluer. Le résultat de cette expression est automatiquement retourné.

```mermaid
graph TD
    subgraph "Fonction `def` classique"
        A[def addition(x, y):]
        B[    return x + y]
        A --> B
    end

    subgraph "Fonction Lambda équivalente"
        C["lambda x, y: x + y"]
    end
    
    style C fill:#f9f,stroke:#333,stroke-width:2px
```

### Pourquoi
L'intérêt principal n'est pas de remplacer toutes les fonctions `def`, mais de fournir une syntaxe légère pour les fonctions "jetables". Elles sont particulièrement utiles lorsqu'une fonction en attend une autre en paramètre (ce sont les fonctions d'ordre supérieur, ou *higher-order functions*).

### Comment
**Comparaison `def` vs `lambda`**
```python
# Version classique avec def
def carre(x):
    return x * x

# Version avec lambda, assignée à une variable (pour la démonstration)
carre_lambda = lambda x: x * x

# Les deux sont appelables de la même manière
print(f"Avec def: {carre(5)}")
print(f"Avec lambda: {carre_lambda(5)}")

# Une lambda peut avoir plusieurs arguments
addition = lambda a, b: a + b
print(f"Addition avec lambda: {addition(10, 3)}")
```

### Zone de Danger
*   **Limitation à une seule expression** : Une fonction lambda ne peut pas contenir d'instructions comme des boucles `for`, des conditions `if/else` complexes (bien qu'une expression ternaire soit possible : `lambda x: "pair" if x % 2 == 0 else "impair"`), ou des `print`. Elle est faite pour des calculs simples et directs.
*   **Lisibilité (PEP 8)** : La convention officielle de style Python (PEP 8) déconseille d'assigner une lambda à une variable (comme nous l'avons fait avec `carre_lambda`). Si une fonction a besoin d'un nom, il faut utiliser `def`. La vraie puissance des lambdas se révèle lorsqu'elles sont utilisées directement comme arguments.

---

## 2. Lambdas avec les Fonctions d'Ordre Supérieur {#lambdas-fonctions-ordre-superieur-19}

Le cas d'usage le plus courant et le plus puissant des lambdas est de les combiner avec des fonctions intégrées comme `sorted()`, `map()`, et `filter()`.

### Quoi | Pourquoi
Ces fonctions acceptent une autre fonction comme argument pour personnaliser leur comportement. `sorted()` peut trier selon une clé de tri personnalisée, `map()` applique une transformation à chaque élément, et `filter()` sélectionne des éléments selon une condition. Une lambda est parfaite pour définir cette logique de manière concise et "sur le vif".

### Comment

#### Cas réel 1 : `sorted()`
Pour trier des données complexes (comme une liste de dictionnaires), on doit spécifier *sur quelle base* trier. La fonction lambda est idéale pour extraire cette "clé" de tri.

```python
# Liste de dictionnaires représentant des produits
produits = [
    {'nom': 'Clavier', 'prix': 75},
    {'nom': 'Souris', 'prix': 25},
    {'nom': 'Écran', 'prix': 300},
]

# Trier les produits par prix, du moins cher au plus cher
produits_tries = sorted(produits, key=lambda produit: produit['prix'])

print(produits_tries)
# Résultat: [{'nom': 'Souris', 'prix': 25}, {'nom': 'Clavier', 'prix': 75}, ...]
```
Ici, `key=lambda produit: produit['prix']` crée une fonction anonyme qui, pour chaque `produit` de la liste, retourne la valeur de sa clé `'prix'`. `sorted()` utilise ensuite cette valeur pour l'ordre de tri.

#### Cas réel 2 : `map()`
La fonction `map(fonction, iterable)` applique une fonction à chaque élément d'un itérable.

```python
nombres = [1, 2, 3, 4, 5]

# Mettre au carré chaque nombre de la liste
nombres_au_carre = map(lambda x: x ** 2, nombres)

# map retourne un objet 'map', il faut le convertir en liste pour l'afficher
print(list(nombres_au_carre))
# Résultat: [1, 4, 9, 16, 25]
```

#### Cas réel 3 : `filter()`
La fonction `filter(fonction, iterable)` construit un itérable à partir des éléments pour lesquels la fonction retourne `True`.

```python
nombres = [1, 2, 3, 4, 5, 6, 7, 8]

# Garder uniquement les nombres pairs
nombres_pairs = filter(lambda x: x % 2 == 0, nombres)

print(list(nombres_pairs))
# Résultat: [2, 4, 6, 8]
```

### Zone de Danger
*   **List Comprehensions vs `map`/`filter`** : En Python, les compréhensions de liste sont souvent considérées comme plus "pythoniques" et plus lisibles que `map` et `filter` pour des cas simples.
    *   `map(lambda x: x ** 2, nombres)` devient `[x ** 2 for x in nombres]`.
    *   `filter(lambda x: x % 2 == 0, nombres)` devient `[x for x in nombres if x % 2 == 0]`.
    
    Le choix est souvent une question de style, mais il est crucial de connaître l'alternative par compréhension de liste.

---

## Validation des Acquis {#validation-19}

### 3 Questions Clés
1.  Quelle est la principale contrainte syntaxique d'une fonction lambda ?
2.  Dans quel contexte une fonction lambda est-elle plus appropriée qu'une fonction `def` classique ?
3.  Pour trier une liste de listes en fonction du deuxième élément de chaque sous-liste, quelle fonction lambda utiliseriez-vous avec `sorted()` ?

### 3 Exercices Progressifs

#### Exercice 1 : Tri par Longueur
Vous avez une liste de mots. Utilisez `sorted()` avec une fonction lambda pour trier cette liste en fonction de la longueur de chaque mot, du plus court au plus long.

```python
mots = ["python", "est", "un", "langage", "puissant"]
```

<details>
<summary>Découvrir la solution commentée</summary>

```python
mots = ["python", "est", "un", "langage", "puissant"]

# La clé de tri est la longueur du mot, obtenue avec la fonction len().
# La lambda prend un mot 'm' et retourne len(m).
mots_tries = sorted(mots, key=lambda m: len(m))

print(mots_tries)
# Résultat attendu : ['un', 'est', 'python', 'langage', 'puissant']
```
</details>

#### Exercice 2 : Filtrer des Données
Vous avez une liste de dictionnaires représentant des utilisateurs. Utilisez `filter()` avec une fonction lambda pour ne garder que les utilisateurs actifs (`'est_actif': True`).

```python
utilisateurs = [
    {'nom': 'Alice', 'est_actif': True},
    {'nom': 'Bob', 'est_actif': False},
    {'nom': 'Charlie', 'est_actif': True},
    {'nom': 'David', 'est_actif': False},
]
```

<details>
<summary>Découvrir la solution commentée</summary>

```python
utilisateurs = [
    {'nom': 'Alice', 'est_actif': True},
    {'nom': 'Bob', 'est_actif': False},
    {'nom': 'Charlie', 'est_actif': True},
    {'nom': 'David', 'est_actif': False},
]

# La lambda prend un utilisateur 'u' et retourne la valeur de sa clé 'est_actif'.
# filter() ne conservera que les utilisateurs pour lesquels cette valeur est True.
utilisateurs_actifs = filter(lambda u: u['est_actif'], utilisateurs)

print(list(utilisateurs_actifs))
# Résultat attendu : [{'nom': 'Alice', 'est_actif': True}, {'nom': 'Charlie', 'est_actif': True}]
```
</details>

#### Exercice 3 : Transformation de Données
Vous avez une liste de tuples, où chaque tuple contient un nom de produit et son prix en euros. Utilisez `map()` avec une fonction lambda pour transformer cette liste en une liste de chaînes de caractères formatées : `"PRODUIT: PRIX €"`.

```python
produits_tuples = [
    ("Laptop", 1200),
    ("Souris", 25),
    ("Casque", 150)
]
```

<details>
<summary>Découvrir la solution commentée</summary>

```python
produits_tuples = [
    ("Laptop", 1200),
    ("Souris", 25),
    ("Casque", 150)
]

# La lambda prend un tuple 'p' en entrée.
# p[0] est le nom et p[1] est le prix.
# Elle retourne une f-string formatée avec ces valeurs.
produits_formates = map(lambda p: f"{p[0].upper()}: {p[1]} €", produits_tuples)

print(list(produits_formates))
# Résultat attendu : ['LAPTOP: 1200 €', 'SOURIS: 25 €', 'CASQUE: 150 €']
```
</details>