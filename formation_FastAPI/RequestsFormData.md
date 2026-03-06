---
sidebar_label: "Requêtes avec Form Data"
sidebar_position: 22
difficulty: "confirmé"
---

# Requêtes avec Form Data {#requetes-avec-form-data-22}

Bien que les API modernes privilégient largement le format JSON pour les corps de requête, il existe des scénarios cruciaux où les données sont envoyées sous forme de "form data" (`application/x-www-form-urlencoded`). C'est le format utilisé par les navigateurs pour soumettre des formulaires HTML traditionnels, et il est également un standard pour certaines parties de protocoles comme OAuth2.

FastAPI fournit un moyen simple et cohérent de déclarer et de valider ces données en utilisant la fonction `Form`.

## Prérequis : Installation {#prerequis--installation-22}

Pour que FastAPI puisse parser les corps de requête de type "form data", il a besoin d'une bibliothèque supplémentaire. Vous devez l'installer :

```bash
pip install python-multipart
```

Sans cette bibliothèque, toute tentative de déclarer un champ de formulaire lèvera une erreur au démarrage de votre application.

## Concept 1 : Déclarer des Champs de Formulaire avec `Form` {#concept-1-declarer-des-champs-de-formulaire-avec-form-22}

### Quoi ? {#quoi-22}
`Form` est une fonction qui s'utilise de la même manière que `Body`, `Query` ou `Path`. Vous l'utilisez comme valeur par défaut pour un paramètre de votre fonction d'opération de chemin pour déclarer qu'il doit être extrait d'un corps de requête de type "form data".

### Pourquoi ? {#pourquoi-22}
-   **Compatibilité OAuth2 :** Le flux "Resource Owner Password Credentials" d'OAuth2, souvent utilisé pour les connexions natives, exige que `username`, `password` et `grant_type` soient envoyés en tant que "form data".
-   **Intégration avec des Systèmes Existants :** Permet de créer des endpoints qui peuvent être appelés directement depuis des formulaires HTML simples, sans nécessiter de JavaScript pour construire un payload JSON.
-   **Clarté et Validation :** En utilisant `Form`, vous bénéficiez de toute la puissance de FastAPI/Pydantic pour la validation, la conversion de type et la documentation automatique, tout comme pour les autres types de paramètres.

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-22}
On importe `Form` de `fastapi` et on l'utilise pour annoter les paramètres de la fonction.

**Cas Réel : Un endpoint de connexion simple**

```python
from fastapi import FastAPI, Form

app = FastAPI()

@app.post("/login/")
async def login(username: str = Form(...), password: str = Form(...)):
    # Dans une vraie application, vous vérifieriez ici le username et le password
    return {"username": username, "token_type": "bearer"}
```
Dans la documentation Swagger UI, FastAPI ne montrera pas un champ de saisie JSON. À la place, il affichera des champs de saisie individuels pour `username` et `password` sous la section "Request body", avec le type de média `application/x-www-form-urlencoded`.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Documentation Swagger UI de l'endpoint `/login/`.
> **Alt Text** : Interface Swagger montrant l'endpoint POST /login/ avec un corps de requête de type "application/x-www-form-urlencoded" et des champs de saisie distincts pour "username" et "password".

### Zone de Danger {#zone-de-danger-22}
La règle la plus importante : **vous ne pouvez pas mélanger `Body` (ou un modèle Pydantic) et `Form` dans la signature d'un même endpoint.** Le corps d'une requête HTTP ne peut avoir qu'un seul type de média. Soit il est `application/json` (géré par `Body`), soit il est `application/x-www-form-urlencoded` (géré par `Form`). Tenter de déclarer les deux entraînera une erreur de FastAPI.

---

## Concept 2 : Combiner `Form` avec d'autres Paramètres {#concept-2-combiner-form-avec-dautres-parametres-22}

### Quoi ? {#quoi-23}
Alors que vous ne pouvez pas mélanger `Form` et `Body`, vous pouvez sans aucun problème combiner des champs de formulaire avec tous les autres types de paramètres qui ne proviennent pas du corps de la requête : `Path`, `Query`, `Header`, `Cookie`.

### Pourquoi ? {#pourquoi-23}
C'est un scénario très courant. Une action peut nécessiter une ressource spécifique identifiée dans le chemin (`Path`), des options supplémentaires dans les paramètres de requête (`Query`), et des données à soumettre dans le corps du formulaire (`Form`). FastAPI gère cette composition de manière transparente.

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-23}
Il suffit de déclarer tous les paramètres nécessaires dans la signature de la fonction, chacun avec son "marqueur" approprié (`Path`, `Query`, `Form`, etc.).

**Cas Réel : Poster un commentaire sur un article de blog**

```python
from fastapi import FastAPI, Path, Query, Form

app = FastAPI()

@app.post("/articles/{article_id}/comments")
async def post_comment(
    article_id: int = Path(..., title="The ID of the article to comment on", ge=1),
    author: str = Form(..., min_length=3, max_length=50),
    text: str = Form(..., min_length=10),
    notify_author: bool = Query(True, description="Send an email notification to the article author")
):
    return {
        "action": "Posted comment",
        "article_id": article_id,
        "comment_author": author,
        "comment_text": text,
        "notification_sent": notify_author
    }
```
Dans ce cas :
-   `article_id` vient du chemin.
-   `author` et `text` viennent du corps du formulaire.
-   `notify_author` est un paramètre de requête optionnel dans l'URL (ex: `?notify_author=false`).

### Zone de Danger {#zone-de-danger-24}
Lorsque vous utilisez `Form`, rappelez-vous que les données sont envoyées sous forme de paires clé-valeur plates. Vous ne pouvez pas envoyer de structures JSON imbriquées complexes comme vous le feriez avec `Body` et un modèle Pydantic. Si vous avez besoin d'envoyer à la fois des données structurées et des champs de formulaire (un cas d'usage typique est le téléversement de fichiers avec des métadonnées JSON), vous devrez utiliser une approche plus avancée, souvent en combinant `File` et des `Form` dont le contenu est une chaîne JSON à parser manuellement.

---

### 3 Questions Clés {#3-questions-cles-22}
1.  Quelle bibliothèque doit être installée pour permettre à FastAPI de gérer les requêtes "form data" ?
2.  Un même endpoint peut-il recevoir simultanément un champ `username: str = Form(...)` et un modèle `Item(BaseModel)` en tant que corps de requête ? Pourquoi ?
3.  Dans quel flux d'authentification standard est-il courant d'utiliser `Form` pour envoyer un nom d'utilisateur et un mot de passe ?

### 3 Exercices Progressifs {#3-exercices-progressifs-22}

**Exercice 1 : Formulaire de Newsletter**
Créez un endpoint `POST /subscribe` qui accepte un seul champ de formulaire : `email`. Validez que l'email est une chaîne de caractères et retournez un message de confirmation.

<details>
<summary>Découvrir la solution commentée</summary>

```python
from fastapi import FastAPI, Form
from pydantic import EmailStr

app = FastAPI()

@app.post("/subscribe")
async def subscribe_to_newsletter(email: EmailStr = Form(...)):
    # L'utilisation de 'EmailStr' de Pydantic fournit une validation d'email gratuite !
    return {"message": f"'{email}' has been subscribed to the newsletter."}
```
</details>

**Exercice 2 : Soumettre un Avis sur un Produit**
Créez un endpoint `POST /products/{product_id}/reviews`.
-   Il doit accepter `product_id` comme un entier provenant du chemin.
-   Il doit accepter une `rating` (note) comme un entier provenant du formulaire, qui doit être compris entre 1 et 5.
-   Il doit accepter un `comment` optionnel comme une chaîne de caractères provenant du formulaire.
-   Retournez un résumé de l'avis soumis.

<details>
<summary>Découvrir la solution commentée</summary>

```python
from typing import Optional
from fastapi import FastAPI, Path, Form

app = FastAPI()

@app.post("/products/{product_id}/reviews")
async def submit_review(
    product_id: int = Path(..., ge=1),
    rating: int = Form(..., ge=1, le=5),
    comment: Optional[str] = Form(None)
):
    review = {
        "product_id": product_id,
        "rating": rating
    }
    if comment:
        review["comment"] = comment
    
    return {"message": "Review submitted successfully", "review": review}
```
</details>

**Exercice 3 : Simulation d'un Endpoint de Token OAuth2**
Créez un endpoint `POST /token` qui simule l'obtention d'un jeton d'accès.
-   Il doit accepter un champ de formulaire `grant_type`. Si la valeur n'est pas `"password"`, levez une `HTTPException` 400.
-   Il doit accepter des champs `username` et `password` via le formulaire.
-   Si tout est correct, retournez un dictionnaire contenant un `access_token` factice et `token_type` à `"bearer"`.

<details>
<summary>Découvrir la solution commentée</summary>

```python
from fastapi import FastAPI, Form, HTTPException, status

app = FastAPI()

@app.post("/token")
async def get_access_token(
    grant_type: str = Form(...),
    username: str = Form(...),
    password: str = Form(...) # Dans une vraie app, on ne ferait rien de ce mdp
):
    if grant_type != "password":
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail="Unsupported grant type"
        )
    
    # Ici, vous valideriez le username/password
    print(f"Authenticating user '{username}'...")
    
    return {
        "access_token": "fake-jwt-token-for-" + username,
        "token_type": "bearer"
    }
```
*Cet exercice imite parfaitement la spécification OAuth2 pour le flux de mot de passe, un cas d'utilisation très courant pour les données de formulaire dans les API.*
</details>