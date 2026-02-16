---
sidebar_label: Opérateurs de Comparaison et Logiques
sidebar_position: 6
---

# Chapitre 6 : Opérateurs de Comparaison et Logiques

Égalité, Inégalité, Supérieur/Inférieur, Opérateurs logiques (and, or, not)

Un programme informatique ne se contente pas de calculer ; il doit prendre des décisions. "Si l'utilisateur est majeur", "Tant qu'il reste du stock", "Si le mot de passe est correct ET l'utilisateur est actif".

Pour formuler ces hypothèses, Python utilise des **opérateurs de comparaison** (qui comparent deux valeurs) et des **opérateurs logiques** (qui combinent ces comparaisons). Le résultat de ces opérations est toujours un booléen : `True` ou `False`.

---

## 1. Opérateurs de Comparaison {#operateurs-de-comparaison}

### 1. Quoi
Les outils permettant de comparer deux valeurs. Ils répondent à des questions comme "Est-ce égal ?", "Est-ce plus grand ?", "Est-ce différent ?".

### 2. Pourquoi
Sans comparaison, un programme serait linéaire et incapable de réagir à des données variables (saisie utilisateur, lecture de capteur, résultat de base de données).

### 3. Comment

#### A. Syntaxe de base

| Opérateur | Signification | Exemple | Résultat (si a=10, b=20) |
| :--- | :--- | :--- | :--- |
| `==` | Égal à | `a == b` | `False` |
| `!=` | Différent de | `a != b` | `True` |
| `>` | Supérieur strict | `b > a` | `True` |
| `<` | Inférieur strict | `a < b` | `True` |
| `>=` | Supérieur ou égal | `a >= 10` | `True` |
| `<=` | Inférieur ou égal | `b <= 15` | `False` |

#### B. Cas concret : Vérification d'âge

```python
user_age: int = 17
legal_age: int = 18

is_adult: bool = user_age >= legal_age # False

print(f"L'utilisateur est majeur : {is_adult}")
```

#### C. Comparaisons Chaînées (Spécificité Python)
Python permet d'enchaîner les comparaisons de manière mathématique naturelle, ce qui est rare dans d'autres langages.

```python
temperature: int = 22

# Vérifie si 18 <= temperature ET temperature < 25
# Beaucoup plus lisible que : temperature >= 18 and temperature < 25
is_comfortable: bool = 18 <= temperature < 25

print(f"Confortable : {is_comfortable}") # True
```

### 4. Zone de Danger

❌ **Confusion `=` vs `==`** :
*   `=` est l'**assignation** (donner une valeur).
*   `==` est la **comparaison** (vérifier l'égalité).
*   `if a = 10:` provoquera une `SyntaxError`.

❌ **Typage Fort** :
`10 == "10"` retourne `False`. Python ne convertit pas implicitement les types lors d'une comparaison (contrairement au Javascript `==`).

✅ **Bonne Pratique** :
Pour vérifier si une variable est `None`, utilisez l'opérateur d'identité `is` et non l'égalité `==`.
```python
value = None
if value is None: # ✅ Correct
    pass
if value == None: # ❌ Déconseillé (bien que fonctionnel souvent)
    pass
```

---

## 2. Opérateurs Logiques (and, or, not) {#operateurs-logiques-and-or-not}

### 1. Quoi
Ces opérateurs permettent de combiner plusieurs conditions booléennes pour former des expressions complexes. Ce sont les portes logiques de votre code.

### 2. Pourquoi
La vie réelle est rarement binaire. Une condition d'accès peut être : "Avoir un billet valide" **ET** ("Être majeur" **OU** "Être accompagné").

### 3. Comment

#### A. Table de vérité simplifiée

*   **`and`** : `True` seulement si **TOUT** est Vrai.
*   **`or`** : `True` si **AU MOINS UN** est Vrai.
*   **`not`** : Inverse le résultat (Vrai devient Faux).

#### B. Exemple complet

```python
has_ticket: bool = True
is_vip: bool = False
is_staff: bool = True

# Accès si on a un billet OU si on fait partie du staff
can_enter: bool = has_ticket or is_staff  # True

# Accès VIP : Billet ET statut VIP
can_enter_vip: bool = has_ticket and is_vip # False

# Refus d'entrée (Négation)
is_banned: bool = False
can_enter_safe: bool = can_enter and not is_banned # True
```

#### C. Évaluation "Paresseuse" (Short-circuit evaluation)
Python optimise les tests logiques. Il s'arrête dès que le résultat est certain.

```python
def check_database():
    print("Vérification BDD...")
    return True

# Ici, check_database() n'est JAMAIS appelée
# car la première condition (False) rend le "and" impossible à valider.
if False and check_database():
    print("Ok")
```

### 4. Zone de Danger

❌ **Priorité des opérateurs** :
`not` est prioritaire sur `and`, qui est prioritaire sur `or`.
En cas de doute, utilisez des parenthèses `()` pour rendre l'intention explicite.

```python
# Ambigu
result = True or False and False 

# Explicite
result = True or (False and False)
```

---

## 3. Truthiness : Valeurs de Vérité Implicites {#truthiness-valeurs-de-verite-implicites}

### 1. Quoi
En Python, tout objet peut être évalué dans un contexte booléen. Certains objets sont considérés comme "Faux" par nature, tous les autres sont "Vrais".

### 2. Pourquoi
Cela permet d'écrire du code extrêmement concis et lisible ("Pythonic").

### 3. Comment

#### A. Les valeurs "Falsy" (considérées comme False)
*   `False`
*   `None`
*   Le zéro numérique : `0`, `0.0`
*   Les séquences vides : `""` (string vide), `[]` (liste vide), `()` (tuple vide), `{}` (dico vide)

#### B. Utilisation pratique

```python
username: str = ""
emails: list[str] = []

# Au lieu de : if username != "":
if not username:
    print("Nom d'utilisateur requis")

# Au lieu de : if len(emails) > 0:
if emails:
    print(f"Vous avez {len(emails)} emails")
else:
    print("Boîte vide")
```

### 4. Zone de Danger

❌ **Le piège du Zéro** :
Si `0` est une valeur valide (ex: score d'un jeu, position d'un index), ne vérifiez pas l'existence avec `if value:`.

```python
score: int = 0

# Mauvais si 0 est un score valide
if score: 
    print("Le joueur a joué") 
# Ne s'affichera pas car 0 est Falsy !

# Bon
if score is not None:
    print("Le joueur a joué")
```

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-6}

1.  **Quel est le résultat de `10 == "10"` en Python ?**
    `False`. Python est fortement typé et ne considère pas qu'un entier est égal à une chaîne de caractères contenant les mêmes chiffres.

2.  **Quelle est la particularité de l'expression `5 < x < 10` ?**
    C'est une comparaison chaînée. Python vérifie si `x` est strictement supérieur à 5 **ET** strictement inférieur à 10 en une seule expression lisible.

3.  **Qu'est-ce que l'évaluation paresseuse (short-circuit) ?**
    C'est le fait que Python arrête d'évaluer une expression logique dès que le résultat est définitif (ex: `True or ...` s'arrête tout de suite car le résultat sera forcément True).

4.  **Citez trois valeurs considérées comme `False` (Falsy) en Python.**
    `None`, `0`, `""` (chaîne vide), `[]` (liste vide), `False`.

---

## Exercices : {#exercices-6}

### Exercice 1 - Le Contrôleur de Train {#exercice-1---le-controleur-de-train}

🎯 **Objectif** : Combiner comparaisons et opérateurs logiques.

💼 **Mise en situation** : Vous codez la borne d'accès d'un métro. Le tarif réduit s'applique aux moins de 12 ans OU aux plus de 65 ans.

📝 **Énoncé** :
1.  Déclarez `age` (int) à 70.
2.  Déclarez un booléen `is_student` à `False`.
3.  Créez une variable `has_discount` qui vaut `True` si :
    *   L'âge est < 12 OU l'âge est > 65
    *   OU si la personne est étudiante.
4.  Affichez le résultat.

📺 **Résultat attendu** :
```text
Droit au tarif réduit : True
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
age: int = 70
is_student: bool = False

# Les parenthèses clarifient que l'âge est un groupe de conditions
# Bien que 'or' soit associatif, c'est une bonne pratique de grouper la logique métier
has_discount: bool = (age < 12 or age > 65) or is_student

print(f"Droit au tarif réduit : {has_discount}")
```
</details>

### Exercice 2 - Validation de commande E-commerce {#exercice-2---validation-de-commande-e-commerce}

🎯 **Objectif** : Utiliser la logique `and` et la négation `not`.

💼 **Mise en situation** : Une commande ne peut être validée que si le panier n'est pas vide ET si l'utilisateur a renseigné une adresse.

📝 **Énoncé** :
1.  Déclarez `cart_amount` (int) à 0 (panier vide).
2.  Déclarez `has_address` (bool) à `True`.
3.  Déclarez `can_checkout` qui vérifie que le montant est > 0 ET qu'il a une adresse.
4.  Affichez un message d'erreur explicite en utilisant `not` si le checkout est impossible.

📺 **Résultat attendu** :
```text
Commande impossible : Panier vide ou adresse manquante.
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
cart_amount: int = 0
has_address: bool = True

# La condition de succès
can_checkout: bool = (cart_amount > 0) and has_address

if not can_checkout:
    print("Commande impossible : Panier vide ou adresse manquante.")
else:
    print("Commande validée !")
```
</details>

### Exercice 3 - Le Système d'Alarme (Truthiness) {#exercice-3---le-systeme-d-alarme-truthiness}

🎯 **Objectif** : Utiliser la valeur de vérité implicite (Falsy/Truthy).

💼 **Mise en situation** : Une alarme doit sonner si la liste des `active_sensors` n'est pas vide.

📝 **Énoncé** :
1.  Déclarez une liste `active_sensors` contenant `["Motion_Hall", "Door_Front"]`.
2.  Déclarez une variable `is_system_armed` à `True`.
3.  Vérifiez si le système est armé ET s'il y a des capteurs actifs en utilisant la syntaxe Pythonique (pas de `len() > 0`).
4.  Si oui, affichez "ALERTE !".

📺 **Résultat attendu** :
```text
ALERTE ! Capteurs actifs détectés.
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
active_sensors: list[str] = ["Motion_Hall", "Door_Front"]
is_system_armed: bool = True

# Python évalue 'active_sensors' comme True car la liste n'est pas vide
if is_system_armed and active_sensors:
    print("ALERTE ! Capteurs actifs détectés.")
else:
    print("Système calme.")
```
</details>