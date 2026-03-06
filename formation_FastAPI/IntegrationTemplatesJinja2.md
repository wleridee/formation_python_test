---
sidebar_label: "Intégration de Templates (Jinja2)"
sidebar_position: 31
difficulty: "confirmé"
---

# Intégration de Templates (Jinja2) {#integration-de-templates-jinja2-31}

FastAPI est conçu pour créer des API JSON, mais il est tout à fait capable de rendre des pages HTML côté serveur. C'est particulièrement utile pour construire des panneaux d'administration, des pages de connexion, ou des applications web simples sans avoir recours à un framework frontend lourd.

La bibliothèque de templating la plus populaire en Python est Jinja2, et FastAPI s'intègre avec elle de manière transparente grâce à la classe `Jinja2Templates`.

**Prérequis :** Vous devez d'abord installer `jinja2`.
```bash
pip install jinja2
```

## Concept 1 : Configuration et Rendu d'un Template {#concept-1-configuration-et-rendu-dun-template-31}

### Quoi ? {#quoi-31}
L'intégration se fait en deux étapes :
1.  **Initialisation :** On crée une instance de `Jinja2Templates` en lui indiquant le répertoire où se trouvent nos fichiers de templates HTML.
2.  **Rendu :** Dans une opération de chemin, au lieu de retourner un dictionnaire (qui devient du JSON), on retourne une instance de `TemplateResponse`. Cette classe spéciale indique à FastAPI de rendre un template HTML.

### Pourquoi ? {#pourquoi-31}
Cette approche sépare clairement la logique de votre application (dans les fichiers Python) de la présentation (dans les fichiers HTML). Le moteur de template Jinja2 se charge de combiner les deux pour produire la page HTML finale, ce qui rend le code plus propre et plus facile à maintenir.

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-31}
1.  **Structure du projet :**
    ```
    .
    ├── main.py
    └── templates
        └── index.html
    ```

2.  **Contenu de `templates/index.html` :**
    ```html
    <!DOCTYPE html>
    <html>
    <head>
        <title>Page d'Accueil</title>
    </head>
    <body>
        <h1>Bienvenue sur notre site !</h1>
        <p>{{ message }}</p>
    </body>
    </html>
    ```

3.  **Code dans `main.py` :**
    ```python
    from fastapi import FastAPI, Request
    from fastapi.templating import Jinja2Templates
    from fastapi.responses import HTMLResponse

    app = FastAPI()

    # 1. Initialise Jinja2Templates en pointant vers le répertoire "templates"
    templates = Jinja2Templates(directory="templates")

    @app.get("/", response_class=HTMLResponse)
    async def read_root(request: Request):
        # 2. Retourne une TemplateResponse
        context = {
            "request": request,
            "message": "Ceci est un message dynamique venant de FastAPI."
        }
        return templates.TemplateResponse("index.html", context)
    ```
Quand vous visitez `/`, FastAPI utilise `index.html`, remplace `{{ message }}` par la valeur fournie, et envoie le HTML résultant.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Page web affichant le titre "Bienvenue sur notre site !" et le paragraphe "Ceci est un message dynamique venant de FastAPI."
> **Alt Text** : Rendu de la page index.html dans un navigateur, montrant le contenu statique et la variable dynamique injectée par Jinja2.

### Zone de Danger {#zone-de-danger-31}
L'erreur la plus commune est de mal spécifier le chemin du répertoire des templates. Si le chemin est incorrect, FastAPI lèvera une erreur `TemplateNotFound`. Assurez-vous que le chemin dans `Jinja2Templates(directory="...")` est correct par rapport à l'endroit où vous lancez votre application.

---

## Concept 2 : Contexte et Logique dans les Templates {#concept-2-contexte-et-logique-dans-les-templates-31}

### Quoi ? {#quoi-32}
Le "contexte" est le dictionnaire que vous passez à `TemplateResponse`. Chaque clé de ce dictionnaire devient une variable accessible à l'intérieur du template. Jinja2 ne se limite pas à la simple substitution de variables ; il supporte également les structures logiques comme les conditions (`if/else`) et les boucles (`for`).

L'objet `request` **doit obligatoirement** être inclus dans le contexte. Il est utilisé en interne par `TemplateResponse` et pour des fonctionnalités avancées comme `url_for`.

### Pourquoi ? {#pourquoi-32}
La logique dans les templates permet de construire des pages riches et dynamiques. Vous pouvez afficher une liste d'utilisateurs, montrer un bouton "Connexion" ou "Déconnexion" selon l'état de l'utilisateur, ou afficher des messages d'erreur sans avoir à créer plusieurs fichiers HTML.

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-32}
**Cas Réel : Afficher une liste de produits**

1.  **`main.py`**
    ```python
    # ... (initialisation de l'app et des templates)
    
    # Données factices
    db_products = [
        {"id": 1, "name": "Laptop", "price": 1200, "in_stock": True},
        {"id": 2, "name": "Souris", "price": 25, "in_stock": True},
        {"id": 3, "name": "Clavier", "price": 75, "in_stock": False},
    ]

    @app.get("/products", response_class=HTMLResponse)
    async def get_products(request: Request):
        context = {
            "request": request,
            "products": db_products,
            "is_admin": True # Simule un utilisateur admin
        }
        return templates.TemplateResponse("products.html", context)
    ```

2.  **`templates/products.html`**
    ```html
    <!DOCTYPE html>
    <html>
    <head><title>Nos Produits</title></head>
    <body>
        <h1>Liste des produits</h1>
        <ul>
            {% for product in products %}
                <li>
                    {{ product.name }} - {{ product.price }}€
                    {% if product.in_stock %}
                        <span style="color: green;">En stock</span>
                    {% else %}
                        <span style="color: red;">Rupture de stock</span>
                    {% endif %}
                </li>
            {% endfor %}
        </ul>

        {% if is_admin %}
            <p><a href="#">Ajouter un produit</a></p>
        {% endif %}
    </body>
    </html>
    ```
Ce template va boucler sur la liste `products`, afficher les détails de chacun, et utiliser des conditions pour afficher le statut du stock et le lien d'administration.

### Zone de Danger {#zone-de-danger-33}
Oublier la clé `"request": request` dans le dictionnaire de contexte est une erreur fréquente qui provoque une erreur interne du serveur. FastAPI a besoin de l'objet `request` pour fonctionner correctement. Traitez-le comme un élément obligatoire de chaque contexte de template.

---

## Concept 3 : Utilisation de `url_for` avec des Fichiers Statiques {#concept-3-utilisation-de-url_for-avec-des-fichiers-statiques-31}

### Quoi ? {#quoi-33}
Pour que vos templates soient robustes, vous ne devriez jamais coder en dur les URLs vers vos fichiers statiques (CSS, JS, images) ou vers d'autres pages de votre application. Jinja2, grâce à l'objet `request` que vous lui passez, peut utiliser la fonction `url_for` pour générer dynamiquement ces URLs en se basant sur le `name` de vos montages statiques ou le nom de vos fonctions d'opération de chemin.

### Pourquoi ? {#pourquoi-34}
Si vous décidez de changer le chemin de montage de vos fichiers statiques (de `/static` à `/assets`), vous n'aurez qu'à changer une seule ligne dans votre `main.py` au lieu de devoir modifier tous vos fichiers HTML. Cela rend votre application beaucoup plus maintenable.

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-34}
On combine le montage de `StaticFiles` avec la syntaxe `{{ url_for(...) }}` dans le template.

1.  **`main.py`**
    ```python
    from fastapi import FastAPI, Request
    from fastapi.templating import Jinja2Templates
    from fastapi.staticfiles import StaticFiles

    app = FastAPI()
    templates = Jinja2Templates(directory="templates")

    # Monte les fichiers statiques avec un nom
    app.mount("/static", StaticFiles(directory="static"), name="static")

    @app.get("/item/{item_id}")
    async def get_item_page(request: Request, item_id: str):
        context = {"request": request, "item_id": item_id}
        return templates.TemplateResponse("item.html", context)
    ```

2.  **`templates/item.html`**
    ```html
    <!DOCTYPE html>
    <html>
    <head>
        <title>Détail de l'article</title>
        <!-- Génère l'URL pour un fichier statique -->
        <link href="{{ url_for('static', path='/css/main.css') }}" rel="stylesheet">
    </head>
    <body>
        <h1>Détails pour l'article : {{ item_id }}</h1>
        <img src="{{ url_for('static', path='/images/item_placeholder.png') }}" alt="Image produit">

        <!-- Génère l'URL pour une autre page/endpoint -->
        <a href="{{ url_for('get_item_page', item_id=item_id) }}">Lien permanent vers cette page</a>
    </body>
    </html>
    ```

### Zone de Danger {#zone-de-danger-35}
Pour que `url_for` fonctionne, trois conditions doivent être remplies :
1.  L'objet `request` doit être dans le contexte.
2.  Pour les fichiers statiques, le `name` dans `app.mount` doit correspondre au premier argument de `url_for`.
3.  Pour les autres endpoints, le premier argument de `url_for` doit correspondre au **nom de la fonction Python** de l'opération de chemin (ex: `get_item_page`).

---

### 3 Questions Clés {#3-questions-cles-31}
1.  Quelle classe de réponse est spécifiquement utilisée pour rendre un template HTML avec FastAPI, et quels sont ses deux arguments principaux ?
2.  Pourquoi est-il crucial d'inclure la paire clé-valeur `"request": request` dans le dictionnaire de contexte ?
3.  Comment générer, depuis un template Jinja2, un lien vers un autre endpoint de votre application qui accepte un paramètre de chemin ?

### 3 Exercices Progressifs {#3-exercices-progressifs-31}

**Exercice 1 : Page de Profil Basique**
Créez un endpoint `GET /user/{username}` qui rend un template `user.html`. Le template doit afficher un titre `<h1>Profil de {{ username }}</h1>`, où `username` est passé depuis l'URL.

<details>
<summary>Découvrir la solution commentée</summary>

**`templates/user.html`**
```html
<!DOCTYPE html><html><head><title>Profil</title></head>
<body><h1>Profil de {{ username }}</h1></body></html>
```
**`main.py`**
```python
from fastapi import FastAPI, Request
from fastapi.templating import Jinja2Templates

app = FastAPI()
templates = Jinja2Templates(directory="templates")

@app.get("/user/{username}")
async def get_user_profile(request: Request, username: str):
    return templates.TemplateResponse(
        "user.html", 
        {"request": request, "username": username}
    )
```
</details>

**Exercice 2 : Liste de Tâches avec CSS**
1.  Créez un endpoint `GET /todos` qui affiche une liste de tâches (un tableau de dictionnaires).
2.  Le template `todos.html` doit utiliser une boucle `for` pour afficher chaque tâche dans un `<li>`.
3.  Créez un fichier `static/style.css` qui met les tâches en `list-style-type: none;` et ajoute un peu de `padding`.
4.  Chargez cette feuille de style dans votre template en utilisant `url_for`.

<details>
<summary>Découvrir la solution commentée</summary>

**`static/style.css`**
```css
ul { list-style-type: none; padding-left: 0; }
li { padding: 8px; border-bottom: 1px solid #ccc; }
```
**`templates/todos.html`**
```html
<!DOCTYPE html><html><head><title>Tâches</title>
<link href="{{ url_for('static', path='/style.css') }}" rel="stylesheet">
</head><body>
    <h1>Liste de Tâches</h1><ul>
    {% for todo in todos %}
        <li>{{ todo.title }}</li>
    {% endfor %}
    </ul></body></html>
```
**`main.py`**
```python
from fastapi import FastAPI, Request
from fastapi.templating import Jinja2Templates
from fastapi.staticfiles import StaticFiles

app = FastAPI()
templates = Jinja2Templates(directory="templates")
app.mount("/static", StaticFiles(directory="static"), name="static")

@app.get("/todos")
async def get_todos(request: Request):
    todo_list = [{"id": 1, "title": "Acheter du lait"}, {"id": 2, "title": "Apprendre Jinja2"}]
    return templates.TemplateResponse(
        "todos.html", 
        {"request": request, "todos": todo_list}
    )
```
</details>

**Exercice 3 : Héritage de Templates**
Créez une application avec une structure de base réutilisable.
1.  Créez un template `base.html` avec une structure HTML de base, un titre, un en-tête (`<header>`) et un pied de page (`<footer>`). Il doit définir un bloc de contenu : `{% block content %}{% endblock %}`.
2.  Créez un template `home.html` qui hérite de `base.html` (`{% extends "base.html" %}`).
3.  Ce template `home.html` doit remplir le bloc de contenu avec un message de bienvenue.
4.  Créez un endpoint `GET /` qui rend `home.html`.

<details>
<summary>Découvrir la solution commentée</summary>

**`templates/base.html`**
```html
<!DOCTYPE html><html><head>
<title>{% block title %}Mon Site{% endblock %}</title>
</head><body>
    <header><h1>Mon Application Web</h1></header>
    <main>
        {% block content %}{% endblock %}
    </main>
    <footer><p>&copy; 2024</p></footer>
</body></html>
```
**`templates/home.html`**
```html
{% extends "base.html" %}

{% block title %}Accueil - {{ super() }}{% endblock %}

{% block content %}
    <h2>Bienvenue sur la page d'accueil !</h2>
    <p>Ceci est le contenu spécifique à la page d'accueil.</p>
{% endblock %}
```
**`main.py`**
```python
from fastapi import FastAPI, Request
from fastapi.templating import Jinja2Templates

app = FastAPI()
templates = Jinja2Templates(directory="templates")

@app.get("/")
async def home(request: Request):
    return templates.TemplateResponse("home.html", {"request": request})
```
</details>