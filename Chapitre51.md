---
sidebar_label: Annotation de Type (Type Hinting)
sidebar_position: 51
---

# Chapitre 51 : Annotation de Type (Type Hinting)

Variables typées, Fonctions typées, Modules typing (List, Dict, Union), Mypy

Python est un langage à **typage dynamique** : vous n'avez pas besoin de déclarer le type d'une variable avant de l'utiliser. C'est génial pour le prototypage rapide, mais cela peut devenir un cauchemar de maintenance sur les gros projets ("Attends, cette fonction `process_data`, elle attend une liste d'ID ou une liste d'objets User ?").

Introduit progressivement depuis Python 3.5, le **Type Hinting** permet d'ajouter des annotations de type optionnelles. Couplé à un analyseur statique comme **Mypy**, cela apporte la robustesse des langages compilés (comme Java ou TypeScript) sans perdre la souplesse de Python.

---

## 1. La Syntaxe de Base : Variables et Fonctions {#syntaxe-base-type-hinting}

### 1. Quoi
Le Type Hinting consiste à ajouter des métadonnées à votre code pour indiquer quel type de donnée est attendu (pour les arguments) et quel type est renvoyé (pour les fonctions).

### 2. Pourquoi
1.  **Documentation vivante** : Plus besoin de docstrings obsolètes ("Retourne un int" alors que le code retourne maintenant un float). La signature *est* la documentation.
2.  **Autocomplétion (IntelliSense)** : Les IDE comme VS Code peuvent vous suggérer les méthodes appropriées car ils savent exactement quel objet vous manipulez.
3.  **Détection d'erreurs avant exécution** : Repérer les bugs de type (ex: additionner un nombre et une chaîne) avant même de lancer le script.

### 3. Comment

#### A. Variables simples
La syntaxe est `nom_variable: type = valeur`.

```python
# Typage explicite
age: int = 25
taux_conversion: float = 1.15
nom_client: str = "Alice Corp"
est_actif: bool = True

# Python 3.14 infère souvent le type tout seul, 
# l'annotation sur les variables simples est surtout utile si la déclaration est séparée
score: int
# ... plus loin ...
score = 100 
```

#### B. Fonctions (Arguments et Retour)
C'est ici que le typage est le plus puissant. On utilise `->` pour le retour.

```python
def calculer_ttc(prix_ht: float, tva: float = 0.20) -> float:
    """Calcule le prix TTC."""
    return prix_ht * (1 + tva)

# L'IDE saura que 'resultat' est un float
resultat = calculer_ttc(100.0)
```

### 4. Zone de Danger
❌ **Ne pas croire que Python vérifie les types à l'exécution** :
```python
x: int = "Je suis une string"
print(x) # Cela fonctionne parfaitement ! Python ignore les types au runtime.
```
✅ **Bonne Pratique** : Les annotations sont destinées aux humains et aux outils d'analyse (Mypy), pas à l'interpréteur Python.

---

## 2. Collections et Types Complexes Modernes {#collections-et-types-complexes}

### 1. Quoi
Comment typer une liste d'entiers ? Un dictionnaire avec des clés string et des valeurs float ? Depuis Python 3.9+, on utilise les classes natives (`list`, `dict`, `tuple`) directement.

### 2. Pourquoi
Pour éviter le `IndexError` ou le `KeyError` classique parce qu'on pensait manipuler une liste de chaînes alors qu'on a reçu une liste de dictionnaires.

### 3. Comment

#### A. Listes, Dictionnaires et Tuples
N'utilisez plus `typing.List` ou `typing.Dict` (dépréciés).

```python
# Une liste contenant uniquement des chaînes
emails: list[str] = ["a@b.com", "c@d.com"]

# Un dictionnaire : Clé (str) -> Valeur (float)
prix_produits: dict[str, float] = {
    "pomme": 1.2,
    "banane": 0.9
}

# Un tuple de taille fixe avec des types précis
# Ici : (latitude, longitude)
coordonnees: tuple[float, float] = (48.8566, 2.3522)
```

#### B. Union et Optionnels (`|`)
Parfois, une variable peut avoir plusieurs types. Depuis Python 3.10, on utilise l'opérateur `|`.

```python
def rechercher_user(user_id: int | str) -> dict[str, str] | None:
    # Accepte un int OU une str
    # Retourne un dict OU None (si pas trouvé)
    if user_id == "admin":
        return {"name": "Admin"}
    return None
```
*Note : `Type | None` remplace l'ancien `Optional[Type]`.*

#### C. Any : La porte de sortie
Si vous ne connaissez pas le type ou si c'est trop dynamique, utilisez `Any` du module `typing`.

```python
from typing import Any

def log_data(data: Any) -> None:
    print(f"Log: {data}")
    # Mypy ne vérifiera rien ici. À utiliser avec parcimonie !
```

---

## 3. L'Analyse Statique avec `mypy` {#analyse-statique-mypy}

### 1. Quoi
**Mypy** est un outil en ligne de commande qui lit vos annotations et vérifie la cohérence logique de votre code. C'est le "compilateur" des types Python.

### 2. Pourquoi
Les annotations ne servent à rien si personne ne les vérifie. Mypy est le gardien qui vous empêche de commiter du code incohérent.

### 3. Comment

#### A. Installation et Exécution
```bash
pip install mypy
```

Créez un fichier `buggy.py` :
```python
def division(a: int, b: int) -> float:
    return a / b

division("10", 2) # Erreur : "10" est une str
```

Lancez Mypy :
```bash
mypy buggy.py
```

#### B. Résultat
Mypy va détecter l'erreur :
```text
buggy.py:4: error: Argument 1 to "division" has incompatible type "str"; expected "int"
Found 1 error in 1 file (checked 1 source file)
```

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Terminal montrant une exécution de Mypy détectant une erreur.
> **Annotation** : Surlignez la ligne "error: Argument ... has incompatible type".
> **Alt Text suggéré** : Rapport d'erreur Mypy dans un terminal CLI.

### 🚨 Limitations de Mypy
*   **Bibliothèques tierces** : Certaines vieilles bibliothèques n'ont pas de types. Mypy peut se plaindre. Il faut parfois installer des paquets "stubs" (ex: `types-requests`) ou ignorer les erreurs.
*   **Courbe d'apprentissage** : Typer des décorateurs ou des fonctions très dynamiques (`**kwargs`) peut être complexe.

---

## 4. Créer ses propres types (Alias et TypedDict) {#types-personnalises}

### 1. Quoi
Pour rendre le code plus lisible, on peut donner un nom à une structure de type complexe.

### 2. Pourquoi
Évite de répéter `dict[str, int | str | float]` partout dans le code.

### 3. Comment

#### A. TypeAlias (Moderne)
```python
# Définition d'un alias
Vector = list[float]
UserId = int | str

def move(v: Vector, uid: UserId) -> None:
    # ...
    pass
```

#### B. TypedDict (Pour des dictionnaires structurés)
Si vos dictionnaires ressemblent à des objets JSON fixes :

```python
from typing import TypedDict

class UserConfig(TypedDict):
    username: str
    is_admin: bool
    retries: int

def create_user(config: UserConfig) -> None:
    print(config['username']) # Autocomplétion disponible ici !

conf: UserConfig = {"username": "bob", "is_admin": False, "retries": 3}
```

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-51}

1.  **Le typage en Python empêche-t-il le programme de planter à l'exécution si les types sont faux ?**
    Non. L'interpréteur Python ignore les annotations. Elles servent uniquement aux outils externes (IDE, Mypy) et aux humains.

2.  **Quelle est la syntaxe moderne pour dire qu'une fonction retourne soit une chaine, soit rien (`None`) ?**
    `-> str | None`.

3.  **Quel outil faut-il installer pour vérifier automatiquement les types ?**
    Mypy (`pip install mypy`).

4.  **Comment typer une liste qui contient des entiers ?**
    `list[int]`.

---

## Exercices : {#exercices-51}

### Exercice 1 - Le Gestionnaire d'Inventaire {#exercice-1-inventaire}

🎯 **Objectif** : Typer correctement des collections complexes.

💼 **Mise en situation** : Vous gérez un stock e-commerce. L'inventaire est un dictionnaire où la clé est le nom du produit (str) et la valeur est la quantité (int).

📝 **Énoncé** :
1.  Écrivez une fonction `mettre_a_jour_stock` qui prend l'inventaire et un tuple `(produit, quantité)` à ajouter.
2.  La fonction ne retourne rien.
3.  Utilisez les annotations modernes (`dict[]`, `tuple[]`).
4.  Ajoutez une ligne de code qui viole le type (ex: quantité sous forme de string) et vérifiez (mentalement ou avec mypy) l'erreur.

📺 **Résultat attendu** :
Le code doit être valide pour Python 3.14 et Mypy.

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
# Définition des types pour la clarté
Inventaire = dict[str, int]
Mouvement = tuple[str, int]

def mettre_a_jour_stock(stock: Inventaire, ajout: Mouvement) -> None:
    produit, quantite = ajout
    if produit in stock:
        stock[produit] += quantite
    else:
        stock[produit] = quantite

# Utilisation correcte
mon_stock: Inventaire = {"pommes": 10}
mettre_a_jour_stock(mon_stock, ("pommes", 5))

# Erreur détectée par Mypy (mais passe à l'exécution en crashant plus tard)
# mettre_a_jour_stock(mon_stock, ("poires", "beaucoup")) 
```
</details>

### Exercice 2 - Traitement de Données API (Union) {#exercice-2-api-union}

🎯 **Objectif** : Gérer les types multiples (`|`) et `Any`.

💼 **Mise en situation** : Une API externe renvoie parfois un code d'erreur (int), parfois un message d'erreur (str), ou une liste de résultats (list).

📝 **Énoncé** :
1.  Créez une fonction `analyser_reponse(data: ...)` qui accepte `int`, `str` ou `list[str]`.
2.  Si c'est un entier -> print "Code erreur".
3.  Si c'est une liste -> print "Données reçues".
4.  Si c'est une string -> print "Message".
5.  Utilisez `isinstance` pour vérifier le type à l'intérieur (Type Guarding).

📺 **Résultat attendu** :
Mypy ne doit signaler aucune erreur, car vous gérez tous les cas possibles de l'Union.

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
def analyser_reponse(data: int | str | list[str]) -> None:
    # Mypy sait que 'data' peut être l'un des 3 types
    
    if isinstance(data, int):
        # Dans ce bloc, Mypy sait que data est un int
        print(f"Code erreur : {data}")
    elif isinstance(data, list):
        # Ici, c'est une liste
        print(f"Données reçues : {len(data)} items")
    else:
        # Par élimination, c'est une str
        print(f"Message : {data}")

analyser_reponse(404)
analyser_reponse(["user1", "user2"])
```
</details>

### Exercice 3 - Le Piège du None {#exercice-3-piege-none}

🎯 **Objectif** : Comprendre pourquoi `| None` est vital.

💼 **Mise en situation** : Une fonction de recherche retourne un utilisateur ou rien. Vous devez empêcher le programme de planter si l'utilisateur n'existe pas.

📝 **Énoncé** :
1.  Fonction `get_user_name(id: int) -> str | None`. Retourne "Alice" si id=1, sinon `None`.
2.  Code principal : Récupère le nom et appelle `.upper()`.
3.  Analysez ce code avec Mypy. Il doit vous interdire d'appeler `.upper()` directement sans vérifier si c'est `None` avant.

📺 **Résultat attendu** :
Mypy Error: `Item "None" of "str | None" has no attribute "upper"`.

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
def get_user_name(uid: int) -> str | None:
    if uid == 1:
        return "Alice"
    return None

nom = get_user_name(99)

# ❌ Ce code est rejeté par Mypy (Safety first !)
# print(nom.upper()) 

# ✅ Ce code est validé
if nom is not None:
    print(nom.upper())
else:
    print("Utilisateur inconnu")
```
</details>