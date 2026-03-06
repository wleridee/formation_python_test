---
sidebar_label: "Votre Première API FastAPI : 'Hello World'"
sidebar_position: 3
difficulty: "junior"
---

# Votre Première API FastAPI : 'Hello World' {#votre-premiere-api-fastapi-hello-world-3}

Dans le chapitre précédent, nous avons mis en place notre environnement de développement et validé que tout fonctionnait. Nous allons maintenant décortiquer le code minimaliste que nous avons utilisé pour comprendre en profondeur les briques fondamentales de toute application FastAPI.

Notre objectif est de maîtriser les trois concepts clés présents dans ce simple fichier `main.py` :
1.  L'instance `FastAPI`.
2.  Le décorateur d'opération de chemin (`@app.get`).
3.  La fonction de l'opération qui gère la logique.

Reprenons notre code de base :
```python
# main.py
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"Hello": "World"}
```

## Concept 1 : L'Instance `FastAPI` {#concept-1-linstance-fastapi-3}

### Quoi ? {#quoi-3}
La ligne `app = FastAPI()` crée l'objet central de votre application. C'est le point d'entrée principal qui va orchestrer toutes les fonctionnalités de votre API. Vous pouvez lui donner un autre nom, mais `app` est une convention largement adoptée.

### Pourquoi ? {#pourquoi-3}
Cette instance `app` est essentielle car c'est elle qui est utilisée pour déclarer les "routes" (ou "endpoints") de votre API. C'est également à travers cet objet que vous pourrez configurer des aspects plus avancés de votre application (middlewares, gestionnaires d'événements, etc.) que nous verrons plus tard.

### Comment ? {#comment-3}
La syntaxe est simple : on importe la classe `FastAPI` du module `fastapi` et on l'instancie.
```python
from fastapi import FastAPI

# Crée une instance de l'application
app = FastAPI()
```
Cette seule ligne suffit à créer une application web ASGI fonctionnelle.

### Zone de Danger {#zone-de-danger-3}
Pour une application simple, vous n'aurez qu'une seule instance de `FastAPI`. Dans des projets plus complexes, vous apprendrez à diviser votre code en plusieurs modules avec des "Routers", mais ils seront tous finalement connectés à cette unique instance `app` principale. Évitez de créer plusieurs instances `FastAPI` dans un même projet simple.

---

## Concept 2 : Le Décorateur d'Opération de Chemin {#concept-2-le-decorateur-doperation-de-chemin-3}

### Quoi ? {#quoi-4}
La ligne `@app.get("/")` est un **décorateur** Python. En FastAPI, on l'appelle un **décorateur d'opération de chemin** (Path Operation Decorator). Son rôle est de lier une URL (le "chemin") et une méthode HTTP (ici, `GET`) à la fonction Python qui se trouve juste en dessous.

### Pourquoi ? {#pourquoi-4}
C'est le mécanisme de **routage**. Il indique à FastAPI : "Quand tu reçois une requête HTTP de type GET sur l'URL racine (`/`), exécute la fonction `read_root()`". Sans ce décorateur, la fonction `read_root` ne serait qu'une simple fonction Python, jamais appelée par le framework.

### Comment ? {#comment-5}
La syntaxe se décompose ainsi :
-   `@app`: On utilise notre instance `FastAPI`.
-   `.get`: C'est la **méthode HTTP**. FastAPI fournit un décorateur pour chaque méthode standard :
    -   `@app.post()`
    -   `@app.put()`
    -   `@app.delete()`
    -   `@app.patch()`
    -   etc.
-   `("/")`: C'est le **chemin** (path) de l'URL. `/` représente la racine du site. Si vous aviez mis `("/users")`, l'URL correspondante serait `http://127.0.0.1:8000/users`.

```python
# Déclare un endpoint qui répond aux requêtes GET sur l'URL "/"
@app.get("/")
def read_root():
    # ... la logique se trouve ici
    pass
```

### Zone de Danger {#zone-de-danger-6}
L'ordre des décorateurs peut avoir de l'importance pour des chemins plus complexes. Par exemple, si vous avez `/users/me` et `/users/{user_id}`, vous devez déclarer `/users/me` *avant* `/users/{user_id}` pour qu'il soit détecté correctement. FastAPI est assez intelligent, mais c'est une bonne pratique à garder en tête.

---

## Concept 3 : La Fonction de l'Opération {#concept-3-la-fonction-de-loperation-3}

### Quoi ? {#quoi-7}
La fonction `read_root()` est appelée "fonction de l'opération de chemin" (Path Operation Function). C'est une fonction Python standard qui contient la logique métier à exécuter lorsque le endpoint est appelé.

### Pourquoi ? {#pourquoi-8}
C'est ici que la "magie" opère. Vous pouvez y mettre n'importe quel code : interroger une base de données, appeler une autre API, effectuer un calcul, etc. Le résultat de cette fonction (`return`) sera la réponse envoyée au client.

### Comment ? {#comment-9}
Dans notre cas, `def read_root():` définit la fonction.
La ligne `return {"Hello": "World"}` renvoie un dictionnaire Python.
FastAPI se charge automatiquement de convertir ce dictionnaire en **JSON** valide et de l'envoyer dans la réponse HTTP avec le bon `Content-Type` (`application/json`).

FastAPI peut retourner de nombreux types de données, qui seront convertis automatiquement :
-   `dict`
-   `list`
-   `str`, `int`, `float`, `bool`
-   Modèles Pydantic (nous verrons cela très bientôt)

### Zone de Danger {#zone-de-danger-10}
La fonction `read_root` est une fonction synchrone (`def`). Pour des opérations simples et rapides, c'est suffisant. Cependant, si votre fonction doit effectuer une opération longue (comme un appel réseau ou une requête en base de données), elle bloquera tout le serveur pendant son exécution. Pour ces cas, FastAPI brille avec les fonctions asynchrones (`async def`), que nous aborderons dans les chapitres avancés.

---

## Conclusion et Validation {#conclusion-et-validation-3}

Vous comprenez maintenant parfaitement comment fonctionne l'API la plus simple avec FastAPI. Vous savez comment lier une URL à une fonction et comment renvoyer une réponse JSON. Il est temps de mettre ces connaissances en pratique.

### 3 Questions Clés {#3-questions-cles-3}

1.  Quel est le rôle de la ligne `app = FastAPI()` ?
2.  Que signifie la partie `.get` dans le décorateur `@app.get("/")` et quelles sont les autres alternatives courantes ?
3.  Si une fonction d'opération de chemin retourne un dictionnaire Python, que reçoit le client (navigateur ou autre) comme réponse ?

### 3 Exercices Progressifs {#3-exercices-progressifs-3}

**Instructions** : Modifiez votre fichier `main.py` pour réaliser les exercices suivants. Gardez votre serveur `uvicorn` lancé avec l'option `--reload` pour voir les changements instantanément.

**Exercice 1 : Créer un Endpoint de Statut**
Créez un nouvel endpoint à l'URL `/status`. Il doit répondre aux requêtes `GET` et retourner le JSON suivant : `{"status": "ok"}`.

<details>
<summary>Découvrir la solution commentée</summary>

```python
# main.py
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"Hello": "World"}

# --- Solution Exercice 1 ---
# On ajoute un nouveau décorateur pour le chemin /status
@app.get("/status")
def get_status():
    # Le nom de la fonction (get_status) est arbitraire mais doit être unique.
    # On retourne un simple dictionnaire qui sera converti en JSON.
    return {"status": "ok"}
```
</details>

**Exercice 2 : Endpoint d'Informations sur l'API**
Créez un endpoint à l'URL `/api/info`. Il doit répondre aux requêtes `GET` et retourner un dictionnaire contenant des informations sur votre API, par exemple :
```json
{
  "api_name": "Formation FastAPI",
  "version": "1.0.0"
}
```
<details>
<summary>Découvrir la solution commentée</summary>

```python
# main.py
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"Hello": "World"}

@app.get("/status")
def get_status():
    return {"status": "ok"}

# --- Solution Exercice 2 ---
# On définit le chemin complet /api/info
@app.get("/api/info")
def get_api_info():
    # On structure le dictionnaire avec les clés et valeurs demandées.
    return {
        "api_name": "Formation FastAPI",
        "version": "1.0.0"
    }
```
</details>

**Exercice 3 : Lister des Données Statiques**
Simulez une base de données en créant un endpoint `GET` sur `/items` qui retourne une liste d'objets JSON. Chaque objet doit représenter un item avec un `id` et un `name`.
Exemple de retour attendu :
```json
[
  { "id": 1, "name": "Laptop" },
  { "id": 2, "name": "Smartphone" },
  { "id": 3, "name": "Tablet" }
]
```
<details>
<summary>Découvrir la solution commentée</summary>

```python
# main.py
from fastapi import FastAPI
from typing import List, Dict, Any

app = FastAPI()

# ... (les autres endpoints) ...

# --- Solution Exercice 3 ---
@app.get("/items")
# La fonction retourne une liste Python.
# Chaque élément de la liste est un dictionnaire.
# FastAPI convertira la liste de dictionnaires en un tableau d'objets JSON.
def list_items():
    return [
      { "id": 1, "name": "Laptop" },
      { "id": 2, "name": "Smartphone" },
      { "id": 3, "name": "Tablet" }
    ]
```
*Note : Vous pouvez vérifier le résultat dans votre navigateur à l'adresse `http://127.0.0.1:8000/items` ou via la documentation sur `/docs`.*
</details>