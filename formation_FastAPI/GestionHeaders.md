---
sidebar_label: "Gestion des Headers"
sidebar_position: 25
difficulty: "confirmé"
---

# Gestion des Headers {#gestion-des-headers-25}

Les en-têtes (headers) HTTP sont des composants fondamentaux de chaque requête et réponse. Ils transportent des métadonnées essentielles comme les informations d'authentification (`Authorization`), le type de contenu accepté par le client (`Accept`), ou des informations sur le client lui-même (`User-Agent`).

Maîtriser la lecture des en-têtes de requête et la définition d'en-têtes de réponse est crucial pour construire des API robustes, sécurisées et performantes. FastAPI, fidèle à sa philosophie, rend ces opérations déclaratives et simples.

## Concept 1 : Lire les En-têtes de Requête avec `Header` {#concept-1-lire-les-en-tetes-de-requete-avec-header-25}

### Quoi ? {#quoi-25}
`Header` est une fonction de dépendance, comme `Cookie` ou `Query`. Elle permet de déclarer qu'un paramètre de votre fonction doit être extrait d'un en-tête de la requête HTTP. FastAPI s'occupe de la recherche (insensible à la casse) et de la validation.

Une particularité importante : les en-têtes HTTP utilisent des tirets (`-`), ce qui n'est pas valide pour les noms de variables en Python. FastAPI gère cela automatiquement : il convertit les noms de paramètres avec des underscores (`_`) en leur équivalent avec des tirets pour la recherche dans les en-têtes. Par exemple, `user_agent` correspondra à l'en-tête `user-agent`.

### Pourquoi ? {#pourquoi-25}
-   **Authentification :** Le plus souvent, pour extraire les jetons d'authentification de l'en-tête `Authorization` (ex: `Bearer <token>`).
-   **Négociation de contenu :** Pour comprendre ce que le client attend en retour, via les en-têtes `Accept` (ex: `application/json`) ou `Accept-Language` (ex: `fr-FR`).
-   **Informations client :** Pour obtenir des informations sur le client qui fait l'appel, comme le `User-Agent`, afin d'adapter la réponse ou pour l'analytique.
-   **En-têtes personnalisés :** Pour passer des identifiants uniques (`X-Request-ID`) ou des clés d'API (`X-API-Key`).

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-25}
On importe `Header` de `fastapi` et on l'utilise pour déclarer un paramètre.

**Cas Réel : Lire le `User-Agent` d'une requête**

```python
from typing import Optional
from fastapi import FastAPI, Header

app = FastAPI()

@app.get("/whoami")
async def who_am_i(user_agent: Optional[str] = Header(None)):
    # FastAPI a automatiquement cherché l'en-tête "user-agent"
    # et l'a injecté dans la variable user_agent.
    return {"user_agent": user_agent}
```

### Zone de Danger {#zone-de-danger-25}
-   **Noms d'en-têtes dupliqués :** Si un client envoie plusieurs en-têtes avec le même nom (ce qui est valide selon la spécification HTTP), FastAPI vous fournira les valeurs dans une liste. Pour gérer ce cas, vous devez typer votre paramètre comme `List[str]`. Exemple : `x_token: List[str] = Header(...)`.
-   **Sensibilité à la casse :** Bien que la spécification HTTP dise que les noms d'en-têtes sont insensibles à la casse, certains serveurs ou proxys peuvent mal se comporter. FastAPI gère cela correctement, mais soyez conscient de l'écosystème autour de votre API.

---

## Concept 2 : Définir des En-têtes de Réponse {#concept-2-definir-des-en-tetes-de-reponse-25}

### Quoi ? {#quoi-26}
Pour ajouter, modifier ou supprimer des en-têtes dans la réponse HTTP, vous ne pouvez pas simplement les retourner dans votre dictionnaire de réponse. Vous devez accéder à l'objet `Response` et manipuler son attribut `headers`.

### Pourquoi ? {#pourquoi-26}
-   **Métadonnées personnalisées :** Renvoyer un identifiant de corrélation (`X-Request-ID`) pour faciliter le débogage et le suivi des requêtes à travers plusieurs services.
-   **Contrôle du cache :** Définir des en-têtes comme `Cache-Control` ou `ETag` pour indiquer aux clients et aux proxys comment mettre en cache la réponse.
-   **Sécurité :** Ajouter des en-têtes de sécurité comme `Content-Security-Policy` ou `X-Content-Type-Options`.
-   **Téléchargements de fichiers :** Définir `Content-Disposition` pour suggérer au navigateur de télécharger un fichier et de lui donner un nom spécifique.

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-26}
La méthode la plus simple est d'injecter l'objet `Response` dans la signature de votre fonction et d'utiliser son dictionnaire `headers`.

**Cas Réel : Renvoyer un identifiant de corrélation**

```python
import uuid
from fastapi import FastAPI, Response, Header

app = FastAPI()

@app.get("/process")
def process_data(response: Response, x_request_id: Optional[str] = Header(None)):
    # Si le client a fourni un ID, on le réutilise. Sinon, on en génère un.
    correlation_id = x_request_id or str(uuid.uuid4())
    
    # On ajoute l'en-tête à la réponse
    response.headers["X-Request-ID"] = correlation_id
    
    return {"message": "Data processed", "request_id": correlation_id}
```
Si vous inspectez la réponse de cet endpoint dans votre client HTTP, vous verrez l'en-tête `X-Request-ID` présent.

### Zone de Danger {#zone-de-danger-27}
-   **Les valeurs doivent être des chaînes :** Les valeurs des en-têtes HTTP doivent être des chaînes de caractères. Si vous essayez d'assigner un entier ou un autre type, une erreur sera levée. Pensez à convertir vos valeurs avec `str()`.
-   **En-têtes standards vs. personnalisés :** Par convention, les en-têtes non standards créés pour votre application devraient être préfixés par `X-` (ex: `X-My-Custom-Header`) pour éviter les conflits avec les futurs en-têtes standards.

---

### 3 Questions Clés {#3-questions-cles-25}
1.  Si vous définissez un paramètre de fonction `api_key: str = Header(...)`, quel est le nom exact de l'en-tête HTTP que FastAPI va rechercher ?
2.  Quelle est la méthode la plus directe pour ajouter un en-tête `X-API-Version: "2.1"` à une réponse sortante ?
3.  Un client envoie un en-tête `Accept-Language: fr` et aussi `Accept-Language: en`. Comment devriez-vous typer votre paramètre dans FastAPI pour recevoir les deux valeurs ?

### 3 Exercices Progressifs {#3-exercices-progressifs-25}

**Exercice 1 : Négociation de Langue**
Créez un endpoint `GET /greeting` qui lit l'en-tête `Accept-Language`.
-   Si l'en-tête contient `"fr"`, retournez `{"message": "Bonjour !"}`.
-   Sinon, retournez la salutation par défaut `{"message": "Hello !"}`.

<details>
<summary>Découvrir la solution commentée</summary>

```python
from typing import Optional
from fastapi import FastAPI, Header

app = FastAPI()

@app.get("/greeting")
async def get_greeting(accept_language: Optional[str] = Header(None)):
    if accept_language and "fr" in accept_language:
        return {"message": "Bonjour !"}
    return {"message": "Hello !"}
```
</details>

**Exercice 2 : Cache de Données Statiques**
Créez un endpoint `GET /static-data` qui retourne des données qui ne changent pas souvent. Ajoutez un en-tête de réponse `Cache-Control` pour indiquer aux clients qu'ils peuvent mettre la réponse en cache pendant une heure.

<details>
<summary>Découvrir la solution commentée</summary>

```python
from fastapi import FastAPI, Response

app = FastAPI()

@app.get("/static-data")
async def get_static_data(response: Response):
    # 'public' indique que la réponse peut être mise en cache par n'importe quel cache.
    # 'max-age=3600' indique une durée de vie de 3600 secondes (1 heure).
    response.headers["Cache-Control"] = "public, max-age=3600"
    
    return {"data": "This data rarely changes."}
```
</details>

**Exercice 3 : Miroir d'En-têtes Personnalisés**
Créez un endpoint `POST /mirror-headers`.
-   Il doit lire un en-tête personnalisé `X-Client-Info`.
-   Si l'en-tête est présent, il doit le renvoyer dans un en-tête de réponse nommé `X-Server-Mirrored-Info`.
-   Si l'en-tête est absent, il doit renvoyer un en-tête de réponse `X-Error` avec la valeur `"Client info header missing"`.

<details>
<summary>Découvrir la solution commentée</summary>

```python
from typing import Optional
from fastapi import FastAPI, Header, Response

app = FastAPI()

@app.post("/mirror-headers")
async def mirror_headers(response: Response, x_client_info: Optional[str] = Header(None, alias="X-Client-Info")):
    # Note : On utilise 'alias' ici pour la bonne pratique, bien que FastAPI gère la conversion.
    if x_client_info:
        response.headers["X-Server-Mirrored-Info"] = x_client_info
        return {"status": "success", "mirrored_header": x_client_info}
    else:
        response.headers["X-Error"] = "Client info header missing"
        return {"status": "error", "detail": "X-Client-Info header is required."}
```
</details>