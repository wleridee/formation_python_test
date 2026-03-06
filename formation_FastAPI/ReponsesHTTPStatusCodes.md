---
sidebar_label: "Réponses HTTP : Codes de Statut"
sidebar_position: 13
difficulty: "junior"
---

# Réponses HTTP : Codes de Statut {#reponses-http-codes-de-statut-13}

Lorsque votre API répond à une requête, elle ne se contente pas d'envoyer des données (le corps de la réponse). Elle envoie aussi des métadonnées dans les en-têtes, dont la plus importante est le **code de statut HTTP**. Ce code est un nombre à trois chiffres qui informe instantanément le client du résultat de son opération : succès, échec, besoin d'authentification, etc.

Maîtriser les codes de statut est essentiel pour construire des API claires, prévisibles et conformes aux standards du web. FastAPI vous donne tous les outils pour les manipuler avec précision.

```mermaid
graph TD
    subgraph Client
        A[Navigateur / Application]
    end
    subgraph "FastAPI Server"
        B{Endpoint /items/{id}}
    end
    subgraph "Logique Métier"
        C{Item trouvé ?}
    end

    A -- "GET /items/123" --> B
    B -- "Cherche l'item 123" --> C
    C -- "Oui" --> B
    B -- "200 OK" & " { ... } " --> A
    
    A -- "GET /items/999" --> B
    B -- "Cherche l'item 999" --> C
    C -- "Non" --> B
    B -- "404 Not Found" & " { detail: '...' } " --> A
```

## Concept 1 : Définir le Code de Statut de Succès {#concept-1-definir-le-code-de-statut-de-succes-13}

### Quoi ? {#quoi-13}
Par défaut, FastAPI retourne un code de statut `200 OK` pour toutes les opérations réussies. Cependant, ce n'est pas toujours le code le plus sémantiquement correct. Par exemple, lors de la création d'une ressource (`POST`), le code `201 Created` est plus approprié. Vous pouvez définir ce code par défaut en utilisant le paramètre `status_code` dans le décorateur de l'opération de chemin.

### Pourquoi ? {#pourquoi-13}
Utiliser le code de statut correct rend votre API plus expressive et conforme aux standards REST. Un client qui reçoit un `201 Created` sait non seulement que sa requête a réussi, mais aussi qu'une nouvelle ressource a été créée. Pour une opération `DELETE` réussie, un `204 No Content` est souvent préférable à un `200 OK` si aucune donnée n'est retournée.

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-13}
Il suffit d'ajouter `status_code=<CODE>` à votre décorateur. Pour la lisibilité, il est recommandé d'utiliser le module `status` de FastAPI qui contient des constantes pour tous les codes HTTP.

**Cas Réel : Créer un produit et retourner `201 Created`**

```python
from fastapi import FastAPI, status
from pydantic import BaseModel

class Product(BaseModel):
    name: str

app = FastAPI()

# On spécifie que le code de succès pour cet endpoint sera 201
@app.post("/products", status_code=status.HTTP_201_CREATED)
async def create_product(product: Product):
    # Logique pour sauvegarder le produit...
    new_product_id = 123
    return {"id": new_product_id, **product.dict()}
```
Désormais, toute requête `POST` réussie à `/products` recevra une réponse avec le code de statut `201 Created`.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Documentation Swagger UI pour le endpoint POST /products.
> **Alt Text** : Section "Responses" de l'interface Swagger UI, montrant clairement "201" comme code de réponse en cas de succès, avec l'exemple de corps de réponse.

### Zone de Danger {#zone-de-danger-13}
Le paramètre `status_code` ne définit que le code de **succès**. Il n'a aucun effet sur les codes d'erreur. Si votre logique rencontre un problème, vous devrez utiliser un autre mécanisme pour renvoyer un code d'erreur (voir le concept suivant).

---

## Concept 2 : Gérer les Erreurs avec `HTTPException` {#concept-2-gerer-les-erreurs-avec-http-exception-13}

### Quoi ? {#quoi-14}
`HTTPException` est une classe d'exception spéciale fournie par FastAPI. Lorsque vous la "levez" (`raise`) dans votre code, FastAPI intercepte cette exception et génère une réponse HTTP d'erreur avec le code de statut et le message que vous avez spécifiés.

### Pourquoi ? {#pourquoi-14}
C'est la manière propre et recommandée de gérer les erreurs dans FastAPI.
-   **Clarté du code :** Votre logique métier n'est pas polluée par la construction manuelle de réponses d'erreur.
-   **Centralisation :** Vous pouvez gérer toutes les conditions d'erreur (ex: ressource non trouvée, permission refusée) de la même manière.
-   **Intégration :** FastAPI documente automatiquement ces erreurs possibles dans votre documentation OpenAPI.

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-14}
1.  Importez `HTTPException` de `fastapi`.
2.  Dans votre logique, si une condition d'erreur est remplie, levez l'exception avec `raise HTTPException(...)`.
3.  Les deux arguments principaux sont `status_code` et `detail` (le message d'erreur).

**Cas Réel : Récupérer un item qui n'existe pas**

```python
from fastapi import FastAPI, HTTPException, status

app = FastAPI()

fake_items_db = {"a": "Item A", "b": "Item B"}

@app.get("/items/{item_id}")
async def get_item(item_id: str):
    if item_id not in fake_items_db:
        # L'item n'existe pas, on arrête tout et on renvoie une 404.
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"Item with id '{item_id}' not found"
        )
    
    return {"item": fake_items_db[item_id]}
```
Si vous appelez `/items/c`, FastAPI retournera une réponse JSON avec un code `404 Not Found` :
```json
{
  "detail": "Item with id 'c' not found"
}
```

### Zone de Danger {#zone-de-danger-15}
Ne mettez jamais d'informations sensibles ou de détails d'implémentation dans le message `detail` de l'exception. Ce message est visible par le client. Par exemple, au lieu de "Erreur de base de données : colonne 'user_email' n'existe pas", préférez un message générique comme "Une erreur interne est survenue."

---

## Concept 3 : Les Codes de Statut les Plus Courants {#concept-3-les-codes-de-statut-les-plus-courants-13}

### Quoi ? {#quoi-16}
Voici un guide rapide des codes que vous utiliserez 99% du temps dans une API REST.

-   **`2xx` - Succès**
    -   `200 OK` : Succès générique pour une requête `GET` ou `PUT`.
    -   `201 Created` : La ressource a été créée avec succès (`POST`).
    -   `204 No Content` : La requête a réussi mais il n'y a rien à retourner (`DELETE`).

-   **`4xx` - Erreurs Client**
    -   `400 Bad Request` : La requête du client est mal formée (syntaxe incorrecte, paramètre manquant).
    -   `401 Unauthorized` : Le client doit s'authentifier pour obtenir la ressource demandée.
    -   `403 Forbidden` : Le client est authentifié, mais n'a pas les droits nécessaires pour accéder à cette ressource.
    -   `404 Not Found` : La ressource spécifiée dans l'URL n'existe pas.
    -   `422 Unprocessable Entity` : La syntaxe de la requête est correcte, mais les données sont sémantiquement invalides (ex: un email mal formé). C'est le code que FastAPI utilise pour les erreurs de validation Pydantic.

### Pourquoi ? {#pourquoi-16}
Connaître et utiliser ces codes correctement est une marque de professionnalisme. Cela permet aux développeurs utilisant votre API de comprendre immédiatement le résultat de leurs appels sans même avoir à lire le corps de la réponse.

---

### 3 Questions Clés {#3-questions-cles-13}
1.  Comment pouvez-vous faire en sorte qu'un endpoint `POST` retourne par défaut un code de statut `201 Created` en cas de succès ?
2.  Quelle est la différence fondamentale de signification entre un code d'erreur `401 Unauthorized` et un `403 Forbidden` ?
3.  Quelle classe FastAPI utilisez-vous pour interrompre le flux d'une requête et renvoyer immédiatement une réponse d'erreur, comme une `404 Not Found` ?

### 3 Exercices Progressifs {#3-exercices-progressifs-13}

**Exercice 1 : Statut de Création**
Créez un endpoint `POST /messages`. Il doit accepter un corps JSON simple comme `{"text": "Hello World"}`. En cas de succès, il doit retourner une réponse avec un code de statut `201 Created` et un corps contenant `{"status": "Message created"}`.

<details>
<summary>Découvrir la solution commentée</summary>

```python
from fastapi import FastAPI, status
from pydantic import BaseModel

class Message(BaseModel):
    text: str

app = FastAPI()

@app.post("/messages", status_code=status.HTTP_201_CREATED)
async def create_message(message: Message):
    print(f"Received message: {message.text}")
    return {"status": "Message created"}
```
*L'élément clé est l'ajout de `status_code=status.HTTP_201_CREATED` dans le décorateur `@app.post`.*
</details>

**Exercice 2 : Gestion de l'Absence d'une Ressource**
Créez un endpoint `GET /users/{user_id}`. Simulez une base de données avec un dictionnaire.
-   Si l'`user_id` existe dans le dictionnaire, retournez les données de l'utilisateur avec un `200 OK`.
-   Si l'`user_id` n'existe pas, retournez une erreur `404 Not Found` avec le message `"User not found"`.

<details>
<summary>Découvrir la solution commentée</summary>

```python
from fastapi import FastAPI, HTTPException, status

app = FastAPI()

fake_users_db = {
    1: {"username": "zack", "email": "zack@example.com"},
    2: {"username": "ana", "email": "ana@example.com"},
}

@app.get("/users/{user_id}")
async def get_user(user_id: int):
    user = fake_users_db.get(user_id)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="User not found"
        )
    return user
```
*L'utilisation de `dict.get()` est plus sûre que `dict[key]` car elle retourne `None` si la clé n'existe pas, ce qui permet un test `if not user:` propre.*
</details>

**Exercice 3 : Validation de Paramètre et Statut Personnalisé**
Créez un endpoint `DELETE /tasks/{task_id}`.
-   Il doit accepter un paramètre de requête `?force=...` qui doit être un booléen.
-   Si `task_id` n'existe pas, retournez une erreur `404`.
-   Si la tâche existe mais que `force` n'est pas à `True` (c'est-à-dire `False` ou non fourni), retournez une erreur `403 Forbidden` avec le message `"Deletion not allowed without force flag"`.
-   Si la tâche existe et que `force=true`, supprimez la tâche et retournez une réponse `204 No Content`.

<details>
<summary>Découvrir la solution commentée</summary>

```python
from fastapi import FastAPI, HTTPException, status, Response

app = FastAPI()

fake_tasks_db = {1: "Faire le ménage", 2: "Sortir le chien"}

@app.delete("/tasks/{task_id}")
async def delete_task(task_id: int, force: bool = False):
    if task_id not in fake_tasks_db:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="Task not found"
        )
    
    if not force:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="Deletion not allowed without force flag"
        )
        
    del fake_tasks_db[task_id]
    
    # Pour un 204, on retourne un objet Response vide.
    return Response(status_code=status.HTTP_204_NO_CONTENT)
```
*Cet exercice combine la gestion de plusieurs conditions d'erreur avec des codes de statut différents, et montre comment retourner explicitement une réponse `204`.*
</details>