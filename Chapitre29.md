---
sidebar_label: Module `collections` : Types de Conteneurs Spécialisés
sidebar_position: 29
---

# Chapitre 29 : Module `collections` : Types de Conteneurs Spécialisés

Counter, defaultdict, namedtuple, deque

Python est célèbre pour ses structures de données natives puissantes : listes, dictionnaires et tuples. Cependant, pour des besoins spécifiques (performance, lisibilité, regroupement), la bibliothèque standard propose le module `collections`. Ce sont les "piles incluses" survitaminées de Python.

Ce chapitre explore quatre outils indispensables qui permettent d'écrire un code plus performant ("Pythonique") et plus expressif que les structures de base.

---

## 1. `Counter` : Le Comptable Automatique {#counter}

### 1. Quoi
`Counter` est une sous-classe de dictionnaire spécialement conçue pour compter des objets hachables. Il stocke les éléments comme clés et leur nombre d'occurrences comme valeurs.

### 2. Pourquoi
Avant `Counter`, compter la fréquence des éléments dans une liste nécessitait une boucle `for` et des conditions `if/else` verbeuses. `Counter` fait cela en une ligne et offre des méthodes d'analyse statistique (comme "les N plus fréquents").

### 3. Comment

#### A. Syntaxe de base

```python
from collections import Counter

# Création directe à partir d'une liste
votes = ['Alice', 'Bob', 'Alice', 'Charlie', 'Alice', 'Bob']
c = Counter(votes)

print(c) 
# Counter({'Alice': 3, 'Bob': 2, 'Charlie': 1})

# Accès comme un dictionnaire standard
print(c['Alice']) # 3
# ⚠️ Pas de KeyError si la clé n'existe pas, retourne 0
print(c['David']) # 0
```

#### B. Cas concret : Analyse de logs
Imaginez analyser des codes de statut HTTP pour voir quelles erreurs reviennent le plus souvent.

```python
from collections import Counter
from typing import List

http_status_codes: List[int] = [200, 404, 200, 500, 200, 404, 200, 503]

stats = Counter(http_status_codes)

# Récupérer les 2 statuts les plus fréquents
# Retourne une liste de tuples (élément, compte)
top_errors = stats.most_common(2) 

print(f"Top statuts : {top_errors}")
# Top statuts : [(200, 4), (404, 2)]
```

### 4. Zone de Danger
❌ **Utiliser Counter pour tout stocker** :
`Counter` est optimisé pour le comptage. Si vous avez besoin de stocker des données complexes associées à une clé, utilisez un `dict` standard.
✅ **Opérations mathématiques** :
Les objets `Counter` supportent l'addition `+` et la soustraction `-` (ex: additionner les ventes de deux jours différents).

---

## 2. `defaultdict` : Le Dictionnaire Prévoyant {#defaultdict}

### 1. Quoi
`defaultdict` est une sous-classe de dictionnaire qui appelle une "usine" (factory function) pour fournir une valeur par défaut si une clé demandée n'existe pas, au lieu de lever une `KeyError`.

### 2. Pourquoi
Le pattern "Vérifier si la clé existe, sinon créer une liste vide, puis ajouter l'élément" est omniprésent en Python. `defaultdict` supprime cette friction. C'est idéal pour **regrouper** des données.

### 3. Comment

#### A. Syntaxe de base

```python
from collections import defaultdict

# On définit que la valeur par défaut sera une liste vide (list())
dd = defaultdict(list)

# On accède à une clé inexistante 'fruits' -> Python crée la liste automatiquement
dd['fruits'].append('Pomme')
dd['fruits'].append('Banane')

print(dd)
# defaultdict(<class 'list'>, {'fruits': ['Pomme', 'Banane']})
```

#### B. Cas concret : Regroupement par catégorie (E-commerce)

```python
from collections import defaultdict

products = [
    ('Tech', 'Laptop'),
    ('Maison', 'Lampe'),
    ('Tech', 'Souris'),
    ('Jardin', 'Pelle'),
    ('Maison', 'Tapis')
]

# Sans defaultdict : 4 lignes de logique "if key not in..."
# Avec defaultdict :
catalog = defaultdict(list)

for category, item in products:
    # Pas besoin de vérifier si 'category' existe !
    catalog[category].append(item)

print(dict(catalog)) # Conversion en dict normal pour affichage propre
# {'Tech': ['Laptop', 'Souris'], 'Maison': ['Lampe', 'Tapis'], 'Jardin': ['Pelle']}
```

### 🚨 Limitations de `defaultdict`
Attention : le simple fait d'interroger une clé crée une entrée.
`val = my_defaultdict['cle_inexistante']` va créer l'entrée `'cle_inexistante': []` dans le dictionnaire, ce qui peut faire gonfler la mémoire inutilement si vous faites juste de la lecture.

---

## 3. `namedtuple` : Le Tuple Lisible {#namedtuple}

### 1. Quoi
`namedtuple` est une fonction fabrique qui génère une nouvelle classe héritant de `tuple`. Elle permet d'accéder aux éléments par **nom** (`p.x`) en plus de l'index (`p[0]`).

### 2. Pourquoi
Les tuples classiques `(25, "Alice", True)` sont opaques : que signifie `25` ? L'âge ? L'ID ? `namedtuple` donne du sens aux données sans le surcoût mémoire d'une classe complète.

### 3. Comment

#### A. Syntaxe de base

```python
from collections import namedtuple

# Définition de la structure
User = namedtuple('User', ['id', 'username', 'is_admin'])

# Instanciation
u1 = User(101, 'admin_sys', True)

# Accès par nom (Lisible)
print(f"User: {u1.username}") 

# Accès par index (Rétro-compatible avec tuple)
print(f"ID: {u1[0]}")
```

#### B. Cas concret : Coordonnées GPS ou CSV
Idéal pour représenter une ligne d'un fichier CSV ou une configuration immuable.

```python
Point = namedtuple('Point', ['x', 'y', 'z'])
p = Point(10, 20, 5)

# p.x = 12 # ❌ AttributeError : les tuples sont immuables !
```

### D. Tableau Comparatif : Tuple vs NamedTuple vs Class vs DataClass

| Type | Mutabilité | Accès par nom | Charge Mémoire | Cas d'usage |
| :--- | :--- | :--- | :--- | :--- |
| **Tuple** | Immuable | Non (`t[0]`) | Très faible | Données brutes, temporaires |
| **NamedTuple** | Immuable | Oui (`t.name`) | Faible | Structure de données simple |
| **Class** | Mutable | Oui | Moyenne | Logique métier + Données |
| **DataClass** | Mutable* | Oui | Moyenne | DTO moderne (Python 3.7+) |

*(DataClass peut être rendue immuable avec `frozen=True`)*

---

## 4. `deque` : La File à Double Entrée {#deque}

### 1. Quoi
`deque` (prononcer "deck") signifie *Double Ended Queue*. C'est une liste optimisée pour ajouter et retirer des éléments **aux deux extrémités** (début et fin).

### 2. Pourquoi
Les listes Python (`list`) sont rapides pour ajouter/retirer à la *fin* (`append`, `pop`), mais très lentes pour le faire au *début* (`insert(0, x)`, `pop(0)`), car il faut décaler tous les autres éléments en mémoire (complexité O(n)).
`deque` réalise ces opérations instantanément (O(1)).

### 3. Comment

#### A. Syntaxe de base

```python
from collections import deque

file_attente = deque(["Client 2", "Client 3"])

# Ajout rapide au début (Haute priorité)
file_attente.appendleft("Client 1 (VIP)")

# Ajout normal à la fin
file_attente.append("Client 4")

# Retrait rapide au début
served = file_attente.popleft()
print(f"Service de : {served}") # Client 1 (VIP)
```

#### B. La fonctionnalité "Historique Glissant" (`maxlen`)
Une caractéristique puissante de `deque` est la limite de taille.

```python
# Garde seulement les 3 dernières actions
history = deque(maxlen=3)

history.append("Page 1")
history.append("Page 2")
history.append("Page 3")
history.append("Page 4") 

print(history) 
# deque(['Page 2', 'Page 3', 'Page 4'], maxlen=3)
# "Page 1" a été automatiquement éjectée !
```

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-29}

1.  **Pourquoi utiliser `defaultdict(list)` au lieu d'un `dict` classique ?**
    Pour éviter de devoir vérifier manuellement si une clé existe avant d'y ajouter un élément (`append`). Le dictionnaire initialise la liste automatiquement.

2.  **Quelle est la différence de performance majeure entre `list.pop(0)` et `deque.popleft()` ?**
    `list.pop(0)` est lent (O(n), il décale tout le tableau). `deque.popleft()` est instantané (O(1)).

3.  **Que retourne `Counter['cle_inexistante']` ?**
    Il retourne `0` (zéro), au lieu de lever une erreur.

4.  **Peut-on modifier la valeur d'un champ dans un `namedtuple` après sa création (ex: `p.x = 10`) ?**
    Non, comme les tuples standards, les `namedtuple` sont immuables. Il faut créer une nouvelle instance ou utiliser `_replace()`.

---

## Exercices : {#exercices-29}

### Exercice 1 - Analyseur de Fréquence de Mots {#exercice-1---analyseur-frequence}

🎯 **Objectif** : Maîtriser `Counter` pour l'analyse de texte.

💼 **Mise en situation** : Vous développez un outil SEO. Vous devez analyser un texte pour trouver les 3 mots les plus utilisés (hors mots courts de moins de 3 lettres).

📝 **Énoncé** :
1.  Voici le texte : `"python est super python est puissant python est facile code code"`.
2.  Découpez le texte en liste de mots.
3.  Filtrez pour ne garder que les mots de 3 lettres ou plus.
4.  Utilisez `Counter` pour trouver les fréquences.
5.  Affichez les 3 mots les plus communs avec leur score.

📺 **Résultat attendu** :
```text
[('python', 3), ('est', 3), ('code', 2)]
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
from collections import Counter

text = "python est super python est puissant python est facile code code"

# 1. Nettoyage et découpage
words = text.split()

# 2. Filtrage (mots >= 3 lettres)
long_words = [w for w in words if len(w) >= 3]

# 3. Comptage
counter = Counter(long_words)

# 4. Top 3
top_3 = counter.most_common(3)

print(f"Top mots : {top_3}")
```
</details>

### Exercice 2 - Regroupement de Ventes par Ville {#exercice-2---ventes-ville}

🎯 **Objectif** : Utiliser `defaultdict` pour agréger des données.

💼 **Mise en situation** : Vous recevez un flux de transactions. Chaque transaction est un tuple `(Ville, Montant)`. Vous devez calculer le chiffre d'affaires total par ville.

📝 **Énoncé** :
1.  Liste des transactions : `[('Paris', 100), ('Lyon', 50), ('Paris', 20), ('Marseille', 80), ('Lyon', 10)]`.
2.  Créez un `defaultdict` avec `int` comme valeur par défaut (pour démarrer à 0).
3.  Parcourez la liste et additionnez les montants par ville.
4.  Affichez le résultat final.

📺 **Résultat attendu** :
```text
Paris : 120€
Lyon : 60€
Marseille : 80€
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
from collections import defaultdict

transactions = [
    ('Paris', 100),
    ('Lyon', 50),
    ('Paris', 20),
    ('Marseille', 80),
    ('Lyon', 10)
]

# Le type 'int' instancié sans argument retourne 0
# C'est parfait pour un compteur ou une somme
revenue_by_city = defaultdict(int)

for city, amount in transactions:
    # Pas besoin de if city in revenue_by_city...
    revenue_by_city[city] += amount

# Affichage propre
for city, total in revenue_by_city.items():
    print(f"{city} : {total}€")
```
</details>

### Exercice 3 - Gestionnaire de Tâches Récentes {#exercice-3---taches-recentes}

🎯 **Objectif** : Utiliser `deque` avec `maxlen`.

💼 **Mise en situation** : Dans une application SaaS, vous voulez afficher dans la barre latérale "Les 5 derniers fichiers ouverts" par l'utilisateur.

📝 **Énoncé** :
1.  Créez une fonction `open_file(filename)` qui ajoute le fichier à un historique global.
2.  L'historique doit être un `deque` limité à 5 éléments.
3.  Simulez l'ouverture de 7 fichiers différents (ex: "file1.txt", "file2.txt"... "file7.txt").
4.  Affichez l'historique final pour vérifier que les fichiers 1 et 2 ont disparu.

📺 **Résultat attendu** :
```text
Historique : deque(['file3.txt', 'file4.txt', 'file5.txt', 'file6.txt', 'file7.txt'], maxlen=5)
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
from collections import deque

# Création de l'historique avec taille fixe
# Les nouveaux éléments pousseront les anciens dehors automatiquement
recent_files = deque(maxlen=5)

def open_file(filename: str):
    print(f"Ouverture de {filename}...")
    recent_files.append(filename)

# Simulation
for i in range(1, 8):
    open_file(f"file{i}.txt")

print("\n--- État final ---")
print(f"Historique : {recent_files}")
# Notez que file1.txt et file2.txt ne sont plus là
```
</details>