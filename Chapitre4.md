---
sidebar_label: Opérateurs arithmétiques et de comparaison
sidebar_position: 4
---

# Chapitre 4 : Opérateurs arithmétiques et de comparaison

Opérateurs arithmétiques (+, -, *, /, //, %, **), Opérateurs de comparaison (==, !=, &lt;, &gt;, &lt;=, &gt;=), Priorité des opérateurs

---

## 1. Opérateurs arithmétiques {#operateurs-arithmetiques}

### 1. Quoi
Les opérateurs arithmétiques sont des symboles spéciaux qui effectuent des calculs mathématiques sur des opérandes (nombres ou variables). Python 3 propose, en plus des classiques additions et multiplications, des opérateurs spécifiques pour la division entière et le modulo.

### 2. Pourquoi
Au-delà des calculs financiers évidents (prix, taxes), ces opérateurs sont fondamentaux pour :
- **La logique de pagination** (calculer le nombre de pages avec la division entière).
- **La distribution de tâches** (répartir des items avec le modulo).
- **La manipulation de données** (conversions d'unités, calculs de délais).

### 3. Comment

#### A. Syntaxe de base

| Symbole | Nom | Description | Exemple | Résultat |
| :---: | --- | --- | --- | --- |
| `+` | Addition | Somme de deux nombres | `10 + 2` | `12` |
| `-` | Soustraction | Différence | `10 - 2` | `8` |
| `*` | Multiplication | Produit | `10 * 2` | `20` |
| `/` | Division | **Toujours** un float | `10 / 2` | `5.0` |
| `//` | Division entière | Partie entière du quotient | `10 // 3` | `3` |
| `%` | Modulo | Reste de la division | `10 % 3` | `1` |
| `**` | Puissance | Élévation à la puissance | `2 ** 3` | `8` |

#### B. Cas concret : Calcul de panier E-commerce

```python
# Prix en centimes pour éviter les erreurs de flottants (bonne pratique)
product_price: int = 2999  # 29.99€
quantity: int = 3
tax_rate: float = 0.20     # 20% TVA

# Calculs
subtotal: int = product_price * quantity
tax_amount: float = subtotal * tax_rate
total: float = subtotal + tax_amount

# Affichage formaté (f-string vue au Chapitre 6, anticipée ici)
print(f"Total à payer : {total / 100} €") 
# Résultat : Total à payer : 107.964 €
```

#### C. Exemples pratiques

**Cas 1 : Pagination (Division entière)**
```python
total_users: int = 145
users_per_page: int = 10

# Combien de pages complètes ?
full_pages: int = total_users // users_per_page  # 14
```

**Cas 2 : Cycle / Parité (Modulo)**
```python
current_user_index: int = 42

# Assigner une couleur alternée (pair/impair)
is_row_even: bool = (current_user_index % 2) == 0

# Assigner à un shard de base de données (ex: 3 shards)
shard_id: int = current_user_index % 3  # Résultat: 0, 1 ou 2
```

**Cas 3 : Calcul exponentiel (Puissance)**
```python
side_length: int = 5
cube_volume: int = side_length ** 3  # 125
```

### 4. Zone de Danger

#### ❌ À ne pas faire
```python
# Utiliser / quand vous attendez un entier pour un index
index = 10 / 2
my_list[index] # TypeError: list indices must be integers, not float
```

#### ✅ Bonne Pratique
```python
# Utiliser // pour obtenir un entier garanti
index = 10 // 2
my_list[index] # Fonctionne
```

### 🚨 Limitations des nombres flottants
Les ordinateurs stockent les nombres à virgule flottante (`float`) en binaire, ce qui entraîne des imprécisions.

```python
print(0.1 + 0.2)
# Affiche : 0.30000000000000004
# ET NON : 0.3
```
> **Solution** : Pour les calculs financiers critiques, utilisez le module `decimal` ou travaillez en entiers (centimes).

---

## 2. Opérateurs de comparaison {#operateurs-de-comparaison}

### 1. Quoi
Ces opérateurs comparent deux valeurs et renvoient toujours un **booléen** (`True` ou `False`).

### 2. Pourquoi
Ils sont le moteur de la prise de décision dans vos programmes (conditions `if`, boucles `while`). Sans eux, un programme serait linéaire et incapable de réagir à des données variables (ex: "L'utilisateur est-il admin ?", "Le stock est-il suffisant ?").

### 3. Comment

#### A. Syntaxe de base

| Symbole | Signification | Mathématiques |
| :---: | --- | :---: |
| `==` | Égal à | $=$ |
| `!=` | Différent de | $\neq$ |
| `<` | Strictement inférieur | $\lt$ |
| `>` | Strictement supérieur | $\gt$ |
| `<=` | Inférieur ou égal | $\leq$ |
| `>=` | Supérieur ou égal | $\geq$ |

#### B. Cas concret : Validation d'âge SaaS

```python
user_age: int = 25
min_age: int = 18
max_age: int = 120

# Vérification standard
is_adult: bool = user_age >= min_age

# Comparaison chaînée (Spécificité Pythonique élégante)
is_valid_human_age: bool = min_age <= user_age <= max_age
# Équivalent à : user_age >= min_age AND user_age <= max_age

print(f"Adulte: {is_adult}, Âge valide: {is_valid_human_age}")
```

#### C. Exemples pratiques

**Cas 1 : Comparaison de chaînes (Ordre lexicographique)**
```python
# Attention : 'apple' > 'Apple' car 'a' (97) > 'A' (65) en ASCII
print("apple" > "Apple")  # True
print("a" < "b")          # True
```

**Cas 2 : Vérification de statut**
```python
status_code: int = 404
is_error: bool = status_code != 200  # True
```

**Cas 3 : Comparaison de types différents**
```python
# Un entier peut être égal à un float mathématiquement
print(10 == 10.0)  # True

# Mais une chaîne n'est jamais égale à un nombre
print(10 == "10")  # False
```

### 4. Zone de Danger

#### ❌ Confusion Assignation vs Comparaison
```python
# Erreur classique de débutant
if x = 5:  # SyntaxError
    pass
```

#### ✅ Bonne Pratique
```python
# Utiliser le double égal pour comparer
if x == 5:
    print("x vaut 5")
```

---

## 3. Priorité des opérateurs {#priorite-des-operateurs}

### 1. Quoi
La priorité détermine l'ordre dans lequel Python évalue les différentes parties d'une expression mathématique complexe. C'est l'équivalent des règles de priorité en mathématiques (PEMDAS).

### 2. Pourquoi
Ne pas maîtriser la priorité conduit à des bugs logiques silencieux où le programme ne plante pas, mais le résultat est faux.

### 3. Comment

#### A. Hiérarchie simplifiée (du plus prioritaire au moins prioritaire)

1.  **Parenthèses** `()` : Forcent la priorité.
2.  **Puissance** `**` : Attention, s'évalue de droite à gauche.
3.  **Multiplicatifs** `*`, `/`, `//`, `%`.
4.  **Additifs** `+`, `-`.
5.  **Comparaisons** `==`, `<`, `>`, etc.

#### B. Cas concret : Calcul de moyenne pondérée

```python
note_math: int = 15
coef_math: int = 3
note_anglais: int = 12
coef_anglais: int = 2

# ❌ Mauvais calcul (Priorité de la division sur l'addition)
moyenne_fausse = note_math * coef_math + note_anglais * coef_anglais / coef_math + coef_anglais
# Interprété comme : (15*3) + (12*2/3) + 2 = 45 + 8 + 2 = 55 (Impossible sur 20)

# ✅ Bon calcul (Parenthèses explicites)
total_points = (note_math * coef_math) + (note_anglais * coef_anglais)
total_coef = coef_math + coef_anglais
moyenne_reelle = total_points / total_coef  # (45 + 24) / 5 = 13.8
```

### 4. Zone de Danger : `**`

L'opérateur puissance est associatif à droite, contrairement aux autres.

```python
print(2 ** 3 ** 2) 
# Est évalué comme 2 ** (3 ** 2) -> 2 ** 9 -> 512
# ET NON PAS (2 ** 3) ** 2 -> 8 ** 2 -> 64
```

> **Conseil** : Dans le doute, utilisez **toujours** des parenthèses. Cela rend votre code lisible et sans ambiguïté.

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-validation-des-acquis-du-chapitre-4}

1.  **Quel type de donnée retourne l'expression `10 / 2` en Python 3 ?**
    *   Un `float` (`5.0`), même si la division est parfaite. Pour un entier, il faut utiliser `//`.

2.  **Que retourne `15 % 4` et à quoi cela sert-il ?**
    *   Cela retourne `3`. C'est le reste de la division. Utile pour déterminer la parité ou cycler sur des index.

3.  **L'expression `10 < x < 20` est-elle valide en Python ?**
    *   Oui, Python supporte le chaînage d'opérateurs de comparaison. C'est équivalent à `10 < x and x < 20`.

4.  **Pourquoi `0.1 + 0.2 == 0.3` retourne `False` ?**
    *   À cause de l'imprécision des nombres flottants en binaire. `0.1 + 0.2` donne `0.30000000000000004`.

---

## Exercices : {#exercices-:-4}

### Exercice 1 - Le Calculateur de TVA SaaS {#exercice-1---le-calculateur-de-tva-saas}

**🎯 Objectif** : Maîtriser les opérateurs arithmétiques et le typage.

**💼 Mise en situation** : Vous développez le module de facturation d'un SaaS B2B. Vous devez calculer le montant HT, la TVA et le TTC à partir d'un prix unitaire et d'une quantité.

**📝 Énoncé** :
1.  Déclarez un prix unitaire hors taxe de **50** (entier).
2.  Déclarez une quantité de **12** (entier).
3.  Déclarez un taux de TVA de **20.0** (float, pour 20%).
4.  Calculez le total HT.
5.  Calculez le montant de la TVA (Total HT * Taux / 100).
6.  Calculez le total TTC.
7.  Affichez les trois montants.

**📺 Résultat attendu** :
```text
Total HT: 600
Montant TVA: 120.0
Total TTC: 720.0
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
# Déclaration des variables
price_unit_ht: int = 50
quantity: int = 12
vat_rate: float = 20.0

# Calculs arithmétiques
total_ht: int = price_unit_ht * quantity
vat_amount: float = total_ht * (vat_rate / 100)
total_ttc: float = total_ht + vat_amount

# Affichage
print(f"Total HT: {total_ht}")
print(f"Montant TVA: {vat_amount}")
print(f"Total TTC: {total_ttc}")
```
</details>

---

### Exercice 2 - Le Gestionnaire de Pagination {#exercice-2---le-gestionnaire-de-pagination}

**🎯 Objectif** : Utiliser `/`, `//` et `%` pour résoudre un problème logique.

**💼 Mise en situation** : Votre application E-commerce affiche des produits. Vous avez 45 produits et vous en affichez 10 par page. Vous devez calculer le nombre de pages nécessaires.

**📝 Énoncé** :
1.  Variable `total_products = 45`.
2.  Variable `products_per_page = 10`.
3.  Calculez le nombre de pages pleines (utilisez `//`).
4.  Calculez le nombre de produits restants sur la dernière page (utilisez `%`).
5.  Déterminez le nombre total de pages nécessaires (si il reste des produits, il faut une page de plus). *Astuce: Essayez de le faire sans `if` pour l'instant, ou simplement calculez les deux valeurs séparément.*

**📺 Résultat attendu** :
```text
Pages pleines : 4
Produits sur la dernière page : 5
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
total_products: int = 45
products_per_page: int = 10

# Division entière pour les pages complètes
full_pages: int = total_products // products_per_page

# Modulo pour le reste
remaining_products: int = total_products % products_per_page

# Calcul astucieux du total de pages (avancé)
# Si remaining_products > 0, on ajoute 1, sinon 0. 
# En Python, True vaut 1 et False vaut 0.
total_pages_needed = full_pages + (remaining_products > 0)

print(f"Pages pleines : {full_pages}")
print(f"Produits sur la dernière page : {remaining_products}")
print(f"Total pages nécessaires : {total_pages_needed}")
```
</details>

---

### Exercice 3 - Le Validateur de Stock Logistique {#exercice-3---le-validateur-de-stock-logistique}

**🎯 Objectif** : Maîtriser les opérateurs de comparaison et le chaînage.

**💼 Mise en situation** : Dans un entrepôt automatisé, un robot ne doit déplacer une palette que si son poids est conforme aux normes de sécurité : entre 10kg (inclus) et 100kg (inclus), et différent de 50kg (une valeur réservée pour la maintenance).

**📝 Énoncé** :
1.  Définissez `min_weight = 10` et `max_weight = 100`.
2.  Définissez `forbidden_weight = 50`.
3.  Testez avec une variable `pallet_weight = 50`.
4.  Créez un booléen `is_compliant` qui est `True` si le poids est dans la fourchette ET n'est pas interdit. Utilisez le chaînage pour la fourchette.
5.  Affichez le résultat.

**📺 Résultat attendu** :
```text
Poids palette : 50
Conforme aux normes : False
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
# Configuration des règles métier
min_weight: int = 10
max_weight: int = 100
forbidden_weight: int = 50

# Cas de test
pallet_weight: int = 50

# Logique de validation en une ligne
# 1. On vérifie la fourchette avec le chaînage
# 2. On vérifie la valeur interdite
is_compliant: bool = (min_weight <= pallet_weight <= max_weight) and (pallet_weight != forbidden_weight)

print(f"Poids palette : {pallet_weight}")
print(f"Conforme aux normes : {is_compliant}")
```
</details>