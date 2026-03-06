---
sidebar_label: "Configuration Avancée des Opérations : Tags, Summary, Description"
sidebar_position: 20
difficulty: "confirmé"
---

# Configuration Avancée des Opérations : Tags, Summary, Description {#configuration-avancee-des-operations--tags-summary-description-20}

La documentation automatique de FastAPI est l'une de ses fonctionnalités phares. Cependant, pour qu'elle soit réellement utile, il faut la nourrir avec des métadonnées riches et précises. Par défaut, FastAPI utilise le nom de la fonction et les docstrings, mais vous pouvez aller bien plus loin en utilisant les paramètres du décorateur d'opération.

Ce chapitre explore comment transformer une documentation fonctionnelle en une documentation professionnelle et facile à naviguer pour les consommateurs de votre API.

## Concept 1 : Organiser avec les `tags` {#concept-1-organiser-avec-les-tags-20}

### Quoi ? {#quoi-20}
Le paramètre `tags` est une liste de chaînes de caractères (`list[str]`) que vous pouvez passer au décorateur d'opération (`@app.get`, `@app.post`, etc.). Chaque chaîne représente une "étiquette" ou une catégorie. Les opérations de chemin partageant le même tag seront regroupées sous une section commune dans l'interface de la documentation.

### Pourquoi ? {#pourquoi-20}
Dans une API de taille conséquente, avoir des dizaines d'endpoints dans une seule liste plate devient rapidement ingérable. Les tags permettent de créer une structure logique et sémantique, rendant l'API beaucoup plus facile à explorer. Les regroupements courants sont par ressource (`Users`, `Items`, `Invoices`) ou par fonctionnalité (`Auth`, `Admin`, `Reports`).

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-20}
Il suffit de fournir une liste au paramètre `tags`.

**Cas Réel : Structurer une API de e-commerce**

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/users/", tags=["Users"])
async def get_users():
    return [{"username": "john.doe"}]

@app.post("/users/", tags=["Users"])
async def create_user():
    return {"status": "User created"}

@app.get("/products/", tags=["Products"])
async def get_products():
    return [{"name": "Laptop"}]

@app.post("/products/", tags=["Products"])
async def create_product():
    return {"status": "Product created"}
```
Dans l'interface Swagger UI, vous verrez maintenant deux sections distinctes et repliables : "Users" et "Products", chacune contenant ses propres endpoints.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Documentation Swagger UI montrant les groupes "Users" et "Products".
> **Alt Text** : Interface Swagger UI avec deux sections bien définies : "Users" contenant les endpoints GET et POST pour les utilisateurs, et "Products" pour les produits.

### Zone de Danger {#zone-de-danger-20}
Faites attention aux fautes de frappe et à la casse. `tags=["User"]` et `tags=["Users"]` créeront deux groupes distincts. Il est judicieux de définir des constantes pour vos tags dans un projet de grande envergure afin de garantir la cohérence.

---

## Concept 2 : `summary` et `description` pour la Clarté {#concept-2-summary-et-description-pour-la-clarte-20}

### Quoi ? {#quoi-21}
-   `summary` : Une courte phrase (un titre) décrivant l'opération. Elle remplace le nom de la fonction dans la vue principale de la documentation.
-   `description` : Une explication plus détaillée de ce que fait l'opération. Elle supporte le format Markdown, vous permettant d'inclure des listes, des blocs de code, des liens, etc.

FastAPI lit la `description` en priorité depuis le paramètre du décorateur. Si ce dernier n'est pas fourni, il se rabat sur le docstring de la fonction.

### Pourquoi ? {#pourquoi-21}
-   Un bon `summary` rend l'intention de l'endpoint immédiatement évidente, sans être contraint par les conventions de nommage de Python (ex: `get_all_active_users_with_pagination` peut devenir "Get Active Users").
-   Une `description` détaillée est votre meilleure opportunité d'expliquer la logique métier, les cas particuliers, les permissions requises ou tout autre détail crucial pour utiliser l'endpoint correctement.

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-21}
On ajoute les paramètres directement au décorateur.

**Cas Réel : Documenter un endpoint de création complexe**

```python
from fastapi import FastAPI

app = FastAPI()

@app.post(
    "/reports/",
    tags=["Reports"],
    summary="Create a new financial report",
    description="""
Generates a new financial report based on the provided parameters.

### Permissions:
* Requires **admin** or **reporter** role.
* The system will perform an audit log of this action.

### Notes:
- The generation process is asynchronous.
- The endpoint returns a task ID to check the status later.
    """
)
async def create_report():
    return {"task_id": "some-uuid-1234"}
```

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Endpoint `/reports/` déplié dans Swagger UI.
> **Alt Text** : La documentation de l'endpoint POST /reports/ montrant le résumé "Create a new financial report" et une description détaillée formatée en Markdown avec des titres, des listes à puces et du texte en gras.

### Zone de Danger {#zone-de-danger-22}
Ne mettez pas d'informations sensibles (détails d'implémentation, noms de serveurs internes) dans la `description`, car elle est publiquement visible. La documentation doit être une interface pour le consommateur, pas un commentaire de code interne.

---

## Concept 3 : `response_description` et `deprecated` {#concept-3-response_description-et-deprecated-20}

### Quoi ? {#quoi-23}
-   `response_description` : Une chaîne de caractères décrivant ce que signifie une réponse réussie (typiquement, un code de statut 2xx). La valeur par défaut est "Successful Response".
-   `deprecated` : Un booléen (`True`/`False`). Si mis à `True`, l'opération est marquée comme obsolète dans la documentation, généralement avec un style barré.

### Pourquoi ? {#pourquoi-23}
-   Personnaliser la `response_description` ajoute un contexte métier important. Au lieu de "Successful Response", vous pouvez avoir "User created successfully" pour un `POST` 201 ou "List of active items" pour un `GET` 200.
-   Marquer un endpoint comme `deprecated` est la manière standard et non-intrusive d'indiquer aux développeurs qu'ils doivent migrer vers une nouvelle version de l'API. Cela fait partie d'un bon cycle de vie d'API, permettant une évolution en douceur sans casser les intégrations existantes du jour au lendemain.

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-23}
Encore une fois, ce sont de simples paramètres du décorateur.

**Cas Réel : Déprécier un ancien endpoint d'utilisateur**

```python
from fastapi import FastAPI

app = FastAPI()

@app.get(
    "/user/{user_id}",
    tags=["Users", "Legacy"],
    summary="Get user by ID (DEPRECATED)",
    deprecated=True,
    description="This endpoint is deprecated. Please use `/users/{user_id}` instead."
)
async def get_old_user(user_id: int):
    return {"id": user_id, "username": "old_user"}

@app.get(
    "/users/{user_id}",
    tags=["Users"],
    summary="Get a specific user",
    response_description="The user object was found and is returned."
)
async def get_user(user_id: int):
    return {"id": user_id, "username": "new_user"}
```
Dans Swagger, l'endpoint `/user/{user_id}` sera affiché barré et potentiellement replié par défaut, signalant clairement son statut.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Vue de l'endpoint `/user/{user_id}` déprécié dans Swagger.
> **Alt Text** : L'interface Swagger UI montrant l'opération GET /user/{user_id} avec son nom barré, indiquant qu'elle est obsolète.

### Zone de Danger {#zone-de-danger-24}
Lorsque vous marquez un endpoint comme `deprecated`, il est crucial de fournir une alternative dans la `description`. Simplement le déprécier sans guider les utilisateurs vers le nouvel endpoint crée de la confusion et de la frustration.

---

### 3 Questions Clés {#3-questions-cles-20}
1.  Quel paramètre du décorateur permet de regrouper les endpoints `/users/`, `/users/{id}` et `/users/{id}/orders` sous la même section "Users" dans la documentation ?
2.  Quelle est la différence entre les paramètres `summary` et `description` ? Lequel supporte le format Markdown ?
3.  Comment signalez-vous aux consommateurs de votre API qu'un endpoint est obsolète et sera supprimé dans une future version ?

### 3 Exercices Progressifs {#3-exercices-progressifs-20}

**Exercice 1 : Documenter une API de Blog**
Créez trois endpoints :
-   `GET /posts`
-   `POST /posts`
-   `GET /posts/{post_id}`
Ajoutez un `summary` clair à chacun, et regroupez-les tous sous le tag "Blog Posts".

<details>
<summary>Découvrir la solution commentée</summary>

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/posts", tags=["Blog Posts"], summary="List all blog posts")
async def get_all_posts():
    return [{"id": 1, "title": "First Post"}]

@app.post("/posts", tags=["Blog Posts"], summary="Create a new blog post")
async def create_post():
    return {"status": "created"}

@app.get("/posts/{post_id}", tags=["Blog Posts"], summary="Get a single post by ID")
async def get_single_post(post_id: int):
    return {"id": post_id, "title": "First Post"}
```
</details>

**Exercice 2 : Description Détaillée et Réponse**
Améliorez l'endpoint `POST /posts` de l'exercice précédent.
-   Ajoutez une `description` détaillée en utilisant le docstring de la fonction, expliquant que le titre du post doit être unique et que le contenu supporte le Markdown.
-   Ajoutez une `response_description` pour le code 200 qui soit plus explicite que "Successful Response", par exemple : "The blog post was created successfully."

<details>
<summary>Découvrir la solution commentée</summary>

```python
from fastapi import FastAPI

app = FastAPI()

@app.post(
    "/posts", 
    tags=["Blog Posts"], 
    summary="Create a new blog post",
    response_description="The blog post was created successfully."
)
async def create_post():
    """
    Creates a new blog post.

    - The **title** of the post must be unique.
    - The `content` field supports GitHub-flavored Markdown.
    """
    return {"status": "created"}
```
*Note : Nous utilisons le docstring ici pour la description. FastAPI le détectera automatiquement car le paramètre `description` n'est pas fourni au décorateur.*
</details>

**Exercice 3 : Cycle de Vie d'une API**
Vous avez une API de gestion de tâches. L'endpoint `POST /add-task` est obsolète.
-   Créez cet endpoint et marquez-le comme `deprecated=True`.
-   Dans sa `description`, indiquez que les utilisateurs doivent maintenant utiliser `POST /tasks`.
-   Créez le nouvel endpoint `POST /tasks` avec une documentation complète (`tags`, `summary`, `response_description`).

<details>
<summary>Découvrir la solution commentée</summary>

```python
from fastapi import FastAPI

app = FastAPI()

@app.post(
    "/add-task",
    tags=["Tasks (Legacy)"],
    summary="Add a new task (DEPRECATED)",
    deprecated=True,
    description="This endpoint is obsolete. Please use the new `POST /tasks` endpoint."
)
async def add_task_legacy():
    return {"warning": "This endpoint is deprecated"}

@app.post(
    "/tasks",
    tags=["Tasks"],
    summary="Create a new task",
    response_description="Task created and returned with its new ID."
)
async def create_task():
    return {"id": 123, "title": "My new task"}
```
</details>