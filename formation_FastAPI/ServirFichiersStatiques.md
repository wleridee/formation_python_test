---
sidebar_label: "Servir des Fichiers Statiques"
sidebar_position: 30
difficulty: "confirmé"
---

# Servir des Fichiers Statiques {#servir-des-fichiers-statiques-30}

Bien que FastAPI soit avant tout un framework pour construire des API, il est courant d'avoir besoin de servir des fichiers statiques directement depuis la même application. Il peut s'agir d'images, de feuilles de style CSS, de fichiers JavaScript, ou même d'une application frontend complète (Single Page Application).

FastAPI s'appuie sur Starlette pour fournir un moyen simple et efficace de "monter" un répertoire de fichiers statiques afin qu'il soit accessible via des URLs spécifiques.

## Concept 1 : Monter un Répertoire avec `StaticFiles` {#concept-1-monter-un-repertoire-avec-staticfiles-30}

### Quoi ? {#quoi-30}
La classe `StaticFiles` de Starlette est une application ASGI spécialisée qui peut servir des fichiers à partir d'un répertoire sur le disque. FastAPI vous permet de "monter" cette application à un chemin spécifique de votre API. Une fois montée, toute requête commençant par ce chemin sera gérée par `StaticFiles` au lieu de vos opérations de chemin.

### Pourquoi ? {#pourquoi-30}
-   **Simplicité :** Pour les petits projets, les panneaux d'administration, ou en développement, servir les fichiers statiques avec FastAPI évite d'avoir à configurer un serveur web distinct comme Nginx.
-   **Contenu généré :** Pour servir des fichiers générés par les utilisateurs (avatars, rapports, etc.) stockés dans un répertoire spécifique.
-   **Couplage :** Permet de lier une petite interface utilisateur (HTML/CSS/JS) directement à l'API qui la soutient, créant une application autonome.

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-30}
La syntaxe est `app.mount("/prefixe-url", StaticFiles(directory="chemin-local"), name="nom-unique")`.

**Cas Réel : Servir des images et des CSS**

1.  Créez la structure de répertoires suivante :
    ```
    .
    ├── main.py
    └── static
        ├── css
        │   └── styles.css
        └── images
            └── logo.png
    ```

2.  Remplissez `static/css/styles.css` avec :
    ```css
    body { background-color: #f0f0f0; }
    img { border: 2px solid #333; }
    ```

3.  Dans `main.py`, montez le répertoire `static` :
    ```python
    from fastapi import FastAPI
    from fastapi.staticfiles import StaticFiles

    app = FastAPI()

    # Monte le répertoire "static" sur le chemin d'URL "/static"
    app.mount("/static", StaticFiles(directory="static"), name="static")

    @app.get("/")
    async def root():
        return {"message": "Hello World"}
    ```

4.  Lancez l'application. Vous pouvez maintenant accéder à vos fichiers statiques :
    -   `http://127.0.0.1:8000/static/css/styles.css`
    -   `http://127.0.0.1:8000/static/images/logo.png`

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Navigateur affichant le contenu brut du fichier `styles.css` après avoir accédé à l'URL correspondante.
> **Alt Text** : Page web simple montrant le texte "body { background-color: #f0f0f0; }" à l'URL http://127.0.0.1:8000/static/css/styles.css.

### Zone de Danger {#zone-de-danger-30}
**L'ordre des opérations compte !** Les applications montées sont vérifiées **avant** les opérations de chemin de l'API. Si vous montez une application sur un chemin qui entre en conflit avec l'un de vos endpoints, c'est l'application montée qui prendra le dessus. Par exemple, si vous avez un endpoint `@app.get("/static/special")` et que vous montez un répertoire sur `/static`, votre endpoint ne sera jamais atteint.

---

## Concept 2 : Servir une Application Frontend (SPA) {#concept-2-servir-une-application-frontend-spa-30}

### Quoi ? {#quoi-31}
Un cas d'usage très courant est de déployer une application web moderne (créée avec React, Vue, Svelte, etc.) avec une API FastAPI dans un seul conteneur. L'API est servie sous un préfixe (ex: `/api`), et l'application frontend est servie sur toutes les autres routes, y compris la racine (`/`).

### Pourquoi ? {#pourquoi-31}
Cela simplifie considérablement le déploiement et l'infrastructure pour de nombreux projets. Vous n'avez qu'un seul service à gérer, à surveiller et à mettre à l'échelle.

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-31}
La clé est de **monter l'API en premier**, puis de monter les fichiers statiques du frontend. En utilisant l'option `html=True` pour `StaticFiles`, Starlette servira automatiquement le fichier `index.html` pour les chemins qui correspondent à des répertoires, ce qui est parfait pour le routage côté client des SPAs.

**Cas Réel : Servir une API et une SPA React**

1.  Imaginez que la commande `npm run build` de votre projet React a généré les fichiers dans un répertoire `frontend/build`.
    ```
    .
    ├── main.py
    └── frontend
        └── build
            ├── static/
            │   ├── css/
            │   └── js/
            └── index.html
    ```
2.  Dans votre `main.py`, structurez le code comme suit :

    ```python
    from fastapi import FastAPI
    from fastapi.staticfiles import StaticFiles

    app = FastAPI()

    # 1. Définir et monter le routeur de l'API (important de le faire avant les statiques)
    # Pour cet exemple, nous mettons un endpoint directement sur l'app
    @app.get("/api/v1/data")
    def get_data():
        return {"data": "from the API"}

    # 2. Monter les fichiers statiques du frontend (après l'API)
    app.mount("/", StaticFiles(directory="frontend/build", html=True), name="static-frontend")
    ```

L'ordre est essentiel :
1.  Une requête `GET /api/v1/data` est interceptée par l'opération de chemin de FastAPI.
2.  Une requête `GET /static/css/main.css` ne correspond à aucune route de l'API. Elle est donc passée au gestionnaire `StaticFiles` qui trouve et sert le fichier.
3.  Une requête `GET /user/profile` ne correspond ni à une route API, ni à un fichier physique. Grâce à `html=True`, `StaticFiles` sert le fichier `index.html` à la racine, et le routeur de React prend le relais côté client.

### Zone de Danger {#zone-de-danger-32}
Si vous montez les fichiers statiques **avant** de définir vos routes d'API, le montage sur `"/"` interceptera **toutes** les requêtes, y compris celles destinées à votre API (comme `/api/v1/data`). Il essaiera de trouver un fichier correspondant sur le disque, échouera, et renverra une erreur 404. L'API deviendra alors inaccessible.

---

## Concept 3 : Générer des URLs pour les Fichiers Statiques {#concept-3-generer-des-urls-pour-les-fichiers-statiques-30}

### Quoi ? {#quoi-32}
FastAPI peut générer dynamiquement les URLs complètes pour vos fichiers statiques. Cela se fait en utilisant la fonction `request.url_for()` et le paramètre `name` que vous avez fourni lors du montage.

### Pourquoi ? {#pourquoi-33}
-   **Flexibilité :** Vous pouvez changer le chemin de montage (par exemple de `/static` à `/assets`) en un seul endroit sans avoir à rechercher et remplacer des URLs codées en dur dans tout votre code.
-   **Robustesse :** Cela garantit que les URLs sont toujours construites correctement, y compris le nom d'hôte et le port.
-   **Templating :** C'est particulièrement utile lorsque vous utilisez un moteur de template comme Jinja2 pour générer des pages HTML côté serveur.

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-33}
Il faut injecter l'objet `Request` dans votre opération de chemin pour pouvoir utiliser `url_for`.

**Cas Réel : Renvoyer le lien d'un avatar utilisateur**

```python
from fastapi import FastAPI, Request
from fastapi.staticfiles import StaticFiles

app = FastAPI()

# Monte un répertoire où sont stockés les avatars des utilisateurs
app.mount("/user-content", StaticFiles(directory="user_uploads"), name="user_content")

@app.get("/users/{username}")
async def get_user_profile(username: str, request: Request):
    # Imaginez que le nom du fichier est stocké en base de données
    avatar_filename = f"{username}_avatar.jpg"
    
    # Génère l'URL complète vers le fichier
    avatar_url = request.url_for('user_content', path=avatar_filename)
    
    return {
        "username": username,
        "avatar_url": avatar_url # ex: http://127.0.0.1:8000/user-content/john_doe_avatar.jpg
    }
```

### Zone de Danger {#zone-de-danger-34}
Oublier de spécifier le paramètre `name` lors de `app.mount()` est une erreur courante. Si vous essayez d'appeler `request.url_for('some_name', ...)` sans que le nom `'some_name'` n'ait été défini dans un `mount`, FastAPI lèvera une exception `NoMatchFound`. Assurez-vous que chaque montage destiné à la génération d'URL a un `name` unique.

---

### 3 Questions Clés {#3-questions-cles-30}
1.  Quelle est la fonction et quels sont les trois arguments principaux utilisés pour rendre un répertoire de fichiers accessible via des URLs ?
2.  Lorsque vous servez une API sous `/api` et une SPA à la racine `/`, dans quel ordre devez-vous déclarer/monter ces deux éléments dans votre code FastAPI, et pourquoi ?
3.  Comment pouvez-vous obtenir l'URL complète d'un fichier situé dans `static/images/logo.png` à l'intérieur d'une opération de chemin, en supposant que le montage a été fait avec `name="assets"` ?

### 3 Exercices Progressifs {#3-exercices-progressifs-30}

**Exercice 1 : Créer une Page de Profil Simple**
1.  Créez un répertoire `static` avec un fichier `profile.css`.
2.  Montez ce répertoire sur `/static`.
3.  Créez un endpoint `GET /profile` qui retourne une réponse HTML (`from fastapi.responses import HTMLResponse`).
4.  Le HTML doit inclure une balise `<link>` qui charge `profile.css`.
5.  Le CSS doit simplement changer la couleur de fond de la page.

<details>
<summary>Découvrir la solution commentée</summary>

**Structure :**
```
.
├── main.py
└── static
    └── profile.css
```
**`static/profile.css`**
```css
h1 { color: steelblue; }
```
**`main.py`**
```python
from fastapi import FastAPI
from fastapi.staticfiles import StaticFiles
from fastapi.responses import HTMLResponse

app = FastAPI()
app.mount("/static", StaticFiles(directory="static"), name="static")

@app.get("/profile", response_class=HTMLResponse)
async def get_profile_page():
    html_content = """
    <html>
        <head>
            <title>Mon Profil</title>
            <link href="/static/profile.css" rel="stylesheet">
        </head>
        <body>
            <h1>Page de Profil</h1>
        </body>
    </html>
    """
    return html_content
```
</details>

**Exercice 2 : Générateur de Lien d'Image Dynamique**
1.  Créez un répertoire `assets/icons`. Placez-y deux images : `file.png` et `folder.png`.
2.  Montez le répertoire `assets` sur l'URL `/assets`. Donnez-lui le nom `assets`.
3.  Créez un endpoint `GET /icon-url/{icon_type}`.
4.  Si `icon_type` est "file", il doit retourner un JSON avec l'URL complète de `file.png`.
5.  Si `icon_type` est "folder", il doit retourner l'URL de `folder.png`.
6.  Utilisez `request.url_for()` pour générer les URLs.

<details>
<summary>Découvrir la solution commentée</summary>

**Structure :**
```
.
├── main.py
└── assets
    └── icons
        ├── file.png
        └── folder.png
```
**`main.py`**
```python
from fastapi import FastAPI, Request, HTTPException

from fastapi.staticfiles import StaticFiles

app = FastAPI()
app.mount("/assets", StaticFiles(directory="assets"), name="assets")

@app.get("/icon-url/{icon_type}")
async def get_icon_url(icon_type: str, request: Request):
    if icon_type not in ["file", "folder"]:
        raise HTTPException(status_code=404, detail="Icon type not found")
    
    file_path = f"icons/{icon_type}.png"
    url = request.url_for('assets', path=file_path)
    
    return {"icon_type": icon_type, "url": url}
```
</details>

**Exercice 3 : Structure de Projet pour une SPA et une API**
Simulez la structure de service pour une application "Todo".
1.  Créez un routeur pour l'API dans `routers/todos.py` avec un endpoint `GET /` qui retourne une liste de tâches.
2.  Créez un répertoire `spa/build` avec un `index.html` factice et un sous-répertoire `css/style.css`.
3.  Dans `main.py` :
    a. Incluez le routeur de l'API avec le préfixe `/api/todos`.
    b. Montez le répertoire `spa/build` à la racine (`/`) de l'application pour servir le frontend.
4.  Vérifiez que `http://.../api/todos` retourne le JSON et que `http://.../` retourne le contenu de votre `index.html`.

<details>
<summary>Découvrir la solution commentée</summary>

**Structure :**
```
.
├── main.py
├── routers
│   └── todos.py
└── spa
    └── build
        ├── css
        │   └── style.css
        └── index.html
```
**`routers/todos.py`**
```python
from fastapi import APIRouter
router = APIRouter()
@router.get("/")
def get_todos(): return [{"id": 1, "task": "Learn FastAPI"}]
```
**`spa/build/index.html`**
```html
<!DOCTYPE html>
<html>
<head><title>Todo App</title></head>
<body><h1>My SPA is loading...</h1></body>
</html>
```
**`main.py`**
```python
from fastapi import FastAPI
from fastapi.staticfiles import StaticFiles
from routers import todos

app = FastAPI()

# Étape 1: Inclure l'API en premier
app.include_router(todos.router, prefix="/api/todos", tags=["todos"])

# Étape 2: Monter les fichiers statiques en dernier
app.mount("/", StaticFiles(directory="spa/build", html=True), name="spa")
```
</details>