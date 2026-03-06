---
sidebar_label: "Query Parameters : Définition et Valeurs par Défaut"
sidebar_position: 7
difficulty: "junior"
---

# Query Parameters : Définition et Valeurs par Défaut {#query-parameters-definition-et-valeurs-par-defaut-7}

Nous avons appris à identifier une ressource unique avec les paramètres de chemin (par exemple, `/users/123`). Mais que faire si nous voulons filtrer une collection de ressources, la trier ou la paginer ? Par exemple, comment demander "tous les utilisateurs actifs, triés par nom" ? C'est le rôle des **paramètres de requête** (Query Parameters).

Les paramètres de requête sont des paires clé-valeur qui apparaissent dans l'URL après un point d'interrogation `?`. Ils sont optionnels et constituent le principal moyen de personnaliser une requête `GET`.

`http://example.com/items?skip=0&limit=10`
Ici, `skip` et `limit` sont des paramètres de requête.

## Concept 1 : Définir des Paramètres de Requête {#concept-1-definir-des-parametres-de-requete-7}

### Quoi ? {#quoi-7}
Un paramètre de requête est une variable déclarée dans la signature de votre fonction d'opération qui **ne fait pas partie** du paramètre de chemin. FastAPI est assez intelligent pour faire la distinction tout seul.

### Pourquoi ? {#pourquoi-7}
Ils sont utilisés pour passer des données optionnelles qui modifient la requête. Contrairement aux paramètres de chemin qui *identifient* une ressource, les paramètres de requête *filtrent*, *trient*, ou *paginent* une collection de ressources. C'est le standard pour personnaliser la sortie d'une liste.

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-7}
La déclaration est très simple : ajoutez un argument à votre fonction. Si le nom de l'argument ne correspond à aucun paramètre dans le chemin du décorateur, FastAPI le traitera comme un paramètre de requête.

**Cas Réel : Pagination d'une liste d'items**
Imaginons que nous avons des milliers d'items et que nous ne voulons en retourner qu'une petite partie à la fois (une "page").

```python
from fastapi import FastAPI

app = FastAPI()

# Une fausse base de données d'items
fake_items_db = [{"item_name": f"Item {i}"} for i in range(100)]


@app.get("/items/")
def read_items(skip: int = 0, limit: int = 10):
    return fake_items_db[skip : skip + limit]
```
Dans cet exemple :
1.  `skip` et `limit` ne sont pas dans le chemin `/items/`.
2.  FastAPI les identifie donc comme des paramètres de requête.
3.  Grâce aux `type hints` (`int`), il va valider et convertir les valeurs.

Vous pouvez maintenant appeler votre API avec différentes URL :
-   `http://127.0.0.1:8000/items/` : Utilise les valeurs par défaut (`skip=0`, `limit=10`).
-   `http://127.0.0.1:8000/items/?skip=20` : Saute les 20 premiers items.
-   `http://127.0.0.1:8000/items/?skip=0&limit=5` : Retourne les 5 premiers items.
-   `http://127.0.0.1:8000/items/?limit=5&skip=0` : L'ordre n'a pas d'importance.

### Zone de Danger {#zone-de-danger-7}
Comme pour les paramètres de chemin, si un utilisateur passe une valeur qui ne peut pas être convertie au type attendu (ex: `/items/?skip=vingt`), FastAPI retournera une erreur de validation 422 claire et automatique. Vous n'avez pas besoin de gérer ce cas.

---

## Concept 2 : Paramètres Optionnels vs. Paramètres avec Défaut {#concept-2-parametres-optionnels-vs-parametres-avec-defaut-7}

### Quoi ? {#quoi-8}
Les paramètres de requête peuvent être soit obligatoires, soit optionnels, soit avoir une valeur par défaut.
-   **Avec valeur par défaut** (`limit: int = 10`): Le paramètre est optionnel. S'il n'est pas fourni, sa valeur sera `10`.
-   **Purement optionnel** (`q: str | None = None`): Le paramètre est optionnel. S'il n'est pas fourni, sa valeur sera `None`.
-   **Obligatoire** (`q: str`): Le paramètre doit être fourni. Si ce n'est pas le cas, FastAPI renverra une erreur.

### Pourquoi ? {#pourquoi-8}
Rendre les paramètres optionnels rend votre API plus flexible et plus simple à utiliser. Forcer un paramètre obligatoire est utile quand il est essentiel à la logique de la fonction (par exemple, un terme de recherche).

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-8}
On utilise les fonctionnalités standards de Python : les valeurs par défaut et le module `typing` (ou `|` en Python 3.10+).

**Cas Réel : Endpoint de recherche avec un terme optionnel**

```python
from fastapi import FastAPI
from typing import Optional # Pour les anciennes versions de Python

app = FastAPI()

@app.get("/items/search")
# En Python 3.10+ on peut écrire q: str | None = None
def search_items(q: Optional[str] = None):
    if q:
        return {"message": f"Searching for items containing: {q}"}
    return {"message": "No search query provided. Returning all items."}
```
Ici, `q` est purement optionnel.
-   `http://127.0.0.1:8000/items/search` : `q` sera `None`.
-   `http://127.0.0.1:8000/items/search?q=laptop` : `q` sera la chaîne `"laptop"`.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Documentation Swagger UI montrant le paramètre de requête `q`.
> **Alt Text** : Interface Swagger UI pour le endpoint /items/search, montrant un champ de saisie pour le paramètre "q" qui est marqué comme "string or null" et non requis.

### Zone de Danger {#zone-de-danger-9}
Un paramètre de requête déclaré sans valeur par défaut (ex: `def search(q: str):`) devient **obligatoire**. Si l'utilisateur appelle l'endpoint sans fournir ce paramètre, il recevra une erreur 422 lui indiquant que le paramètre est manquant. Cela peut être le comportement souhaité, mais assurez-vous que c'est bien intentionnel.

---

## Concept 3 : Conversion des Paramètres Booléens {#concept-3-conversion-des-parametres-booleens-7}

### Quoi ? {#quoi-10}
Un cas d'usage très courant pour les paramètres de requête est d'activer ou désactiver une fonctionnalité via un "flag" booléen.

### Pourquoi ? {#pourquoi-10}
Cela permet de modifier le comportement d'un endpoint de manière simple et lisible, par exemple pour demander une version détaillée ou concise de la réponse (`?full_profile=true`).

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-10}
Vous déclarez simplement le paramètre avec le `type hint` `bool`. FastAPI est assez intelligent pour interpréter différentes valeurs "truthy".

**Cas Réel : Afficher les détails d'un produit**
```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/products/{product_id}")
def get_product(product_id: int, show_details: bool = False):
    details = {
        "id": product_id,
        "name": f"Product {product_id}"
    }
    if show_details:
        details.update({
            "description": "This is a detailed description.",
            "price": 99.99
        })
    return details
```
FastAPI convertira les valeurs suivantes en `True`:
-   `?show_details=true`
-   `?show_details=True`
-   `?show_details=1`
-   `?show_details=yes`
-   `?show_details=on`

Et les valeurs "falsy" (`false`, `0`, `no`, `off`) en `False`.

### Zone de Danger {#zone-de-danger-11}
Attention aux URL mal formées. Si un utilisateur appelle `.../?show_details=`, le paramètre sera une chaîne vide `""`, ce qui n'est ni `True` ni `False`. FastAPI lèvera une erreur de validation. Si le paramètre n'est pas du tout présent, il prendra sa valeur par défaut (`False` dans notre exemple).

---

### 3 Questions Clés {#3-questions-cles-7}
1.  Comment FastAPI fait-il la différence entre un paramètre de chemin et un paramètre de requête dans la déclaration d'une fonction ?
2.  Quelle est la syntaxe pour déclarer un paramètre de requête nommé `filter` qui est une chaîne de caractères, mais qui est complètement optionnel (sa valeur doit être `None` s'il n'est pas fourni) ?
3.  Un utilisateur appelle votre endpoint `/users?active=true`. Comment devez-vous déclarer le paramètre `active` dans votre fonction pour qu'il soit reçu comme une valeur booléenne `True` ?

### 3 Exercices Progressifs {#3-exercices-progressifs-7}

**Exercice 1 : Salutations Personnalisées**
Créez un endpoint `/greeting` qui accepte un paramètre de requête optionnel `name`.
-   Si `name` est fourni (ex: `/greeting?name=Alice`), il doit retourner `{"message": "Hello, Alice"}`.
-   Si `name` n'est pas fourni (`/greeting`), il doit retourner `{"message": "Hello, World"}`.

<details>
<summary>Découvrir la solution commentée</summary>

```python
from fastapi import FastAPI
from typing import Optional

app = FastAPI()

@app.get("/greeting")
def get_greeting(name: Optional[str] = None):
    # On utilise une valeur par défaut de "World" si 'name' est None.
    greet_name = name if name else "World"
    return {"message": f"Hello, {greet_name}"}

# Autre approche avec une valeur par défaut directe
@app.get("/greeting2")
def get_greeting_v2(name: str = "World"):
    return {"message": f"Hello, {name}"}
```
*La deuxième approche est plus concise pour ce cas précis. La première est plus flexible si vous avez une logique différente quand le paramètre est absent (`None`) vs quand il est une chaîne vide (`""`).*
</details>

**Exercice 2 : Filtrage de Blog Posts**
Créez un endpoint `/posts` pour un blog. Il doit accepter deux paramètres de requête optionnels :
1.  `author_id` (un entier).
2.  `published` (un booléen, qui doit être `True` par défaut).

La fonction doit simplement retourner un dictionnaire qui reflète les filtres reçus.

<details>
<summary>Découvrir la solution commentée</summary>

```python
from fastapi import FastAPI
from typing import Optional

app = FastAPI()

@app.get("/posts")
def get_posts(author_id: Optional[int] = None, published: bool = True):
    # Cette fonction simule la récupération des filtres.
    # Dans une vraie application, vous les utiliseriez pour une requête en base de données.
    return {
        "filters": {
            "author_id": author_id,
            "published": published
        }
    }
```
*Testez-le avec différentes combinaisons :*
-   `/posts`
-   `/posts?author_id=5`
-   `/posts?published=false`
-   `/posts?author_id=10&published=false`
</details>

**Exercice 3 : API de Recherche Avancée**
Créez un endpoint de recherche `/advanced-search` qui combine plusieurs types de paramètres.
-   `q` : Un terme de recherche, qui doit être **obligatoire**.
-   `category` : Une chaîne de caractères, optionnelle.
-   `min_price` : Un `float`, optionnel, valeur par défaut `0.0`.
-   `max_price` : Un `float`, optionnel (peut être `None`).
-   `in_stock` : Un booléen, valeur par défaut `True`.

La fonction doit retourner un dictionnaire résumant tous les critères de recherche.

<details>
<summary>Découvrir la solution commentée</summary>

```python
from fastapi import FastAPI
from typing import Optional

app = FastAPI()

@app.get("/advanced-search")
def advanced_search(
    q: str,  # Pas de valeur par défaut, donc c'est obligatoire.
    category: Optional[str] = None,
    min_price: float = 0.0,
    max_price: Optional[float] = None,
    in_stock: bool = True
):
    search_criteria = {
        "query": q,
        "category": category,
        "price_range": {
            "min": min_price,
            "max": max_price
        },
        "in_stock": in_stock
    }
    return search_criteria
```
*Testez un appel valide : `/advanced-search?q=smartwatch&category=electronics&max_price=299.99`*
*Testez un appel invalide (sans le paramètre 'q') : `/advanced-search`. Vous devriez obtenir une erreur 422.*
</details>