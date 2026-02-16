---
sidebar_label: Structures de Données : Listes
sidebar_position: 8
---

# Chapitre 8 : Structures de Données : Listes

Création, Accès aux éléments, Modification, Méthodes de liste

Jusqu'à présent, nos variables ne stockaient qu'une seule valeur à la fois (un nombre, une chaîne). Mais dans la vraie vie, nous manipulons des **collections** : une liste de tâches, un panier d'achats, un tableau de scores.

La **liste (`list`)** est la structure de données la plus polyvalente de Python. Elle est ordonnée, modifiable (mutable) et peut contenir n'importe quel type d'objet.

---

## 1. Création et Structure {#creation-et-structure}

### 1. Quoi
Une liste est une séquence ordonnée d'éléments séparés par des virgules et entourés de crochets `[]`.

### 2. Pourquoi
Pour regrouper des données liées entre elles et les manipuler comme un tout unique.

### 3. Comment

#### A. Syntaxe de base

```python
# Liste vide
empty_list: list = []

# Liste homogène (même type)
numbers: list[int] = [1, 2, 3, 4, 5]

# Liste hétérogène (types variés - moins fréquent en typage strict)
mixed: list = [1, "Python", 3.14, True]
```

#### B. Cas concret : Panier d'achat
En pratique, on utilise souvent des listes d'objets ou de dictionnaires (que nous verrons plus tard), mais pour l'instant, restons sur des types simples.

```python
shopping_cart: list[str] = ["Pommes", "Bananes", "Lait", "Pain"]
print(shopping_cart)
# ['Pommes', 'Bananes', 'Lait', 'Pain']
```

#### C. Listes imbriquées (Matrices)
Une liste peut contenir d'autres listes.

```python
matrix: list[list[int]] = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
]
```

### 4. Zone de Danger
❌ **Nommer une liste `list`** :
Ne faites jamais `list = [1, 2, 3]`. Cela écrase le type `list` natif de Python et vous empêchera de l'utiliser ailleurs. Préférez `my_list`, `items` ou un nom pluriel (`users`, `scores`).

---

## 2. Accès et Modification (Indexing/Slicing) {#acces-et-modification}

### 1. Quoi
Comme pour les chaînes de caractères, on accède aux éléments par leur position (**index**), commençant à 0. Contrairement aux chaînes, on peut modifier les éléments.

### 2. Comment

#### A. Lecture par index

```python
colors: list[str] = ["Rouge", "Vert", "Bleu"]

first: str = colors[0]   # "Rouge"
last: str = colors[-1]   # "Bleu" (Index négatif = depuis la fin)
```

#### B. Modification (Mutabilité)

```python
colors[1] = "Jaune"
print(colors) # ['Rouge', 'Jaune', 'Bleu']
```

#### C. Slicing (Sous-listes)
Extraction d'une portion de la liste : `liste[début:fin:pas]`.

```python
days: list[str] = ["Lun", "Mar", "Mer", "Jeu", "Ven", "Sam", "Dim"]

week_days: list[str] = days[0:5]  # ['Lun', 'Mar', 'Mer', 'Jeu', 'Ven']
weekend: list[str] = days[5:]     # ['Sam', 'Dim']
reversed_days: list[str] = days[::-1] # Copie inversée
```

### 4. Zone de Danger
❌ **IndexError** : Tenter d'accéder à `days[10]` alors que la liste n'a que 7 éléments plantera le programme. Vérifiez toujours la taille avec `len(days)` si l'index est dynamique.

---

## 3. Ajout et Suppression d'éléments {#ajout-et-suppression}

### 1. Quoi
Les listes sont dynamiques : leur taille peut changer au cours de l'exécution.

### 2. Pourquoi
Pour empiler des résultats, traiter une file d'attente, ou filtrer des données.

### 3. Comment

#### A. Ajouter

*   `append(x)` : Ajoute `x` à la fin.
*   `insert(i, x)` : Insère `x` à l'index `i`.
*   `extend(iterable)` : Fusionne une autre liste à la fin.

```python
todos: list[str] = ["Dormir"]

todos.append("Coder")       # ['Dormir', 'Coder']
todos.insert(0, "Manger")   # ['Manger', 'Dormir', 'Coder']

others: list[str] = ["Lire", "Sortir"]
todos.extend(others)        # ['Manger', 'Dormir', 'Coder', 'Lire', 'Sortir']
```

#### B. Supprimer

*   `remove(x)` : Supprime la **première** occurrence de la valeur `x`.
*   `pop(i)` : Supprime et **retourne** l'élément à l'index `i` (par défaut le dernier).
*   `del liste[i]` : Instruction générique de suppression.
*   `clear()` : Vide toute la liste.

```python
stack: list[int] = [10, 20, 30, 20]

stack.remove(20)    # [10, 30, 20] (Seul le premier 20 part)

last_item = stack.pop() # last_item = 20, stack = [10, 30]

del stack[0]        # stack = [30]
stack.clear()       # stack = []
```

### 4. Zone de Danger
❌ **Modifier une liste en itérant dessus** :
Ne supprimez jamais d'éléments d'une liste pendant que vous bouclez dessus avec un `for`. Cela décale les index et saute des éléments. Créez plutôt une nouvelle liste ou itérez sur une copie.

---

## 4. Recherche et Tri {#recherche-et-tri}

### 1. Quoi
Outils pour organiser et interroger les données.

### 2. Comment

#### A. Recherche

```python
guests: list[str] = ["Alice", "Bob", "Charlie"]

# Vérifier la présence
if "Bob" in guests:
    print("Bob est invité.")

# Trouver l'index
position: int = guests.index("Charlie") # 2
```

#### B. Tri
*   `sort()` : Trie la liste **sur place** (modifie l'originale).
*   `sorted()` : Crée une **nouvelle** liste triée (ne touche pas l'originale).

```python
scores: list[int] = [5, 1, 9, 3]

# Méthode non destructive (souvent préférée)
sorted_scores: list[int] = sorted(scores) # [1, 3, 5, 9]
print(scores) # [5, 1, 9, 3] (Intact)

# Méthode destructive
scores.sort() # scores devient [1, 3, 5, 9]
scores.sort(reverse=True) # [9, 5, 3, 1]
```

### 🚨 Limitations de `sort()`
`sort()` ne retourne rien (`None`).
Ne faites pas `my_list = my_list.sort()`, sinon vous perdez vos données !

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-8}

1.  **Quelle est la différence entre `append()` et `extend()` ?**
    `append` ajoute son argument comme *un seul élément* (même si c'est une liste), alors que `extend` ajoute *chacun des éléments* de la séquence fournie.

2.  **Les listes sont-elles mutables ?**
    Oui. On peut changer, ajouter ou supprimer des éléments après la création de la liste.

3.  **Comment accéder au dernier élément d'une liste sans connaître sa taille ?**
    En utilisant l'index négatif : `liste[-1]`.

4.  **Que se passe-t-il si on fait `my_list.sort()` ?**
    La liste est triée "in-place" (modifiée directement). La méthode ne renvoie aucune valeur (`None`).

---

## Exercices : {#exercices-8}

### Exercice 1 - La Playlist Musicale {#exercice-1---playlist}

🎯 **Objectif** : Manipulation basique (ajout, retrait, accès).

💼 **Mise en situation** : Vous gérez la file d'attente d'un jukebox collaboratif.

📝 **Énoncé** :
1.  Créez une liste `playlist` vide.
2.  Ajoutez "Bohemian Rhapsody" et "Stairway to Heaven".
3.  Insérez "Imagine" au tout début de la liste.
4.  Le premier morceau vient d'être joué : supprimez-le (pop) et affichez "Lecture de : [Titre]".
5.  Affichez la playlist restante.

📺 **Résultat attendu** :
```text
Lecture de : Imagine
Reste : ['Bohemian Rhapsody', 'Stairway to Heaven']
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
playlist: list[str] = []

# Ajout à la fin
playlist.append("Bohemian Rhapsody")
playlist.append("Stairway to Heaven")

# Insertion au début (index 0)
playlist.insert(0, "Imagine")

# Lecture et suppression du premier
current_song: str = playlist.pop(0)
print(f"Lecture de : {current_song}")

print(f"Reste : {playlist}")
```
</details>

### Exercice 2 - Analyse de Notes {#exercice-2---analyse-notes}

🎯 **Objectif** : Utiliser `min`, `max`, `sum` et `len` sur une liste.

💼 **Mise en situation** : Un professeur veut des statistiques rapides sur les notes de sa classe.

📝 **Énoncé** :
1.  Soit la liste `grades = [12, 15, 8, 19, 10, 14]`.
2.  Ajoutez une note oubliée : 13.
3.  Calculez la moyenne de la classe (somme / nombre d'élèves).
4.  Affichez la meilleure note, la pire note et la moyenne (arrondie à 2 décimales).

📺 **Résultat attendu** :
```text
Min : 8
Max : 19
Moyenne : 13.0
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
grades: list[int] = [12, 15, 8, 19, 10, 14]

# Ajout
grades.append(13)

# Calculs statistiques
best_grade: int = max(grades)
worst_grade: int = min(grades)

# Moyenne = Somme totale / Nombre d'éléments
average: float = sum(grades) / len(grades)

print(f"Min : {worst_grade}")
print(f"Max : {best_grade}")
# Formatage de la moyenne (chiffre rond ici, mais bon réflexe à avoir)
print(f"Moyenne : {round(average, 2)}")
```
</details>

### Exercice 3 - Le Podium Olympique {#exercice-3---podium}

🎯 **Objectif** : Tri et Slicing.

💼 **Mise en situation** : Vous recevez les temps d'arrivée d'une course en désordre. Vous devez afficher le podium (les 3 meilleurs temps).

📝 **Énoncé** :
1.  Liste `times = [10.5, 9.8, 11.2, 9.9, 10.0, 12.1]`.
2.  Triez la liste du plus rapide (petit chiffre) au plus lent.
3.  Extrayez les 3 premiers éléments dans une nouvelle liste `podium`.
4.  Affichez "Podium : [t1, t2, t3]".

📺 **Résultat attendu** :
```text
Podium : [9.8, 9.9, 10.0]
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
times: list[float] = [10.5, 9.8, 11.2, 9.9, 10.0, 12.1]

# Tri en place (modifie 'times')
times.sort()
# Note : Pour les temps de course, plus petit = meilleur, donc tri ascendant par défaut

# Slicing des 3 premiers
podium: list[float] = times[:3]

print(f"Podium : {podium}")
```
</details>