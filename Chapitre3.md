---
sidebar_label: Variables, types de données primitifs et commentaires
sidebar_position: 3
---

# Chapitre 3 : Variables, types de données primitifs et commentaires

Variables, Types (int, float, bool, None), Commentaires

Dans ce chapitre, nous allons construire les fondations de tout programme Python. Nous apprendrons à stocker de l'information (variables), à comprendre la nature de cette information (types) et à documenter notre pensée pour nous-mêmes et nos collaborateurs (commentaires).

---

## Les Variables {#les-variables}

### 1. Quoi
Une **variable** en Python est une référence (une étiquette) vers une valeur stockée en mémoire. Contrairement à d'autres langages comme C ou Java, une variable en Python n'est pas une "boîte" qui contient une valeur, mais plutôt une "étiquette" collée sur un objet.

### 2. Pourquoi
Les variables permettent de :
- **Stocker** des données pour les réutiliser plus tard.
- **Donner du sens** aux données grâce à des noms explicites.
- **Manipuler** l'état de votre application (ex: le contenu d'un panier d'achat).

### 3. Comment

#### A. Syntaxe de base
En Python, l'assignation se fait avec le signe `=`. Il n'y a pas besoin de déclarer le type (c'est le **typage dynamique**).

```python
# Syntaxe : nom_variable = valeur
product_name = "iPhone 15"
price = 999
```

#### B. Cas concret (SaaS B2B)
Imaginons un système de facturation pour une startup SaaS.

```python
# Déclaration de variables pour une facture
client_company = "TechCorp SAS"
subscription_price_monthly = 49.99
is_active_subscriber = True
discount_rate = 0.10  # 10% de réduction

# Calcul du prix final (stocké dans une nouvelle variable)
final_price = subscription_price_monthly * (1 - discount_rate)

print(f"Facture pour {client_company} : {final_price}€")
```

#### C. Exemples pratiques

**Cas 1 : E-commerce (Stock)**
```python
current_stock = 150
items_sold_today = 12
remaining_stock = current_stock - items_sold_today
```

**Cas 2 : Analytics (KPI)**
```python
total_visitors = 15430
bounce_rate = 0.45  # 45%
retained_visitors = total_visitors * (1 - bounce_rate)
```

**Cas 3 : Configuration (Feature Flag)**
```python
enable_dark_mode = True
max_upload_size_mb = 50
api_endpoint = "https://api.monsaas.com/v1"
```

### 4. Zone de Danger

❌ **À ne pas faire : Noms cryptiques ou camelCase (hors classes)**
```python
x = 10              # Qu'est-ce que x ?
n = "John"          # n pour name ?
userAge = 25        # Non respect de PEP 8 (convention Python)
Customer_List = []  # Mélange de styles
```

✅ **Bonne Pratique : Snake case et noms explicites (PEP 8)**
```python
user_count = 10
first_name = "John"
user_age = 25
customer_list = []
```

### 🚨 Limitations : Typage Dynamique
Python est dynamiquement typé, ce qui signifie qu'une variable peut changer de type au cours du temps.
```python
data = 10       # int
data = "Hello"  # devient str
```
Bien que flexible, cela peut causer des bugs difficiles à détecter dans de gros projets. C'est pourquoi nous verrons plus tard les **Type Hints** (Chapitre 49) pour ajouter de la rigueur.

---

## Les Types Primitifs {#les-types-primitifs}

Python possède quelques types fondamentaux ("atomiques") pour représenter les données.

### 1. Entiers (`int`) {#type-int}
Représente les nombres entiers, positifs ou négatifs, sans limite de taille (sauf la mémoire disponible).

```python
user_id = 42
huge_number = 999_999_999_999  # Les underscores sont ignorés, servent à la lisibilité
temperature = -5
```

### 2. Nombres à virgule flottante (`float`) {#type-float}
Représente les nombres réels (avec décimales).

```python
average_score = 4.5
pi_approximation = 3.14159
scientific_notation = 1.5e3  # 1.5 * 10^3 = 1500.0
```

#### 🚨 Limitations des Floats
Les ordinateurs stockent les flottants en binaire (norme IEEE 754), ce qui entraîne des problèmes de précision.

```python
print(0.1 + 0.2)
# Résultat : 0.30000000000000004 (et non 0.3 !)
```
> **Conseil Pro** : Pour des calculs financiers (monnaie), ne jamais utiliser de `float`. Stockez les prix en centimes dans des `int` (ex: `1999` pour 19.99€) ou utilisez le module `decimal`.

### 3. Booléens (`bool`) {#type-bool}
Représente une valeur de vérité logique. Seulement deux valeurs possibles : `True` ou `False`. Notez les majuscules !

```python
is_admin = True
has_paid = False
is_greater = (10 > 5)  # True
```
> **Info technique** : En Python, `bool` est une sous-classe de `int`. `True` vaut 1 et `False` vaut 0. `True + True` donne `2` (mais ne faites jamais ça).

### 4. Le type Null (`None`) {#type-none}
`None` est un type spécial représentant **l'absence de valeur**. C'est l'équivalent de `null` ou `nil` dans d'autres langages.

```python
current_user = None  # Pas d'utilisateur connecté
discount_code = None # Pas de code promo appliqué
```

### 5. Vérifier le type
La fonction `type()` permet de connaître le type d'une variable.

```python
print(type(42))      # <class 'int'>
print(type(3.14))    # <class 'float'>
print(type(True))    # <class 'bool'>
print(type(None))    # <class 'NoneType'>
```

---

## Les Commentaires {#les-commentaires}

### 1. Quoi
Des lignes de texte ignorées par l'interpréteur Python, destinées aux humains qui lisent le code.

### 2. Pourquoi
Le code nous dit *comment* le programme fonctionne. Les commentaires doivent nous dire *pourquoi* il fonctionne ainsi.

### 3. Comment

#### A. Commentaire en ligne (`#`)
Tout ce qui suit le caractère `#` sur une ligne est ignoré.

```python
# Calcul du ROI pour la campagne Q3
marketing_spend = 5000
revenue = 15000
roi = (revenue - marketing_spend) / marketing_spend # Formule standard
```

#### B. Docstrings (Documentation de module/fonction)
Bien que techniquement ce soient des chaînes de caractères, les triples guillemets `"""` sont utilisés pour documenter des blocs de code.

```python
"""
Ce script calcule les taxes pour les employés freelances
selon la législation 2024.
"""
```

### 4. Zone de Danger

❌ **Commentaires inutiles (Bruit)**
```python
i = i + 1  # Incrémente i de 1
```
*On sait lire le code, merci.*

✅ **Commentaires "Pourquoi" (Contexte métier)**
```python
i = i + 1  # On passe au client suivant dans la file d'attente
```
*Ici, on comprend l'intention métier.*

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-3}

1. **Quelle est la différence entre `10` et `10.0` en Python ?**
   - `10` est un entier (`int`), `10.0` est un flottant (`float`). Ils ont des comportements différents en mémoire et lors des opérations.

2. **Comment nommer une variable contenant le "prix total" selon la convention officielle Python (PEP 8) ?**
   - `total_price` (en snake_case : minuscules séparées par des underscores).

3. **Que renvoie l'instruction `type(False)` ?**
   - `<class 'bool'>`.

4. **Pourquoi le calcul `0.1 + 0.2 == 0.3` renvoie-t-il `False` ?**
   - À cause de l'imprécision des nombres flottants (représentation binaire IEEE 754).

5. **Quelle valeur utilise-t-on pour signifier qu'une variable est vide ou non définie ?**
   - `None`.

---

## Exercices {#exercices-3}

### Exercice 1 - Création de Profil Utilisateur {#exercice-1-creation-profil}

**🎯 Objectif** : Déclarer des variables de différents types primitifs.

**💼 Mise en situation** : Vous développez le backend d'une application de réseau social. Vous devez initialiser le profil d'un nouvel utilisateur avec des données par défaut.

**📝 Énoncé** :
Créez un script qui définit les variables suivantes avec les valeurs appropriées :
1. `username` (chaîne) : "PyFan2026"
2. `followers_count` (entier) : 0
3. `engagement_rate` (flottant) : 0.0
4. `is_verified` (booléen) : Faux
5. `bio` (NoneType) : Aucune valeur pour l'instant

Affichez ensuite le type de la variable `is_verified` et `engagement_rate` dans la console.

**📺 Résultat attendu** :
```text
<class 'bool'>
<class 'float'>
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
# Déclaration des variables du profil utilisateur
username = "PyFan2026"          # Type str (Chaîne de caractères - vu au prochain chapitre)
followers_count = 0             # Type int (Nouveau compte)
engagement_rate = 0.0           # Type float (Pas encore de stats)
is_verified = False             # Type bool (Compte standard)
bio = None                      # Type NoneType (Pas encore remplie)

# Vérification des types pour le debugging
print(type(is_verified))
print(type(engagement_rate))
```
</details>

---

### Exercice 2 - La TVA Buggée {#exercice-2-tva-buggee}

**🎯 Objectif** : Comprendre et contourner les limitations des floats.

**💼 Mise en situation** : Vous travaillez pour une plateforme e-commerce. Un client achète 3 articles à 0.10€. Votre patron vous signale que le total affiché est étrange.

**📝 Énoncé** :
1. Déclarez une variable `item_price` à `0.10`.
2. Calculez la somme de 3 articles : `total = item_price + item_price + item_price`.
3. Affichez le `total`.
4. Créez une variable `total_corrected` en utilisant des entiers (prix en centimes) pour obtenir le résultat exact, puis convertissez-le en euros pour l'affichage.

**📺 Résultat attendu** :
```text
Total buggé : 0.30000000000000004
Total corrigé : 0.3
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
# --- Approche naïve (Float) ---
item_price = 0.10
total_bugged = item_price + item_price + item_price

print(f"Total buggé : {total_bugged}")
# Le résultat montre l'erreur d'approximation flottante

# --- Approche robuste (Entiers / Centimes) ---
# On manipule des centimes pour rester en entiers
item_price_cents = 10 
total_cents = item_price_cents + item_price_cents + item_price_cents

# On repasse en euros uniquement pour l'affichage final
total_corrected = total_cents / 100

print(f"Total corrigé : {total_corrected}")
```
</details>

---

### Exercice 3 - Code Review Cleanup {#exercice-3-code-review}

**🎯 Objectif** : Appliquer les conventions de nommage et les bonnes pratiques de commentaires.

**💼 Mise en situation** : Un stagiaire vient de pousser ce code pour un système de gestion de température de serveur. Le lead dev refuse la Pull Request. Vous devez corriger le code.

**📝 Énoncé** :
Réécrivez le code suivant en respectant le PEP 8 (snake_case), en donnant des noms explicites, et en remplaçant les commentaires inutiles par des commentaires de contexte ("pourquoi").

*Code à corriger :*
```python
# Variable T
T = 85.5 
# Variable S
S = True
# Si S est vrai
if S:
    # Affiche alerte
    print("Alerte")
```

**📺 Résultat attendu** : Le code doit faire la même chose, mais être lisible par un professionnel.

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
# Définition du seuil de température critique du CPU
cpu_temperature = 85.5

# Indicateur si le système de refroidissement est en panne
is_cooling_system_down = True

# On vérifie si une intervention d'urgence est nécessaire
if is_cooling_system_down:
    # TODO: Connecter ceci au système d'alerte Slack/Email
    print("Alerte")
```
</details>