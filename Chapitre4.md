---
sidebar_label: Opérateurs en Python
sidebar_position: 4
---

# Chapitre 4 : Opérateurs en Python

Opérateurs arithmétiques, Opérateurs de comparaison, Opérateurs logiques

## Opérateurs Arithmétiques {#operateurs-arithmetiques}

### 1. Quoi
Les symboles qui permettent d'effectuer des calculs mathématiques sur des nombres (entiers ou décimaux). Python propose les classiques (`+`, `-`, `*`, `/`) mais aussi des outils plus puissants spécifiques au développement.

### 2. Pourquoi
Au-delà de la simple calculatrice, ces opérateurs sont au cœur de la logique métier : calcul de TVA, pagination d'une interface (modulo), conversion d'unités ou calculs de ROI.

### 3. Comment

#### A. Syntaxe de base

| Opérateur | Nom | Description | Exemple | Résultat |
| :--- | :--- | :--- | :--- | :--- |
| `+` | Addition | Ajoute deux nombres | `10 + 2` | `12` |
| `-` | Soustraction | Soustrait le deuxième nombre du premier | `10 - 2` | `8` |
| `*` | Multiplication | Multiplie deux nombres | `10 * 2` | `20` |
| `/` | Division | Divise (résultat **toujours** `float`) | `10 / 2` | `5.0` |
| `//` | Division entière | Divise et tronque la partie décimale | `10 // 3` | `3` |
| `%` | Modulo | Retourne le **reste** de la division | `10 % 3` | `1` |
| `**` | Puissance | Élève un nombre à une puissance | `10 ** 2` | `100` |

#### B. Cas concret : Calcul de panier E-commerce

```python
# Prix hors taxe
price_ht = 150.00
# Quantité
quantity = 3
# Taux de TVA (20%)
vat_rate = 0.20

# 1. Calcul du total HT
total_ht = price_ht * quantity 

# 2. Calcul du montant de la TVA
vat_amount = total_ht * vat_rate

# 3. Calcul du total TTC
total_ttc = total_ht + vat_amount

print(f"Total TTC : {total_ttc} €") 
# Affiche: Total TTC : 540.0 €
```

#### C. Exemples pratiques

**Cas 1 : Pagination (Division entière et Modulo)**
```python
total_users = 145
users_per_page = 10

# Combien de pages complètes ?
full_pages = total_users // users_per_page # Résultat: 14

# Combien d'utilisateurs sur la dernière page ?
remaining_users = total_users % users_per_page # Résultat: 5
```

**Cas 2 : Alternance de styles (Modulo)**
```python
# Utile pour appliquer une couleur 'zébrée' dans un tableau
row_index = 4
is_even = (row_index % 2) == 0 # True si pair, False si impair
```

**Cas 3 : Calculs financiers exponentiels (Puissance)**
```python
# Intérêts composés : Capital * (1 + taux)^années
principal = 1000
rate = 0.05
years = 10

future_value = principal * ((1 + rate) ** years)
```

### 4. Zone de Danger

#### ❌ L'imprécision des flottants
En informatique, les nombres à virgule flottante (`float`) ne sont pas exacts à 100% à cause de leur stockage en binaire.

```python
# ❌ À ne pas faire pour des comparaisons strictes
print(0.1 + 0.2) 
# Affiche: 0.30000000000000004 (et non 0.3)

print(0.1 + 0.2 == 0.3) 
# Affiche: False 😱
```

#### ✅ La solution robuste
Utilisez `math.isclose()` pour comparer des flottants, ou le module `decimal` pour l'argent.

```python
import math

# ✅ Bonne pratique
# On vérifie si les nombres sont "suffisamment proches"
is_equal = math.isclose(0.1 + 0.2, 0.3)
print(is_equal) # True
```

---

## Opérateurs de Comparaison {#operateurs-de-comparaison}

### 1. Quoi
Outils permettant de comparer deux valeurs. Ils retournent **toujours** un booléen : `True` ou `False`.

### 2. Pourquoi
Indispensables pour prendre des décisions (conditions `if`) : l'utilisateur est-il majeur ? Le stock est-il suffisant ? Le mot de passe est-il correct ?

### 3. Comment

#### A. Syntaxe de base

| Opérateur | Signification | Exemple (`a=10`) | Résultat |
| :--- | :--- | :--- | :--- |
| `==` | Égal à (valeur) | `a == 10` | `True` |
| `!=` | Différent de | `a != 5` | `True` |
| `>` | Strictement supérieur | `a > 10` | `False` |
| `<` | Strictement inférieur | `a < 20` | `True` |
| `>=` | Supérieur ou égal | `a >= 10` | `True` |
| `<=` | Inférieur ou égal | `a <= 10` | `True` |

#### B. Cas concret : Validation d'âge (SaaS)

```python
user_age = 17
min_age = 18
has_parental_consent = True

# Vérification simple
is_adult = user_age >= min_age

# Vérification composite (Logique métier)
can_access = is_adult or has_parental_consent

print(f"Accès autorisé : {can_access}")
```

#### C. Comparaison chaînée (Feature unique à Python 🐍)
Python permet d'enchaîner les comparaisons de manière mathématique naturelle.

```python
temperature = 22

# ❌ Style classique (Verbeux)
is_comfortable = temperature >= 18 and temperature <= 25

# ✅ Style Pythonique (Élégant)
is_comfortable = 18 <= temperature <= 25
```

### 4. Zone de Danger : `==` vs `is`

C'est la confusion la plus fréquente en entretien d'embauche.

*   `==` compare les **valeurs** (le contenu est-il le même ?).
*   `is` compare l'**identité** (est-ce le même objet en mémoire ?).

```python
# Deux listes distinctes avec le même contenu
list_a = [1, 2, 3]
list_b = [1, 2, 3]

print(list_a == list_b) # ✅ True (Même contenu)
print(list_a is list_b) # ❌ False (Deux objets différents en mémoire)

# Cas des singletons (None, True, False)
status = None
print(status is None)   # ✅ Bonne pratique pour None
print(status == None)   # ⚠️ Fonctionne mais moins "Pythonique"
```

---

## Opérateurs Logiques {#operateurs-logiques}

### 1. Quoi
Les mots-clés `and`, `or`, et `not` permettent de combiner plusieurs conditions booléennes.

### 2. Pourquoi
La logique métier est rarement simple. Une action requiert souvent plusieurs pré-conditions (Ex: "Utilisateur connecté" ET ("Admin" OU "Modérateur")).

### 3. Comment

#### A. Table de vérité simplifiée

*   `and` : Tout doit être Vrai pour être Vrai.
*   `or` : Au moins un élément doit être Vrai pour être Vrai.
*   `not` : Inverse le résultat (Vrai devient Faux).

#### B. Cas concret : Système de permissions

```python
is_logged_in = True
is_admin = False
has_premium_plan = True

# L'utilisateur peut voir le dashboard s'il est connecté ET (est admin OU a un plan premium)
can_view_dashboard = is_logged_in and (is_admin or has_premium_plan)

print(f"Accès Dashboard : {can_view_dashboard}") # True
```

#### C. "Truthiness" (La Vérité implicite)
En Python, toute valeur a une "valeur de vérité".
*   **Falsy (Considéré Faux)** : `0`, `0.0`, `""` (chaîne vide), `[]` (liste vide), `None`, `False`.
*   **Truthy (Considéré Vrai)** : Tout le reste.

```python
username = ""
# Pas besoin d'écrire: if username == "":
if not username:
    print("Le nom d'utilisateur est vide !")
```

#### D. Évaluation "Short-Circuit" (Court-circuit)
Python est paresseux (dans le bon sens du terme). Il arrête d'évaluer dès qu'il connaît la réponse finale.

```python
def expensive_operation():
    print("Opération coûteuse lancée...")
    return True

# Avec OR : Si le premier est Vrai, Python n'exécute PAS le second
result = True or expensive_operation() 
# "Opération coûteuse lancée..." NE S'AFFICHE PAS
```

### 🚨 Limitations de l'assignation par `or`
Une technique courante pour définir des valeurs par défaut est :
`name = input_name or "Anonyme"`

Attention : si `input_name` vaut `0` (zéro), Python le considère comme `False` et prendra la valeur par défaut. Si `0` est une valeur valide pour votre métier, n'utilisez pas cette astuce.

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-4}

1.  **Quel est le résultat de `15 // 2` ?**
    *   `7` (C'est une division entière).
2.  **Quelle est la différence entre `==` et `is` ?**
    *   `==` compare les valeurs, `is` compare l'identité (l'adresse mémoire).
3.  **Pourquoi `0.1 + 0.2 == 0.3` retourne `False` ?**
    *   À cause de l'imprécision du stockage des nombres flottants en binaire.
4.  **Si `a = True` et `b = False`, est-ce que `a or b` évalue `b` ?**
    *   Non, grâce au court-circuit (short-circuit), Python s'arrête dès que `a` est Vrai.
5.  **Quelle est la valeur booléenne d'une liste vide `[]` ?**
    *   `False`.

---

## Exercices : {#exercices-:-4}

### Exercice 1 - La pagination intelligente {#exercice-1---la-pagination-intelligente}
**🎯 Objectif** : Maîtriser la division entière et le modulo.
**💼 Mise en situation** : Vous développez le backend d'un blog. Vous avez 53 articles et vous devez afficher 10 articles par page. Vous devez calculer le nombre de pages nécessaires et identifier l'index du dernier article.

**📝 Énoncé** :
1. Déclarez `total_articles = 53` et `articles_per_page = 10`.
2. Calculez le nombre de pages nécessaires (Attention : s'il reste des articles, il faut une page de plus !). *Indice: Regardez du côté des mathématiques ou de la logique pure.*
3. Affichez le résultat.

**📺 Résultat attendu** :
```text
Nombre de pages nécessaires : 6
```

<details>
<summary>💡 Voir la solution</summary>

```python
import math

total_articles = 53
articles_per_page = 10

# Approche 1 : Logique pure avec opérateurs
# On divise. S'il y a un reste (modulo > 0), on ajoute 1 page.
pages = (total_articles // articles_per_page) + (1 if total_articles % articles_per_page > 0 else 0)

# Approche 2 : Math (Plus propre pour la production)
# On divise et on arrondit à l'entier supérieur (ceil = ceiling = plafond)
pages_math = math.ceil(total_articles / articles_per_page)

print(f"Nombre de pages nécessaires : {pages}")
```
</details>

---

### Exercice 2 - Le videur de boîte de nuit {#exercice-2---le-videur-de-boite-de-nuit}
**🎯 Objectif** : Combiner opérateurs de comparaison et logiques.
**💼 Mise en situation** : Vous codez le système d'entrée d'un club select.

**📝 Énoncé** :
Déclarez les variables suivantes :
*   `age` (int)
*   `has_id` (bool) : A sa carte d'identité
*   `is_vip` (bool) : Est sur la liste VIP

Les règles d'entrée (`can_enter`) sont :
1. Il faut être majeur (>= 18) ET avoir sa carte d'identité.
2. OU ALORS, être VIP (les VIP rentrent peu importe l'âge ou la carte dans cet exercice simplifié).

Testez avec `age = 17`, `has_id = True`, `is_vip = False`.

**📺 Résultat attendu** :
```text
Peut entrer : False
```

<details>
<summary>💡 Voir la solution</summary>

```python
age = 17
has_id = True
is_vip = False

# Les parenthèses clarifient la priorité :
# (Majeur ET Carte) OU VIP
can_enter = (age >= 18 and has_id) or is_vip

print(f"Peut entrer : {can_enter}")
```
</details>

---

### Exercice 3 - Le simulateur d'épargne (Intérêts composés) {#exercice-3---le-simulateur-depargne}
**🎯 Objectif** : Utiliser l'opérateur de puissance et gérer l'affichage.
**💼 Mise en situation** : Une Fintech veut montrer à ses clients combien ils gagneront en plaçant leur argent.

**📝 Énoncé** :
Formule : `Montant Final = Capital * (1 + Taux/100) puissance Années`
1. Capital : 5000 €
2. Taux : 3 %
3. Années : 5
4. Calculez le montant final.
5. Utilisez une comparaison pour afficher `True` si le montant final dépasse 5500 €.

**📺 Résultat attendu** :
```text
Montant final : 5796.37...
Objectif atteint : True
```

<details>
<summary>💡 Voir la solution</summary>

```python
capital = 5000
rate = 3
years = 5

# Calcul avec l'opérateur puissance **
# Attention à diviser le taux par 100 pour avoir 0.03
final_amount = capital * ((1 + rate/100) ** years)

is_goal_reached = final_amount > 5500

print(f"Montant final : {final_amount}")
print(f"Objectif atteint : {is_goal_reached}")
```
</details>