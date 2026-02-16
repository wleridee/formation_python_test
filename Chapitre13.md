---
sidebar_label: Structures de Données : Dictionnaires
sidebar_position: 13
---

# Chapitre 13 : Structures de Données : Dictionnaires

Création, Accès par clé, Modification, Méthodes de dictionnaire

Imaginez que vous deviez stocker les informations d'un utilisateur : son nom, son âge et son email. Avec une liste, vous auriez `["Alice", 30, "alice@email.com"]`. Mais comment savoir que l'index `1` correspond à l'âge ? Et si on ajoute le numéro de téléphone au début, tout se décale !

Le **Dictionnaire (`dict`)** est la solution. C'est une structure de données associative qui relie une **Clé** unique à une **Valeur**. C'est le cœur battant de Python, utilisé partout, de la gestion interne des variables à la manipulation de données JSON.

---

## 1. Création et Structure (Clé-Valeur) {#creation-et-structure}

### 1. Quoi
Une collection d'éléments où chaque entrée est une paire `clé: valeur`. Les clés doivent être **uniques** et **immuables** (hachables). Les valeurs peuvent être de n'importe quel type.

### 2. Pourquoi
*   **Sémantique** : `user["age"]` est plus lisible que `user[1]`.
*   **Performance** : Retrouver une valeur par sa clé est une opération quasi-instantanée (O(1)), quelle que soit la taille du dictionnaire.

### 3. Comment

#### A. Syntaxe de base

```python
# Dictionnaire vide
empty_dict: dict = {}

# Syntaxe littérale (la plus courante)
user: dict[str, str | int] = {
    "name": "Alice",
    "role": "Admin",
    "level": 42
}

# Constructeur dict() (utile pour convertir des listes de tuples)
config = dict(host="localhost", port=8080)
```

#### B. Clés valides vs Invalides
Une clé doit être **hashable** (immuable).

```python
# ✅ VALIDE : Strings, Nombres, Tuples, Frozensets
valid_dict = {
    "id": 1,
    404: "Not Found",
    (10, 20): "Pixel Coords"
}

# ❌ INVALIDE : Listes, Dictionnaires (car mutables)
# invalid_dict = { ["a", "b"]: "Erreur" } # TypeError: unhashable type: 'list'
```

### 4. Zone de Danger
❌ **Clés dupliquées** :
Si vous définissez deux fois la même clé à la création, la dernière valeur écrase la précédente sans avertissement.

```python
d = {"a": 1, "a": 2}
print(d) # {'a': 2}
```

---

## 2. Accès aux Éléments {#acces-aux-elements}

### 1. Quoi
Récupérer la valeur associée à une clé donnée.

### 2. Pourquoi
Contrairement aux listes où l'on cherche par position, ici on "appelle" la donnée par son nom.

### 3. Comment

#### A. Accès direct (Crochets)
Utilisé quand on est **sûr** que la clé existe.

```python
scores: dict[str, int] = {"Alice": 10, "Bob": 8}

print(scores["Alice"]) # 10
```

#### B. La méthode `get()` (Sécurité)
Utilisée quand la clé pourrait ne pas exister. Elle évite de faire planter le programme.

```python
# Renvoie None si la clé n'existe pas
print(scores.get("Charlie")) # None

# Renvoie une valeur par défaut si la clé n'existe pas
print(scores.get("Charlie", 0)) # 0
```

### 4. Zone de Danger
❌ **KeyError** :
Accéder à une clé inexistante avec `[]` lève une exception bloquante.
✅ **Bonne pratique** : Utilisez `.get()` ou vérifiez avec `in`.

```python
# if "Charlie" in scores: print(scores["Charlie"])
```

---

## 3. Modification, Ajout et Fusion {#modification-ajout-fusion}

### 1. Quoi
Les dictionnaires sont **mutables**. On peut changer la valeur d'une clé, ajouter de nouvelles paires ou supprimer des entrées.

### 2. Comment

#### A. Ajouter ou Modifier
La syntaxe est identique : si la clé existe, on écrase la valeur ; sinon, on la crée.

```python
settings: dict[str, bool] = {"dark_mode": False}

settings["dark_mode"] = True  # Modification
settings["sound"] = True      # Ajout
```

#### B. Suppression
*   `pop(key)` : Enlève la clé et retourne sa valeur.
*   `del d[key]` : Supprime la clé (ne retourne rien).
*   `popitem()` : Enlève la dernière paire ajoutée (LIFO).

```python
inventory = {"Pomme": 5, "Poire": 2}
count = inventory.pop("Pomme") # count = 5, inventory = {"Poire": 2}
```

#### C. Fusion de dictionnaires (Merge)
Depuis Python 3.9+, on utilise l'opérateur d'union `|`.

```python
default_options = {"env": "prod", "debug": False}
user_options = {"debug": True, "theme": "blue"}

# Les valeurs de user_options écrasent celles de default_options en cas de conflit
final_config = default_options | user_options
# {'env': 'prod', 'debug': True, 'theme': 'blue'}
```

### 🚨 Limitations
Les dictionnaires préservent l'ordre d'insertion (depuis Python 3.7+). Cependant, ne vous fiez pas à cet ordre pour la logique algorithmique (sauf besoin spécifique), car sémantiquement, un dictionnaire est une association, pas une séquence.

---

## 4. Itération et Méthodes de Vue {#iteration-et-methodes}

### 1. Quoi
Parcourir le contenu du dictionnaire.

### 2. Pourquoi
Pour afficher un résumé, filtrer des données ou transformer les valeurs.

### 3. Comment

#### A. Sur les clés (Par défaut)

```python
prices = {"Pain": 1.10, "Lait": 0.90}

for item in prices: # Équivalent à prices.keys()
    print(item) # Affiche "Pain" puis "Lait"
```

#### B. Sur les valeurs (`.values()`)

```python
for price in prices.values():
    print(price)
```

#### C. Sur les paires (`.items()`) - Le plus utile
Permet de dépaqueter la clé et la valeur simultanément.

```python
for item, price in prices.items():
    print(f"L'article {item} coûte {price}€")
```

#### D. Tableau Comparatif des Méthodes

| Méthode | Retourne | Type de retour (Vue) |
| :--- | :--- | :--- |
| `.keys()` | Liste des clés | `dict_keys` |
| `.values()` | Liste des valeurs | `dict_values` |
| `.items()` | Liste de tuples `(clé, val)` | `dict_items` |

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-13}

1.  **Quelle est la différence majeure entre une liste et un dictionnaire ?**
    La liste est indexée par des entiers (position), le dictionnaire est indexé par des clés (noms, IDs, etc.).

2.  **Que se passe-t-il si on fait `my_dict["inconnu"]` ? Et `my_dict.get("inconnu")` ?**
    Le premier lève une erreur `KeyError`. Le second renvoie `None` (ou une valeur par défaut) sans erreur.

3.  **Peut-on utiliser une liste comme clé d'un dictionnaire ?**
    Non, car une liste est mutable (non hashable). On peut utiliser un tuple à la place.

4.  **Comment fusionner deux dictionnaires `d1` et `d2` en Python moderne ?**
    En utilisant l'opérateur pipe : `new_dict = d1 | d2`.

---

## Exercices : {#exercices-13}

### Exercice 1 - Le Gestionnaire de Stock {#exercice-1---gestion-stock}

🎯 **Objectif** : Manipulation basique (accès, modification, get).

💼 **Mise en situation** : Vous gérez l'inventaire d'un e-commerce. Des commandes arrivent et décrémentent le stock.

📝 **Énoncé** :
1.  Dict initial : `stock = {"Laptop": 10, "Mouse": 50, "Keyboard": 20}`.
2.  Un client achète 2 "Laptop" (Mettez à jour).
3.  Un client veut acheter 1 "Screen" (Article non référencé). Utilisez `.get()` pour vérifier le stock sans planter.
4.  Si l'article n'existe pas, affichez "Article inconnu".

📺 **Résultat attendu** :
```text
Stock Laptop restant : 8
Stock Screen : Article inconnu
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
stock: dict[str, int] = {"Laptop": 10, "Mouse": 50, "Keyboard": 20}

# 1. Achat de 2 Laptops
stock["Laptop"] -= 2
print(f"Stock Laptop restant : {stock['Laptop']}")

# 2. Vérification d'un article inconnu
item_requested = "Screen"

# On récupère le stock, ou None si pas trouvé
qty = stock.get(item_requested)

if qty is None:
    print(f"Stock {item_requested} : Article inconnu")
else:
    print(f"Stock {item_requested} : {qty}")
```
</details>

### Exercice 2 - Analyse de Fréquence (Compteur) {#exercice-2---frequence-mots}

🎯 **Objectif** : Algorithme classique utilisant l'itération.

💼 **Mise en situation** : Vous analysez des avis clients et voulez savoir quels mots reviennent le plus souvent.

📝 **Énoncé** :
1.  Liste de mots : `words = ["bien", "super", "moyen", "bien", "super", "bien"]`.
2.  Créez un dictionnaire vide `frequency`.
3.  Bouclez sur les mots.
4.  Pour chaque mot : s'il est déjà dans le dictionnaire, incrémentez sa valeur. Sinon, initialisez-la à 1.
5.  Affichez le résultat.

📺 **Résultat attendu** :
```text
{'bien': 3, 'super': 2, 'moyen': 1}
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
words: list[str] = ["bien", "super", "moyen", "bien", "super", "bien"]
frequency: dict[str, int] = {}

for word in words:
    # Méthode explicite
    if word in frequency:
        frequency[word] += 1
    else:
        frequency[word] = 1
        
    # Astuce Pro : frequency[word] = frequency.get(word, 0) + 1

print(frequency)
```
</details>

### Exercice 3 - Configuration SaaS (Fusion) {#exercice-3---config-saas}

🎯 **Objectif** : Utiliser la fusion de dictionnaires (`|`).

💼 **Mise en situation** : Votre application a une configuration par défaut. L'utilisateur peut fournir ses propres réglages qui doivent surcharger les défauts.

📝 **Énoncé** :
1.  `default_config = {"theme": "light", "notifications": True, "version": "1.0"}`.
2.  `user_prefs = {"theme": "dark", "version": "2.0"}`.
3.  Créez `final_config` en fusionnant les deux (priorité à l'utilisateur).
4.  Affichez la configuration finale en itérant avec `.items()`.

📺 **Résultat attendu** :
```text
Clé: theme -> Val: dark
Clé: notifications -> Val: True
Clé: version -> Val: 2.0
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
default_config: dict = {"theme": "light", "notifications": True, "version": "1.0"}
user_prefs: dict = {"theme": "dark", "version": "2.0"}

# Fusion : user_prefs à droite gagne en cas de conflit
final_config = default_config | user_prefs

# Affichage propre
for key, value in final_config.items():
    print(f"Clé: {key} -> Val: {value}")
```
</details>