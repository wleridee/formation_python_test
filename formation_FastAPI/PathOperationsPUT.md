---
sidebar_label: "Path Operations : Requêtes PUT pour la Mise à Jour"
sidebar_position: 11
difficulty: "junior"
---

# Path Operations : Requêtes PUT pour la Mise à Jour {#path-operations-requetes-put-pour-la-mise-a-jour-11}

Nous savons maintenant comment créer des ressources avec `POST`. L'étape logique suivante est de les modifier. En HTTP, la méthode sémantiquement correcte pour la **mise à jour complète** d'une ressource existante est `PUT`.

Une requête `PUT` est envoyée à une URL qui identifie la ressource à modifier. Le corps de la requête contient la **nouvelle représentation complète** de cette ressource. Si la ressource existe, elle est remplacée. Si elle n'existe pas, une API peut choisir de la créer.

Ce chapitre se concentre sur le cas d'usage principal de `PUT` : le remplacement total d'une ressource existante.

## Concept 1 : Le Décorateur `@app.put()` et sa Sémantique {#concept-1-le-decorateur-appput-et-sa-semantique-11}

### Quoi ? {#quoi-11}
Le décorateur `@app.put("/items/{item_id}")` déclare une opération de chemin qui répond aux requêtes HTTP `PUT`. Il est conçu pour les endpoints qui mettent à jour une ressource spécifique, identifiée par un paramètre dans l'URL.

### Pourquoi ? {#pourquoi-11}
La distinction entre `POST` et `PUT` est fondamentale en REST :
-   `POST /items` : "Crée un nouvel item dans la collection `items`."
-   `PUT /items/123` : "Remplace entièrement l'item avec l'ID `123` par les nouvelles données que je fournis."

Une caractéristique clé de `PUT` est l'**idempotence**. Cela signifie que si vous envoyez la même requête `PUT` plusieurs fois, le résultat sur le serveur sera le même après la première requête. Mettre à jour un objet avec les mêmes données 10 fois de suite ne change rien après la première fois. Ceci est différent de `POST`, où chaque requête créerait une nouvelle ressource.

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-11}
La syntaxe est très similaire à celle de `POST`, mais elle inclut presque toujours un paramètre de chemin pour identifier la ressource à remplacer.

**Cas Réel : Mettre à jour les informations d'un utilisateur**
Nous allons créer un endpoint pour mettre à jour le profil d'un utilisateur.

```python
from fastapi import FastAPI
from pydantic import BaseModel

class UserProfile(BaseModel):
    full_name: str
    email: str
    is_active: bool

app = FastAPI()

# Une "fausse" base de données pour l'exemple
fake_users_db = {
    1: {"full_name": "John Doe", "email": "john.doe@example.com", "is_active": True}
}

@app.put("/users/{user_id}")
async def update_user(user_id: int, user_profile: UserProfile):
    if user_id not in fake_users_db:
        # Idéalement, on retournerait une erreur 404 (Not Found)
        return {"error": "User not found"}
    
    # Remplacement des données existantes par les nouvelles
    fake_users_db[user_id] = user_profile.dict()
    
    # On retourne les nouvelles données
    return {"user_id": user_id, **user_profile.dict()}
```

### Zone de Danger {#zone-de-danger-11}
`PUT` est destiné au **remplacement complet**. Si votre modèle `UserProfile` a un champ `is_active` et que le client ne l'envoie pas dans sa requête `PUT`, Pydantic lèvera une erreur de validation car le champ est manquant. Le client doit envoyer la représentation complète de la ressource. Pour les mises à jour partielles (ne changer que l'email, par exemple), la méthode `PATCH` est plus appropriée, mais elle est plus complexe à mettre en œuvre et sera vue dans des chapitres avancés.

---

## Concept 2 : Valider les Données d'Entrée et Gérer les Réponses {#concept-2-valider-les-donnees-dentree-et-gerer-les-reponses-11}

### Quoi ? {#quoi-12}
Comme pour `POST`, FastAPI et Pydantic valident automatiquement le corps de la requête. Votre logique métier doit ensuite gérer ce qui se passe après la validation : la ressource existe-t-elle ? Si oui, mettez-la à jour. Si non, que faire ? La réponse de l'API doit clairement indiquer le résultat de l'opération.

### Pourquoi ? {#pourquoi-12}
Une gestion correcte des cas de figure (succès, échec) est cruciale pour une API robuste. Si un client essaie de mettre à jour une ressource qui n'existe pas, il doit recevoir une erreur claire (comme un code statut HTTP 404) au lieu d'une erreur inattendue du serveur (code 500). Retourner la ressource mise à jour confirme au client l'état final des données.

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-12}
Nous allons améliorer l'exemple précédent en utilisant `HTTPException` pour retourner une erreur 404 propre si l'utilisateur n'existe pas.

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    price: float
    description: str | None = None

app = FastAPI()

fake_items_db = {
    1: {"name": "Clavier", "price": 49.99, "description": "Un clavier mécanique."},
    2: {"name": "Souris", "price": 25.00, "description": "Une souris optique."},
}

@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
    if item_id not in fake_items_db:
        # Si l'ID n'est pas trouvé, on lève une exception HTTP 404
        raise HTTPException(status_code=404, detail="Item not found")
    
    # On met à jour l'item dans notre "DB"
    updated_item_data = item.dict()
    fake_items_db[item_id] = updated_item_data
    
    # On retourne l'ID et les nouvelles données
    return {"id": item_id, **updated_item_data}
```
Avec `HTTPException`, FastAPI arrête l'exécution de la fonction et envoie immédiatement une réponse HTTP formatée avec le code de statut et le message que vous avez spécifiés.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Documentation Swagger UI pour le endpoint `PUT /items/{item_id}`.
> **Alt Text** : Interface Swagger UI montrant les champs pour le paramètre de chemin `item_id` et le corps de la requête attendu (le schéma du modèle `Item`).

### Zone de Danger {#zone-de-danger-13}
Attention à la confusion entre l'ID dans l'URL et un éventuel ID dans le corps de la requête.
```json
// PUT /items/123
{
  "id": 456, // Incohérence !
  "name": "Nouveau Nom",
  ...
}
```
En général, l'ID provenant de l'URL (`/items/123`) est la source de vérité. Vous devriez ignorer tout ID présent dans le corps de la requête pour éviter toute ambiguïté.

---

### 3 Questions Clés {#3-questions-cles-11}
1.  Quelle est la principale différence de sémantique entre les méthodes HTTP `PUT` et `POST` ?
2.  Quel concept décrit le fait qu'exécuter une requête `PUT` plusieurs fois avec les mêmes données produit le même résultat sur le serveur ?
3.  Si un client tente de faire un `PUT` sur une ressource qui n'existe pas (ex: `PUT /items/999`), quel code de statut HTTP est le plus approprié pour la réponse ?

### 3 Exercices Progressifs {#3-exercices-progressifs-11}

**Exercice 1 : Mise à jour d'une Tâche**
Créez une API de "To-Do list". Vous aurez une fausse base de données (dictionnaire) pré-remplie. Créez un endpoint `PUT /todos/{todo_id}`.
-   Le modèle `Todo` doit contenir `title` (str) and `completed` (bool).
-   L'endpoint doit mettre à jour la tâche correspondant à `todo_id`.
-   S'il n'y a pas de tâche avec cet ID, retournez une `HTTPException` 404.
-   En cas de succès, retournez la tâche mise à jour.

<details>
<summary>Découvrir la solution commentée</summary>

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel

class Todo(BaseModel):
    title: str
    completed: bool

app = FastAPI()

fake_todos_db = {
    1: {"title": "Apprendre FastAPI", "completed": True},
    2: {"title": "Faire les courses", "completed": False},
}

@app.put("/todos/{todo_id}")
async def update_todo(todo_id: int, todo: Todo):
    if todo_id not in fake_todos_db:
        raise HTTPException(status_code=404, detail="Todo not found")
    
    fake_todos_db[todo_id] = todo.dict()
    return {"id": todo_id, **fake_todos_db[todo_id]}
```
*Testez en mettant à jour la tâche 2 (`PUT /todos/2`) et en essayant de mettre à jour une tâche inexistante (`PUT /todos/99`).*
</details>

**Exercice 2 : Remplacer une Configuration**
Créez un endpoint `PUT /config/{config_name}` pour mettre à jour une configuration.
-   Le `config_name` est un `str` dans le chemin.
-   Le corps de la requête est un modèle `Config` qui contient :
    -   `value` (str)
    -   `is_enabled` (bool)
-   La particularité de `PUT` ici est que si la configuration n'existe pas, elle doit être créée.
-   L'endpoint doit retourner un message de statut ("Config updated" ou "Config created").

<details>
<summary>Découvrir la solution commentée</summary>

```python
from fastapi import FastAPI, Response, status
from pydantic import BaseModel

class Config(BaseModel):
    value: str
    is_enabled: bool

app = FastAPI()

fake_config_db = {
    "site_name": {"value": "Mon API", "is_enabled": True}
}

@app.put("/config/{config_name}")
async def upsert_config(config_name: str, config: Config, response: Response):
    message = ""
    if config_name in fake_config_db:
        response.status_code = status.HTTP_200_OK # 200 OK pour une mise à jour
        message = "Config updated"
    else:
        response.status_code = status.HTTP_201_CREATED # 201 Created pour une création
        message = "Config created"
        
    fake_config_db[config_name] = config.dict()
    
    return {"message": message, "config_name": config_name, **config.dict()}
```
*Cet exemple montre comment `PUT` peut aussi être utilisé pour "Upsert" (Update or Insert). Le changement de code de statut (`200 OK` vs `201 Created`) est une bonne pratique pour informer le client du type d'opération effectuée.*
</details>

**Exercice 3 : Mettre à Jour un Inventaire avec un Query Parameter**
Créez un endpoint `PUT /inventory/{item_code}`.
-   `item_code` est une chaîne de caractères (ex: "A-123-Z").
-   Le corps de la requête est un modèle `InventoryItem` avec `name` (str) et `quantity` (int).
-   Ajoutez un paramètre de requête optionnel `?update_source=...` (une chaîne de caractères).
-   L'endpoint doit mettre à jour l'item. S'il n'existe pas, levez une erreur 404.
-   La réponse doit inclure l'item mis à jour ainsi que la source de la mise à jour si elle a été fournie.

<details>
<summary>Découvrir la solution commentée</summary>

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from typing import Optional

class InventoryItem(BaseModel):
    name: str
    quantity: int

app = FastAPI()

fake_inventory_db = {
    "A-123-Z": {"name": "Écran 24 pouces", "quantity": 15},
    "B-456-Y": {"name": "Dock USB-C", "quantity": 30},
}

@app.put("/inventory/{item_code}")
async def update_inventory_item(
    item_code: str,
    item: InventoryItem,
    update_source: Optional[str] = None
):
    if item_code not in fake_inventory_db:
        raise HTTPException(status_code=404, detail=f"Item with code '{item_code}' not found")

    fake_inventory_db[item_code] = item.dict()
    
    response_data = {
        "item_code": item_code,
        "updated_item": item.dict()
    }
    
    if update_source:
        response_data["update_source"] = update_source
        
    return response_data
```
*Testez avec `PUT /inventory/A-123-Z?update_source=manual_sync` pour voir la réponse complète.*
</details>