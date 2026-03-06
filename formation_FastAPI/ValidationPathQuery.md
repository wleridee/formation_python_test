---
sidebar_label: "Validation de Données : Path et Query Parameters"
sidebar_position: 8
difficulty: "junior"
---

# Validation de Données : Path et Query Parameters {#validation-de-donnees-path-et-query-parameters-8}

Jusqu'à présent, nous avons utilisé les `type hints` pour valider le *type* de nos paramètres. C'est déjà une protection formidable. Mais que faire si nous avons besoin de contraintes plus fines ?
- Un `item_id` doit-il être un entier positif, et non pas -5 ?
- Un terme de recherche `q` doit-il avoir une longueur minimale pour être pertinent ?
- Une page `limit` ne devrait-elle pas dépasser 100 pour protéger notre base de données ?

C'est là qu'intervient la validation de données avancée. FastAPI, en s'appuyant sur Pydantic, nous offre des outils déclaratifs et puissants pour définir ces règles : les fonctions `Path` et `Query`.

## Concept 1 : Validation Avancée des `Query Parameters` avec `Query` {#concept-1-validation-avancee-des-query-parameters-avec-query-8}

### Quoi ? {#quoi-8}
`Query` est une fonction que vous importez de `fastapi` et que vous utilisez comme valeur par défaut d'un paramètre de requête. Elle vous permet d'ajouter des contraintes de validation supplémentaires directement dans la signature de votre fonction.

### Pourquoi ? {#pourquoi-8}
Cela permet de garder votre code de validation propre, déclaratif et séparé de votre logique métier. Au lieu d'écrire des `if` dans votre fonction pour vérifier la longueur d'une chaîne, vous le déclarez une seule fois. Si la validation échoue, FastAPI retourne automatiquement une erreur 422 descriptive, avant même que votre code ne soit exécuté.

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-8}
On importe `Query` et on l'assigne comme valeur par défaut au paramètre souhaité.

**Cas Réel : Endpoint de recherche avec contraintes de longueur**
Imaginons un endpoint `/search` où le terme de recherche `q` doit être fourni et doit faire entre 3 et 50 caractères.

```python
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/search")
async def do_search(q: str = Query(..., min_length=3, max_length=50)):
    # Le code ici ne sera exécuté que si 'q' passe la validation.
    return {"query_received": q}
```
Décortiquons `q: str = Query(..., min_length=3, max_length=50)` :
-   `q: str`: Le paramètre est toujours typé comme une chaîne.
-   `= Query(...)`: On assigne le résultat de la fonction `Query` comme valeur par défaut.
-   `...`: C'est une syntaxe spéciale de Python (Ellipsis) qui indique à FastAPI que ce paramètre est **obligatoire**.
-   `min_length=3`, `max_length=50`: Ce sont les règles de validation. D'autres existent, comme `regex` pour valider contre une expression régulière.

Si vous appelez `/search?q=ab`, vous recevrez une erreur 422 indiquant que la chaîne est trop courte.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Documentation Swagger UI pour le endpoint `/search`.
> **Alt Text** : Interface Swagger UI montrant le paramètre 'q' avec les contraintes "minLength: 3" et "maxLength: 50" clairement indiquées dans la description.

### Zone de Danger {#zone-de-danger-8}
L'erreur la plus commune est d'oublier que `Query` est utilisé *à la place* de la valeur par défaut.
-   Pour un paramètre obligatoire : `q: str = Query(..., min_length=3)`
-   Pour un paramètre optionnel avec une valeur par défaut : `q: str = Query("default_value", min_length=3)`
-   Pour un paramètre optionnel qui peut être `None` : `q: str | None = Query(None, min_length=3)`

---

## Concept 2 : Validation Avancée des `Path Parameters` avec `Path` {#concept-2-validation-avancee-des-path-parameters-avec-path-8}

### Quoi ? {#quoi-9}
`Path` est l'équivalent de `Query`, mais spécifiquement conçu pour les paramètres de chemin. Il permet d'ajouter des contraintes numériques comme "supérieur à", "inférieur ou égal à", etc.

### Pourquoi ? {#pourquoi-9}
Les identifiants dans les URL (comme les ID d'utilisateurs ou de produits) sont souvent des entiers qui doivent respecter certaines règles métier (par exemple, être toujours positifs). `Path` permet de garantir ces règles au niveau le plus bas de l'application.

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-9}
La syntaxe est très similaire à celle de `Query`.

**Cas Réel : Récupérer un item par son ID**
L'ID de l'item doit être un entier strictement supérieur à 0 et inférieur ou égal à 1000.

```python
from fastapi import FastAPI, Path

app = FastAPI()

@app.get("/items/{item_id}")
async def read_item(
    item_id: int = Path(..., title="The ID of the item to get", gt=0, le=1000)
):
    return {"item_id": item_id}
```
Ici :
-   `item_id: int = Path(...)`: On utilise `Path`.
-   `...`: Un paramètre de chemin est **toujours** obligatoire, donc on utilise `...`. On ne peut pas lui donner de valeur par défaut.
-   `title="..."`: Un paramètre supplémentaire pour améliorer la documentation.
-   `gt=0`: "Greater Than 0". L'ID doit être supérieur à 0.
-   `le=1000`: "Less Than or Equal to 1000". L'ID doit être inférieur ou égal à 1000.

Les opérateurs numériques disponibles sont : `gt` (greater than), `ge` (greater than or equal), `lt` (less than), `le` (less than or equal).

### Zone de Danger {#zone-de-danger-10}
Il est impossible de définir une valeur par défaut pour un paramètre de chemin. Il fait partie intégrante de l'URL. Par conséquent, le premier argument de `Path` doit toujours être `...` pour signifier qu'il est obligatoire. Tenter de mettre une valeur par défaut comme `Path(1, ...)` lèvera une erreur.

---

## Concept 3 : Combiner `Path`, `Query` et les Types Python {#concept-3-combiner-path-query-et-les-types-python-8}

### Quoi ? {#quoi-11}
La véritable puissance de ce système réside dans sa capacité à être combiné. Un seul endpoint peut avoir plusieurs paramètres de chemin et de requête, chacun avec ses propres règles de validation complexes, tout en gardant une signature de fonction claire et lisible.

### Pourquoi ? {#pourquoi-11}
Les endpoints du monde réel sont rarement simples. Une route typique peut nécessiter un ID d'utilisateur validé, un terme de recherche optionnel mais contraint, et des paramètres de pagination numériques. FastAPI gère cette complexité avec élégance.

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-11}
Il suffit de déclarer chaque paramètre avec la validation appropriée.

**Cas Réel : Lister les articles d'un utilisateur avec recherche**
-   On veut les articles de l'utilisateur `user_id`, qui doit être un entier positif.
-   On veut pouvoir filtrer par un terme `q`, qui est optionnel.
-   On veut paginer avec `page` et `page_size`, qui sont des entiers positifs avec des valeurs par défaut.

```python
from fastapi import FastAPI, Path, Query
from typing import Optional

app = FastAPI()

@app.get("/users/{user_id}/items")
async def list_user_items(
    user_id: int = Path(..., title="User ID", ge=1),
    q: Optional[str] = Query(None, min_length=2, max_length=50),
    page: int = Query(1, ge=1),
    page_size: int = Query(10, ge=1, le=50)
):
    results = {"user_id": user_id, "page": page, "page_size": page_size}
    if q:
        results.update({"query": q})
    return results
```
Cette signature de fonction est un véritable contrat pour votre API :
-   `user_id`: Path param, entier, >= 1.
-   `q`: Query param, string ou None, si fourni doit faire entre 2 et 50 caractères.
-   `page`: Query param, entier, optionnel (défaut 1), >= 1.
-   `page_size`: Query param, entier, optionnel (défaut 10), entre 1 et 50.

### Zone de Danger {#zone-de-danger-12}
L'ordre des paramètres dans la fonction Python est important. Les paramètres sans valeur par défaut (comme `user_id` et potentiellement `q` s'il était obligatoire) doivent être placés avant les paramètres avec des valeurs par défaut (`page`, `page_size`). Python l'exige.

---

### 3 Questions Clés {#3-questions-cles-8}
1.  Quelle fonction importez-vous de `fastapi` pour valider qu'un paramètre de requête de type chaîne de caractères a une longueur maximale de 100 ?
2.  Comment spécifiez-vous qu'un paramètre de chemin, `version_id`, doit être un entier supérieur ou égal à 1 (`>= 1`) ?
3.  À quoi sert la valeur spéciale `...` (Ellipsis) lorsqu'elle est utilisée comme premier argument de `Query` ou `Path` ?

### 3 Exercices Progressifs {#3-exercices-progressifs-8}

**Exercice 1 : Validation de Pseudo**
Créez un endpoint `/users/check-username` qui accepte un paramètre de requête `username`. Ce paramètre doit être **obligatoire** et respecter les règles suivantes :
-   Longueur minimale de 3 caractères.
-   Longueur maximale de 20 caractères.
-   Doit correspondre à une expression régulière qui n'autorise que les caractères alphanumériques (`^[a-zA-Z0-9]+$`).

<details>
<summary>Découvrir la solution commentée</summary>

```python
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/users/check-username")
async def check_username(
    username: str = Query(
        ...,  # Obligatoire
        min_length=3,
        max_length=20,
        regex="^[a-zA-Z0-9]+$"  # Expression régulière pour alphanumérique
    )
):
    return {"username": username, "status": "valid"}
```
*Testez avec des valeurs valides (`alex123`), trop courtes (`al`), trop longues, ou avec des symboles (`alex-123`) pour voir les erreurs de validation.*
</details>

**Exercice 2 : API de Calendrier**
Créez un endpoint `/events/{year}/{month}`.
-   `year` (path param) doit être un entier d'une valeur minimale de 2020.
-   `month` (path param) doit être un entier compris entre 1 et 12 (inclus).
-   Ajoutez un paramètre de requête optionnel `is_public` (booléen) avec une valeur par défaut à `True`.

<details>
<summary>Découvrir la solution commentée</summary>

```python
from fastapi import FastAPI, Path, Query

app = FastAPI()

@app.get("/events/{year}/{month}")
async def get_events(
    year: int = Path(..., title="Year", ge=2020),
    month: int = Path(..., title="Month", ge=1, le=12),
    is_public: bool = Query(True)
):
    return {
        "date": f"{month:02d}-{year}",
        "filters": {"is_public": is_public}
    }
```
*Testez avec une date valide comme `/events/2024/10` et une date invalide comme `/events/2019/13`.*
</details>

**Exercice 3 : API de notation de produits**
Créez un endpoint `GET` `/products/{product_id}/reviews`.
-   `product_id` (path param) doit être un entier positif (`> 0`).
-   `rating` (query param) doit être un entier **obligatoire** compris entre 1 et 5 (inclus).
-   `limit` (query param) est optionnel, avec une valeur par défaut de 10, et doit être un entier positif.

<details>
<summary>Découvrir la solution commentée</summary>

```python
from fastapi import FastAPI, Path, Query

app = FastAPI()

@app.get("/products/{product_id}/reviews")
async def get_product_reviews(
    product_id: int = Path(..., title="Product ID", gt=0),
    rating: int = Query(..., title="Filter by rating", ge=1, le=5),
    limit: int = Query(10, title="Number of reviews to return", gt=0)
):
    return {
        "product_id": product_id,
        "filters": {
            "rating": rating,
            "limit": limit
        }
    }
```
*Un appel valide : `/products/123/reviews?rating=5`.*
*Un appel invalide (rating manquant) : `/products/123/reviews`.*
*Un appel invalide (rating hors limites) : `/products/123/reviews?rating=6`.*
</details>