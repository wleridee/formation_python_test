---
sidebar_label: "Gestion des Cookies"
sidebar_position: 24
difficulty: "confirmé"
---

# Gestion des Cookies {#gestion-des-cookies-24}

Les cookies sont un mécanisme essentiel pour la gestion de l'état dans le protocole HTTP, qui est par nature sans état ("stateless"). Ils permettent aux serveurs de stocker de petites informations dans le navigateur du client, qui sont ensuite renvoyées à chaque requête. Les cas d'usage les plus courants sont la gestion des sessions utilisateur, la personnalisation et le suivi.

FastAPI fournit des outils simples et élégants pour lire les cookies entrants et pour définir ou supprimer des cookies dans les réponses sortantes.

## Concept 1 : Lire les Cookies avec `Cookie` {#concept-1-lire-les-cookies-avec-cookie-24}

### Quoi ? {#quoi-24}
`Cookie` est une fonction de dépendance, similaire à `Path`, `Query` ou `Header`. Elle permet de déclarer qu'une valeur doit être extraite des cookies de la requête HTTP. FastAPI s'occupe de trouver le cookie correspondant par son nom (la clé) et d'injecter sa valeur dans votre paramètre de fonction.

### Pourquoi ? {#pourquoi-24}
C'est le mécanisme standard pour récupérer des informations d'état. Votre API peut ainsi :
-   Identifier un utilisateur via un token de session.
-   Lire les préférences de l'utilisateur (ex: thème sombre, langue).
-   Accéder à des identifiants de suivi pour l'analyse.

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-24}
On importe `Cookie` de `fastapi` et on l'utilise comme valeur par défaut pour un paramètre.

**Cas Réel : Lire un identifiant de session**

```python
from typing import Optional
from fastapi import FastAPI, Cookie

app = FastAPI()

@app.get("/items/")
async def read_items(session_id: Optional[str] = Cookie(None)):
    # La variable 'session_id' contiendra la valeur du cookie 'session_id'
    # ou None si le cookie n'est pas présent dans la requête.
    if not session_id:
        return {"message": "Welcome, new user! No session found."}
    return {"message": f"Welcome back! Your session ID is: {session_id}"}
```

### Zone de Danger {#zone-de-danger-24}
Un cookie peut ne pas être présent dans la requête. Si vous déclarez `session_id: str = Cookie(...)`, FastAPI retournera une erreur de validation 422 si le cookie est manquant. Il est presque toujours préférable de déclarer les cookies comme optionnels (`Optional[str] = Cookie(None)`) pour gérer gracieusement les cas où le client n'envoie pas le cookie attendu.

---

## Concept 2 : Définir des Cookies avec `response.set_cookie` {#concept-2-definir-des-cookies-avec-response.set_cookie-24}

### Quoi ? {#quoi-25}
Pour définir un cookie, vous ne pouvez pas simplement le retourner dans votre réponse JSON. Vous devez modifier les en-têtes de la réponse HTTP. FastAPI vous donne accès à l'objet `Response` sur lequel vous pouvez appeler la méthode `set_cookie`.

### Pourquoi ? {#pourquoi-25}
C'est la seule façon standard de demander au navigateur du client de stocker une information. C'est la base de la connexion utilisateur : après avoir vérifié les identifiants, le serveur définit un cookie de session que le navigateur renverra ensuite.

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-25}
Il faut injecter l'objet `Response` dans la signature de votre fonction.

**Cas Réel : Créer une session utilisateur lors de la connexion**

```python
import uuid
from fastapi import FastAPI, Response

app = FastAPI()

@app.post("/login")
def login(response: Response):
    session_id = str(uuid.uuid4())
    
    # Définit un cookie
    response.set_cookie(
        key="session_id", 
        value=session_id,
        httponly=True,       # Le cookie n'est pas accessible via JavaScript (sécurité XSS)
        secure=True,         # Le cookie est envoyé uniquement via HTTPS
        samesite="lax",      # Protection contre les attaques CSRF
        max_age=1800         # Durée de vie en secondes (30 minutes)
    )
    
    return {"message": "Login successful"}
```
Les paramètres `httponly`, `secure` et `samesite` sont **cruciaux** pour la sécurité de votre application.

### Zone de Danger {#zone-de-danger-26}
Ne stockez jamais de données sensibles (mots de passe, informations personnelles) directement dans un cookie. Les cookies sont stockés sur la machine du client et peuvent être inspectés. Stockez uniquement un identifiant de session sécurisé et non devinable, et utilisez cet identifiant côté serveur pour récupérer les données de l'utilisateur depuis une base de données. **Toujours utiliser `httponly=True` pour les cookies d'authentification.**

---

## Concept 3 : Supprimer des Cookies avec `response.delete_cookie` {#concept-3-supprimer-des-cookies-avec-response.delete_cookie-24}

### Quoi ? {#quoi-27}
Pour supprimer un cookie, vous devez indiquer au navigateur de l'expirer. La méthode `response.delete_cookie(key)` est un raccourci pratique qui fait exactement cela : elle définit un cookie avec le même nom, mais avec une date d'expiration dans le passé et un contenu vide.

### Pourquoi ? {#pourquoi-27}
Le cas d'usage principal est la déconnexion (`logout`). En supprimant le cookie de session, vous invalidez la session de l'utilisateur dans son navigateur, le forçant à se reconnecter pour les futures actions protégées.

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-27}
Tout comme pour `set_cookie`, on utilise l'objet `Response` injecté.

**Cas Réel : Endpoint de déconnexion**

```python
from fastapi import FastAPI, Response

app = FastAPI()

# ... (votre endpoint de login qui définit le cookie)

@app.post("/logout")
def logout(response: Response):
    # Supprime le cookie 'session_id'
    response.delete_cookie(key="session_id")
    return {"message": "Logout successful"}
```

### Zone de Danger {#zone-de-danger-28}
Pour que la suppression fonctionne, les attributs `path` et `domain` de `delete_cookie` doivent correspondre exactement à ceux avec lesquels le cookie a été défini. Si vous avez défini un cookie avec `path="/app"`, vous devez appeler `response.delete_cookie(key="...", path="/app")`. Si vous oubliez cela, le navigateur considérera qu'il s'agit de deux cookies différents et ne supprimera pas l'original. C'est une source de bugs très fréquente.

---

### 3 Questions Clés {#3-questions-cles-24}
1.  Quelle est la syntaxe pour lire un cookie nommé `user_preference`, en s'assurant que l'application ne plante pas s'il est absent ?
2.  Pour définir un cookie de session, quel objet devez-vous ajouter aux paramètres de votre fonction ? Citez trois paramètres de `set_cookie` essentiels pour la sécurité.
3.  Vous appelez `response.delete_cookie("session_token")` mais le cookie n'est pas supprimé dans le navigateur. Quelle est la cause la plus probable de ce problème ?

### 3 Exercices Progressifs {#3-exercices-progressifs-24}

**Exercice 1 : Personnalisation du Thème**
Créez deux endpoints :
1.  `POST /theme` : Accepte un paramètre de requête `name`. Il doit définir un cookie nommé `app_theme` avec la valeur de `name`.
2.  `GET /` : Lit le cookie `app_theme`. S'il est présent, retourne `{"message": f"Welcome! Your theme is '{theme_value}'."}`. Sinon, retourne `{"message": "Welcome! No theme set."}`.

<details>
<summary>Découvrir la solution commentée</summary>

```python
from typing import Optional
from fastapi import FastAPI, Cookie, Response

app = FastAPI()

@app.post("/theme")
def set_theme(response: Response, name: str):
    response.set_cookie(key="app_theme", value=name, max_age=31536000) # 1 an
    return {"message": f"Theme set to '{name}'"}

@app.get("/")
def get_home(app_theme: Optional[str] = Cookie(None)):
    if app_theme:
        return {"message": f"Welcome! Your theme is '{app_theme}'."}
    return {"message": "Welcome! No theme set."}
```
</details>

**Exercice 2 : Compteur de Visites**
Créez un seul endpoint `GET /visit-count` qui lit un cookie nommé `visit_counter`.
-   Si le cookie n'existe pas, initialisez le compteur à 1.
-   Si le cookie existe, incrémentez sa valeur (qui est une chaîne, n'oubliez pas de la convertir en entier).
-   Dans tous les cas, redéfinissez le cookie avec la nouvelle valeur et retournez un message indiquant le nombre de visites de l'utilisateur.

<details>
<summary>Découvrir la solution commentée</summary>

```python
from typing import Optional
from fastapi import FastAPI, Cookie, Response

app = FastAPI()

@app.get("/visit-count")
def get_visit_count(response: Response, visit_counter: Optional[str] = Cookie(None)):
    count = 0
    if visit_counter:
        try:
            count = int(visit_counter)
        except ValueError:
            # Si le cookie est corrompu, on réinitialise.
            count = 0
    
    new_count = count + 1
    response.set_cookie(key="visit_counter", value=str(new_count))
    
    return {"message": f"This is your visit number {new_count}."}
```
</details>

**Exercice 3 : Flux de Déconnexion Sécurisé**
Mettez en place un endpoint de déconnexion robuste.
-   Créez un endpoint `POST /login-secure` qui définit un cookie `secure_session` avec `httponly=True` et `path="/api"`.
-   Créez un endpoint `POST /logout-secure` qui supprime ce cookie. Assurez-vous d'utiliser les bons paramètres pour que la suppression fonctionne.

<details>
<summary>Découvrir la solution commentée</summary>

```python
from fastapi import FastAPI, Response

app = FastAPI()

@app.post("/login-secure")
def login_secure(response: Response):
    # En production, secure=True serait activé.
    response.set_cookie(
        key="secure_session", 
        value="fake-secure-token",
        httponly=True,
        path="/api" 
    )
    return {"message": "Secure session started in /api path"}

@app.post("/logout-secure")
def logout_secure(response: Response):
    # Pour que la suppression fonctionne, le path DOIT correspondre à celui de la création.
    response.delete_cookie(key="secure_session", path="/api")
    return {"message": "Secure session ended"}
```
*Note : Pour tester cet exercice, vous aurez besoin d'un client HTTP comme Postman ou Insomnia, ou d'outils de développement de navigateur, car vous ne pouvez pas facilement appeler `/logout-secure` depuis la documentation interactive si le cookie est `httponly` et défini sur un chemin spécifique.*
</details>