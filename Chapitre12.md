---
sidebar_label: Structures de Données : Ensembles (Sets)
sidebar_position: 12
---

# Chapitre 12 : Structures de Données : Ensembles (Sets)

Création, Opérations d'ensembles, Mutabilité des sets, Frozenset

Imaginez un sac de billes où chaque bille est unique. Si vous essayez d'y ajouter une bille rouge alors qu'il y en a déjà une, la nouvelle est magiquement rejetée. De plus, dans ce sac, il n'y a pas de "première" ou de "dernière" bille ; elles sont toutes en vrac.

C'est exactement ce qu'est un **Set (Ensemble)** en Python : une collection **non ordonnée** d'éléments **uniques**. Inspirés des mathématiques, les sets sont incroyablement performants pour tester l'appartenance et comparer des groupes de données.

---

## 1. Création et Propriétés Fondamentales {#creation-et-proprietes}

### 1. Quoi
Un set est défini par des accolades `{}` (comme les dictionnaires, mais sans clés/valeurs) ou par le constructeur `set()`.

### 2. Pourquoi
*   **Unicité garantie** : Dédupliquer une liste instantanément.
*   **Performance** : Vérifier si un élément existe (`x in my_set`) est quasi-instantané (complexité O(1)), bien plus rapide que dans une liste (O(n)).

### 3. Comment

#### A. Syntaxe de base

```python
# Set avec des valeurs
fruits: set[str] = {"Pomme", "Poire", "Banane"}

# Les doublons sont automatiquement supprimés
votes: set[str] = {"Oui", "Non", "Oui", "Oui", "Absention"}
print(votes) 
# Résultat (l'ordre peut varier) : {'Non', 'Oui', 'Absention'}
```

#### B. Le piège de l'ensemble vide
⚠ `empty = {}` crée un **dictionnaire** vide, pas un set !

```python
# FAUX (Ceci est un dict)
empty_dict = {} 

# VRAI (Ceci est un set vide)
empty_set: set = set()
```

#### C. Conversion (Déduplication)
C'est l'usage le plus courant : transformer une liste en set pour retirer les doublons.

```python
raw_emails: list[str] = ["a@a.com", "b@b.com", "a@a.com"]
unique_emails: set[str] = set(raw_emails)
# {'a@a.com', 'b@b.com'}
```

### 4. Zone de Danger
❌ **Pas d'accès par index** :
Un set n'est pas ordonné. Vous ne pouvez **pas** faire `my_set[0]`. Cela lèvera une `TypeError`.
✅ Pour accéder aux éléments, il faut soit itérer dessus (`for item in my_set`), soit vérifier la présence (`if "A" in my_set`).

---

## 2. Opérations d'Ensembles (Algèbre) {#operations-ensembles}

### 1. Quoi
Python implémente la théorie mathématique des ensembles. Vous pouvez réaliser des unions, intersections et différences directement avec des opérateurs.

### 2. Pourquoi
Pour comparer des groupes de données : "Quels utilisateurs sont dans le groupe A ET le groupe B ?", "Quels produits sont en stock MAIS PAS en promotion ?".

### 3. Comment

Prenons deux équipes de développeurs :
```python
devs_frontend: set[str] = {"Alice", "Bob", "Charlie"}
devs_backend: set[str] = {"Bob", "Dave", "Eve"}
```

#### A. Union (`|`) : Tout le monde
Combine les éléments des deux sets (sans doublons).

```python
all_devs = devs_frontend | devs_backend
# {'Alice', 'Bob', 'Charlie', 'Dave', 'Eve'}
# Bob n'apparait qu'une fois
```

#### B. Intersection (`&`) : Les points communs
Les éléments présents dans les DEUX sets.

```python
fullstack_devs = devs_frontend & devs_backend
# {'Bob'}
```

#### C. Différence (`-`) : Ceux qui sont uniquement ici
Les éléments du set de gauche moins ceux du set de droite.

```python
pure_frontend = devs_frontend - devs_backend
# {'Alice', 'Charlie'} (On a retiré Bob car il fait aussi du back)
```

#### D. Différence Symétrique (`^`) : L'un ou l'autre, mais pas les deux
L'opposé de l'intersection.

```python
specialists = devs_frontend ^ devs_backend
# {'Alice', 'Charlie', 'Dave', 'Eve'} (Tout le monde sauf Bob)
```

### 4. Zone de Danger
❌ **Opérateurs vs Méthodes** :
Les opérateurs (`|`, `&`) nécessitent que les deux opérandes soient des `set`. Les méthodes (`.union()`, `.intersection()`) acceptent n'importe quel itérable (liste, tuple) en argument.

```python
s = {1, 2}
l = [2, 3]

# s | l  -> Erreur (TypeError)
res = s.union(l) # Fonctionne -> {1, 2, 3}
```

---

## 3. Modification et Mutabilité {#modification-et-mutabilite}

### 1. Quoi
Les sets sont **mutables**. On peut ajouter ou retirer des éléments après création.

### 2. Comment

#### A. Ajouter

```python
tags: set[str] = {"python", "code"}
tags.add("tutoriel") # Ajout unique
```

#### B. Retirer

```python
tags = {"A", "B", "C"}

tags.remove("A")   # Retire "A". Lève KeyError si "A" n'existe pas.
tags.discard("Z")  # Retire "Z". Ne fait RIEN si "Z" n'existe pas (Safe).

item = tags.pop()  # Retire et retourne un élément ALÉATOIRE.
tags.clear()       # Vide le set
```

### 🚨 Limitations
Un set ne peut contenir que des objets **hashables** (immuables).
*   ✅ Vous pouvez mettre : `int`, `str`, `tuple`, `frozenset`.
*   ❌ Vous ne pouvez PAS mettre : `list`, `dict`, `set`.

```python
# TypeError: unhashable type: 'list'
# invalid_set = { [1, 2] } 
```

---

## 4. Frozenset : L'Ensemble Immuable {#frozenset}

### 1. Quoi
Le `frozenset` est au `set` ce que le `tuple` est à la `list`. C'est un ensemble qu'on ne peut plus modifier une fois créé.

### 2. Pourquoi
1.  Pour utiliser un ensemble comme **clé de dictionnaire**.
2.  Pour placer un ensemble à l'intérieur d'un autre ensemble.
3.  Pour garantir que les données de référence ne changeront pas.

### 3. Comment

```python
# Création
vowels = frozenset({"a", "e", "i", "o", "u", "y"})

# v.add("b") -> AttributeError: 'frozenset' object has no attribute 'add'

# Utilisation comme clé de dictionnaire
locations: dict[frozenset, str] = {
    frozenset({48.85, 2.35}): "Paris",
    frozenset({40.71, -74.00}): "New York"
}
```

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-12}

1.  **Quelle est la différence principale entre une liste et un set ?**
    La liste est ordonnée et accepte les doublons. Le set est non ordonné et garantit l'unicité des éléments.

2.  **Que fait l'instruction `my_set = {}` ?**
    Elle crée un dictionnaire vide, pas un set. Pour un set vide, il faut utiliser `set()`.

3.  **Pourquoi ne peut-on pas accéder à `my_set[0]` ?**
    Parce que les sets n'ont pas d'ordre défini, donc le concept de "position 0" n'existe pas.

4.  **Quelle méthode utiliser pour retirer un élément sans risquer une erreur s'il est absent ?**
    La méthode `.discard(element)`. La méthode `.remove()` lèverait une erreur.

---

## Exercices : {#exercices-12}

### Exercice 1 - Le Nettoyeur d'Emails {#exercice-1---nettoyeur-emails}

🎯 **Objectif** : Utiliser la propriété d'unicité des sets.

💼 **Mise en situation** : Vous gérez une newsletter. Les utilisateurs s'inscrivent parfois plusieurs fois par erreur. Vous devez nettoyer la liste avant l'envoi.

📝 **Énoncé** :
1.  Soit la liste `emails = ["user1@site.com", "admin@site.com", "user1@site.com", "guest@site.com"]`.
2.  Convertissez la liste en set pour supprimer les doublons.
3.  Ajoutez manuellement "new@site.com".
4.  Retirez "guest@site.com".
5.  Affichez le nombre d'emails uniques finaux.

📺 **Résultat attendu** :
```text
3 emails uniques prêts pour envoi.
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
emails_list: list[str] = ["user1@site.com", "admin@site.com", "user1@site.com", "guest@site.com"]

# 1. Conversion en set (Déduplication)
unique_emails: set[str] = set(emails_list)

# 2. Ajout
unique_emails.add("new@site.com")

# 3. Suppression (on utilise discard par sécurité, ou remove si on est sûr)
unique_emails.discard("guest@site.com")

# 4. Affichage
print(f"{len(unique_emails)} emails uniques prêts pour envoi.")
# Contient : user1, admin, new
```
</details>

### Exercice 2 - Analyse de Compétences (Intersection) {#exercice-2---analyse-competences}

🎯 **Objectif** : Utiliser les opérateurs d'ensembles (`&`).

💼 **Mise en situation** : Pour un projet critique, vous cherchez un développeur qui maîtrise À LA FOIS Python et SQL.

📝 **Énoncé** :
1.  Créez le set `skills_candidate_a = {"Python", "Java", "Docker"}`.
2.  Créez le set `skills_candidate_b = {"Python", "SQL", "Git"}`.
3.  Créez le set `required_skills = {"Python", "SQL"}`.
4.  Vérifiez pour chaque candidat s'il possède **toutes** les compétences requises.
    *   *Astuce : Si (Candidat & Requis) == Requis, alors c'est bon.* ou utilisez `issubset`.

📺 **Résultat attendu** :
```text
Candidat A éligible : False
Candidat B éligible : True
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
skills_candidate_a: set[str] = {"Python", "Java", "Docker"}
skills_candidate_b: set[str] = {"Python", "SQL", "Git"}
required_skills: set[str] = {"Python", "SQL"}

# Méthode 1 : Intersection
# On regarde si l'intersection contient TOUTES les compétences requises
is_a_qualified: bool = (skills_candidate_a & required_skills) == required_skills
is_b_qualified: bool = (skills_candidate_b & required_skills) == required_skills

# Méthode 2 (Plus élégante) : issubset (<=)
# is_a_qualified = required_skills.issubset(skills_candidate_a)

print(f"Candidat A éligible : {is_a_qualified}")
print(f"Candidat B éligible : {is_b_qualified}")
```
</details>

### Exercice 3 - Audit de Sécurité (Différence) {#exercice-3---audit-securite}

🎯 **Objectif** : Utiliser la différence (`-`) pour trouver des anomalies.

💼 **Mise en situation** : Vous comparez la liste des personnes présentes dans le bâtiment avec la liste des employés autorisés ce jour-là.

📝 **Énoncé** :
1.  `authorized_ids = {101, 102, 103, 104}`.
2.  `scanned_ids = {101, 103, 105, 106}` (Personnes badgées à l'entrée).
3.  Identifiez les "Intrus" (ceux qui sont scannés mais PAS autorisés).
4.  Identifiez les "Absents" (ceux qui sont autorisés mais PAS scannés).

📺 **Résultat attendu** :
```text
Intrus détectés : {105, 106}
Absents : {102, 104}
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
authorized_ids: set[int] = {101, 102, 103, 104}
scanned_ids: set[int] = {101, 103, 105, 106}

# Intrus : Présents DANS scannés MAIS PAS DANS autorisés
intruders = scanned_ids - authorized_ids

# Absents : Présents DANS autorisés MAIS PAS DANS scannés
absents = authorized_ids - scanned_ids

print(f"Intrus détectés : {intruders}")
print(f"Absents : {absents}")
```
</details>