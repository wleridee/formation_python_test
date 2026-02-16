---
sidebar_label: Structures de Données : Tuples
sidebar_position: 7
---

# Chapitre 7 : Structures de Données : Tuples

Définition et usage, Immuabilité, Packed/Unpacked tuples

## 1. Définition et Usage {#definition-et-usage}

### 1. Quoi {#definition-quoi}
Un **Tuple** est une collection ordonnée et **immuable** (non modifiable) d'éléments. Contrairement aux listes qui sont définies avec des crochets `[]`, les tuples sont généralement définis avec des parenthèses `()`, bien que ce soit techniquement la virgule `,` qui définit le tuple.

En Python moderne (3.12), les tuples sont typés avec `tuple[Type1, Type2]` pour une structure fixe, ou `tuple[Type, ...]` pour une collection homogène de longueur indéfinie.

### 2. Pourquoi {#definition-pourquoi}
- **Intégrité des données** : Garantit qu'une séquence de données ne sera pas modifiée accidentellement au cours de l'exécution du programme (ex: coordonnées GPS, configuration serveur).
- **Performance** : Les tuples sont légèrement plus rapides et consomment moins de mémoire que les listes car leur taille est fixe.
- **Clés de Dictionnaire** : Parce qu'ils sont immuables (et hachables), les tuples peuvent être utilisés comme clés dans un dictionnaire, contrairement aux listes.

### 3. Comment {#definition-comment}

#### A. Syntaxe de base

```python
# Création simple
coordinates: tuple[float, float] = (48.8566, 2.3522)

# Accès (comme les listes)
latitude = coordinates[0]  # 48.8566
```

#### B. Cas concret : Configuration immuable

```python
# Définition d'une configuration serveur qui ne doit pas bouger
# Typage : Un tuple contenant (IP, Port, Mode Debug)
server_config: tuple[str, int, bool] = ("192.168.1.10", 8080, False)

def connect_to_server(config: tuple[str, int, bool]) -> None:
    ip_address = config[0]
    port = config[1]
    print(f"Connexion à {ip_address}:{port}...")

connect_to_server(server_config)
```

#### C. Exemples pratiques

1.  **Retourner plusieurs valeurs** : Une fonction qui renvoie un résultat et un statut.
2.  **Enregistrements (Records)** : Stocker les données d'une ligne de base de données (ID, Nom, Email).
3.  **Coordonnées** : Géométrie, points 3D (x, y, z).

### 4. Zone de Danger {#definition-danger}

❌ **Le piège du tuple à un seul élément**
Python utilise les parenthèses pour les mathématiques `(1 + 2)`. Pour créer un tuple d'un seul élément, la **virgule est obligatoire**.

```python
# ❌ Ceci est un entier (int), pas un tuple !
not_a_tuple = (42)
print(type(not_a_tuple)) # <class 'int'>

# ✅ Ceci est un tuple
is_a_tuple = (42,) 
print(type(is_a_tuple)) # <class 'tuple'>
```

---

## 2. Immuabilité {#immuabilite}

### 1. Quoi {#immuabilite-quoi}
L'immuabilité signifie qu'une fois le tuple créé, on ne peut ni ajouter, ni supprimer, ni modifier ses éléments. L'identité mémoire de la structure est scellée.

### 2. Pourquoi {#immuabilite-pourquoi}
Cela permet de passer des données complexes à travers plusieurs fonctions sans craindre "d'effets de bord" (side effects). Si vous passez un tuple à une fonction tierce, vous avez la certitude qu'elle ne pourra pas le modifier à votre insu.

### 3. Comment {#immuabilite-comment}

#### A. Tentative de modification (Erreur)

```python
user_data = ("admin", 1234)

try:
    # ❌ Interdit
    user_data[0] = "guest"
except TypeError as e:
    print(f"Erreur attrapée : {e}")
    # Output: 'tuple' object does not support item assignment
```

#### B. La nuance "Cheval de Troie" (Objets mutables dans un tuple)
Si un tuple contient un objet mutable (comme une liste), cet objet interne **peut** être modifié. Le tuple protège la *référence* vers la liste, pas le *contenu* de la liste.

```python
# Un tuple contenant une liste de notes
student_record: tuple[str, list[int]] = ("Alice", [10, 15])

# ✅ On peut modifier la liste À L'INTÉRIEUR du tuple
student_record[1].append(20)

print(student_record) 
# ('Alice', [10, 15, 20]) -> La liste a changé, mais le tuple pointe toujours vers la même liste.
```

---

## 3. Packed / Unpacked Tuples {#packed-unpacked}

### 1. Quoi {#unpacked-quoi}
Le "Packing" consiste à regrouper des valeurs dans un tuple. L'"Unpacking" (déballage) permet d'extraire les valeurs d'un tuple directement dans des variables individuelles en une seule ligne.

### 2. Pourquoi {#unpacked-pourquoi}
C'est l'une des fonctionnalités les plus "Pythoniques". Elle rend le code extrêmement lisible, évite l'utilisation d'index numériques obscurs (`t[0], t[1]`) et permet d'échanger des variables sans variable temporaire.

### 3. Comment {#unpacked-comment}

#### A. Syntaxe de base

```python
# Packing
coordinates = 10, 20  # Les parenthèses sont optionnelles ici

# Unpacking
x, y = coordinates
print(f"x={x}, y={y}")
```

#### B. Cas concret : Retour multiple de fonction

```python
def get_min_max(data: list[int]) -> tuple[int, int]:
    # On renvoie deux valeurs (Packing implicite)
    return min(data), max(data)

temperatures = [12, 14, 9, 20, 18]

# Unpacking direct au retour de la fonction
temp_min, temp_max = get_min_max(temperatures)

print(f"Min: {temp_min}°C, Max: {temp_max}°C")
```

#### C. Unpacking Avancé (L'opérateur `*`)
Depuis Python 3, on peut capturer "le reste" des éléments avec `*`.

```python
# Exemple : Le podium et les autres
race_results = ("Mario", "Luigi", "Peach", "Toad", "Yoshi")

winner, runner_up, *others = race_results

print(winner)   # "Mario"
print(others)   # ['Peach', 'Toad', 'Yoshi'] (C'est une liste)
```

#### D. Ignorer des valeurs (`_`)
Par convention, on utilise l'underscore `_` pour les variables dont on ne se servira pas.

```python
full_name = ("Jean", "Pierre", "Dupont")

# On veut juste le prénom et le nom, on ignore le deuxième prénom
first, _, last = full_name
```

---

## Questions clés (validation des acquis du chapitre) {#questions-clés-(validation-des-acquis-du-chapitre)-7}

1.  **Quelle est la différence syntaxique principale entre une liste et un tuple lors de la définition ?**
    *   Les listes utilisent des crochets `[]` et les tuples utilisent des parenthèses `()` (ou juste des virgules).

2.  **Comment créer un tuple contenant un seul élément, par exemple le nombre 5 ?**
    *   Il faut ajouter une virgule après l'élément : `t = (5,)`. `(5)` est simplement un entier parenthésé.

3.  **Peut-on modifier le contenu d'une liste qui se trouve à l'intérieur d'un tuple ?**
    *   **Oui**. L'immuabilité du tuple ne concerne que les références qu'il contient. Si l'élément référencé est mutable (comme une liste), il peut être modifié.

4.  **Que se passe-t-il si on tente de faire `mon_tuple[0] = 10` ?**
    *   Python lève une exception `TypeError` car les tuples ne supportent pas l'assignation d'items.

5.  **Comment échanger la valeur de deux variables `a` et `b` en une seule ligne grâce aux tuples ?**
    *   `a, b = b, a` (C'est du packing suivi d'un unpacking).

---

## Exercices : {#exercices-:-7}

### Exercice 1 - Le Journal de Vol du Drone {#exercice-1---le-journal-de-vol-du-drone}

**🎯 Objectif** : Manipuler la création de tuples et l'accès par index.

**💼 Mise en situation** :
Vous développez le système de navigation d'un drone de livraison. Le drone enregistre son parcours sous forme d'une liste de coordonnées GPS immuables. Chaque point est un tuple `(latitude, longitude)`.

**📝 Énoncé** :
1.  Créez une liste nommée `flight_path` contenant 3 tuples de coordonnées (float).
2.  Le drone revient à sa base. Ajoutez (via `append`) le point de départ `(48.85, 2.35)` à la fin de la liste.
3.  Affichez le dernier point enregistré sans connaître la longueur de la liste (utilisez l'indexation négative).
4.  Tentez de modifier la latitude du premier point pour voir l'erreur, et gérez-la avec un bloc `try/except`.

**📺 Résultat attendu** :
```text
Dernier point : (48.85, 2.35)
ALERTE : Impossible de modifier un point GPS historique !
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
# 1. Création de la liste de tuples (tuples immuables dans une liste mutable)
flight_path: list[tuple[float, float]] = [
    (48.86, 2.36),
    (48.87, 2.37),
    (48.88, 2.38)
]

# 2. Ajout du point de retour
base_station: tuple[float, float] = (48.85, 2.35)
flight_path.append(base_station)

# 3. Accès au dernier élément (-1)
last_point = flight_path[-1]
print(f"Dernier point : {last_point}")

# 4. Tentative de modification (Immuabilité)
try:
    # On essaie de changer la latitude du premier tuple
    # flight_path[0] est un tuple, donc immuable
    flight_path[0][0] = 0.0 
except TypeError:
    print("ALERTE : Impossible de modifier un point GPS historique !")
```
</details>

---

### Exercice 2 - L'extracteur de Base de Données {#exercice-2---l'extracteur-de-base-de-données}

**🎯 Objectif** : Maîtriser l'Unpacking et l'ignoring de variables (`_`).

**💼 Mise en situation** :
Vous travaillez pour une startup SaaS. Votre API reçoit des lignes brutes d'une base de données héritée (legacy). Chaque ligne contient beaucoup trop d'informations : `(id, username, password_hash, email, created_at, last_login, is_active)`. Vous ne voulez extraire que l'**id**, le **username** et l'**email**.

**📝 Énoncé** :
1.  Définissez un tuple `db_row` simulant une ligne complète.
2.  Utilisez l'unpacking avec `_` (ou `*_` pour ignorer plusieurs éléments d'un coup) pour extraire uniquement `user_id`, `username` et `email`.
    *   *Astuce* : L'email est au milieu, attention à l'ordre.
3.  Affichez les données extraites formatées.

**📺 Résultat attendu** :
```text
User #42 : admin_sys (contact: admin@startup.io)
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
# Simulation d'une ligne de DB
# Structure: (id, username, password, email, created, login, active)
db_row = (42, "admin_sys", "sha256:xyz...", "admin@startup.io", "2023-01-01", "2023-10-27", True)

# Option A : Ignorer variable par variable (verbeux)
# user_id, username, _, email, _, _, _ = db_row

# Option B : Utiliser l'unpacking étendu (*) pour ignorer la fin
# Attention : l'email est en position 3 (index 3), pas à la fin.
# Structure précise : id, user, pass, email, ... reste

user_id, username, _, email, *meta_data = db_row

# Note : *meta_data va capturer ("2023-01-01", "2023-10-27", True)

print(f"User #{user_id} : {username} (contact: {email})")
```
</details>

---

### Exercice 3 - Analyse de Ventes E-commerce {#exercice-3---analyse-de-ventes-e-commerce}

**🎯 Objectif** : Utiliser les tuples pour retourner plusieurs valeurs depuis une fonction.

**💼 Mise en situation** :
Votre plateforme e-commerce a besoin d'un dashboard rapide. Vous devez écrire une fonction qui prend une liste de montants de commandes et retourne un résumé statistique contenant : le montant total, la commande moyenne, et la plus grosse commande.

**📝 Énoncé** :
1.  Écrivez une fonction `analyze_orders(orders: list[float])` qui retourne un `tuple[float, float, float]`.
2.  Calculez la somme, la moyenne (arrondie à 2 décimales) et le max.
3.  Dans le programme principal, appelez la fonction avec une liste de ventes et "déballez" (unpack) le résultat dans trois variables : `total`, `avg`, `max_order`.

**📺 Résultat attendu** :
```text
Analyse sur 5 commandes :
Total CA : 450.50 €
Panier moyen : 90.10 €
Record : 150.00 €
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
def analyze_orders(orders: list[float]) -> tuple[float, float, float]:
    """
    Calcule des statistiques sur une liste de commandes.
    Retourne (Total, Moyenne, Max)
    """
    if not orders:
        return (0.0, 0.0, 0.0)
    
    total_revenue = sum(orders)
    average_cart = round(total_revenue / len(orders), 2)
    max_cart = max(orders)
    
    # On retourne un tuple (packing automatique)
    return total_revenue, average_cart, max_cart

# Données de test
sales_data = [29.99, 150.00, 12.50, 99.99, 158.02]

# Appel et Unpacking immédiat
total, avg, max_order = analyze_orders(sales_data)

print(f"Analyse sur {len(sales_data)} commandes :")
print(f"Total CA : {total} €")
print(f"Panier moyen : {avg} €")
print(f"Record : {max_order} €")
```
</details>