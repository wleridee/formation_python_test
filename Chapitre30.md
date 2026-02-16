---
sidebar_label: Module `itertools` : Fonctions pour Itérateurs
sidebar_position: 30
---

# Chapitre 30 : Module `itertools` : Fonctions pour Itérateurs

Combinations, Permutations, Product, Cycle, repeat, chain

Le module `itertools` est souvent considéré comme le joyau caché de la bibliothèque standard Python. Il fournit une collection de briques fondamentales pour créer des itérateurs efficaces et rapides.

Inspirés de la programmation fonctionnelle, ces outils permettent de manipuler des flux de données (listes, tuples, générateurs) de manière concise et optimisée pour la mémoire, sans jamais charger toutes les données en RAM. Maîtriser `itertools`, c'est passer du niveau "débutant qui écrit des boucles imbriquées" au niveau "expert qui traite des données massivement".

---

## 1. Itérateurs Infinis : `cycle`, `repeat`, `count` {#iterateurs-infinis}

### 1. Quoi
Ce sont des fonctions qui génèrent des séquences de données qui ne s'arrêtent jamais (sauf si vous les arrêtez explicitement avec un `break`).
*   **`cycle(iterable)`** : Répète les éléments d'une séquence en boucle indéfiniment.
*   **`repeat(elem, [n])`** : Répète un seul élément indéfiniment (ou `n` fois).
*   **`count(start, [step])`** : Compte à partir de `start` avec un pas de `step` (par défaut 1).

### 2. Pourquoi
Utile pour :
*   Assigner des tâches à tour de rôle (Round Robin).
*   Générer des données de test constantes.
*   Créer des identifiants uniques séquentiels.

### 3. Comment

#### A. Syntaxe de base

```python
import itertools

# 1. cycle
colors = itertools.cycle(['Rouge', 'Vert', 'Bleu'])
# next(colors) -> Rouge, next(colors) -> Vert, next(colors) -> Bleu, next(colors) -> Rouge...

# 2. repeat
constant = itertools.repeat(42, 3)
# list(constant) -> [42, 42, 42]

# 3. count
counter = itertools.count(start=10, step=2)
# next(counter) -> 10, next(counter) -> 12...
```

#### B. Cas concret : Répartition de charge (Round Robin)

Imaginez répartir des requêtes entrantes entre 3 serveurs disponibles.

```python
import itertools
import time

servers = ["Server-A", "Server-B", "Server-C"]
server_pool = itertools.cycle(servers)

incoming_requests = ["Req-001", "Req-002", "Req-003", "Req-004", "Req-005"]

print("--- Répartition des tâches ---")
for req in incoming_requests:
    assigned_server = next(server_pool)
    print(f"La requête {req} est traitée par {assigned_server}")
```

### 🚨 Limitations de `cycle` et `count`
⚠️ **Boucles infinies** : Ne faites jamais `list(itertools.count())` ou `for x in itertools.cycle(...)` sans condition d'arrêt (`break`). Cela remplira votre RAM jusqu'au crash de l'application.

---

## 2. Combinatoire : `product`, `permutations`, `combinations` {#combinatoire}

### 1. Quoi
Ces fonctions génèrent toutes les dispositions possibles d'un ensemble d'éléments.
*   **`product()`** : Produit cartésien (équivalent à des boucles `for` imbriquées).
*   **`permutations()`** : Tous les ordres possibles (AB, BA).
*   **`combinations()`** : Tous les groupes possibles sans ordre (AB est pareil que BA).

### 2. Pourquoi
Indispensable pour :
*   Le "Brute Force" (tester tous les mots de passe possibles).
*   Générer des configurations de test (toutes les combinaisons d'options).
*   Problèmes d'optimisation (voyageur de commerce).

### 3. Comment

#### A. Syntaxe de base

```python
import itertools

data = [1, 2, 3]

# Product : "Avec remise" (1,1 existe)
print(list(itertools.product(data, repeat=2)))
# [(1, 1), (1, 2), (1, 3), (2, 1), (2, 2), (2, 3), (3, 1), (3, 2), (3, 3)]

# Permutations : L'ordre compte, pas de répétition d'index
print(list(itertools.permutations(data, 2)))
# [(1, 2), (1, 3), (2, 1), (2, 3), (3, 1), (3, 2)]

# Combinations : L'ordre ne compte pas
print(list(itertools.combinations(data, 2)))
# [(1, 2), (1, 3), (2, 3)]
```

#### B. Cas concret : Générateur de grille de variantes E-commerce
Vous vendez un T-shirt avec plusieurs attributs. Vous devez générer tous les SKUs (Stock Keeping Units) possibles.

```python
import itertools

colors = ["Red", "Blue"]
sizes = ["S", "M", "L"]
materials = ["Cotton", "Polyester"]

# product remplace 3 boucles for imbriquées
variants = itertools.product(colors, sizes, materials)

print("--- SKUs Générés ---")
for c, s, m in variants:
    sku = f"TSHIRT-{c}-{s}-{m}"
    print(sku)
    # Ex: TSHIRT-Red-S-Cotton
    # Ex: TSHIRT-Red-S-Polyester
    # ...
```

### 4. Zone de Danger
❌ **Explosion combinatoire** :
`permutations(range(20))` génère 2,4 trilliards de possibilités. N'essayez jamais de convertir ces itérateurs en liste (`list()`) si la taille des données d'entrée est grande. Itérez dessus un par un.

---

## 3. Chaînage et Découpage : `chain`, `islice` {#chainage-et-decoupage}

### 1. Quoi
*   **`chain()`** : Prend plusieurs itérables et les "soude" bout à bout pour n'en former qu'un seul.
*   **`islice()`** : Effectue un "slicing" (`[start:stop:step]`) sur un itérateur (ce que la syntaxe standard `[:]` ne peut pas faire sur un générateur).

### 2. Pourquoi
*   **`chain`** : Pour traiter des données provenant de sources différentes (ex: une liste locale + un résultat BDD) sans créer une nouvelle liste géante en mémoire.
*   **`islice`** : Pour prendre les "N premiers" éléments d'un flux infini ou très long.

### 3. Comment

#### A. Syntaxe `chain`

```python
import itertools

list1 = [1, 2, 3]
tuple1 = (4, 5, 6)
gen1 = range(7, 9)

# Itère sur 1, 2, 3, puis 4, 5, 6, puis 7, 8
combined = itertools.chain(list1, tuple1, gen1)

print(list(combined)) 
# [1, 2, 3, 4, 5, 6, 7, 8]
```

#### B. Syntaxe `islice`

```python
import itertools

def infinite_numbers():
    n = 0
    while True:
        yield n
        n += 1

gen = infinite_numbers()

# gen[:5] # ❌ TypeError: 'generator' object is not subscriptable

# ✅ islice permet de découper un itérateur
first_five = itertools.islice(gen, 5)
print(list(first_five)) # [0, 1, 2, 3, 4]
```

---

## 4. Manipulation de Données : `groupby` {#manipulation-groupby}

### 1. Quoi
Regroupe des éléments consécutifs ayant la même clé. Fonctionne comme le `GROUP BY` en SQL, **mais exige que les données soient triées au préalable**.

### 2. Pourquoi
Pour aggréger des données (logs, ventes) par catégorie, date ou identifiant.

### 3. Comment

```python
import itertools

# Données TRIÉES par catégorie (impératif !)
inventory = [
    {'item': 'Pomme', 'type': 'Fruit'},
    {'item': 'Banane', 'type': 'Fruit'},
    {'item': 'Carotte', 'type': 'Légume'},
    {'item': 'Poireau', 'type': 'Légume'},
]

# Fonction pour extraire la clé de tri/groupement
key_func = lambda x: x['type']

# Groupement
grouped_data = itertools.groupby(inventory, key=key_func)

print("--- Inventaire ---")
for key, group in grouped_data:
    # 'group' est un itérateur, on le convertit en liste pour voir le contenu
    items = list(group)
    print(f"Catégorie : {key} ({len(items)} articles)")
    for i in items:
        print(f" - {i['item']}")
```

### 🚨 Limitations de `groupby`
Si votre liste n'est pas triée par la clé de groupement, `groupby` créera un nouveau groupe à chaque changement de valeur, ce qui ne donnera pas le résultat attendu.
✅ **Toujours trier (`sorted()`) avant de `groupby()`**.

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-30}

1.  **Quelle est la différence majeure entre `itertools.product` et `zip` ?**
    `zip` associe les éléments index par index (le 1er avec le 1er). `product` associe *chaque* élément du premier groupe avec *tous* les éléments du second groupe (produit cartésien).

2.  **Pourquoi faut-il trier une liste avant d'utiliser `itertools.groupby` ?**
    Car `groupby` ne regroupe que les éléments *consécutifs* identiques. Si les données sont mélangées, vous obtiendrez des groupes fragmentés.

3.  **Quel est l'avantage de `itertools.chain(list1, list2)` par rapport à `list1 + list2` ?**
    `chain` ne crée pas de nouvelle liste intermédiaire en mémoire. Il itère simplement sur la première, puis passe à la seconde. C'est beaucoup plus efficace pour de grandes listes.

4.  **Comment récupérer les 10 premiers éléments d'un générateur infini ?**
    En utilisant `itertools.islice(generateur, 10)`.

---

## Exercices : {#exercices-30}

### Exercice 1 - Le Cassage de Code PIN (Brute Force) {#exercice-1-brute-force}

🎯 **Objectif** : Utiliser `product` pour générer des combinaisons.

💼 **Mise en situation** : Vous devez tester la sécurité d'un système protégé par un code PIN à 4 chiffres (0-9). Vous devez générer tous les codes possibles.

📝 **Énoncé** :
1.  Utilisez `itertools.product` pour générer toutes les combinaisons de 4 chiffres (de 0 à 9).
2.  Le code "secret" est le tuple `(7, 2, 5, 9)`.
3.  Itérez sur les combinaisons générées.
4.  Arrêtez-vous quand vous trouvez le code secret et affichez "Code trouvé : [CODE] après X tentatives".

📺 **Résultat attendu** :
```text
Recherche en cours...
Code trouvé : (7, 2, 5, 9) après 7260 tentatives.
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
import itertools

# Chiffres possibles
digits = range(10) # 0, 1, ... 9
secret_code = (7, 2, 5, 9)

# repeat=4 équivaut à passer 4 fois la liste 'digits'
# product(digits, digits, digits, digits)
combinations = itertools.product(digits, repeat=4)

attempts = 0

print("Recherche en cours...")

for combo in combinations:
    attempts += 1
    if combo == secret_code:
        print(f"Code trouvé : {combo} après {attempts} tentatives.")
        break
```
</details>

### Exercice 2 - Ordonnancement d'Équipes {#exercice-2-teams}

🎯 **Objectif** : Utiliser `combinations` pour créer des matchs.

💼 **Mise en situation** : Vous organisez un tournoi e-sport. Il y a 5 équipes. Chaque équipe doit affronter toutes les autres une seule fois.

📝 **Énoncé** :
1.  Liste des équipes : `["Tigers", "Dragons", "Wolves", "Eagles", "Sharks"]`.
2.  Utilisez `combinations` pour générer les paires de matchs uniques.
3.  Affichez le planning.

📺 **Résultat attendu** :
```text
Match 1 : Tigers vs Dragons
Match 2 : Tigers vs Wolves
...
Match 10 : Eagles vs Sharks
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
import itertools

teams = ["Tigers", "Dragons", "Wolves", "Eagles", "Sharks"]

# combinations(iterable, 2) prend des paires uniques sans ordre
# (A vs B est pareil que B vs A, donc on n'aura pas de doublon)
matchups = list(itertools.combinations(teams, 2))

for i, match in enumerate(matchups, 1):
    print(f"Match {i} : {match[0]} vs {match[1]}")

print(f"\nTotal de matchs : {len(matchups)}")
```
</details>

### Exercice 3 - Pagination Intelligente {#exercice-3-pagination}

🎯 **Objectif** : Utiliser `chain` et `islice` pour manipuler des flux.

💼 **Mise en situation** : Vous affichez une page de résultats de recherche. Les résultats viennent de deux sources : "Produits Sponsorisés" (Prioritaires) et "Produits Organiques" (Secondaires). Vous voulez afficher la page 2 (résultats 10 à 20) du flux combiné.

📝 **Énoncé** :
1.  Générez une liste `sponsored` de 5 éléments ("Sponsor 1" à "Sponsor 5").
2.  Générez un itérateur `organic` de 100 éléments ("Item 1" à "Item 100").
3.  Utilisez `chain` pour créer un flux unique : d'abord les sponsors, puis les organiques.
4.  Utilisez `islice` pour extraire uniquement les éléments de l'index 10 à 20 (non inclus) de ce flux global.
5.  Affichez ces éléments sous forme de liste.

📺 **Résultat attendu** :
Les 5 premiers étaient les sponsors, les 5 suivants (6-10) étaient organiques.
Donc l'index 10 commence à "Item 6".
```text
Page 2 : ['Item 6', 'Item 7', 'Item 8', 'Item 9', 'Item 10', 'Item 11', 'Item 12', 'Item 13', 'Item 14', 'Item 15']
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
import itertools

# 1. Sources de données
sponsored = [f"Sponsor {i}" for i in range(1, 6)] # 5 éléments
organic = (f"Item {i}" for i in range(1, 101))    # Générateur (100 éléments)

# 2. Fusion des flux (Virtuelle, pas de copie en mémoire)
full_stream = itertools.chain(sponsored, organic)

# 3. Extraction de la page 2 (Index 10 à 20)
# islice(iterable, start, stop, [step])
# Attention : islice "consomme" l'itérateur jusqu'à 'start' avant de renvoyer des valeurs
page_2 = list(itertools.islice(full_stream, 10, 20))

print(f"Page 2 : {page_2}")
```
</details>