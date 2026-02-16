---
sidebar_label: Fonctions : Définition, Appel et Paramètres
sidebar_position: 14
---

# Chapitre 14 : Fonctions : Définition, Appel et Paramètres

Mot-clé def, Appel de fonction, Paramètres, Valeurs de retour

Jusqu'ici, nous avons écrit du code séquentiel : une suite d'instructions lues de haut en bas. Mais que se passe-t-il si vous devez exécuter la même logique (par exemple, calculer une TVA ou formater un email) à 15 endroits différents de votre application ? Copier-coller le code est la pire solution (principe **DRY : Don't Repeat Yourself**).

La **fonction** est l'unité fondamentale de réutilisation et d'organisation du code en Python. Elle permet d'encapsuler une logique, de lui donner un nom, et de l'appeler à la demande avec différentes données d'entrée.

---

## 1. Définition et Appel (`def`) {#definition-et-appel}

### 1. Quoi
Une fonction est un bloc de code nommé qui ne s'exécute que lorsqu'il est appelé. Elle est définie par le mot-clé **`def`**.

### 2. Pourquoi
*   **Organisation** : Découper un gros problème en sous-tâches gérables.
*   **Réutilisabilité** : Écrire la logique une fois, l'utiliser partout.
*   **Maintenance** : Si la logique change, on ne modifie le code qu'à un seul endroit.

### 3. Comment

#### A. Syntaxe de base

```python
# Définition
def say_hello():
    # Corps de la fonction (indenté)
    print("Bonjour Python !")

# Appel de la fonction
say_hello() 
```

#### B. Cas concret : Initialisation de système (Logging)

```python
import datetime

def log_system_start():
    """Affiche un message de démarrage avec l'heure actuelle."""
    current_time = datetime.datetime.now()
    # f-string pour le formatage
    print(f"[INFO] Système démarré à {current_time}")

# Simulation du démarrage
print("Booting...")
log_system_start() 
# Affiche : [INFO] Système démarré à 2023-10-27 10:00:00.123456
```

### 4. Zone de Danger
❌ **Oublier les parenthèses à l'appel** :
En Python, les fonctions sont des objets. Si vous écrivez `log_system_start` sans `()`, vous ne l'exécutez pas, vous faites référence à l'objet fonction lui-même.

```python
# Ne fait rien (pas d'affichage)
func = log_system_start 
print(func) # <function log_system_start at 0x...>
```

---

## 2. Paramètres et Arguments {#parametres-et-arguments}

### 1. Quoi
Les **paramètres** sont les variables définies entre les parenthèses de la fonction (les "réceptacles"). Les **arguments** sont les valeurs réelles envoyées lors de l'appel.

### 2. Pourquoi
Une fonction qui fait toujours exactement la même chose (comme afficher "Bonjour") est limitée. Les paramètres permettent de **dynamiser** la fonction en lui passant des données.

### 3. Comment

#### A. Paramètres positionnels et Type Hinting
Python 3.14 encourage fortement le **typage statique** (Type Hinting) pour documenter ce que la fonction attend.

```python
# name est un paramètre de type str
def greet_user(name: str):
    print(f"Bienvenue, {name} !")

greet_user("Alice") # "Alice" est l'argument
```

#### B. Paramètres par défaut
On peut rendre certains paramètres optionnels en leur donnant une valeur par défaut. Ils doivent toujours être placés **après** les paramètres obligatoires.

```python
def create_file(filename: str, extension: str = "txt"):
    full_name = f"{filename}.{extension}"
    print(f"Création de {full_name}")

create_file("rapport")          # Utilise "txt" par défaut -> rapport.txt
create_file("image", "png")     # Surcharge la valeur -> image.png
```

#### C. Exemples pratiques

**Cas 1 : Calculatrice e-commerce**
```python
def calculate_price_ttc(price_ht: float, tax_rate: float = 0.20):
    total = price_ht * (1 + tax_rate)
    print(f"Prix TTC : {total:.2f}€")

calculate_price_ttc(100)      # 120.00€
calculate_price_ttc(100, 0.05) # 105.00€
```

**Cas 2 : Envoi d'email simulé**
```python
def send_email(to_email: str, subject: str, urgent: bool = False):
    prefix = "[URGENT] " if urgent else ""
    print(f"Envoi à {to_email} : {prefix}{subject}")

send_email("admin@corp.com", "Server Down", urgent=True)
```

### 4. Zone de Danger
❌ **Le piège des arguments par défaut mutables** :
C'est l'erreur la plus classique en Python. Si vous utilisez une liste ou un dictionnaire vide comme valeur par défaut, **cette même liste est réutilisée à chaque appel**.

```python
# ❌ MAUVAISE PRATIQUE
def add_student(name: str, class_list: list = []):
    class_list.append(name)
    print(class_list)

add_student("Bob")   # ['Bob']
add_student("Alice") # ['Bob', 'Alice'] !!! Alice a rejoint la liste de Bob !

# ✅ BONNE PRATIQUE
def add_student_safe(name: str, class_list: list | None = None):
    if class_list is None:
        class_list = [] # Nouvelle liste créée à chaque appel
    class_list.append(name)
    print(class_list)
```

---

## 3. Valeurs de Retour (`return`) {#valeurs-de-retour}

### 1. Quoi
L'instruction `return` met fin à l'exécution de la fonction et renvoie une valeur au code appelant. Si `return` est absent, la fonction renvoie implicitement `None`.

### 2. Pourquoi
Une fonction bien conçue doit souvent **calculer** une donnée sans l'afficher, pour que le programme principal puisse décider quoi en faire (l'enregistrer, l'envoyer sur le réseau, l'afficher, etc.). Séparation des responsabilités.

### 3. Comment

#### A. Retour simple
On indique le type de retour avec `-> Type`.

```python
def square(number: int) -> int:
    return number * number

result = square(4) # result vaut 16
# On peut utiliser le résultat dans un calcul
print(square(4) + 10) # 26
```

#### B. Retour multiple (Tuple)
Comme vu au Chapitre 11, Python permet de renvoyer plusieurs valeurs via un tuple.

```python
def get_user_info(user_id: int) -> tuple[str, int]:
    # Simulation DB
    name = "Alice"
    age = 30
    return name, age

user_name, user_age = get_user_info(1)
```

#### C. Sortie anticipée (Guard Clause)
Le `return` arrête immédiatement la fonction. On l'utilise pour gérer les cas d'erreurs au début.

```python
def divide(a: float, b: float) -> float | None:
    if b == 0:
        print("Erreur : Division par zéro")
        return None # Arrêt immédiat
    
    return a / b
```

### 4. Zone de Danger
❌ **Confusion `print` vs `return`** :
Un débutant écrit souvent `print(result)` dans la fonction au lieu de `return result`.
*   `print` : Affiche à l'écran (pour l'humain). La variable récupérée vaudra `None`.
*   `return` : Transmet la donnée (pour le programme).

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-14}

1.  **Quelle est la différence entre un paramètre et un argument ?**
    Le paramètre est la variable définie dans la déclaration de la fonction (`def foo(x):`). L'argument est la valeur passée lors de l'appel (`foo(5)`).

2.  **Que renvoie une fonction qui ne contient pas d'instruction `return` ?**
    Elle renvoie la valeur `None` par défaut.

3.  **Pourquoi ne faut-il jamais utiliser `list = []` comme paramètre par défaut ?**
    Car la liste est créée une seule fois à la définition de la fonction et partagée entre tous les appels, ce qui cause des effets de bord inattendus. Il faut utiliser `None` et initialiser dans la fonction.

4.  **Comment spécifier qu'une fonction renvoie un entier ?**
    En utilisant l'annotation de type après les parenthèses : `def ma_fonction() -> int:`.

---

## Exercices : {#exercices-14}

### Exercice 1 - Le Calculateur de Réductions {#exercice-1---calculateur-reduction}

🎯 **Objectif** : Paramètres, calcul et retour de valeur.

💼 **Mise en situation** : Vous développez le module de paiement d'un site e-commerce. Vous devez calculer le prix final après application d'une réduction en pourcentage.

📝 **Énoncé** :
1.  Créez une fonction `apply_discount(price: float, discount_percent: float) -> float`.
2.  La fonction doit calculer le montant de la réduction et le soustraire au prix.
3.  Si la réduction est négative ou supérieure à 100, renvoyez le prix original (validation basique).
4.  Testez avec un prix de 100€ et 20% de remise.

📺 **Résultat attendu** :
```text
Prix original : 100, Remise : 20% -> Final : 80.0
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
def apply_discount(price: float, discount_percent: float) -> float:
    # Validation des entrées (Guard clause)
    if discount_percent < 0 or discount_percent > 100:
        print("Erreur : Pourcentage invalide. Prix inchangé.")
        return price

    discount_amount = price * (discount_percent / 100)
    final_price = price - discount_amount
    return final_price

# Test
p = 100.0
d = 20.0
result = apply_discount(p, d)
print(f"Prix original : {p}, Remise : {d}% -> Final : {result}")
```
</details>

### Exercice 2 - Générateur de Slug (SEO) {#exercice-2---generateur-slug}

🎯 **Objectif** : Manipulation de chaînes et paramètres par défaut.

💼 **Mise en situation** : Pour les URLs de votre blog, vous devez transformer un titre comme "Vive Python 3.14 !" en "vive-python-3-14".

📝 **Énoncé** :
1.  Créez une fonction `generate_slug(text: str, separator: str = "-") -> str`.
2.  La fonction doit :
    - Mettre le texte en minuscules.
    - Remplacer les espaces par le `separator`.
    - (Bonus simple) Retirer le caractère "!" à la fin si présent (méthode `.strip()`).
3.  Appelez la fonction avec le titre "Mon Super Article".
4.  Appelez la fonction avec "Photo Vacances" et un séparateur `_`.

📺 **Résultat attendu** :
```text
mon-super-article
photo_vacances
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
def generate_slug(text: str, separator: str = "-") -> str:
    # 1. Minuscules
    clean_text = text.lower()
    # 2. Nettoyage basique (optionnel pour l'exercice mais bonne pratique)
    clean_text = clean_text.strip("!?.") 
    # 3. Remplacement
    slug = clean_text.replace(" ", separator)
    
    return slug

print(generate_slug("Mon Super Article !"))       # mon-super-article
print(generate_slug("Photo Vacances", "_"))       # photo_vacances
```
</details>

### Exercice 3 - Le Formateur de Profil Utilisateur {#exercice-3---profil-utilisateur}

🎯 **Objectif** : Gestion des types `None` et logique conditionnelle interne.

💼 **Mise en situation** : Afficher une carte de visite textuelle. Le numéro de téléphone est optionnel.

📝 **Énoncé** :
1.  Fonction `format_profile(name: str, role: str, phone: str | None = None) -> str`.
2.  Construisez une chaîne multiligne.
3.  Si `phone` est `None`, affichez "Non renseigné" à la place du numéro.
4.  Retournez la chaîne formatée.

📺 **Résultat attendu** :
```text
--- USER CARD ---
Nom : Alice
Rôle : DevOps
Tel : Non renseigné
-----------------
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
def format_profile(name: str, role: str, phone: str | None = None) -> str:
    # Gestion de la valeur par défaut pour l'affichage
    display_phone = phone
    if phone is None:
        display_phone = "Non renseigné"
    
    # Construction de la chaîne avec f-string multi-lignes
    card = (
        f"--- USER CARD ---\n"
        f"Nom : {name}\n"
        f"Rôle : {role}\n"
        f"Tel : {display_phone}\n"
        f"-----------------"
    )
    return card

# Appel sans téléphone
print(format_profile("Alice", "DevOps"))

# Appel avec téléphone
# print(format_profile("Bob", "Manager", "0600000000"))
```
</details>