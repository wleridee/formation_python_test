---
sidebar_label: "Path Operations : Gérer les Requêtes GET"
sidebar_position: 5
difficulty: "junior"
---

# Path Operations : Gérer les Requêtes GET {#path-operations-gerer-les-requetes-get-5}

Jusqu'à présent, nous avons créé des endpoints avec des chemins fixes comme `/` ou `/status`. Cependant, la plupart des API doivent gérer des ressources dynamiques. Par exemple, comment récupérer un utilisateur spécifique par son ID, ou un produit par sa référence ? La réponse est : les **paramètres de chemin** (Path Parameters).

Ce chapitre se concentre sur la méthode `GET`, utilisée pour la lecture de données, et vous apprendra à construire des URL dynamiques et robustes.

## Concept 1 : Les Paramètres de Chemin (Path Parameters) {#concept-1-les-parametres-de-chemin-path-parameters-5}

### Quoi ? {#quoi-5}
Un paramètre de chemin est une partie variable de l'URL. Il est défini en utilisant des accolades `{}` dans la chaîne de caractères du chemin du décorateur.

### Pourquoi ? {#pourquoi-5}
Cela vous permet de créer un seul endpoint capable de gérer une infinité de ressources. Au lieu de créer un endpoint pour `/users/1`, un autre pour `/users/2`, etc., vous créez un unique endpoint `/users/{user_id}` qui gère tous les cas. C'est le fondement des API RESTful pour identifier une ressource unique.

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-5}
La syntaxe est très intuitive :
1.  Vous déclarez la variable dans le chemin avec des accolades : `@app.get("/items/{item_id}")`.
2.  Vous ajoutez un paramètre du **même nom** à votre fonction d'opération : `def read_item(item_id):`.

FastAPI fait automatiquement le lien entre les deux.

**Cas Réel : API d'un site e-commerce**
Imaginons que nous voulons récupérer un produit en fonction de son ID.

```python
# main.py
from fastapi import FastAPI

app = FastAPI()

@app.get("/products/{product_id}")
def read_product(product_id): # Le nom 'product_id' doit correspondre
    return {"product_id": product_id}
```
Si vous lancez cette application et visitez `http://127.0.0.1:8000/products/123`, la réponse sera :
```json
{ "product_id": "123" }
```
Notez que la valeur `"123"` est une chaîne de caractères. Nous allons améliorer cela tout de suite.

### Zone de Danger {#zone-de-danger-5}
L'erreur la plus courante est une faute de frappe ou une différence entre le nom dans le chemin et le nom dans la signature de la fonction. Si vous écrivez `@app.get("/items/{item_id}")` et `def read_item(id):`, FastAPI lèvera une erreur au démarrage, car il ne peut pas faire le lien.

---

## Concept 2 : Les `type hints` pour la Validation et Conversion {#concept-2-les-type-hints-pour-la-validation-et-conversion-5}

### Quoi ? {#quoi-6}
Les annotations de type (type hints) en Python ne sont pas juste de la documentation. FastAPI les utilise activement pour **valider** et **convertir** les données entrantes.

### Pourquoi ? {#pourquoi-7}
C'est une des fonctionnalités les plus puissantes de FastAPI. En ajoutant simplement `: int` à votre paramètre, vous demandez à FastAPI de s'assurer que la valeur reçue est bien un entier.
-   Si c'est le cas, il la convertit (`"123"` devient `123`).
-   Si ce n'est pas le cas (ex: `/products/abc`), FastAPI bloque la requête et retourne **automatiquement** une erreur HTTP `422 Unprocessable Entity` avec un message JSON très clair expliquant le problème. Cela vous évite d'écrire du code de validation fastidieux.

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-6}
Reprenons notre exemple précédent en ajoutant une annotation de type.

```python
# main.py
from fastapi import FastAPI

app = FastAPI()

@app.get("/products/{product_id}")
def read_product(product_id: int): # On ajoute : int
    return {"product_id": product_id, "type": str(type(product_id))}
```
Maintenant, si vous visitez `http://127.0.0.1:8000/products/123` :
-   Le `product_id` dans l'URL est la chaîne `"123"`.
-   FastAPI voit l'annotation `product_id: int`.
-   Il essaie de convertir `"123"` en `123`. Ça fonctionne.
-   Il appelle votre fonction `read_product(123)`.
-   La réponse est `{"product_id": 123, "type": "<class 'int'>"}`. La valeur est maintenant un nombre !

Si vous essayez de visiter `http://127.0.0.1:8000/products/chaussure` :
-   FastAPI essaie de convertir `"chaussure"` en `int`. Ça échoue.
-   Il ne va même pas appeler votre fonction. Il renvoie directement une erreur 422.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : La documentation Swagger UI (/docs) montrant une erreur de validation.
> **Alt Text** : Interface Swagger UI après avoir tenté d'exécuter le endpoint /products/{product_id} avec la valeur "chaussure", affichant une réponse 422 avec un message d'erreur "value is not a valid integer".

### Zone de Danger {#zone-de-danger-8}
Les `type hints` fonctionnent pour tous les types standards de Python ( `str`, `int`, `float`, `bool`). `bool` est intéressant : les valeurs "true", "True", "1", "yes" seront converties en `True`, et "false", "False", "0", "no" en `False`. Cependant, n'essayez pas de typer un paramètre de chemin avec un type complexe comme `dict` ou `list`. Ce n'est pas supporté car une URL ne peut pas contenir nativement une structure complexe dans cette partie.

---

## Concept 3 : L'Ordre des Opérations est Crucial {#concept-3-lordre-des-operations-est-crucial-5}

### Quoi ? {#quoi-9}
FastAPI parcourt les opérations de chemin dans l'ordre où elles sont déclarées dans votre code, de haut en bas. La première qui correspond à l'URL de la requête est utilisée.

### Pourquoi ? {#pourquoi-10}
Si vous avez un chemin fixe qui ressemble à un chemin variable, l'ordre est primordial pour éviter les conflits. Par exemple, `/users/me` (pour l'utilisateur courant) et `/users/{user_id}` (pour un utilisateur spécifique). Si une requête arrive pour `/users/me`, elle pourrait correspondre aux deux.

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-10}
La règle est simple : **déclarez toujours les chemins les plus spécifiques (fixes) avant les chemins les plus génériques (variables).**

**Mauvais ordre (bug) :**
```python
# NE FAITES PAS ÇA
@app.get("/users/{user_id}")
def read_user(user_id: str):
    return {"user_id": user_id}

@app.get("/users/me")
def read_current_user():
    return {"user_id": "the current user"}
```
Avec cet ordre, si vous appelez `/users/me`, la première route `@app.get("/users/{user_id}")` va correspondre. FastAPI pensera que `"me"` est la valeur de `user_id`, et c'est la fonction `read_user` qui sera exécutée. La fonction `read_current_user` ne sera jamais atteinte.

**Bon ordre :**
```python
# BONNE PRATIQUE
@app.get("/users/me")
def read_current_user():
    return {"user_id": "the current user"}

@app.get("/users/{user_id}")
def read_user(user_id: str):
    return {"user_id": user_id}
```
Ici, si la requête est `/users/me`, elle correspondra d'abord au chemin fixe et la bonne fonction sera appelée. Si la requête est `/users/123`, elle ne correspondra pas au premier chemin, mais elle correspondra au second.

### Zone de Danger {#zone-de-danger-11}
Ce type de bug peut être subtil. Votre API semble fonctionner, mais certains endpoints spécifiques ne sont jamais appelés. Si vous avez un comportement étrange où la mauvaise fonction semble être appelée, vérifiez immédiatement l'ordre de vos déclarations de routes.

---

### 3 Questions Clés {#3-questions-cles-5}
1.  Comment définiriez-vous un endpoint pour récupérer un livre par son identifiant numérique, par exemple `/books/42` ?
2.  Un utilisateur fait une requête GET sur `/items/3.14`. Votre fonction est `def get_item(item_id: int):`. Que se passe-t-il ?
3.  Vous avez deux endpoints : un pour lister les articles en promotion (`/articles/promo`) et un pour afficher un article par son nom (`/articles/{article_name}`). Dans quel ordre devez-vous les déclarer dans votre fichier Python et pourquoi ?

### 3 Exercices Progressifs {#3-exercices-progressifs-5}

**Exercice 1 : Profil Utilisateur Simple**
Créez un endpoint `GET` à l'URL `/users/{username}`. Il doit accepter un nom d'utilisateur (chaîne de caractères) et le retourner dans une réponse JSON.
Par exemple, un appel à `/users/alex` doit retourner `{"username": "alex"}`.

<details>
<summary>Découvrir la solution commentée</summary>

```python
from fastapi import FastAPI

app = FastAPI()

# On définit le paramètre de chemin {username}.
# Le type hint 'str' est utilisé pour la clarté,
# bien que ce soit le comportement par défaut si aucun type n'est spécifié.
@app.get("/users/{username}")
def get_user_profile(username: str):
    # La fonction retourne un dictionnaire avec la valeur reçue.
    return {"username": username}
```
</details>

**Exercice 2 : Calculatrice d'IMC Basique**
Créez un endpoint `GET` qui prend un poids (en kg, `int`) et une taille (en cm, `int`) comme paramètres de chemin. L'URL doit être `/imc/{weight_kg}/{height_cm}`. La fonction doit calculer l'IMC (Indice de Masse Corporelle) et le retourner.
*Formule de l'IMC : poids (kg) / (taille (m))^2*

Par exemple, un appel à `/imc/70/175` doit retourner environ `{"imc": 22.86}`.

<details>
<summary>Découvrir la solution commentée</summary>

```python
from fastapi import FastAPI

app = FastAPI()

# On définit deux paramètres de chemin, chacun avec son type hint.
@app.get("/imc/{weight_kg}/{height_cm}")
def calculate_imc(weight_kg: int, height_cm: int):
    # On convertit la taille de cm en m pour la formule.
    height_m = height_cm / 100
    # On calcule l'IMC. On utilise round() pour un résultat plus propre.
    imc = round(weight_kg / (height_m ** 2), 2)
    return {"imc": imc}

# Testez avec http://127.0.0.1:8000/imc/70/175
# Essayez aussi avec des valeurs non entières dans l'URL pour voir la validation de FastAPI en action.
# Exemple : http://127.0.0.1:8000/imc/70.5/175
```
</details>

**Exercice 3 : Gestion de Conflits de Routes**
Créez une API pour un blog qui a deux types de routes pour les articles :
1.  Une route `GET` sur `/posts/latest` qui retourne `{"post": "Voici le dernier article"}`.
2.  Une route `GET` sur `/posts/{post_id}` qui prend un ID entier et retourne `{"post_id": post_id}`.

Implémentez ces deux routes dans le bon ordre pour que les deux fonctionnent correctement. Testez en appelant `/posts/latest` et `/posts/123`.

<details>
<summary>Découvrir la solution commentée</summary>

```python
from fastapi import FastAPI

app = FastAPI()

# On déclare le chemin fixe en PREMIER.
# Si une requête arrive pour /posts/latest, elle correspondra ici et s'arrêtera.
@app.get("/posts/latest")
def get_latest_post():
    return {"post": "Voici le dernier article"}

# On déclare le chemin variable en SECOND.
# Une requête comme /posts/123 ne correspondra pas à la route précédente,
# mais elle correspondra à celle-ci.
@app.get("/posts/{post_id}")
def get_post_by_id(post_id: int):
    return {"post_id": post_id}

# Si on avait inversé l'ordre, un appel à /posts/latest aurait été capturé
# par la route /posts/{post_id}, et FastAPI aurait essayé de convertir "latest" en int,
# ce qui aurait provoqué une erreur de validation 422.
```
</details>