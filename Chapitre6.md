---
sidebar_label: Structures de Données : Listes
sidebar_position: 6
---

# Chapitre 6 : Structures de Données : Listes

Création, Manipulation, Indexation, Slicing, Méthodes natives

Les **listes** sont la structure de données la plus fondamentale et la plus polyvalente de Python. Ce sont des séquences ordonnées et **mutables** (modifiables) qui peuvent contenir n'importe quel type d'objet.

## 1. Création et Accès Basique {#creation-et-acces-basique}

### 1. Quoi
Une liste est une collection d'éléments séparés par des virgules et entourés de crochets `[]`. Contrairement aux tableaux (arrays) dans d'autres langages (comme C ou Java), une liste Python peut contenir des éléments de types différents (hétérogènes), bien qu'en pratique, on stocke souvent des données de même type.

### 2. Pourquoi
Dans presque tous les programmes, vous aurez besoin de regrouper des données :
- Une liste de produits dans un panier d'achat.
- Une liste d'utilisateurs connectés.
- Une liste de lignes lues depuis un fichier CSV.

La liste permet de manipuler cet ensemble comme une seule variable tout en gardant l'ordre d'insertion.

### 3. Comment

#### A. Syntaxe de base

```python
# Liste vide
empty_list: list = []

# Liste homogène (recommandé)
users: list[str] = ["Alice", "Bob", "Charlie"]

# Liste hétérogène (possible mais moins courant en typage strict)
mixed_data = [1, "Python", 3.14, True]
```

#### B. Accès aux éléments (Indexation)

Python utilise une **indexation base-0** (le premier élément est à l'index 0). Il supporte aussi l'**indexation négative** pour accéder aux éléments depuis la fin.

```python
products: list[str] = ["Laptop", "Mouse", "Keyboard", "Monitor"]

# Premier élément
first = products[0]  # "Laptop"

# Dernier élément (Très pythonique !)
last = products[-1]  # "Monitor"

# Avant-dernier
second_to_last = products[-2] # "Keyboard"
```

#### C. Exemples Pratiques

**Cas 1 : Panier E-commerce**
```python
cart_items: list[str] = ["Apple Watch", "MacBook Pro", "USB-C Cable"]
print(f"Article principal : {cart_items[0]}") 
# Affiche: Article principal : Apple Watch
```

**Cas 2 : Pile d'exécution (Logs)**
```python
error_logs: list[str] = [
    "Error 404: Page not found",
    "Error 500: Internal Server Error"
]
# Récupérer la dernière erreur survenue
latest_error = error_logs[-1]
```

### 4. Zone de Danger

❌ **Erreur fréquente : IndexError**
Accéder à un index qui n'existe pas lève une erreur bloquante.

```python
users = ["Alice", "Bob"]
# print(users[2])  # ❌ IndexError: list index out of range (car 0, 1 sont les seuls index)
```

---

## 2. Le Slicing (Découpage) {#le-slicing-decoupage}

### 1. Quoi
Le **slicing** permet d'extraire une sous-partie d'une liste en créant une *nouvelle* liste. C'est une fonctionnalité signature de Python.

### 2. Pourquoi
Pour traiter des lots de données, faire de la pagination, ou inverser des séquences sans boucles complexes.
*Exemple : Afficher les 10 premiers résultats d'une recherche.*

### 3. Comment

Syntaxe : `liste[start:stop:step]`
- `start` : Index de début (inclus). Défaut = 0.
- `stop` : Index de fin (**exclus**). Défaut = fin de la liste.
- `step` : Pas (optionnel). Défaut = 1.

#### A. Exemples de Slicing

```python
numbers: list[int] = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

# Du début à l'index 5 (exclus) -> 5 premiers éléments
first_five = numbers[:5]  # [0, 1, 2, 3, 4]

# De l'index 5 jusqu'à la fin
from_five = numbers[5:]   # [5, 6, 7, 8, 9]

# Tranche du milieu
middle = numbers[3:6]     # [3, 4, 5] (indices 3, 4, 5)

# Un élément sur deux (Step = 2)
evens = numbers[::2]      # [0, 2, 4, 6, 8]

# Inverser une liste (Idiome très connu)
reversed_list = numbers[::-1] # [9, 8, 7, ..., 0]
```

### 4. Zone de Danger

✅ **Bonne pratique** : Le slicing ne lève jamais d'`IndexError`. Si les bornes sont hors limites, Python renvoie gentiment une liste vide ou tronquée.

```python
data = [1, 2, 3]
print(data[10:20]) # ✅ Affiche [] (pas d'erreur)
```

---

## 3. Modification et Méthodes {#modification-et-methodes}

### 1. Quoi
Les listes sont des objets riches possédant des méthodes intégrées pour ajouter, supprimer, trier et chercher des éléments.

### 2. Pourquoi
Ne réinventez pas la roue. Les méthodes natives sont optimisées en C (CPython) et sont beaucoup plus rapides que d'écrire vos propres boucles pour ces tâches.

### 3. Comment

#### A. Ajouter des éléments

```python
todo_list: list[str] = ["Email client", "Call boss"]

# 1. append(x) : Ajoute un élément à la fin (Opération O(1) - très rapide)
todo_list.append("Coffee break") 
# -> ["Email client", "Call boss", "Coffee break"]

# 2. insert(i, x) : Insère à une position spécifique
todo_list.insert(0, "Wake up") 
# -> ["Wake up", "Email client", "Call boss", "Coffee break"]

# 3. extend(iterable) : Fusionne une autre liste à la fin
more_tasks = ["Go home", "Sleep"]
todo_list.extend(more_tasks)
# -> ["Wake up", ..., "Go home", "Sleep"]
```

#### B. Supprimer des éléments

```python
stack = ["A", "B", "C", "D"]

# 1. pop() : Retire et RETOURNE le dernier élément (utile pour les piles LIFO)
last_item = stack.pop()  # Retourne "D", stack devient ["A", "B", "C"]
specific_item = stack.pop(0) # Retourne "A", stack devient ["B", "C"]

# 2. remove(x) : Cherche et retire la PREMIÈRE occurrence de la valeur
stack.remove("B") # stack devient ["C"]
# ⚠️ Lève ValueError si "B" n'est pas trouvé !

# 3. clear() : Vide toute la liste
stack.clear() # []
```

#### C. Trier et Chercher

```python
prices = [99.99, 10.00, 50.50]

# Trier en place (modifie la liste originale !)
prices.sort() 
# prices est maintenant [10.00, 50.50, 99.99]

# Trier sans modifier l'original (fonction built-in sorted)
sorted_prices = sorted(prices, reverse=True)

# Compter les occurrences
count_10 = prices.count(10.00) # 1

# Trouver l'index
idx_50 = prices.index(50.50) # 1
```

### 4. Zone de Danger

❌ **Confusion "In-place" vs "Return"**
La méthode `.sort()` modifie la liste et retourne `None`.

```python
my_list = [3, 1, 2]
sorted_list = my_list.sort() # ❌ ERREUR CLASSIQUE

print(sorted_list) # Affiche: None
print(my_list)     # Affiche: [1, 2, 3] (La liste originale a changé)
```

✅ **Solution** : Si vous voulez garder la liste originale intacte, utilisez `sorted(my_list)`.

---

## 4. Pièges de la Mutabilité (Référence vs Copie) {#pieges-de-la-mutabilite}

### 1. Quoi
En Python, assigner une liste à une nouvelle variable ne copie pas les données, cela copie la **référence** (l'adresse mémoire).

### 2. Pourquoi
C'est efficace en mémoire, mais dangereux si on ne s'y attend pas.

### 3. Comment (Le problème)

```python
original = [1, 2, 3]
copie = original # ⚠️ Ce n'est PAS une vraie copie, c'est un alias

copie[0] = 999

print(original) # Affiche [999, 2, 3] -> L'original a été modifié !
```

### 4. Comment (La solution)

Pour travailler sur une copie indépendante :

```python
# Méthode explicite (Python 3.3+)
vraie_copie = original.copy()

# Méthode via slicing (Old school mais courant)
vraie_copie_2 = original[:]

# Modification sans danger
vraie_copie[0] = 555
print(original) # Affiche toujours [999, 2, 3]
```

### 🚨 Limitations de `.copy()`
`.copy()` effectue une **Shallow Copy** (copie superficielle).
- Si la liste contient d'autres listes (objets imbriqués), seules les références de ces objets sont copiées.
- Une modification *profonde* (dans une sous-liste) affectera les deux variables.
- Pour une **Deep Copy** (Chapitre 37), il faut utiliser `import copy`.

---

## 5. Cas Pratiques Réels {#cas-pratiques-reels}

### A. File de Traitement (SaaS)

Simule une file d'attente FIFO (First-In, First-Out) pour des traitements asynchrones.

```python
processing_queue: list[str] = []

# Nouveaux utilisateurs s'inscrivent
processing_queue.append("user_42@example.com")
processing_queue.append("user_99@example.com")

# Traiter le premier
user_to_process = processing_queue.pop(0) 
# Traitement de "user_42@example.com"...

print(f"Reste à traiter : {len(processing_queue)}")
```
> ⚠️ Note : Pour de très grandes files FIFO, `collections.deque` (Chapitre 30) est bien plus performant que `list.pop(0)`.

### B. Pagination de Résultats (API)

Utiliser le slicing pour renvoyer une "page" de données.

```python
all_posts: list[str] = [f"Post #{i}" for i in range(1, 101)] # 100 posts fictifs

# Page 1 (0-10)
page_1 = all_posts[:10] 
# Page 2 (10-20)
page_2 = all_posts[10:20]

print(f"Page 2 contient : {page_2}")
```

### C. Gestion d'État (Jeu Vidéo / App)

Gérer un historique d'actions pour un "Undo/Redo".

```python
history: list[str] = []

# Action utilisateur
history.append("MOVE_UP")
history.append("ATTACK")

# Annuler dernière action (Undo)
if history:
    undone_action = history.pop() 
    print(f"Annulation de : {undone_action}") 
    # Reste ["MOVE_UP"]
```

---

## Questions clés (validation des acquis du chapitre) {#questions-clés-(validation-des-acquis-du-chapitre)-6}

1. **Comment récupérer le dernier élément d'une liste sans connaître sa longueur ?**
   - Réponse : `ma_liste[-1]`

2. **Quelle est la différence entre `ma_liste.sort()` et `sorted(ma_liste)` ?**
   - Réponse : `sort()` trie la liste **en place** et retourne `None`. `sorted()` crée une **nouvelle liste** triée et laisse l'originale inchangée.

3. **Si je fais `b = a`, modifier `b` modifie-t-il `a` ?**
   - Réponse : **Oui**, car `b` est une référence vers le même objet en mémoire que `a`. Utilisez `b = a.copy()` pour éviter cela.

4. **Quelle méthode utiliser pour ajouter un élément à une position précise (par ex: au début) ?**
   - Réponse : `ma_liste.insert(index, element)`.

5. **Que se passe-t-il si j'accède à `ma_liste[10]` alors qu'elle ne contient que 3 éléments ?**
   - Réponse : Une exception `IndexError: list index out of range` est levée.

---

## Exercices : {#exercices-:-6}

### Exercice 1 - Le Tableau Kanban (Startup) {#exercice-1---le-tableau-kanban}

🎯 **Objectif** : Manipuler les listes comme des piles/files et déplacer des éléments.

💼 **Mise en situation** :
Vous développez un outil de gestion de projet type Trello pour une startup. Vous devez gérer le flux des tâches entre les colonnes "À Faire", "En Cours" et "Terminé".

📝 **Énoncé** :
1. Créez une liste `todo` avec les tâches : "Maquette UI", "Backend API", "Tests E2E".
2. Créez deux listes vides : `in_progress` et `done`.
3. Une tâche est prise en charge : déplacez la **première** tâche de `todo` vers `in_progress`.
4. Une autre tâche est prise : déplacez la **nouvelle première** tâche de `todo` vers `in_progress`.
5. La première tâche prise est terminée : déplacez la **dernière** tâche ajoutée à `in_progress` vers `done`.
6. Affichez l'état final des 3 listes.

📺 **Résultat attendu** :
```text
Todo: ['Tests E2E']
In Progress: ['Maquette UI']
Done: ['Backend API']
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
# 1. État initial
todo: list[str] = ["Maquette UI", "Backend API", "Tests E2E"]
in_progress: list[str] = []
done: list[str] = []

# 3. Déplacer "Maquette UI" (index 0) vers in_progress
task1 = todo.pop(0)       # Retire l'élément à l'index 0
in_progress.append(task1) # Ajoute à la fin de la nouvelle liste

# 4. Déplacer "Backend API" (devenu index 0) vers in_progress
task2 = todo.pop(0)
in_progress.append(task2)

# 5. Déplacer la dernière tâche entrée ("Backend API") vers done
# On utilise pop() sans argument pour prendre le dernier élément
finished_task = in_progress.pop() 
done.append(finished_task)

# 6. Affichage
print(f"Todo: {todo}")
print(f"In Progress: {in_progress}")
print(f"Done: {done}")
```
</details>

---

### Exercice 2 - Analyse de Prix (E-commerce) {#exercice-2---analyse-de-prix}

🎯 **Objectif** : Utiliser les méthodes de recherche, tri et slicing.

💼 **Mise en situation** :
Votre site e-commerce a récupéré les prix d'un produit chez différents concurrents. Vous devez nettoyer ces données pour afficher le "Meilleur Prix" et le "Prix Médian".

📝 **Énoncé** :
1. Soit la liste brute `prices = [120, 50, 80, 50, 110, 120, 90]`.
2. Le prix `50` est une erreur (trop bas), supprimez **toutes** ses occurrences (astuce : utilisez `while` ou `remove` intelligemment, ou recréez la liste). *Note : Comme nous n'avons pas encore vu les boucles `while` complexes, utilisez `remove` deux fois manuellement pour cet exercice.*
3. Triez la liste propre du plus petit au plus grand.
4. Récupérez le prix minimum (le premier de la liste triée).
5. Récupérez les 3 prix les plus chers (slicing depuis la fin).

📺 **Résultat attendu** :
```text
Prix nettoyés et triés : [80, 90, 110, 120, 120]
Meilleur prix : 80
Top 3 les plus chers : [110, 120, 120]
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
prices = [120, 50, 80, 50, 110, 120, 90]

# 2. Suppression des erreurs (50)
# Note: remove() ne supprime que la première occurrence trouvée
prices.remove(50) 
prices.remove(50)

# 3. Tri en place
prices.sort()
print(f"Prix nettoyés et triés : {prices}")

# 4. Meilleur prix (le premier après tri)
best_price = prices[0]
print(f"Meilleur prix : {best_price}")

# 5. Les 3 plus chers (les 3 derniers)
# Slicing : commence à -3 (3e avant la fin) jusqu'à la fin
most_expensive = prices[-3:] 
print(f"Top 3 les plus chers : {most_expensive}")
```
</details>

---

### Exercice 3 - Rotation de Logs (DevOps) {#exercice-3---rotation-de-logs}

🎯 **Objectif** : Slicing avancé et gestion de la taille d'une liste.

💼 **Mise en situation** :
Un système de logs ne doit conserver que les 5 dernières entrées pour économiser la mémoire.

📝 **Énoncé** :
1. Créez une liste `logs` contenant 8 messages : `"Log 1"`, `"Log 2"`, ..., `"Log 8"`.
2. Simulez une "rotation" : conservez uniquement les 5 **derniers** éléments de la liste. (Indice : Slicing avec index négatif ou positif).
3. Ajoutez un nouveau log : `"Log 9"`.
4. Refaites la rotation pour ne garder que les 5 derniers (donc `"Log 5"` à `"Log 9"`).
5. Affichez la liste finale inversée (du plus récent au plus ancien) pour l'affichage dashboard.

📺 **Résultat attendu** :
```text
Logs après première rotation : ['Log 4', 'Log 5', 'Log 6', 'Log 7', 'Log 8']
Logs finaux (récent -> ancien) : ['Log 9', 'Log 8', 'Log 7', 'Log 6', 'Log 5']
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
# 1. Création (on peut utiliser une compréhension ou écrire à la main)
logs: list[str] = [
    "Log 1", "Log 2", "Log 3", "Log 4", 
    "Log 5", "Log 6", "Log 7", "Log 8"
]

# 2. Rotation : Garder les 5 derniers
# [-5:] signifie "des 5 derniers jusqu'à la fin"
logs = logs[-5:]
print(f"Logs après première rotation : {logs}")

# 3. Nouveau log
logs.append("Log 9")

# 4. Nouvelle rotation (car maintenant on en a 6)
logs = logs[-5:]

# 5. Affichage inversé pour le dashboard
# [::-1] crée une copie inversée
dashboard_view = logs[::-1]
print(f"Logs finaux (récent -> ancien) : {dashboard_view}")
```
</details>