---
sidebar_label: Générateurs et Itérateurs
sidebar_position: 40
---

# Chapitre 40 : Générateurs et Itérateurs

Mot-clé yield, Objets itérateurs, Protocoles d'itération, Avantages des générateurs

L'itération est au cœur de Python. Quand vous faites une boucle `for`, vous utilisez sans le savoir le puissant protocole d'itération. Mais que faire si vous devez traiter un fichier de 50 Go ligne par ligne, ou calculer une suite infinie de nombres ? Charger tout cela en mémoire dans une liste ferait planter votre programme instantanément.

C'est là qu'interviennent les **Générateurs**. Ils permettent de produire des données "à la demande" (lazy evaluation), une par une, sans jamais tout stocker en mémoire. C'est l'outil secret des développeurs Python pour écrire du code performant et scalable.

---

## 1. Le Protocole d'Itération : Itérables et Itérateurs {#le-protocole-d-iteration}

### 1. Quoi
En Python, deux concepts distincts permettent aux boucles de fonctionner :
*   **Itérable** : Tout objet capable de renvoyer ses membres un par un (Listes, Tuples, Strings, Dictionnaires). Il possède une méthode `__iter__()`.
*   **Itérateur** : L'objet "curseur" qui effectue le travail de traversée. Il possède une méthode `__next__()` qui renvoie l'élément suivant ou lève l'exception `StopIteration` à la fin.

### 2. Pourquoi
Comprendre cette mécanique permet de créer vos propres objets compatibles avec `for`, et de manipuler des flux de données infinis ou très volumineux.

### 3. Comment

#### A. La mécanique sous le capot (ce que fait `for`)

```python
# Un itérable simple
fruits = ["Pomme", "Banane", "Cerise"]

# 1. Obtenir l'itérateur à partir de l'itérable
iterator = iter(fruits) 
# iterator est maintenant un objet prêt à délivrer des fruits un par un

# 2. Consommer manuellement
print(next(iterator)) # Pomme
print(next(iterator)) # Banane
print(next(iterator)) # Cerise

# 3. La fin
try:
    print(next(iterator))
except StopIteration:
    print("Plus de fruits !")
```

#### B. Créer son propre Itérateur (Approche POO classique)
C'est verbeux, mais instructif pour comprendre le protocole.

```python
class CompteArebours:
    def __init__(self, start):
        self.current = start

    def __iter__(self):
        return self # Un itérateur est son propre itérable

    def __next__(self):
        if self.current <= 0:
            raise StopIteration
        val = self.current
        self.current -= 1
        return val

# Utilisation
for n in CompteArebours(3):
    print(n)
# Affiche 3, 2, 1
```

---

## 2. Les Générateurs et le mot-clé `yield` {#les-generateurs-et-yield}

### 1. Quoi
Un générateur est une **fonction** qui ressemble à une fonction normale, mais qui utilise `yield` au lieu de `return`.
Quand `yield` est exécuté, la fonction :
1.  Renvoie la valeur.
2.  Met son exécution en **pause**, en sauvegardant tout son état (variables locales, ligne de code).
3.  Reprendra exactement là où elle s'est arrêtée au prochain appel de `next()`.

### 2. Pourquoi
Pour simplifier drastiquement l'écriture d'itérateurs. Plus besoin de créer une classe avec `__iter__` et `__next__`. C'est aussi l'outil idéal pour gérer la mémoire (Memory Efficiency).

### 3. Comment

#### A. Syntaxe de base

```python
def simple_generator():
    yield 1
    print("On reprend ici...")
    yield 2
    print("Encore une fois...")
    yield 3

gen = simple_generator()
# Rien ne se passe encore ! La fonction ne démarre pas à l'appel.

print(next(gen)) 
# Sortie: 1
# La fonction est en pause ligne 2.

print(next(gen)) 
# Sortie: 
# On reprend ici...
# 2
```

#### B. Générateur Infini (Exemple concret)
Impossible avec une liste classique (mémoire saturée), trivial avec un générateur.

```python
def id_generator(prefix="USER"):
    num = 1
    while True: # Boucle infinie
        yield f"{prefix}_{num:04d}"
        num += 1

gen = id_generator()

print(next(gen)) # USER_0001
print(next(gen)) # USER_0002
# On peut en générer des milliards sans jamais saturer la RAM.
```

#### D. Comparatif : Fonction vs Générateur

| Aspect | Fonction Classique (`return`) | Générateur (`yield`) |
| :--- | :--- | :--- |
| **Exécution** | S'exécute entièrement puis renvoie 1 résultat | S'exécute par morceaux, renvoie N résultats |
| **État** | Perd ses variables locales après `return` | Conserve ses variables entre deux `yield` |
| **Mémoire** | Stocke tout le résultat (ex: liste) | Ne stocke que l'élément courant |
| **Usage** | Calculs finis | Flux de données, grands fichiers, streams |

---

## 3. Les Expressions Génératrices {#les-expressions-generatrices}

### 1. Quoi
C'est la version "une ligne" (lambda-like) des générateurs. Syntaxiquement identique aux list comprehensions, mais avec des parenthèses `()` au lieu de crochets `[]`.

### 2. Pourquoi
Pour des transformations rapides et légères sans définir une fonction `def` complète.

### 3. Comment

```python
# Liste (Gourmand en RAM)
# squares_list = [x**2 for x in range(1000000)]

# Générateur (RAM constante, quasi nulle)
squares_gen = (x**2 for x in range(1000000))

print(squares_gen) # <generator object ...>

# On peut itérer dessus
total = sum(squares_gen) 
print(total)
```

### 4. Zone de Danger
❌ **Réutilisation** : Un générateur est à **usage unique**. Une fois consommé, il est vide. Vous ne pouvez pas itérer deux fois dessus.

```python
gen = (x for x in range(3))
list(gen) # [0, 1, 2]
list(gen) # [] -> VIDE ! Il faut recréer le générateur.
```

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-40}

1.  **Quelle est la différence fondamentale entre `return` et `yield` ?**
    `return` termine la fonction et détruit son contexte local. `yield` renvoie une valeur mais met la fonction en pause, conservant tout son état pour le prochain appel.

2.  **Comment un générateur gère-t-il la mémoire différemment d'une liste ?**
    Une liste stocke tous les éléments en mémoire vive. Un générateur calcule et produit un seul élément à la fois, ne consommant de la mémoire que pour cet élément courant.

3.  **Que se passe-t-il si on appelle `next()` sur un générateur qui a fini son travail ?**
    Il lève l'exception `StopIteration` (c'est le signal pour la boucle `for` de s'arrêter).

4.  **Peut-on accéder au 10ème élément d'un générateur directement (ex: `gen[9]`) ?**
    Non. Les générateurs n'ont pas d'accès aléatoire (random access). Il faut itérer 9 fois pour atteindre le 10ème élément.

---

## Exercices : {#exercices-40}

### Exercice 1 - Le Lecteur de Logs Géants {#exercice-1-lecteur-logs}

🎯 **Objectif** : Créer un générateur pour filtrer un flux de données.

💼 **Mise en situation** : Vous devez analyser un fichier de log de 20 Go. Vous cherchez les lignes contenant "ERROR". Charger le fichier en mémoire est impossible.

📝 **Énoncé** :
1.  Simulez un fichier de log avec une liste de chaînes (imaginez qu'elle est immense).
2.  Créez une fonction génératrice `filter_errors(logs)` qui prend un itérable de lignes.
3.  Elle doit `yield` uniquement les lignes contenant le mot "ERROR".
4.  Utilisez une boucle pour afficher les résultats.

📺 **Résultat attendu** :
```text
Ligne trouvée : 2023-10-01 ERROR Connection lost
Ligne trouvée : 2023-10-02 ERROR Database timeout
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
# Simulation d'un flux de données (pourrait être un fichier ouvert)
fake_huge_file = [
    "2023-10-01 INFO Starting",
    "2023-10-01 ERROR Connection lost",
    "2023-10-02 INFO Processing",
    "2023-10-02 ERROR Database timeout",
    "2023-10-03 INFO Done"
]

def filter_errors(lines):
    """Générateur qui filtre les lignes à la volée."""
    for line in lines:
        if "ERROR" in line:
            # On met en pause ici et on renvoie la ligne
            yield line

# Utilisation : aucune liste n'est créée en mémoire
# On branche le générateur sur la source
error_gen = filter_errors(fake_huge_file)

for error in error_gen:
    print(f"Ligne trouvée : {error}")
```
</details>

### Exercice 2 - La Suite de Fibonacci Infinie {#exercice-2-fibonacci}

🎯 **Objectif** : Implémenter un algorithme mathématique sous forme de générateur infini.

💼 **Mise en situation** : Vous avez besoin de nombres de la suite de Fibonacci (0, 1, 1, 2, 3, 5, 8...) pour un algorithme de cryptographie, mais vous ne savez pas à l'avance combien il en faudra.

📝 **Énoncé** :
1.  Créez un générateur `fibonacci()`.
2.  Il doit produire la suite indéfiniment.
3.  Dans le programme principal, utilisez `next()` pour afficher les 10 premiers nombres, ou une boucle avec un `break` si la valeur dépasse 100.

📺 **Résultat attendu** :
```text
0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, Stop (>100)
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
def fibonacci():
    """Génère la suite de Fibonacci à l'infini."""
    a, b = 0, 1
    while True:
        yield a
        # Calcul du terme suivant
        a, b = b, a + b

gen = fibonacci()

print("Suite de Fibonacci jusqu'à 100 :")
for num in gen:
    if num > 100:
        print("Stop (>100)")
        break
    print(num, end=", ")
```
</details>

### Exercice 3 - Pipeline de Traitement de Données {#exercice-3-pipeline}

🎯 **Objectif** : Chaîner plusieurs générateurs (Pipelining).

💼 **Mise en situation** : Traitement de données SaaS. 
Source -> Convertir en Entier -> Carré du nombre -> Convertir en String.
Vous voulez construire ce pipeline de manière modulaire.

📝 **Énoncé** :
1.  Source : `data = ["1", "2", "3", "4", "5"]` (chaînes).
2.  Générateur 1 : Convertit en `int`.
3.  Générateur 2 : Prend le flux du Gen 1 et calcule le carré.
4.  Générateur 3 : Prend le flux du Gen 2 et formate en chaîne `"Value: X"`.
5.  Affichez le résultat final en itérant sur le dernier générateur.

📺 **Résultat attendu** :
```text
Value: 1
Value: 4
Value: 9
Value: 16
Value: 25
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
data = ["1", "2", "3", "4", "5"]

# Pipeline utilisant des expressions génératrices (plus concis que def)

# Étape 1 : Conversion str -> int
integers = (int(x) for x in data)

# Étape 2 : Calcul (consomme integers)
squares = (x * x for x in integers)

# Étape 3 : Formatage (consomme squares)
formatted = (f"Value: {x}" for x in squares)

# Exécution du pipeline
# Rien n'a été calculé avant cette boucle !
for item in formatted:
    print(item)
```
</details>