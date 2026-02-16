---
sidebar_label: Module `enum` : Énumérations
sidebar_position: 36
---

# Chapitre 36 : Module `enum` : Énumérations

Création d'énumérations, Accès aux membres, Itération sur les enums, Comparaison

En programmation, nous utilisons souvent des constantes pour représenter des états, des types ou des options : `0` pour "Inactif", `1` pour "Actif", "draft" pour un brouillon, etc. C'est ce qu'on appelle des "nombres magiques" ou des "chaînes magiques". Ils polluent le code et sont sources d'erreurs (typos, confusion).

Le module `enum` (disponible dans la bibliothèque standard) permet de créer des **Énumérations** : un ensemble de noms symboliques liés à des valeurs uniques et constantes. C'est l'outil indispensable pour rendre votre code plus lisible, plus sûr et auto-documenté.

---

## 1. Création d'Énumérations {#creation-d-enumerations}

### 1. Quoi
Une énumération est une classe qui hérite de `Enum`. Chaque attribut de cette classe devient un **membre** de l'énumération, composé d'un nom (name) et d'une valeur (value).

### 2. Pourquoi
Pour remplacer des listes de constantes disparates par une structure typée et regroupée. Cela garantit que vous n'utilisez que des valeurs valides définies à l'avance.

### 3. Comment

#### A. Syntaxe de base

```python
from enum import Enum

class Status(Enum):
    PENDING = 1
    ACTIVE = 2
    ARCHIVED = 3

print(Status.ACTIVE) # Status.ACTIVE
```

#### B. Utilisation de `auto()` et `StrEnum` (Python 3.11+)
Souvent, la valeur exacte (1, 2, 3) importe peu, on veut juste qu'elle soit unique. `auto()` gère cela.
Depuis Python 3.11, `StrEnum` est recommandé si vos valeurs doivent être interopérables avec des chaînes de caractères (ex: JSON, API).

```python
from enum import Enum, StrEnum, auto

# Enum classique avec valeurs automatiques
class Color(Enum):
    RED = auto()
    GREEN = auto()
    BLUE = auto()

# StrEnum : Les membres se comportent comme des strings
class HttpMethod(StrEnum):
    GET = auto()  # La valeur sera 'GET' (en minuscule par défaut avec auto())
    POST = auto()
    DELETE = auto()

print(f"Méthode choisie : {HttpMethod.GET}") 
# Affiche : Méthode choisie : get
```

### 4. Zone de Danger
❌ **Duplication de clés** : Il est interdit d'avoir deux clés avec le même nom (ex: deux fois `ACTIVE`).
✅ **Alias** : Il est permis d'avoir deux clés avec la même *valeur*. La seconde clé sera un alias de la première.

```python
class Response(Enum):
    YES = 1
    OUI = 1 # OUI est un alias de YES
```

---

## 2. Accès aux Membres {#acces-aux-membres}

### 1. Quoi
On peut accéder aux membres de l'énumération de trois manières : par attribut, par valeur, ou par nom (chaîne de caractères).

### 2. Pourquoi
*   **Par attribut** : Pour le code en dur (`Status.ACTIVE`).
*   **Par valeur** : Pour convertir une donnée brute venant d'une BDD (`Status(1)`).
*   **Par nom** : Pour convertir une configuration textuelle (`Status['ACTIVE']`).

### 3. Comment

```python
from enum import Enum

class Role(Enum):
    ADMIN = 1
    USER = 2
    GUEST = 3

# 1. Accès direct (le plus courant)
current_role = Role.ADMIN

# 2. Accès par valeur (utile lors de la lecture d'une BDD)
db_value = 2
user_role = Role(db_value) # Renvoie Role.USER

# 3. Accès par nom (utile pour le parsing de config/JSON)
config_str = "GUEST"
guest_role = Role[config_str] # Renvoie Role.GUEST

# 4. Inspection
print(f"Nom: {user_role.name}, Valeur: {user_role.value}")
# Nom: USER, Valeur: 2
```

### 🚨 Limitations
Si vous tentez d'accéder à une valeur ou un nom qui n'existe pas (ex: `Role(99)` ou `Role['SUPER_ADMIN']`), Python lèvera une `ValueError` ou une `KeyError`. Il faut toujours gérer ces exceptions lors de la conversion de données externes.

---

## 3. Itération et Comparaison {#iteration-et-comparaison}

### 1. Quoi
Les Enums sont itérables. On peut boucler dessus pour lister toutes les options possibles. Ils supportent aussi les comparaisons d'identité (`is`) et d'égalité (`==`).

### 2. Pourquoi
*   **Itération** : Pour générer automatiquement des menus déroulants dans une interface ou valider des entrées.
*   **Comparaison** : Pour vérifier l'état d'un objet de manière lisible.

### 3. Comment

#### A. Itération

```python
class Planet(Enum):
    EARTH = "Terre"
    MARS = "Mars"
    VENUS = "Vénus"

# Afficher toutes les options disponibles
for p in Planet:
    print(f"Destination possible : {p.value}")
```

#### B. Comparaison

```python
selected_planet = Planet.MARS

# ✅ Comparaison d'identité (recommandé pour les Enums)
if selected_planet is Planet.MARS:
    print("Décollage vers Mars !")

# ✅ Comparaison d'égalité (fonctionne aussi)
if selected_planet == Planet.MARS:
    pass

# ❌ Comparaison avec la valeur brute (sauf pour IntEnum/StrEnum)
# Ceci renvoie False avec une classe Enum standard !
if selected_planet == "Mars":
    print("Ceci ne s'affichera pas avec un Enum standard")
```

#### C. Cas particulier : `IntEnum`
Si vous avez besoin de comparer des ordres de grandeur (`<`, `>`), utilisez `IntEnum`.

```python
from enum import IntEnum

class LogLevel(IntEnum):
    INFO = 10
    WARNING = 30
    ERROR = 50

# Possible car héritant de IntEnum
if LogLevel.ERROR > LogLevel.WARNING:
    print("Erreur plus grave que Warning")
```

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-36}

1.  **Quelle est la différence fondamentale entre `Enum` et `StrEnum` (ou `IntEnum`) ?**
    Un membre d'`Enum` standard n'est égal à rien d'autre que lui-même. Un membre de `StrEnum` (ou `IntEnum`) se comporte aussi comme le type natif (`str` ou `int`) et peut être comparé directement à une valeur brute.

2.  **Comment récupérer un membre d'Enum à partir de sa valeur (ex: `1`) ?**
    En appelant la classe comme une fonction : `MonEnum(1)`. Cela lève `ValueError` si la valeur est invalide.

3.  **À quoi sert la fonction `auto()` ?**
    Elle assigne automatiquement une valeur au membre (souvent un entier incrémental ou le nom en minuscule pour `StrEnum`), ce qui évite de gérer manuellement les IDs.

4.  **Peut-on modifier la valeur d'un membre d'Enum après sa création (`Color.RED.value = 5`) ?**
    Non. Les Enums sont immuables. C'est leur force principale : garantir des constantes fiables.

---

## Exercices : {#exercices-36}

### Exercice 1 - Gestion des Commandes {#exercice-1-gestion-commandes}

🎯 **Objectif** : Créer un Enum basique et l'utiliser dans une logique conditionnelle.

💼 **Mise en situation** : Vous gérez le workflow d'un site e-commerce. Une commande peut être "Nouvelle", "Payée", "Expédiée" ou "Annulée".

📝 **Énoncé** :
1.  Créez une Enum `OrderStatus` avec les 4 états cités (valeurs libres ou `auto()`).
2.  Créez une fonction `check_cancellation(status: OrderStatus)` qui :
    - Renvoie `True` si la commande peut être annulée (seulement si "Nouvelle" ou "Payée").
    - Renvoie `False` sinon.
3.  Testez la fonction avec le statut "Expédiée".

📺 **Résultat attendu** :
```text
Peut annuler EXPEDIEE ? False
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
from enum import Enum, auto

# 1. Définition de l'Enum
class OrderStatus(Enum):
    NEW = auto()
    PAID = auto()
    SHIPPED = auto()
    CANCELLED = auto()

# 2. Fonction logique
def check_cancellation(status: OrderStatus) -> bool:
    # On vérifie l'appartenance à un ensemble d'états autorisés
    cancellable_states = {OrderStatus.NEW, OrderStatus.PAID}
    
    # Retourne True si le statut est dans l'ensemble
    return status in cancellable_states

# 3. Test
current_status = OrderStatus.SHIPPED
can_cancel = check_cancellation(current_status)

print(f"Peut annuler {current_status.name} ? {can_cancel}")
```
</details>

### Exercice 2 - Permissions avec `IntEnum` {#exercice-2-permissions-intenum}

🎯 **Objectif** : Utiliser `IntEnum` pour des comparaisons de hiérarchie.

💼 **Mise en situation** : Système de droits utilisateurs. Un `ADMIN` (niveau 100) a plus de droits qu'un `EDITOR` (50), qui a plus de droits qu'un `VIEWER` (10).

📝 **Énoncé** :
1.  Créez un `IntEnum` nommé `UserRole`.
2.  Créez une fonction `has_access(user_role, required_role)` qui retourne `True` si le rôle de l'utilisateur est supérieur ou égal au rôle requis.
3.  Vérifiez si un `EDITOR` a accès à une ressource nécessitant `ADMIN`.

📺 **Résultat attendu** :
```text
EDITOR peut accéder à zone ADMIN ? False
EDITOR peut accéder à zone VIEWER ? True
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
from enum import IntEnum

# 1. Utilisation de IntEnum pour permettre les opérateurs <, >, >=
class UserRole(IntEnum):
    VIEWER = 10
    EDITOR = 50
    ADMIN = 100

# 2. Comparaison arithmétique directe grâce à IntEnum
def has_access(user_role: UserRole, required_role: UserRole) -> bool:
    return user_role >= required_role

# 3. Tests
my_role = UserRole.EDITOR

print(f"EDITOR peut accéder à zone ADMIN ? {has_access(my_role, UserRole.ADMIN)}")
print(f"EDITOR peut accéder à zone VIEWER ? {has_access(my_role, UserRole.VIEWER)}")
```
</details>

### Exercice 3 - API et `StrEnum` {#exercice-3-api-strenum}

🎯 **Objectif** : Mapper des données JSON (chaînes) vers un `StrEnum`.

💼 **Mise en situation** : Vous recevez un JSON d'une API météo : `{"day": "monday", "weather": "rainy"}`. Vous voulez sécuriser le code en convertissant le jour en Enum.

📝 **Énoncé** :
1.  Créez un `StrEnum` nommé `WeekDay` (MONDAY, TUESDAY...). Les valeurs doivent être les noms des jours en minuscules ("monday", etc.). Utilisez `auto()` si vous êtes en Python 3.11+, sinon explicite.
2.  Simulez la réception de la donnée `raw_day = "monday"`.
3.  Convertissez cette chaîne en membre de l'Enum `WeekDay`.
4.  Gérez le cas où la chaîne est invalide (ex: "funday") avec un `try/except` affichant "Jour inconnu".

📺 **Résultat attendu** :
```text
Jour détecté : monday (Instance de WeekDay)
Erreur : 'funday' n'est pas un jour valide.
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
from enum import StrEnum, auto

# 1. StrEnum : les valeurs générées par auto() sont les noms en minuscules
class WeekDay(StrEnum):
    MONDAY = auto()
    TUESDAY = auto()
    WEDNESDAY = auto()
    # ... autres jours

def process_day(raw_value: str):
    try:
        # 2 & 3. Conversion String -> Enum
        # StrEnum permet la conversion directe si la valeur correspond
        day = WeekDay(raw_value)
        print(f"Jour détecté : {day.value} (Instance de {type(day).__name__})")
        
    except ValueError:
        # 4. Gestion d'erreur
        print(f"Erreur : '{raw_value}' n'est pas un jour valide.")

process_day("monday")
process_day("funday")
```
</details>