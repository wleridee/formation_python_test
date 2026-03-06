---
sidebar_label: "Router d'API : Organisation Modulaire avec APIRouter"
sidebar_position: 26
difficulty: "confirmé"
---

# Router d'API : Organisation Modulaire avec APIRouter {#router-dapi--organisation-modulaire-avec-apirouter-26}

Lorsque votre application FastAPI grandit, le fichier `main.py` peut rapidement devenir un monolithe de plusieurs centaines ou milliers de lignes, contenant des dizaines d'opérations de chemin. Cette approche n'est pas scalable : elle rend le code difficile à lire, à maintenir et à tester. La collaboration en équipe devient également un cauchemar.

La solution à ce problème est `APIRouter`. C'est l'outil fourni par FastAPI pour diviser votre application en plusieurs "mini-applications" modulaires, chacune responsable d'un domaine métier spécifique.

## Concept 1 : Le Rôle de `APIRouter` {#concept-1-le-role-de-apirouter-26}

### Quoi ? {#quoi-26}
`APIRouter` est une classe qui fonctionne de manière très similaire à la classe `FastAPI` elle-même. Vous pouvez y déclarer des opérations de chemin (`@router.get`, `@router.post`, etc.) comme vous le feriez avec `app`. La différence fondamentale est qu'un `APIRouter` est un module portable que vous pouvez ensuite "brancher" sur votre application principale `FastAPI`.

```mermaid
graph TD
    subgraph Fichier main.py
        A[app = FastAPI()]
    end
    
    subgraph Fichier routers/users.py
        B[users_router = APIRouter()]
        B1["@users_router.get('/me')"] --> B
        B2["@users_router.post('/')"] --> B
    end

    subgraph Fichier routers/items.py
        C[items_router = APIRouter()]
        C1["@items_router.get('/')"] --> C
    end
    
    B -->|app.include_router(users_router)| A
    C -->|app.include_router(items_router)| A

    A --> D{API Complète}
```

### Pourquoi ? {#pourquoi-26}
-   **Organisation du Code :** Séparez votre logique par domaine. Par exemple, toute la gestion des utilisateurs dans `routers/users.py`, les produits dans `routers/products.py`, etc.
-   **Maintenabilité :** Les fichiers plus petits sont plus faciles à comprendre et à modifier.
-   **Collaboration :** Plusieurs développeurs peuvent travailler sur différents fichiers de routeurs simultanément avec moins de risques de conflits.
-   **Réutilisabilité :** Un routeur bien conçu peut potentiellement être réutilisé dans d'autres projets.

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-26}
La mise en place se fait en deux étapes : créer le routeur, puis l'inclure.

**1. Créez un fichier pour le routeur (ex: `routers/users.py`)**

```python
# routers/users.py
from fastapi import APIRouter

# Créez une instance de APIRouter
router = APIRouter()

@router.get("/users/")
async def get_all_users():
    return [{"username": "jdoe"}, {"username": "asmith"}]

@router.get("/users/{user_id}")
async def get_user_by_id(user_id: int):
    return {"user_id": user_id, "username": f"user{user_id}"}
```

**2. Incluez le routeur dans votre application principale (`main.py`)**

```python
# main.py
from fastapi import FastAPI
from routers import users # Importez le module contenant votre routeur

app = FastAPI()

# Incluez le routeur dans l'application principale
app.include_router(users.router)

@app.get("/")
async def root():
    return {"message": "Welcome to the main application"}
```
Votre application a maintenant les endpoints `/`, `/users/`, et `/users/{user_id}`.

### Zone de Danger {#zone-de-danger-26}
L'erreur la plus commune est d'oublier l'étape `app.include_router(...)` dans `main.py`. Vos opérations de chemin existent dans le fichier du routeur, mais FastAPI ne les connaît pas et ne peut donc pas diriger les requêtes vers elles. Résultat : vous obtiendrez des erreurs 404 "Not Found".

---

## Concept 2 : Préfixes et Tags pour les Routeurs {#concept-2-prefixes-et-tags-pour-les-routeurs-26}

### Quoi ? {#quoi-27}
Lorsque vous incluez un routeur, vous pouvez fournir des paramètres supplémentaires à `app.include_router` pour simplifier votre code et améliorer votre documentation. Les deux plus importants sont :
-   `prefix`: Une chaîne de caractères qui sera préfixée à *tous* les chemins des opérations de ce routeur.
-   `tags`: Une liste de chaînes de caractères qui seront utilisées pour regrouper les opérations de ce routeur dans la documentation OpenAPI (Swagger UI).

### Pourquoi ? {#pourquoi-27}
-   **Code DRY (Don't Repeat Yourself) :** Au lieu d'écrire `@router.get("/users/{user_id}")` et `@router.post("/users/")`, vous pouvez définir `prefix="/users"` une seule fois et simplifier vos décorateurs en `@router.get("/{user_id}")` et `@router.post("/")`.
-   **Documentation Claire :** L'utilisation de `tags` est essentielle pour une documentation lisible. Sans cela, tous vos endpoints apparaissent en vrac. Avec les tags, ils sont regroupés par fonctionnalité (ex: "users", "items", "auth").

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-27}
Améliorons l'exemple précédent.

**1. Simplifiez le fichier `routers/users.py`**

```python
# routers/users.py
from fastapi import APIRouter

router = APIRouter()

# Notez que "/users" a disparu des chemins
@router.get("/")
async def get_all_users():
    return [{"username": "jdoe"}, {"username": "asmith"}]

@router.get("/{user_id}")
async def get_user_by_id(user_id: int):
    return {"user_id": user_id, "username": f"user{user_id}"}
```

**2. Ajoutez `prefix` et `tags` dans `main.py`**

```python
# main.py
from fastapi import FastAPI
from routers import users

app = FastAPI()

app.include_router(
    users.router,
    prefix="/users",  # Tous les chemins de ce routeur commenceront par /users
    tags=["users"]      # Ils seront groupés sous "users" dans la doc
)
```
Le résultat final est le même, mais le code est plus propre et la documentation est mieux organisée.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Documentation Swagger UI montrant les endpoints regroupés sous le tag "users".
> **Alt Text** : Interface Swagger avec une section "users" dépliée, affichant les endpoints GET /users/ et GET /users/{user_id}.

### Zone de Danger {#zone-de-danger-28}
Faites attention aux barres obliques (`/`). La meilleure pratique est de mettre un préfixe **sans** barre oblique à la fin (`prefix="/users"`) et de faire commencer les chemins de vos opérations **sans** barre oblique au début (`@router.get("/{user_id}")`). FastAPI est assez intelligent pour gérer les erreurs, mais cette convention évite les doubles barres obliques (`//`) dans vos chemins.

---

## Concept 3 : Dépendances au Niveau du Routeur {#concept-3-dependances-au-niveau-du-routeur-26}

### Quoi ? {#quoi-28}
Vous pouvez appliquer une liste de dépendances à un `APIRouter` entier lors de son instanciation. Ces dépendances seront exécutées pour chaque opération de chemin définie dans ce routeur, tout comme si vous les aviez ajoutées au décorateur de chaque fonction.

### Pourquoi ? {#pourquoi-28}
C'est extrêmement utile pour les "cross-cutting concerns" (préoccupations transversales) comme l'authentification. Si toutes les opérations sur les utilisateurs nécessitent qu'un utilisateur soit authentifié, vous pouvez appliquer la dépendance d'authentification une seule fois au niveau du routeur, au lieu de la répéter sur chaque fonction.

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-28}
On passe une liste de dépendances au constructeur `APIRouter`.

**Cas Réel : Protéger un routeur d'administration**

```python
# dependencies.py
from fastapi import Header, HTTPException, status

async def verify_admin_token(x_token: str = Header(...)):
    if x_token != "fake-super-secret-admin-token":
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN, 
            detail="Admin token missing or invalid"
        )
    return True

# routers/admin.py
from fastapi import APIRouter, Depends
from dependencies import verify_admin_token

# Applique la dépendance à toutes les routes de ce routeur
router = APIRouter(
    prefix="/admin",
    tags=["admin"],
    dependencies=[Depends(verify_admin_token)]
)

@router.get("/dashboard")
async def get_admin_dashboard():
    return {"message": "Welcome to the admin dashboard!"}

@router.post("/report")
async def create_admin_report():
    return {"message": "Report created successfully."}
```
Maintenant, toute requête vers `/admin/dashboard` ou `/admin/report` qui n'inclut pas l'en-tête `X-Token: fake-super-secret-admin-token` recevra une erreur 403.

### Zone de Danger {#zone-de-danger-29}
Une dépendance au niveau du routeur est "invisible" dans la signature des fonctions d'opérations de chemin. Cela peut rendre le code un peu plus difficile à comprendre pour quelqu'un qui lit uniquement la fonction `get_admin_dashboard`, car il n'est pas évident qu'une vérification de sécurité a lieu. Utilisez cette fonctionnalité judicieusement, principalement pour des exigences larges et bien documentées comme l'authentification.

---

### 3 Questions Clés {#3-questions-cles-26}
1.  Quelle est la fonction principale utilisée dans `main.py` pour intégrer les opérations de chemin définies dans un `APIRouter` ?
2.  Comment pouvez-vous vous assurer que tous les chemins d'un routeur commencent par `/api/v1` et sont regroupés sous le tag "API V1" dans la documentation, sans modifier chaque décorateur d'opération de chemin ?
3.  Vous avez un ensemble d'endpoints qui doivent tous vérifier la présence d'une clé d'API valide. Quelle est la manière la plus efficace d'appliquer cette vérification à l'ensemble du groupe d'endpoints en utilisant `APIRouter` ?

### 3 Exercices Progressifs {#3-exercices-progressifs-26}

**Exercice 1 : Refactoring de Base**
Prenez le code suivant dans un seul fichier `main.py` et refactorez-le en utilisant un `APIRouter` dans un fichier `routers/products.py`.

```python
# Code initial dans main.py
from fastapi import FastAPI
app = FastAPI()
@app.get("/products/")
def get_products(): return [{"name": "Laptop"}, {"name": "Mouse"}]
@app.get("/products/{product_id}")
def get_product(product_id: int): return {"id": product_id}
```

<details>
<summary>Découvrir la solution commentée</summary>

**Fichier `routers/products.py`**
```python
from fastapi import APIRouter

router = APIRouter()

@router.get("/products/")
def get_products():
    return [{"name": "Laptop"}, {"name": "Mouse"}]

@router.get("/products/{product_id}")
def get_product(product_id: int):
    return {"id": product_id}
```

**Fichier `main.py`**
```python
from fastapi import FastAPI
from routers import products

app = FastAPI()

app.include_router(products.router)
```
</details>

**Exercice 2 : Utilisation de Préfixes et Tags**
Reprenez la solution de l'exercice 1 et améliorez-la en utilisant les paramètres `prefix` et `tags` pour que les chemins dans `routers/products.py` soient simplifiés (`/` et `/{product_id}`) et que la documentation soit regroupée sous le tag "products".

<details>
<summary>Découvrir la solution commentée</summary>

**Fichier `routers/products.py` (modifié)**
```python
from fastapi import APIRouter

router = APIRouter()

# Les préfixes "/products" ont été retirés
@router.get("/")
def get_products():
    return [{"name": "Laptop"}, {"name": "Mouse"}]

@router.get("/{product_id}")
def get_product(product_id: int):
    return {"id": product_id}
```

**Fichier `main.py` (modifié)**
```python
from fastapi import FastAPI
from routers import products

app = FastAPI()

app.include_router(
    products.router,
    prefix="/products",
    tags=["products"]
)
```
</details>

**Exercice 3 : Routeur Protégé pour les Factures**
Créez une mini-application pour la gestion de factures.
1.  Créez un routeur dans `routers/invoices.py`.
2.  Définissez une dépendance `verify_api_key` qui vérifie la présence d'un en-tête `X-API-Key` avec la valeur `topsecretkey`. Si la clé est invalide, levez une `HTTPException` 401.
3.  Appliquez cette dépendance à l'ensemble du routeur `invoices`.
4.  Le routeur doit avoir le préfixe `/invoices` et le tag "invoices".
5.  Créez un endpoint `GET /` dans ce routeur qui retourne une liste de factures factices.

<details>
<summary>Découvrir la solution commentée</summary>

**Fichier `dependencies.py` (créé pour la propreté)**
```python
from fastapi import Header, HTTPException, status

async def verify_api_key(x_api_key: str = Header(...)):
    if x_api_key != "topsecretkey":
        raise HTTPException(status_code=status.HTTP_401_UNAUTHORIZED, detail="Invalid API Key")
```

**Fichier `routers/invoices.py`**
```python
from fastapi import APIRouter, Depends
from dependencies import verify_api_key

router = APIRouter(
    dependencies=[Depends(verify_api_key)]
)

@router.get("/")
async def get_invoices():
    return [
        {"id": "inv_123", "amount": 100},
        {"id": "inv_456", "amount": 250},
    ]
```

**Fichier `main.py`**
```python
from fastapi import FastAPI
from routers import invoices

app = FastAPI()

app.include_router(
    invoices.router,
    prefix="/invoices",
    tags=["invoices"]
)
```
*Pour tester, une requête à `GET /invoices` sans l'en-tête `X-API-Key: topsecretkey` doit échouer avec une erreur 401.*
</details>