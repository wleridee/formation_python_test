---
sidebar_label: Module `json` : Sérialisation et Désérialisation
sidebar_position: 25
---

# Chapitre 25 : Module `json` : Sérialisation et Désérialisation

JSON et Python, Sérialisation (dumps), Désérialisation (loads), Fichiers JSON

Dans le monde du développement moderne, les applications ne vivent pas en vase clos. Elles communiquent : une API web envoie des données à une application mobile, un script de configuration stocke des préférences, ou un serveur sauvegarde l'état d'une partie. Le langage universel pour ces échanges est le **JSON** (JavaScript Object Notation).

Le module `json` de Python est l'outil standard pour traduire les objets Python (dictionnaires, listes) en format texte JSON ("sérialisation") et inversement ("désérialisation"). C'est une compétence fondamentale pour tout développeur backend ou data.

---

## 1. Correspondance des Types (Mapping) {#correspondance-des-types}

### 1. Quoi
Le JSON est très proche de la syntaxe Python, mais il y a des différences subtiles. Le module `json` gère la conversion automatique entre les types natifs des deux langages.

### 2. Pourquoi
Comprendre ce tableau est essentiel pour éviter les surprises (comme un tuple qui devient une liste et ne redevient jamais un tuple).

### 3. Comment

#### D. Tableau Comparatif

| Python | JSON | Remarques |
| :--- | :--- | :--- |
| `dict` | `object` `{...}` | Les clés doivent être des chaînes |
| `list`, `tuple` | `array` `[...]` | **Attention** : Un tuple devient une liste en JSON |
| `str` | `string` | |
| `int`, `float` | `number` | |
| `True` / `False` | `true` / `false` | Différence de casse (minuscule en JSON) |
| `None` | `null` | |

### 🚨 Limitations
Les types complexes comme `datetime`, `set`, ou vos propres classes (`class User`) **ne sont pas compatibles** par défaut. Essayer de les convertir lèvera une erreur `TypeError`.

---

## 2. Sérialisation : De Python vers JSON {#serialisation-python-vers-json}

### 1. Quoi
La **sérialisation** (ou encodage) consiste à transformer un objet Python en une chaîne de caractères au format JSON. On utilise principalement deux fonctions :
*   `json.dumps()` (Dump String) : Renvoie une chaîne de caractères.
*   `json.dump()` (Dump File) : Écrit directement dans un fichier ouvert.

### 2. Pourquoi
Pour envoyer des données sur le réseau (API) ou sauvegarder des configurations.

### 3. Comment

#### A. Syntaxe de base (`dumps`)

```python
import json

data = {
    "username": "Alice",
    "active": True,
    "roles": ("admin", "editor"), # Tuple
    "meta": None
}

# Conversion en chaîne JSON
json_str = json.dumps(data)

print(json_str)
# Résultat (noter les minuscules true/null et le tuple devenu tableau) :
# {"username": "Alice", "active": true, "roles": ["admin", "editor"], "meta": null}
```

#### B. Formatage lisible (Pretty Print)
Pour rendre le JSON lisible par un humain (fichiers de config, logs), utilisez `indent` et `ensure_ascii`.

```python
# indent=4 : Ajoute des sauts de ligne et 4 espaces
# ensure_ascii=False : Permet d'écrire les caractères accentués tels quels (é au lieu de \u00e9)
pretty_json = json.dumps(data, indent=4, ensure_ascii=False)
print(pretty_json)
```

### 4. Zone de Danger
❌ **Sérialiser des objets métier directement** :
```python
user = User("Alice")
json.dumps(user) # ❌ TypeError: Object of type User is not JSON serializable
```
✅ **Convertir en dict d'abord** :
Sérialisez `user.__dict__` ou une méthode `user.to_json()` qui retourne un dictionnaire.

---

## 3. Désérialisation : De JSON vers Python {#deserialisation-json-vers-python}

### 1. Quoi
La **désérialisation** (ou décodage) est l'opération inverse : transformer une chaîne JSON en structures de données Python (dict, list...).
*   `json.loads()` (Load String) : Depuis une chaîne.
*   `json.load()` (Load File) : Depuis un fichier ouvert.

### 2. Pourquoi
Pour lire la réponse d'une API ou charger un fichier de configuration au démarrage de l'application.

### 3. Comment

#### A. Syntaxe de base (`loads`)

```python
import json

json_response = '{"id": 101, "price": 99.99, "in_stock": true}'

try:
    product = json.loads(json_response)
    print(f"Produit : {product['id']} - Prix : {product['price']}€")
    print(type(product)) # <class 'dict'>

except json.JSONDecodeError:
    print("Erreur : La chaîne fournie n'est pas du JSON valide.")
```

### 🚨 Limitations
Le JSON ne supporte pas les commentaires (`//` ou `#`). Si vous essayez de charger un fichier JSON contenant des commentaires, `json.loads` plantera.

---

## 4. Travailler avec des Fichiers JSON {#fichiers-json}

### 1. Quoi
Combinaison du module `json` avec l'ouverture de fichiers (`open`) vue au chapitre 23.

### 2. Pourquoi
C'est le standard de facto pour stocker des configurations légères ou échanger des données entre programmes.

### 3. Comment

#### A. Écriture dans un fichier (`dump`)

```python
import json

config = {
    "theme": "dark",
    "version": 3.14,
    "plugins": ["linter", "formatter"]
}

# On utilise 'w' pour écrire
with open("settings.json", "w", encoding="utf-8") as f:
    json.dump(config, f, indent=4)
```

#### B. Lecture d'un fichier (`load`)

```python
import json

try:
    with open("settings.json", "r", encoding="utf-8") as f:
        # json.load lit le fichier et convertit directement
        loaded_config = json.load(f)
        
    print(f"Thème chargé : {loaded_config['theme']}")

except FileNotFoundError:
    print("Fichier de configuration introuvable.")
```

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-25}

1.  **Quelle est la différence entre `json.dumps()` et `json.dump()` ?**
    `dumps()` (avec un 's' pour String) renvoie une chaîne de caractères JSON en mémoire. `dump()` écrit directement le JSON dans un objet fichier ouvert.

2.  **Comment un tuple Python `(1, 2)` est-il converti en JSON ? Et si on le recharge en Python ?**
    Il est converti en tableau JSON `[1, 2]`. Lors du rechargement (`loads`), il deviendra une **liste** Python `[1, 2]`, pas un tuple.

3.  **Pourquoi utiliser l'argument `ensure_ascii=False` ?**
    Pour que les caractères non-ASCII (accents, emojis) soient écrits lisiblement (ex: `"café"`) au lieu de leur code unicode (ex: `"caf\u00e9"`).

4.  **Quelle exception est levée si le JSON est mal formé (ex: virgule manquante) ?**
    `json.JSONDecodeError`.

---

## Exercices : {#exercices-25}

### Exercice 1 - Gestionnaire de Préférences Utilisateur {#exercice-1---preferences}

🎯 **Objectif** : Lire et écrire un fichier JSON.

💼 **Mise en situation** : Vous créez une petite application CLI. Au premier lancement, elle demande le nom de l'utilisateur et sa couleur préférée, puis sauvegarde ces infos. Au prochain lancement, elle accueille l'utilisateur par son nom.

📝 **Énoncé** :
1.  Utilisez `pathlib` pour vérifier si `user_prefs.json` existe.
2.  Si oui : chargez-le et affichez "Bon retour [Nom], je mets l'interface en [Couleur] !".
3.  Si non : demandez les infos via `input()`, créez le dictionnaire, sauvegardez-le dans le fichier JSON, et affichez "Infos sauvegardées".

📺 **Résultat attendu** :
*Premier lancement :*
```text
Nom ? Alice
Couleur ? Rouge
Infos sauvegardées.
```
*Deuxième lancement :*
```text
Bon retour Alice, je mets l'interface en Rouge !
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
import json
from pathlib import Path

# Chemin du fichier
prefs_file = Path("user_prefs.json")

if prefs_file.exists():
    # Cas 1 : Le fichier existe, on lit
    try:
        # read_text() est un raccourci pathlib, on peut passer le contenu à loads()
        # Ou utiliser la méthode classique open() + json.load()
        with open(prefs_file, "r", encoding="utf-8") as f:
            data = json.load(f)
            
        print(f"Bon retour {data['name']}, je mets l'interface en {data['color']} !")
    except json.JSONDecodeError:
        print("Erreur : Fichier de préférences corrompu.")
else:
    # Cas 2 : Le fichier n'existe pas, on crée
    name = input("Nom ? ")
    color = input("Couleur ? ")
    
    prefs = {"name": name, "color": color}
    
    with open(prefs_file, "w", encoding="utf-8") as f:
        json.dump(prefs, f, indent=4, ensure_ascii=False)
        
    print("Infos sauvegardées.")
```
</details>

### Exercice 2 - Filtrage de Données API {#exercice-2---filtrage-api}

🎯 **Objectif** : Désérialiser une chaîne complexe et manipuler les listes/dicts.

💼 **Mise en situation** : Vous recevez une réponse d'une API e-commerce (simulée par une string) contenant une liste de produits. Vous devez extraire uniquement les produits en stock coûtant moins de 50€.

📝 **Énoncé** :
1.  Copiez la variable `api_response` (voir solution).
2.  Utilisez `json.loads()` pour convertir la chaîne.
3.  Utilisez une compréhension de liste pour filtrer les produits (`in_stock` est vrai ET `price` < 50).
4.  Affichez les noms des produits retenus.

📺 **Résultat attendu** :
```text
Produits abordables en stock :
- Souris sans fil
- Tapis de souris
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
import json

# Simulation de la réponse API (Chaîne JSON)
api_response = """
[
    {"id": 1, "name": "Laptop Gamer", "price": 1500.00, "in_stock": true},
    {"id": 2, "name": "Souris sans fil", "price": 25.50, "in_stock": true},
    {"id": 3, "name": "Écran 4K", "price": 450.00, "in_stock": false},
    {"id": 4, "name": "Clavier Mécanique", "price": 80.00, "in_stock": true},
    {"id": 5, "name": "Tapis de souris", "price": 15.00, "in_stock": true}
]
"""

# 1. Désérialisation
products = json.loads(api_response)

# 2. Filtrage
# On cherche : in_stock == True AND price < 50
affordable_products = [
    p for p in products 
    if p["in_stock"] and p["price"] < 50
]

print("Produits abordables en stock :")
for item in affordable_products:
    print(f"- {item['name']}")
```
</details>

### Exercice 3 - Le Piège des Dates {#exercice-3---dates}

🎯 **Objectif** : Gérer les limitations de sérialisation (`datetime`).

💼 **Mise en situation** : Vous voulez sauvegarder un log d'événement contenant un timestamp précis. JSON ne connait pas les dates.

📝 **Énoncé** :
1.  Créez un dictionnaire avec un message et la date actuelle (`datetime.now()`).
2.  Essayez de faire un `json.dumps()`. Cela va planter (`TypeError`).
3.  Utilisez l'argument `default=str` dans `json.dumps` pour convertir automatiquement ce qui n'est pas reconnu en chaîne de caractères.
4.  Affichez le résultat.

📺 **Résultat attendu** :
```text
{"event": "Login", "timestamp": "2023-10-27 10:30:00.123456"}
```
*(La date sera sous forme de chaîne)*

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
import json
from datetime import datetime

log_entry = {
    "event": "Login",
    "user_id": 42,
    "timestamp": datetime.now() # Objet non sérialisable par défaut
}

print("Tentative de sérialisation...")

try:
    # Ceci va échouer
    print(json.dumps(log_entry))
except TypeError as e:
    print(f"Échec attendu : {e}")

print("\nSérialisation réussie avec conversion :")

# L'argument 'default' prend une fonction à appeler si le type n'est pas reconnu
# Ici, on utilise la fonction 'str' qui convertit la date en chaîne
json_str = json.dumps(log_entry, indent=2, default=str)

print(json_str)
```
</details>