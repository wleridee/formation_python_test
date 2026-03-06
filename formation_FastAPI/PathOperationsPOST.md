---
sidebar_label: "Path Operations : Requêtes POST et Données du Corps"
sidebar_position: 10
difficulty: "junior"
---

# Path Operations : Requêtes POST et Données du Corps {#path-operations-requetes-post-et-donnees-du-corps-10}

Nous avons maîtrisé les requêtes `GET` pour lire des données, en utilisant les paramètres de chemin et de requête pour spécifier et filtrer nos demandes. Il est temps de passer à la création de ressources. Pour cela, nous utilisons la méthode HTTP `POST`, conçue pour envoyer des données au serveur afin qu'il crée une nouvelle entité.

Avec `POST`, les données ne sont plus dans l'URL mais dans le **corps de la requête** (Request Body), généralement au format JSON. Ce chapitre vous montrera comment définir des opérations `POST` et comment FastAPI, grâce à Pydantic, rend la réception et la validation de ces données incroyablement simples et sûres.

## Concept 1 : Le Décorateur `@app.post()` {#concept-1-le-decorateur-apppost-10}

### Quoi ? {#quoi-10}
Le décorateur `@app.post("/path")` est utilisé pour déclarer une fonction d'opération de chemin qui répond aux requêtes HTTP `POST`. C'est la méthode standard pour les endpoints destinés à la création de ressources.

### Pourquoi ? {#pourquoi-10}
La sémantique HTTP dicte que `POST` est utilisé pour créer une nouvelle ressource enfant sous une collection. Par exemple, une requête `POST` à `/users` devrait créer un nouvel utilisateur. Envoyer des données dans le corps est essentiel car :
-   Les données peuvent être complexes et structurées (JSON).
-   La taille des données n'est pas limitée comme elle peut l'être avec une URL.
-   C'est plus sécurisé : les données sensibles (comme un mot de passe lors de la création d'un utilisateur) n'apparaissent pas dans les URL, qui sont souvent enregistrées dans les logs du serveur ou l'historique du navigateur.

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-10}
On combine le décorateur `@app.post()` avec un paramètre de fonction typé par un modèle Pydantic, comme nous l'avons vu dans le chapitre précédent.

**Cas Réel : Ajout d'une nouvelle tâche à une liste de choses à faire (To-Do list)**

```python
from fastapi import FastAPI
from pydantic import BaseModel
from typing import Optional

class Todo(BaseModel):
    title: str
    description: Optional[str] = None
    completed: bool = False

app = FastAPI()

# On utilise @app.post() pour cet endpoint de création
@app.post("/todos")
async def create_todo(todo: Todo):
    # Dans une vraie application, vous inséreriez ici la tâche
    # dans une base de données et lui assigneriez un nouvel ID.
    print(f"Nouvelle tâche à créer : '{todo.title}'")
    
    # On retourne les données reçues pour confirmation.
    # Souvent, on retourne l'objet complet tel qu'il a été créé,
    # avec son nouvel ID.
    todo_in_db = todo.dict()
    todo_in_db.update({"id": 1}) # Simulation de l'ID de la BDD
    
    return todo_in_db
```
Lorsque FastAPI reçoit une requête `POST` sur `/todos`, il :
1.  Prend le corps de la requête (le JSON).
2.  Le valide et le convertit en une instance de la classe `Todo`.
3.  Appelle votre fonction `create_todo` en lui passant cette instance.

### Zone de Danger {#zone-de-danger-10}
Une erreur conceptuelle courante est d'utiliser `POST` pour des actions qui ne créent pas de ressource. Si votre action modifie une ressource existante dans sa totalité, `PUT` est plus approprié. Si elle ne fait que lire des données, c'est un `GET`. Respecter la sémantique HTTP rend votre API prévisible et plus facile à utiliser.

---

## Concept 2 : Combiner Paramètres de Chemin et Corps de Requête {#concept-2-combiner-parametres-de-chemin-et-corps-de-requete-10}

### Quoi ? {#quoi-11}
Il est très courant d'avoir besoin de créer une ressource qui est liée à une autre. Par exemple, poster un commentaire *sur un article de blog spécifique*. Dans ce cas, l'endpoint combine un paramètre de chemin (pour identifier le parent) et un corps de requête (pour les données de la nouvelle ressource).

### Pourquoi ? {#pourquoi-11}
Cette structure d'URL (`/parent/{parent_id}/child`) est au cœur des API RESTful. Elle exprime clairement la relation hiérarchique entre les ressources. Cela rend votre API logique et facile à comprendre.

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-11}
Vous déclarez simplement les deux types de paramètres dans la signature de votre fonction. FastAPI sait automatiquement que si un paramètre correspond à une variable dans le chemin, c'est un paramètre de chemin, et si c'est un modèle Pydantic, c'est le corps de la requête.

**Cas Réel : Poster un commentaire sur un article de blog**

```python
from fastapi import FastAPI, Path
from pydantic import BaseModel

class Comment(BaseModel):
    username: str
    text: str

app = FastAPI()

@app.post("/articles/{article_id}/comments")
async def post_comment(
    article_id: int = Path(..., gt=0),
    comment: Comment
):
    # 'article_id' vient de l'URL
    # 'comment' vient du corps de la requête JSON
    return {
        "message": f"Comment by {comment.username} posted on article {article_id}",
        "comment_text": comment.text
    }
```
Pour appeler cet endpoint, un client enverrait une requête `POST` à une URL comme `/articles/42/comments` avec un corps JSON :
```json
{
  "username": "Alice",
  "text": "Super article, merci !"
}
```

### Zone de Danger {#zone-de-danger-12}
Une fonction ne peut déclarer **qu'un seul paramètre pour représenter le corps de la requête**. Vous ne pouvez pas faire `def create(item: Item, user: User):`. Si vous avez besoin de recevoir plusieurs objets JSON, vous devez les encapsuler dans un modèle Pydantic plus grand.

Exemple :
```python
class RequestData(BaseModel):
    item: Item
    user: User

# Utilisation correcte
@app.post("/create")
async def create(data: RequestData):
    # Accéder via data.item et data.user
    ...
```

---

## Concept 3 : Mélange Complet : Path, Query, et Body {#concept-3-melange-complet--path-query-et-body-10}

### Quoi ? {#quoi-13}
Un endpoint peut être encore plus flexible en acceptant des paramètres des trois sources en même temps :
1.  **Path Parameters** : Pour identifier la ressource principale.
2.  **Request Body** : Pour la charge utile de données.
3.  **Query Parameters** : Pour des options ou des drapeaux qui modifient le comportement de l'opération.

### Pourquoi ? {#pourquoi-14}
Cela permet de créer des endpoints très puissants et configurables. L'URL identifie la *destination*, le corps contient *ce que* vous envoyez, et les paramètres de requête configurent *comment* l'envoi est traité.

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-13}
Il suffit d'ajouter tous les paramètres nécessaires à la signature de la fonction. FastAPI les distribuera correctement.

**Cas Réel : Ajouter un produit à un inventaire avec notification**

```python
from fastapi import FastAPI, Path
from pydantic import BaseModel

class Product(BaseModel):
    name: str
    quantity: int

app = FastAPI()

@app.post("/inventories/{inventory_id}/products")
async def add_product(
    inventory_id: str,
    product: Product,
    notify_supplier: bool = False # Query parameter
):
    message = (
        f"Added {product.quantity} of '{product.name}' "
        f"to inventory '{inventory_id}'."
    )
    if notify_supplier:
        message += " Supplier has been notified."
    
    return {"status": message}
```
Un appel à cet endpoint ressemblerait à une requête `POST` à l'URL `/inventories/warehouse-A/products?notify_supplier=true` avec le corps JSON :
```json
{
  "name": "Laptop Pro",
  "quantity": 50
}
```

### Zone de Danger {#zone-de-danger-15}
L'ordre des paramètres dans la définition de la fonction est important. Python exige que les arguments sans valeur par défaut (comme `inventory_id` et `product`) soient définis avant les arguments avec des valeurs par défaut (comme `notify_supplier`).

---

### 3 Questions Clés {#3-questions-cles-10}
1.  Quelle est la principale différence entre une requête `GET` et une requête `POST` en termes de transport de données ?
2.  Vous voulez créer un endpoint pour ajouter un tag à un utilisateur spécifique. L'URL est `/users/{user_id}/tags`. Le nom du tag est envoyé en JSON. Comment la signature de votre fonction (`def ...():`) doit-elle être structurée ?
3.  Est-il possible d'avoir un endpoint qui utilise `@app.post` mais qui ne prend aucun paramètre de corps de requête ? Si oui, donnez un exemple de cas d'utilisation.

### 3 Exercices Progressifs {#3-exercices-progressifs-10}

**Exercice 1 : Enregistrement d'un utilisateur**
Créez un endpoint `POST /register` qui accepte un corps de requête JSON. Le modèle `User` doit contenir :
-   `username` (str)
-   `email` (str)
-   `password` (str)
L'endpoint doit retourner un message de succès confirmant le nom d'utilisateur enregistré, mais **ne doit pas** renvoyer le mot de passe.

<details>
<summary>Découvrir la solution commentée</summary>

```python
from fastapi import FastAPI
from pydantic import BaseModel

class User(BaseModel):
    username: str
    email: str
    password: str

app = FastAPI()

@app.post("/register")
async def register_user(user: User):
    # Simuler l'enregistrement de l'utilisateur en base de données.
    print(f"Registering user: {user.username}")
    
    # Ne jamais retourner le mot de passe dans la réponse !
    return {
        "message": f"User '{user.username}' registered successfully.",
        "user_info": {
            "username": user.username,
            "email": user.email
        }
    }
```
</details>

**Exercice 2 : Poster un Score dans un Jeu**
Créez un endpoint `POST /games/{game_id}/scores` qui permet de soumettre un score pour un jeu.
-   `game_id` est un paramètre de chemin (str).
-   Le corps de la requête est un modèle `Score` avec :
    -   `player_name` (str)
    -   `points` (int)
L'endpoint doit retourner une phrase confirmant que le score du joueur a été enregistré pour le jeu spécifié.

<details>
<summary>Découvrir la solution commentée</summary>

```python
from fastapi import FastAPI
from pydantic import BaseModel

class Score(BaseModel):
    player_name: str
    points: int

app = FastAPI()

@app.post("/games/{game_id}/scores")
async def submit_score(game_id: str, score: Score):
    # Logique pour sauvegarder le score.
    
    return {
        "message": (
            f"Score of {score.points} for player '{score.player_name}' "
            f"has been submitted for game '{game_id}'."
        )
    }
```
</details>

**Exercice 3 : Commander un Produit**
Créez un endpoint `POST /products/{product_id}/order` qui combine tout.
-   `product_id` (path param, entier, > 0).
-   `priority_shipping` (query param, booléen, défaut `False`).
-   Le corps de la requête est un modèle `Order` avec :
    -   `quantity` (int, > 0).
    -   `shipping_address` (str).
L'endpoint doit retourner un résumé de la commande.

<details>
<summary>Découvrir la solution commentée</summary>

```python
from fastapi import FastAPI, Path
from pydantic import BaseModel, Field

class Order(BaseModel):
    # On peut utiliser Field de Pydantic pour ajouter de la validation
    # directement dans le modèle. C'est comme utiliser Path ou Query.
    quantity: int = Field(..., gt=0)
    shipping_address: str

app = FastAPI()

@app.post("/products/{product_id}/order")
async def order_product(
    product_id: int = Path(..., gt=0),
    order: Order,
    priority_shipping: bool = False
):
    
    shipping_method = "Priority" if priority_shipping else "Standard"
    
    return {
        "order_summary": {
            "product_id": product_id,
            "quantity": order.quantity,
            "shipping_address": order.shipping_address,
            "shipping_method": shipping_method
        },
        "status": "Order placed successfully"
    }
```
</details>