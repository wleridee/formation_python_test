---
sidebar_label: Fonctions Lambda (Anonymes)
sidebar_position: 52
---

# Chapitre 52 : Fonctions Lambda (Anonymes)

Définition de lambda, Cas d'utilisation, Limitations, map(), filter(), sorted()

Jusqu'à présent, nous avons défini des fonctions avec le mot-clé `def`, en leur donnant un nom et un bloc de code indenté. Mais parfois, vous avez besoin d'une petite fonction "jetable", juste pour une opération unique, sans avoir envie de polluer votre code avec une définition formelle.

Python propose pour cela les **fonctions lambda** (aussi appelées fonctions anonymes). Bien que puissantes, elles doivent être utilisées avec parcimonie pour ne pas nuire à la lisibilité.

---

## 1. La Syntaxe Lambda {#syntaxe-lambda}

### 1. Quoi
Une fonction lambda est une fonction **anonyme** (sans nom), définie sur **une seule ligne**, et qui retourne automatiquement le résultat de son expression.

### 2. Pourquoi
Pour passer une logique simple en argument à une autre fonction (comme un critère de tri ou une règle de filtrage) sans avoir à définir une fonction nommée complète 5 lignes plus haut.

### 3. Comment

#### A. Syntaxe de base
La structure est : `lambda arguments: expression`.

Comparaison entre une fonction classique et une lambda :

```python
# Version classique (def)
def ajouter(x: int, y: int) -> int:
    return x + y

# Version Lambda
# Note : On assigne rarement une lambda à une variable, c'est pour l'exemple
ajouter_lambda = lambda x, y: x + y

print(ajouter(5, 3))        # 8
print(ajouter_lambda(5, 3)) # 8
```

#### B. Les règles d'or
1.  **Mot-clé `lambda`** : Remplace `def`.
2.  **Pas de parenthèses** autour des arguments.
3.  **Pas de `return`** : Le retour est implicite. L'évaluation de l'expression *est* la valeur de retour.
4.  **Une seule expression** : Pas de blocs, pas de boucles `for` ou `while` complexes, pas d'assignation de variables à l'intérieur.

### 4. Zone de Danger
❌ **Ne jamais assigner une lambda à un nom** (PEP 8) :
```python
# Mauvais : Si ça a un nom, utilisez def !
carre = lambda x: x**2 

# Bon
def carre(x: int) -> int: return x**2
```
L'assignation d'une lambda supprime son intérêt (être anonyme) et rend le débogage plus difficile car la trace d'erreur affichera `<lambda>` au lieu du nom de la fonction.

---

## 2. Cas d'Usage : `sorted`, `map` et `filter` {#cas-usage-sorted-map-filter}

### 1. Quoi
Les lambdas brillent quand elles sont passées comme arguments à des fonctions d'ordre supérieur (fonctions qui acceptent d'autres fonctions).

### 2. Pourquoi
C'est le cas d'utilisation principal : définir un comportement "à la volée" pour transformer ou trier des données.

### 3. Comment

#### A. Tri personnalisé avec `sorted()` (Le cas roi 👑)
Trier une liste de dictionnaires ou d'objets est impossible sans dire à Python sur *quel champ* trier.

```python
from typing import Dict, List

produits: List[Dict[str, any]] = [
    {"nom": "Laptop", "prix": 999},
    {"nom": "Souris", "prix": 25},
    {"nom": "Clavier", "prix": 50},
]

# Trier par prix croissant
# La lambda prend un élément 'p' et retourne la clé de tri 'p["prix"]'
produits_tries = sorted(produits, key=lambda p: p["prix"])

print(produits_tries) 
# [{'nom': 'Souris', 'prix': 25}, {'nom': 'Clavier', 'prix': 50}, ...]
```

#### B. Filtrage avec `filter()`
`filter(fonction, iterable)` garde les éléments pour lesquels la fonction retourne `True`.

```python
nombres: list[int] = [1, 2, 3, 4, 5, 6]

# Garder uniquement les pairs
pairs = list(filter(lambda x: x % 2 == 0, nombres))
# Résultat : [2, 4, 6]
```

#### C. Transformation avec `map()`
`map(fonction, iterable)` applique la fonction à chaque élément.

```python
# Mettre au carré
carres = list(map(lambda x: x**2, nombres))
# Résultat : [1, 4, 9, 16, 25, 36]
```

#### D. Tableau Comparatif : Lambda vs Compréhension
En Python moderne, les compréhensions de listes sont souvent préférées à `map` et `filter`.

| Opération | Approche Lambda | Approche Compréhension (Pythonique ✅) |
| :--- | :--- | :--- |
| **Map** | `list(map(lambda x: x*2, L))` | `[x*2 for x in L]` |
| **Filter** | `list(filter(lambda x: x>0, L))` | `[x for x in L if x > 0]` |
| **Sorted** | `sorted(L, key=lambda x: x.id)` | **Seule option simple (Lambda gagne)** |

---

## 3. Limitations et Lisibilité {#limitations-lisibilite}

### 1. Quoi
Les lambdas sont syntaxiquement restreintes à une **expression unique**.

### 2. Pourquoi
Guido van Rossum (créateur de Python) voulait empêcher l'obfuscation du code. Si une fonction nécessite plusieurs lignes, des variables temporaires ou des structures de contrôle (`if/else` complexes), elle mérite un nom et une définition `def` propre.

### 3. Comment

#### A. Ce qui est impossible
```python
# ❌ SyntaxError : Impossible de mettre des instructions (statements)
lambda x: x += 1 
lambda x: print(x); return x 
```

#### B. Complexité excessive
Même si c'est techniquement possible, n'écrivez pas ceci :

```python
# ❌ Illisible et difficile à maintenir
f = lambda x: "Grand" if x > 100 else ("Moyen" if x > 50 else "Petit")
```

Préférez toujours :
```python
def classifier_taille(x: int) -> str:
    if x > 100: return "Grand"
    if x > 50: return "Moyen"
    return "Petit"
```

### 🚨 Limitations de Lambda
*   **Pas de documentation** : Impossible d'ajouter une docstring.
*   **Debug difficile** : Dans une stack trace d'erreur, la fonction apparaîtra sous le nom `<lambda>`, ce qui ne vous aide pas à savoir laquelle a planté.
*   **Typage** : On ne peut pas annoter les types des arguments (`lambda x: int: ...` est invalide).

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-52}

1.  **Quel mot-clé est utilisé pour créer une fonction anonyme ?**
    `lambda`.

2.  **Une fonction lambda peut-elle contenir plusieurs instructions (lignes) ?**
    Non, elle est limitée à une seule expression évaluée et retournée.

3.  **Quel est le cas d'usage le plus courant et recommandé pour une lambda ?**
    Comme argument `key` dans les fonctions de tri (`sorted`, `min`, `max`).

4.  **Est-il nécessaire d'écrire `return` dans une lambda ?**
    Non, le retour est implicite.

---

## Exercices : {#exercices-52}

### Exercice 1 - Le Maître du Tri {#exercice-1-maitre-tri}

🎯 **Objectif** : Utiliser `lambda` pour trier une structure de données complexe.

💼 **Mise en situation** : Vous gérez une liste d'utilisateurs pour un SaaS. Chaque utilisateur a un nom et un score d'activité. Vous devez afficher le top 3 des utilisateurs les plus actifs.

📝 **Énoncé** :
1.  Soit la liste `users` ci-dessous.
2.  Utilisez `sorted()` avec une `lambda` pour trier par score **décroissant** (du plus grand au plus petit).
3.  Affichez les 3 premiers.

```python
users = [
    {"name": "Alice", "score": 120},
    {"name": "Bob", "score": 50},
    {"name": "Charlie", "score": 300},
    {"name": "David", "score": 210}
]
```

📺 **Résultat attendu** :
```text
[{'name': 'Charlie', 'score': 300}, {'name': 'David', 'score': 210}, {'name': 'Alice', 'score': 120}]
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
users = [
    {"name": "Alice", "score": 120},
    {"name": "Bob", "score": 50},
    {"name": "Charlie", "score": 300},
    {"name": "David", "score": 210}
]

# On trie sur la clé "score".
# reverse=True pour l'ordre décroissant.
top_users = sorted(users, key=lambda u: u["score"], reverse=True)

# Slicing pour prendre les 3 premiers
print(top_users[:3])
```
</details>

### Exercice 2 - Transformation de Données (Map) {#exercice-2-transformation-map}

🎯 **Objectif** : Transformer une liste avec `map` et `lambda`.

💼 **Mise en situation** : Vous recevez des prix en dollars, vous devez les convertir en euros (Taux : 0.92) et les formater en chaîne de caractères.

📝 **Énoncé** :
1.  Liste `prix_usd = [10.0, 55.5, 100.0]`.
2.  Utilisez `map` et une `lambda` pour multiplier par 0.92.
3.  Utilisez une deuxième `map` (ou intégrez-la) pour formater en string avec le symbole "€" (ex: "9.2 €").
4.  Convertissez le résultat final en liste.

📺 **Résultat attendu** :
`['9.2 €', '51.06 €', '92.0 €']`

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
prix_usd = [10.0, 55.5, 100.0]

# On peut tout faire en une seule lambda, mais décomposons pour l'exercice
# Conversion
prix_eur = map(lambda p: p * 0.92, prix_usd)

# Formatage (f-string dans une lambda)
prix_format = map(lambda p: f"{p} €", prix_eur)

# Exécution du map (lazy evaluation) via list()
print(list(prix_format))

# Version "One-Liner" (pour les curieux) :
# print(list(map(lambda p: f"{p * 0.92} €", prix_usd)))
```
</details>

### Exercice 3 - Le Filtre de Sécurité {#exercice-3-filtre-securite}

🎯 **Objectif** : Filtrer avec une condition composée.

💼 **Mise en situation** : Nettoyage d'une base de logs. Vous ne voulez garder que les logs de niveau "ERROR" ou "CRITICAL".

📝 **Énoncé** :
1.  Liste de logs (tuples) : `logs = [("INFO", "Start"), ("ERROR", "Db fail"), ("WARN", "Slow"), ("CRITICAL", "Fire")]`.
2.  Utilisez `filter` et une `lambda` pour ne garder que les tuples dont le premier élément est dans la liste `["ERROR", "CRITICAL"]`.

📺 **Résultat attendu** :
`[('ERROR', 'Db fail'), ('CRITICAL', 'Fire')]`

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
logs = [("INFO", "Start"), ("ERROR", "Db fail"), ("WARN", "Slow"), ("CRITICAL", "Fire")]

severity_kept = ["ERROR", "CRITICAL"]

# La lambda vérifie si le niveau (log[0]) est dans la liste autorisée
logs_critiques = list(filter(lambda log: log[0] in severity_kept, logs))

print(logs_critiques)
```
</details>