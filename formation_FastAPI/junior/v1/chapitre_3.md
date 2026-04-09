---
sidebar_label: "Introduction aux types Python"
sidebar_position: 3
difficulty: "junior"
---

# Chapitre 3 : Introduction aux types Python {#chapitre-3-:-introduction-aux-types-python}

Type hints, validation automatique, Pydantic, sérialisation

## Le typage dynamique au service de la validation {#le-typage-dynamique-au-service-de-la-validation-3}

### 1. Quoi
Les **type hints** (indices de type) sont une fonctionnalité de Python permettant d'indiquer explicitement le type attendu pour les variables, les arguments de fonctions et les valeurs de retour. FastAPI utilise ces indices pour valider, convertir et documenter les données entrantes.

### 2. Pourquoi
En définissant les types, vous permettez à FastAPI de transformer automatiquement les données (ex: transformer une chaîne "123" en entier `123`) et de générer des erreurs explicites si le format est incorrect, le tout sans écrire de code de validation manuel.

### 3. Comment
A. **Syntaxe de base** :
```python
def saluer(nom: str, age: int) -> str:
    # On indique que 'nom' doit être une chaîne et 'age' un entier
    return f"Bonjour {nom}, vous avez {age} ans."
```

B. **Cas concret dans FastAPI** :
```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/{item_id}")
async def lire_item(item_id: int, q: str | None = None):
    # FastAPI valide que item_id est bien un entier
    # Sinon, il renvoie automatiquement une erreur 422 (Unprocessable Entity)
    return {"item_id": item_id, "q": q}
```

C. **Exemples pratiques** :
- **Validation de chemin** : `item_id: int` garantit que l'URL contient un nombre.
- **Paramètres optionnels** : `q: str | None = None` définit un paramètre de requête optionnel.
- **Conversion automatique** : Si vous passez `?item_id=42`, FastAPI le convertit en `int` avant d'appeler la fonction.

### 4. Zone de Danger
❌ **À ne pas faire** : Ignorer les types ou utiliser `Any` partout, ce qui annule les bénéfices de la validation automatique.
✅ **Bonne Pratique** : Être le plus précis possible dans les types (ex: `list[str]` plutôt que `list`).

---

## Questions clés (validation des acquis du chapitre) {#questions-clés-(validation-des-acquis-du-chapitre)-3}

- **Que se passe-t-il si un utilisateur envoie une chaîne de caractères là où un entier est attendu dans un paramètre de chemin ?**
  *Réponse : FastAPI renvoie automatiquement une réponse HTTP 422 avec un message d'erreur détaillant le problème de validation.*

- **Quel est l'avantage principal d'utiliser les type hints avec FastAPI ?**
  *Réponse : Ils permettent la validation automatique, la conversion des types et la génération automatique de la documentation OpenAPI.*

- **Comment définir un paramètre de requête optionnel en Python moderne ?**
  *Réponse : En utilisant l'union de types `Type | None` et en lui donnant une valeur par défaut de `None`.*

---

## Exercices : {#exercices-:-3}

### Exercice 1 - Typage simple {#exercice-1---typage-simple}

🎯 **Objectif** : Créer un endpoint qui calcule une réduction.

💼 **Mise en situation** : Vous développez une API pour un site e-commerce. Vous devez créer une route `/reduction` qui prend un prix (float) et un pourcentage (float).

📝 **Énoncé** : Créez une fonction qui retourne le prix final après réduction. Utilisez les type hints pour valider les entrées.

📺 **Résultat attendu** : Une requête sur `/reduction?prix=100&pourcentage=20` doit retourner `{"prix_final": 80.0}`.

💡 **Solution** :
<details><summary>Voir le code complet commenté</summary>

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/reduction")
async def calculer_reduction(prix: float, pourcentage: float):
    # On calcule le prix final en utilisant les types validés par FastAPI
    prix_final = prix * (1 - pourcentage / 100)
    return {"prix_final": prix_final}
```
</details>

### Exercice 2 - Paramètres optionnels {#exercice-2---paramètres-optionnels}

🎯 **Objectif** : Gérer des filtres optionnels.

💼 **Mise en situation** : Vous avez une liste de produits. L'utilisateur peut filtrer par catégorie ou par prix maximum.

📝 **Énoncé** : Créez une route `/produits` qui accepte `categorie` (str, optionnel) et `prix_max` (float, optionnel).

📺 **Résultat attendu** : `/produits?categorie=livre` doit fonctionner, tout comme `/produits` seul.

💡 **Solution** :
<details><summary>Voir le code complet commenté</summary>

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/produits")
async def lister_produits(categorie: str | None = None, prix_max: float | None = None):
    # On utilise | None pour indiquer que ces paramètres sont optionnels
    return {"categorie": categorie, "prix_max": prix_max}
```
</details>

### Exercice 3 - Validation stricte {#exercice-3---validation-stricte}

🎯 **Objectif** : Comprendre le comportement de FastAPI face aux erreurs.

💼 **Mise en situation** : Vous voulez vérifier que votre API rejette correctement les mauvaises données.

📝 **Énoncé** : Reprenez l'exercice 1. Que se passe-t-il si vous envoyez `/reduction?prix=cent&pourcentage=20` ? Testez-le via `/docs`.

📺 **Résultat attendu** : Une erreur 422 est renvoyée par le serveur car "cent" ne peut pas être converti en `float`.

💡 **Solution** :
<details><summary>Voir le code complet commenté</summary>

```python
# Aucune modification de code nécessaire.
# FastAPI intercepte la requête avant d'atteindre la fonction
# et renvoie une erreur 422 car 'prix' attend un float.
```
</details>