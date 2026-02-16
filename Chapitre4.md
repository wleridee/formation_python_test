---
sidebar_label: Types Numériques et Opérations
sidebar_position: 4
---

# Chapitre 4 : Types Numériques et Opérations

Entiers (int), Flottants (float), Opérateurs arithmétiques, Opérations sur les nombres

Les nombres sont le cœur battant de l'informatique, de la simple boucle de répétition aux calculs complexes de l'intelligence artificielle. Python brille particulièrement dans ce domaine grâce à une syntaxe intuitive et une gestion puissante des grands nombres.

Dans ce chapitre, nous allons maîtriser les types numériques, comprendre leurs pièges (notamment avec les virgules flottantes) et utiliser les opérateurs arithmétiques comme des professionnels.

---

## 1. Les Entiers (`int`) {#les-entiers-int}

### 1. Quoi
Le type `int` représente les **nombres entiers** (sans partie décimale), qu'ils soient positifs, négatifs ou nuls.

Une particularité majeure de Python est que les entiers ont une **précision arbitraire**. Contrairement au C ou Java où un entier est limité à 32 ou 64 bits (provoquant des "overflows"), un entier Python peut être aussi grand que la mémoire de votre ordinateur le permet.

### 2. Pourquoi
Pour compter des éléments, gérer des identifiants, des index de boucles ou effectuer des calculs mathématiques exacts (arithmétique discrète). La précision infinie est un atout majeur pour la cryptographie ou les calculs scientifiques.

### 3. Comment

#### A. Syntaxe de base

```python
count: int = 10
negative: int = -5
zero: int = 0
```

#### B. Cas concret : Grands nombres
Pour la lisibilité, on peut utiliser le caractère `_` comme séparateur de milliers (purement visuel, ignoré par Python).

```python
# Population mondiale (lisible)
population: int = 8_000_000_000

# Calcul astronomique (automatiquement géré)
distance_to_star: int = 300_000 * 3600 * 24 * 365 * 4  # Années-lumière en km
print(distance_to_star) 
# Affiche: 37843200000000 (pas de dépassement de capacité)
```

#### C. Exemples pratiques

**Cas 1 : Compteurs et Incrémentation**
```python
visitors: int = 0
visitors += 1  # Opérateur in-place (équivaut à visitors = visitors + 1)
```

**Cas 2 : Hexadécimal et Binaire**
Les développeurs bas niveau utilisent souvent d'autres bases.
```python
mask_hex: int = 0xFF   # 255 en hexadécimal
flags_bin: int = 0b101 # 5 en binaire
```

### 4. Zone de Danger
❌ **Division par zéro** : `10 / 0` lève une `ZeroDivisionError`. Toujours vérifier le diviseur si c'est une variable utilisateur.

✅ **Bonne Pratique** : Utilisez des entiers pour tout ce qui est dénombrable (nombre d'articles, ID utilisateur). N'utilisez pas de `float` pour des valeurs qui doivent être exactes par nature (comme un index de liste).

---

## 2. Les Flottants (`float`) {#les-flottants-float}

### 1. Quoi
Le type `float` représente les **nombres réels** (à virgule). Ils sont implémentés selon la norme IEEE 754 (double précision 64 bits), ce qui signifie qu'ils ont une précision limitée.

### 2. Pourquoi
Indispensables pour la physique, les statistiques, les prix (avec précaution), et tout ce qui nécessite une partie fractionnaire.

### 3. Comment

#### A. Syntaxe de base

```python
pi: float = 3.14159
scientific: float = 1.5e2  # Notation scientifique (1.5 * 10^2 = 150.0)
```

#### B. Le piège de l'imprécision
L'ordinateur stocke les flottants en binaire. Certaines valeurs décimales simples (comme 0.1) n'ont pas de représentation binaire finie exacte.

```python
val_a: float = 0.1 + 0.2
print(val_a) 
# Affiche: 0.30000000000000004 (et non 0.3 !)

# Comparaison stricte dangereuse
if val_a == 0.3:
    print("Égal") # Ne s'affichera JAMAIS
```

#### C. Solution pour la comparaison
Pour comparer deux flottants, on vérifie s'ils sont "suffisamment proches" à l'aide d'une tolérance (epsilon) ou de `math.isclose`.

```python
import math

if math.isclose(val_a, 0.3):
    print("C'est (presque) égal !") # S'affiche correctement
```

### 🚨 Limitations des Flottants pour l'Argent
❌ N'utilisez JAMAIS `float` pour des calculs financiers critiques. Les erreurs d'arrondi s'accumulent (0.1 + 0.1 + 0.1 != 0.3).
✅ Pour l'argent, utilisez le module `decimal` (vu dans un chapitre ultérieur) ou stockez les valeurs en centimes dans des `int` (ex: 1000 pour 10.00€).

---

## 3. Opérateurs Arithmétiques {#operateurs-arithmetiques}

### 1. Quoi
Les symboles qui permettent d'effectuer des calculs. Python en possède 7 principaux.

### 2. Pourquoi
Pour transformer la donnée brute en information utile.

### 3. Comment

#### A. Liste des opérateurs

| Symbole | Nom | Exemple | Résultat | Description |
| :--- | :--- | :--- | :--- | :--- |
| `+` | Addition | `10 + 5` | `15` | Somme standard |
| `-` | Soustraction | `10 - 5` | `5` | Différence standard |
| `*` | Multiplication | `10 * 5` | `50` | Produit standard |
| `/` | Division Réelle | `10 / 3` | `3.3333...` | Retourne **toujours** un `float` |
| `//` | Division Entière | `10 // 3` | `3` | Tronque la partie décimale (partie entière) |
| `%` | Modulo | `10 % 3` | `1` | Reste de la division entière |
| `**` | Puissance | `2 ** 3` | `8` | 2 au cube (2*2*2) |

#### B. Cas concret : Convertisseur de temps
Convertir 3665 secondes en heures, minutes, secondes.

```python
total_seconds: int = 3665

# Division entière pour les heures
hours: int = total_seconds // 3600  # 1

# Le reste pour les minutes/secondes
remaining_seconds: int = total_seconds % 3600 # 65

minutes: int = remaining_seconds // 60 # 1
seconds: int = remaining_seconds % 60 # 5

print(f"{hours}h {minutes}m {seconds}s")
# Affiche: 1h 1m 5s
```

### 4. Zone de Danger
❌ **Confusion `/` vs `//`** : En Python 3, `10 / 2` donne `5.0` (float), pas `5` (int). Si vous avez besoin d'un entier (pour un index de liste par exemple), utilisez toujours `//`.

---

## 4. Fonctions Numériques Utiles {#fonctions-numeriques-utiles}

### 1. Quoi
Des fonctions intégrées (built-in) pour manipuler les nombres sans écrire d'algorithmes complexes.

### 2. Comment

#### A. Valeur absolue et Arrondi

```python
temp_diff: int = abs(-5)     # 5
price: float = round(3.14159, 2) # 3.14
score: int = round(3.6)      # 4
```

#### B. Min et Max

```python
scores: list[int] = [10, 5, 20, 15]
best: int = max(scores) # 20
worst: int = min(scores) # 5
```

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-4}

1.  **Quel est le type de retour de l'opération `10 / 2` ?**
    C'est un `float` (`5.0`), même si la division tombe juste. Pour obtenir un entier, il faut utiliser `//`.

2.  **Pourquoi `0.1 + 0.2 == 0.3` retourne-t-il `False` ?**
    À cause de la représentation binaire des nombres flottants (IEEE 754) qui introduit une infime erreur de précision.

3.  **Que fait l'opérateur modulo `%` ?**
    Il retourne le *reste* de la division entière. Très utile pour savoir si un nombre est pair (`n % 2 == 0`) ou pour créer des cycles.

4.  **Quelle est la limite de taille d'un entier (`int`) en Python ?**
    Il n'y a pas de limite théorique fixe (comme 32 bits ou 64 bits). La seule limite est la mémoire RAM disponible sur la machine.

---

## Exercices : {#exercices-4}

### Exercice 1 - Pair ou Impair ? {#exercice-1---pair-ou-impair}

🎯 **Objectif** : Utiliser l'opérateur modulo `%`.

💼 **Mise en situation** : Vous construisez un système de distribution de tâches. Les tâches paires vont au serveur A, les impaires au serveur B.

📝 **Énoncé** :
1.  Déclarez une variable `task_id` avec la valeur `42`.
2.  Calculez le reste de la division par 2.
3.  Stockez le résultat dans une variable `remainder`.
4.  Affichez "Serveur A" si le reste est 0, sinon "Serveur B" (vous pouvez utiliser une condition simple ou juste afficher le résultat booléen de la comparaison pour l'instant).

📺 **Résultat attendu** :
```text
Reste : 0
Est pair : True
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
task_id: int = 42

# Le modulo retourne le reste
remainder: int = task_id % 2

print(f"Reste : {remainder}")

# Test d'égalité
is_even: bool = (remainder == 0)
print(f"Est pair : {is_even}")
```
</details>

### Exercice 2 - Calcul de l'Hypoténuse {#exercice-2---calcul-hypotenuse}

🎯 **Objectif** : Utiliser les puissances et les conversions de types.

💼 **Mise en situation** : Application de géométrie pour architectes. Vous devez calculer la diagonale d'une pièce rectangulaire.

📝 **Énoncé** :
1.  Déclarez `width` = 3.0 et `length` = 4.0.
2.  Calculez l'hypoténuse selon le théorème de Pythagore : $c = \sqrt{a^2 + b^2}$.
    *   Astuce : La racine carrée est équivalente à la puissance 0.5 (`** 0.5`).
3.  Affichez le résultat.

📺 **Résultat attendu** :
```text
Diagonale : 5.0
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
width: float = 3.0
length: float = 4.0

# Calcul des carrés
sum_squares: float = (width ** 2) + (length ** 2)

# Racine carrée (puissance 1/2)
hypotenuse: float = sum_squares ** 0.5

print(f"Diagonale : {hypotenuse}")
```
</details>

### Exercice 3 - Le Répartiteur de Billets {#exercice-3---repartiteur-billets}

🎯 **Objectif** : Maîtriser la division entière (`//`) et le modulo (`%`) combinés.

💼 **Mise en situation** : Un distributeur automatique doit rendre la monnaie de façon optimale en utilisant le moins de billets possible.

📝 **Énoncé** :
1.  Soit une somme `amount` = 137 euros.
2.  Calculez combien de billets de 50€ sont nécessaires.
3.  Calculez le reste à payer.
4.  Répétez pour les billets de 20€ et les pièces de 1€.
5.  Affichez le détail.

📺 **Résultat attendu** :
```text
Somme : 137
Billets de 50 : 2
Billets de 20 : 1
Pièces de 1 : 17
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
amount: int = 137

# Billets de 50
count_50: int = amount // 50     # Combien de fois 50 rentre dans 137 ? -> 2
remaining: int = amount % 50     # Ce qu'il reste -> 37

# Billets de 20 (sur le reste !)
count_20: int = remaining // 20  # Combien de fois 20 rentre dans 37 ? -> 1
remaining = remaining % 20       # Ce qu'il reste -> 17

# Pièces de 1
count_1: int = remaining         # Le reste est constitué d'unités

print(f"Somme : {amount}")
print(f"Billets de 50 : {count_50}")
print(f"Billets de 20 : {count_20}")
print(f"Pièces de 1 : {count_1}")
```
</details>