---
sidebar_label: Structures de Contrôle : Conditions (if/elif/else)
sidebar_position: 7
---

# Chapitre 7 : Structures de Contrôle : Conditions (if/elif/else)

Bloc if, Bloc elif, Bloc else, Expressions booléennes

Jusqu'à présent, nos programmes s'exécutaient de manière linéaire, ligne après ligne. La puissance réelle de la programmation réside dans la capacité à créer des branchements : exécuter un bloc de code seulement si une certaine condition est remplie.

En Python, cette logique conditionnelle repose sur les mots-clés `if`, `elif` et `else`. Ce chapitre marque votre entrée dans l'algorithmique véritable.

---

## 1. Le bloc `if` (Si) {#le-bloc-if}

### 1. Quoi
Le bloc `if` permet d'exécuter une section de code **uniquement si** une condition donnée est évaluée à `True` (Vrai). C'est la structure conditionnelle fondamentale.

En Python, la structure est définie par l'**indentation** (le décalage vers la droite), et non par des accolades `{}` comme en C ou Java.

### 2. Pourquoi
Pour adapter le comportement du programme au contexte : afficher un message de bienvenue *si* l'utilisateur est connecté, sauvegarder les données *si* le fichier a été modifié, etc.

### 3. Comment

#### A. Syntaxe de base

```python
temperature: int = 30

# Notez les "deux points" à la fin de la ligne
if temperature > 25:
    # Ce bloc est indenté (4 espaces)
    print("Il fait chaud !") 
    print("Pensez à boire de l'eau.")

# Ce code s'exécute toujours, car il n'est plus indenté
print("Fin du bulletin météo.")
```

#### B. Cas concret : Vérification de droits

```python
user_role: str = "admin"

if user_role == "admin":
    # Logique sensible
    print("Accès au panneau de configuration autorisé.")
    delete_button_visible: bool = True
```

### 4. Zone de Danger

❌ **Erreur d'indentation** : C'est l'erreur n°1 des débutants Python.
```python
if True:
print("Erreur !") # IndentationError: expected an indented block
```

❌ **Oubli des deux points** :
```python
if True # SyntaxError: invalid syntax
    print("Oups")
```

---

## 2. Le bloc `else` (Sinon) {#le-bloc-else}

### 1. Quoi
Le bloc `else` est optionnel. Il capture **tous les cas** où la condition du `if` est fausse. C'est le plan B, l'alternative par défaut.

### 2. Pourquoi
Pour gérer une alternative binaire stricte : Pair ou Impair, Majeur ou Mineur, Réussite ou Échec. Il garantit qu'un (et un seul) des deux blocs sera exécuté.

### 3. Comment

#### A. Syntaxe de base

```python
battery_level: int = 10

if battery_level > 20:
    print("Batterie OK")
else:
    # S'exécute si battery_level <= 20
    print("Batterie faible, branchez le chargeur !")
```

#### B. Expression conditionnelle (Ternaire)
Pour des assignations simples, Python propose une syntaxe condensée sur une seule ligne.

```python
score: int = 85
# Syntaxe : valeur_si_vrai if condition else valeur_si_faux
status: str = "Reçu" if score >= 50 else "Recalé"

print(status) # "Reçu"
```

### 4. Zone de Danger
❌ **Mettre une condition au `else`** :
Un `else` ne prend JAMAIS de condition. Si vous voulez tester une autre condition, vous avez besoin de `elif`.

```python
# FAUX
else score < 50: 

# VRAI
else:
```

---

## 3. Le bloc `elif` (Sinon Si) {#le-bloc-elif}

### 1. Quoi
Contraction de "else if". Le bloc `elif` permet de tester plusieurs conditions séquentiellement. Dès qu'une condition est vraie, Python exécute le bloc correspondant et **ignore tout le reste** de la structure.

### 2. Pourquoi
Pour gérer des choix multiples mutuellement exclusifs (ex: Menu d'options, Tranches d'imposition, Niveaux de jeu).

### 3. Comment

#### A. Syntaxe de base

```python
traffic_light: str = "orange"

if traffic_light == "red":
    print("Arrêt")
elif traffic_light == "orange":
    print("Ralentir")
elif traffic_light == "green":
    print("Passer")
else:
    print("Feu en panne, prudence !")
```

#### B. Cas concret : Tarification dégressive SaaS

```python
users_count: int = 150
price_per_user: float = 0.0

if users_count < 10:
    price_per_user = 0.0 # Gratuit
elif users_count < 100:
    price_per_user = 10.0 # Standard
elif users_count < 500:
    price_per_user = 8.5 # Business
else:
    price_per_user = 5.0 # Enterprise

print(f"Prix unitaire : {price_per_user}€")
```

### 4. Zone de Danger

❌ **L'ordre compte !**
Python teste de haut en bas. Mettre la condition la plus générale en premier peut masquer les cas spécifiques.

```python
score = 95

# MAUVAISE LOGIQUE
if score > 50:
    print("Passable") # 95 est > 50, donc ceci s'affiche et on sort !
elif score > 90:
    print("Excellent") # Ce bloc ne sera jamais atteint pour 95

# BONNE LOGIQUE (Du plus spécifique au plus général)
if score > 90:
    print("Excellent")
elif score > 50:
    print("Passable")
```

---

## 4. Structures Imbriquées (Nested Ifs) {#structures-imbriquees}

### 1. Quoi
Placer un bloc `if` à l'intérieur d'un autre bloc `if`.

### 2. Pourquoi
Pour vérifier des sous-conditions dépendantes d'une condition principale.

### 3. Comment

#### A. Exemple de validation complexe

```python
is_connected: bool = True
has_permission: bool = False

if is_connected:
    print("Bienvenue utilisateur.")
    
    if has_permission:
        print("Accès dossier confidentiel.")
    else:
        print("Accès refusé : Droits insuffisants.")
else:
    print("Veuillez vous connecter.")
```

#### B. Alternative : Aplatir le code ("Guard Clauses")
Les imbrications profondes rendent le code illisible (le "Hadouken code"). On préfère souvent retourner (ou lever une erreur) tôt.

```python
# Version "Clean Code" (Aplatissement)
def access_file(connected: bool, permission: bool):
    if not connected:
        print("Veuillez vous connecter.")
        return # On arrête là
    
    if not permission:
        print("Accès refusé.")
        return

    print("Accès dossier confidentiel.")
```

### 4. Zone de Danger
❌ **Trop d'imbrication** : Au-delà de 2 ou 3 niveaux d'indentation, refactorisez votre code. C'est un signe que la logique est trop complexe pour un seul bloc.

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-7}

1.  **Quelle est la différence entre `if` et `elif` ?**
    `if` commence une nouvelle chaîne de conditions. `elif` ne s'exécute que si les conditions précédentes (`if` ou autres `elif`) étaient fausses.

2.  **Peut-on avoir un `else` sans `if` ?**
    Non, `else` doit toujours fermer une structure conditionnelle commencée par un `if`.

3.  **Comment Python délimite-t-il le code qui appartient à un bloc `if` ?**
    Par l'indentation (généralement 4 espaces) sous la ligne contenant le `if`.

4.  **Si j'ai trois blocs `if` indépendants à la suite, combien peuvent s'exécuter ?**
    Les trois peuvent s'exécuter si leurs conditions sont toutes vraies. Contrairement à une structure `if/elif/else` où un seul bloc maximum est exécuté.

---

## Exercices : {#exercices-7}

### Exercice 1 - Le Videur de Boîte de Nuit {#exercice-1---le-videur}

🎯 **Objectif** : Maîtriser `if`, `elif`, `else` et les opérateurs logiques.

💼 **Mise en situation** : Vous codez le scanner d'entrée d'un club sélect. Les règles sont strictes :
- Moins de 18 ans : Refusé.
- Entre 18 et 25 ans : Entrée payante (20€).
- Plus de 25 ans : Entrée gratuite.
- **Exception** : Si la personne est "VIP", entrée gratuite quel que soit l'âge (si majeur).

📝 **Énoncé** :
1.  Déclarez `age` (int) et `is_vip` (bool).
2.  Écrivez la logique pour afficher le prix ou le refus.
3.  Testez avec un VIP de 20 ans (doit être gratuit).

📺 **Résultat attendu** (pour age=20, vip=True) :
```text
Bienvenue ! Prix : 0€
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
age: int = 20
is_vip: bool = True

if age < 18:
    print("Entrée refusée : Mineur.")
elif is_vip:
    # VIP majeur (car le premier if a filtré les mineurs)
    print("Bienvenue VIP ! Prix : 0€")
elif age > 25:
    print("Bienvenue ! Prix : 0€")
else:
    # Entre 18 et 25 ans et non VIP
    print("Bienvenue ! Prix : 20€")
```
</details>

### Exercice 2 - Système de Note Américain {#exercice-2---systeme-de-note}

🎯 **Objectif** : Gérer l'ordre des conditions (`elif`) et les intervalles.

💼 **Mise en situation** : Convertir une note sur 100 en lettre (A, B, C, D, F) pour un bulletin scolaire international.
- A : >= 90
- B : >= 80
- C : >= 70
- D : >= 60
- F : < 60

📝 **Énoncé** :
1.  Déclarez `score` (int).
2.  Utilisez une structure conditionnelle pour déterminer la lettre.
3.  Affichez "Note : [Lettre]".
4.  Assurez-vous que la logique fonctionne pour 85 (doit donner B, pas C ou D).

📺 **Résultat attendu** (pour score=85) :
```text
Note : B
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
score: int = 85
grade: str = ""

# On teste du plus haut vers le plus bas
# Si on testait > 60 en premier, tout le monde aurait D !
if score >= 90:
    grade = "A"
elif score >= 80:
    grade = "B"
elif score >= 70:
    grade = "C"
elif score >= 60:
    grade = "D"
else:
    grade = "F"

print(f"Note : {grade}")
```
</details>

### Exercice 3 - Gestionnaire de Stock E-commerce {#exercice-3---stock-ecommerce}

🎯 **Objectif** : Conditions imbriquées vs Logique plate.

💼 **Mise en situation** : Avant d'ajouter un produit au panier, il faut vérifier son stock.
- Si stock == 0 : "Rupture de stock".
- Si stock > 0 mais < quantité demandée : "Stock insuffisant (Reste : X)".
- Sinon : "Produit ajouté".

📝 **Énoncé** :
1.  Déclarez `stock` (int) à 5 et `qty_requested` (int) à 8.
2.  Écrivez la logique pour gérer ces 3 cas.

📺 **Résultat attendu** :
```text
Stock insuffisant (Reste : 5)
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
stock: int = 5
qty_requested: int = 8

if stock == 0:
    print("Rupture de stock")
elif stock < qty_requested:
    print(f"Stock insuffisant (Reste : {stock})")
else:
    # Ici, stock >= qty_requested
    print("Produit ajouté au panier")
    # Simulation de la décrémentation
    # stock -= qty_requested
```
</details>