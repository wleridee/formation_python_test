---
sidebar_label: Variables, Types de Données Primitifs et Commentaires
sidebar_position: 3
---

# Chapitre 3 : Variables, Types de Données Primitifs et Commentaires

Concepts clés : **Assignation de variables**, **Types: int, float, bool, str**, **Commentaires et bonnes pratiques**

## 1. Les Variables et l'Assignation {#variables-et-assignation}

### 1. Quoi
Une variable en Python est une **étiquette** (un nom) attachée à une valeur stockée en mémoire. Contrairement à d'autres langages comme le C ou le Java, une variable Python n'est pas une "boîte" qui contient une valeur, mais une **référence** (comme un post-it) collée sur un objet.

### 2. Pourquoi
Les variables permettent de stocker l'état de votre programme (prix d'un produit, nom d'un utilisateur) pour le manipuler et le réutiliser plus tard. Sans elles, vous devriez répéter les valeurs littérales partout, rendant le code impossible à maintenir.

### 3. Comment

#### A. Syntaxe de base
L'opérateur `=` sert à l'assignation. Le nom est à gauche, la valeur à droite.

```python
# Syntaxe : nom_variable = valeur
first_name = "Alice"
user_age = 25
is_active = True
```

#### B. Cas concret : Références vs Valeurs
Comprendre que les variables sont des références est crucial.

```python
x = 100
y = x  # y est une nouvelle étiquette posée sur l'objet 100

print(id(x)) # Adresse mémoire de x
print(id(y)) # Même adresse mémoire que x

x = 200 # On déplace l'étiquette x sur un nouvel objet 200
# y reste sur 100
```

#### C. Bonnes pratiques de nommage (PEP 8)
Python suit la convention `snake_case` pour les variables.

| Type | Convention | Exemple | ❌ À éviter |
| :--- | :--- | :--- | :--- |
| **Variables** | snake_case | `user_email` | `userEmail`, `UserEmail` |
| **Constantes** | UPPER_CASE | `MAX_RETRY` | `max_retry` |
| **Privé** | _leading_underscore | `_internal_id` | N/A |

#### D. Le Typage Dynamique
Python détermine le type automatiquement à l'exécution. Vous pouvez changer le type d'une variable (bien que ce soit déconseillé pour la lisibilité).

```python
score = 10      # int
score = "Dix"   # str (Valide en Python, mais risqué)
```

:::tip Modern Python : Type Hints
Depuis Python 3.5+ et surtout en 3.12, il est recommandé d'annoter les types pour aider les outils de développement (comme l'auto-complétion), même si Python ne force pas leur respect à l'exécution.

```python
# Annotation de type (optionnel mais recommandé)
product_price: float = 99.99
product_name: str = "SaaS Subscription"
```
:::

### 4. Zone de Danger {#danger-variables}

❌ **À ne pas faire :**
*   Utiliser des noms ambigus : `x`, `data`, `temp`.
*   Écraser des mots-clés réservés : `list = [1, 2]` (Ceci casse la fonction `list()` !).
*   Utiliser `l` (L minuscule), `O` (o majuscule) ou `I` (i majuscule) comme noms de variables seuls (confusion avec 1 et 0).

✅ **Bonne Pratique :**
*   Des noms descriptifs : `days_since_last_login` est mieux que `d`.

---

## 2. Les Types Primitifs (Int, Float, Bool, Str) {#types-primitifs}

Python possède 4 types de base essentiels.

### A. Les Entiers (`int`)
Nombres sans virgule. En Python 3, ils ont une **précision arbitraire** (illimitée tant qu'il y a de la mémoire).

```python
user_count = 1050
# Les underscores sont ignorés, pratique pour la lisibilité des grands nombres
mrr = 1_000_000  # équivaut à 1000000
```

### B. Les Flottants (`float`)
Nombres à virgule flottante (approximation).

```python
conversion_rate = 3.14
scientific_notation = 1.2e-3 # 0.0012
```

### 🚨 Limitations des Floats
Les ordinateurs calculent en binaire (base 2), ce qui crée des erreurs d'arrondi sur certains décimaux.

```python
print(0.1 + 0.2) 
# Affiche: 0.30000000000000004 (et non 0.3)
```
> **Solution** : Pour l'argent, utilisez toujours le module `decimal` (voir Chapitres avancés) ou stockez en centimes (int).

### C. Les Booléens (`bool`)
Représente une valeur de vérité : `True` ou `False` (Notez les majuscules).

```python
is_admin = True
has_paid = False

# Les booléens sont techniquement des sous-types d'entiers
print(True + 1) # Affiche 2 (car True vaut 1)
```

### D. Les Chaînes de caractères (`str`)
Séquences de caractères Unicode **immuables**.

```python
# Guillemets simples ou doubles
company = "TechCorp"
quote = 'L\'innovation est clé' # Échappement nécessaire
quote_easy = "L'innovation est clé" # Pas d'échappement nécessaire

# F-strings (La méthode moderne de formatage)
greeting = f"Bienvenue chez {company}"
```

:::info Nouveauté Python 3.12 : Quotes dans les f-strings
Avant Python 3.12, vous ne pouviez pas réutiliser le même type de guillemet à l'intérieur d'une f-string. C'est maintenant possible :
```python
# Valide en Python 3.12+
data = {"status": "ok"}
message = f"Le status est : {data['status']}" # Plus besoin d'alterner ' et "
```
:::

---

## 3. Les Commentaires {#commentaires}

### 1. Quoi
Lignes ignorées par l'interpréteur, destinées aux humains.

### 2. Pourquoi
Expliquer le **POURQUOI** d'un bloc de code complexe, pas le **COMMENT** (que le code explique déjà).

### 3. Comment

#### A. Commentaire en ligne (`#`)

```python
# Calcul du revenu mensuel récurrent
monthly_revenue = users * price_per_user
```

#### B. Docstrings (`"""`)
Utilisées pour documenter des modules, classes ou fonctions. Elles sont accessibles via l'attribut `__doc__`.

```python
def calculate_vat(price):
    """
    Calcule la TVA de 20% sur un prix donné.
    """
    return price * 0.20
```

### 4. Zone de Danger {#danger-commentaires}

❌ **Code bruyant :**
```python
x = x + 1 # Incrémente x de 1 (INUTILE)
```

✅ **Code cristallin :**
```python
retry_count += 1 # On tente une nouvelle connexion au serveur
```

---

## Questions clés (validation des acquis) {#questions-cles}

1. **Quelle est la différence entre `score = 10` et `score = "10"` ?**
   *Réponse : Le premier est un entier (`int`) pour les calculs, le second est une chaîne (`str`) pour du texte.*

2. **Peut-on modifier une variable définie comme `x = 5` pour qu'elle devienne une chaîne de caractères plus tard ?**
   *Réponse : Oui, Python est dynamiquement typé, mais c'est une pratique à éviter pour la clarté.*

3. **Que retourne l'expression `0.1 + 0.1 + 0.1 == 0.3` ?**
   *Réponse : `False` (à cause des erreurs de précision des nombres flottants).*

4. **Quelle est la convention de nommage pour une variable standard en Python ?**
   *Réponse : `snake_case` (tout en minuscule avec des underscores).*

---

## Exercices {#exercices}

### Exercice 1 - Le Dashboard SaaS {#exercice-1-dashboard}
🎯 **Objectif** : Manipuler des variables `int`, `float` et `str` avec des f-strings.

💼 **Mise en situation** : Vous développez un dashboard pour une startup SaaS. Vous devez afficher un résumé financier pour l'investisseur.

📝 **Énoncé** :
1. Créez une variable `company_name` (str).
2. Créez une variable `mrr` (Monthly Recurring Revenue) de type float (ex: 12500.50).
3. Créez une variable `churn_rate` (taux d'attrition) de type float (ex: 0.05 pour 5%).
4. Calculez le `yearly_revenue` (Revenu Annuel) approximatif : `mrr * 12`.
5. Affichez un message formaté : "Startup [Nom] : Revenu annuel estimé à [Montant]€ avec un churn de [Pourcentage]%."

📺 **Résultat attendu** :
```text
Startup DataFlow : Revenu annuel estimé à 150006.0€ avec un churn de 5.0%.
```

<details>
<summary>💡 Voir la solution</summary>

```python
# Déclaration des variables
company_name: str = "DataFlow"
mrr: float = 12500.50
churn_rate: float = 0.05

# Calculs
yearly_revenue = mrr * 12
churn_percentage = churn_rate * 100

# Affichage avec f-string
print(f"Startup {company_name} : Revenu annuel estimé à {yearly_revenue}€ avec un churn de {churn_percentage}%.")
```
</details>

### Exercice 2 - Le Switch Booléen (Feature Flag) {#exercice-2-feature-flag}
🎯 **Objectif** : Comprendre les booléens et la logique simple.

💼 **Mise en situation** : Vous gérez le déploiement d'une nouvelle fonctionnalité "Dark Mode" sur votre site e-commerce, mais elle n'est disponible que pour les administrateurs pour le moment.

📝 **Énoncé** :
1. Créez une variable `feature_dark_mode_enabled` à `True`.
2. Créez une variable `user_is_admin` à `False`.
3. Créez une variable `can_see_feature` qui est vraie **seulement si** la feature est activée ET l'utilisateur est admin.
4. Affichez la valeur de `can_see_feature`.
5. Changez `user_is_admin` à `True` et ré-affichez le résultat.

📺 **Résultat attendu** :
```text
Accès feature : False
... (après changement)
Accès feature : True
```

<details>
<summary>💡 Voir la solution</summary>

```python
# État initial
feature_dark_mode_enabled = True
user_is_admin = False

# Logique (ET logique)
can_see_feature = feature_dark_mode_enabled and user_is_admin
print(f"Accès feature : {can_see_feature}")

# Changement d'utilisateur
user_is_admin = True
can_see_feature = feature_dark_mode_enabled and user_is_admin # Recalcul nécessaire
print(f"Accès feature : {can_see_feature}")
```
</details>

### Exercice 3 - Debugging de Types {#exercice-3-debugging}
🎯 **Objectif** : Identifier et corriger des erreurs de types courants.

💼 **Mise en situation** : Un stagiaire a écrit un script pour calculer le total d'une commande, mais le code plante ou donne un résultat étrange.

📝 **Énoncé** :
Analysez et corrigez le code suivant pour qu'il affiche "Total: 150.0" et non une erreur ou "10050".

Code cassé :
```python
price = "100"
tax = 50.0
total = price + tax
print("Total: " + total)
```

<details>
<summary>💡 Voir la solution</summary>

```python
# Correction
price = "100"
tax = 50.0

# 1. Conversion explicite de str vers float pour le calcul
total = float(price) + tax 

# 2. Conversion explicite vers str pour la concaténation (ou usage de f-string)
# Option A (classique)
print("Total: " + str(total))

# Option B (moderne - recommandée)
print(f"Total: {total}")
```
</details>