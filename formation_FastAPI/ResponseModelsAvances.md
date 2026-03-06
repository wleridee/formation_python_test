---
sidebar_label: "Modèles de Réponse Avancés : Listes et Types Arbitraires"
sidebar_position: 21
difficulty: "confirmé"
---

# Modèles de Réponse Avancés : Listes et Types Arbitraires {#modeles-de-reponse-avances--listes-et-types-arbitraires-21}

Jusqu'à présent, nous avons utilisé `response_model` pour retourner un objet unique. Cependant, les applications réelles nécessitent des structures de données plus complexes. Les deux cas les plus courants sont le renvoi de listes d'objets et le renvoi de données provenant directement de votre couche de données (comme un ORM) sans conversion manuelle.

FastAPI, en s'appuyant sur Pydantic, gère ces scénarios avancés avec une simplicité et une robustesse remarquables, garantissant que vos réponses sont toujours validées, filtrées et correctement documentées.

## Concept 1 : Renvoyer des Listes de Modèles {#concept-1-renvoyer-des-listes-de-modeles-21}

### Quoi ? {#quoi-21}
Pour déclarer qu'un endpoint renvoie une liste d'objets, vous pouvez utiliser les "types génériques" de Python, en particulier `List`. En définissant `response_model=List[MonModele]`, vous indiquez à FastAPI que la réponse sera un tableau JSON, où chaque élément du tableau doit être conforme à la structure de `MonModele`.

### Pourquoi ? {#pourquoi-21}
-   **Documentation Précise :** La documentation OpenAPI spécifiera clairement que la réponse est un tableau (`array`) d'un certain type (`schema`), ce qui est essentiel pour les clients qui génèrent leur code à partir de votre documentation.
-   **Validation Complète :** FastAPI validera *chaque élément* de la liste avant d'envoyer la réponse. Si un seul objet de la liste ne correspond pas au modèle, une erreur interne sera levée, empêchant l'envoi de données corrompues.
-   **Filtrage des Données :** Le filtrage des données s'applique à chaque élément de la liste, garantissant qu'aucune donnée sensible non définie dans le `response_model` ne soit exposée.

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-21}
On importe `List` depuis le module `typing` et on l'utilise pour "envelopper" notre modèle Pydantic.

**Cas Réel : Lister tous les produits d'un catalogue**

```python
from typing import List, Optional
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Product(BaseModel):
    id: int
    name: str
    price: float
    is_available: bool = True
    description: Optional[str] = None

# Données simulées de la base de données
db_products = [
    {"id": 1, "name": "Laptop", "price": 1200.0, "is_available": True},
    {"id": 2, "name": "Smartphone", "price": 800.0, "is_available": False, "secret_key": "abc"},
    {"id": 3, "name": "Headphones", "price": 150.0},
]

@app.get("/products/", response_model=List[Product])
async def get_all_products():
    # FastAPI va itérer sur cette liste, valider chaque dictionnaire
    # contre le modèle Product, et filtrer les champs inconnus (comme "secret_key").
    return db_products
```
La réponse JSON sera un tableau de produits, et le champ `"secret_key"` du deuxième produit sera automatiquement retiré.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Schéma de réponse pour `GET /products/` dans Swagger UI.
> **Alt Text** : Interface Swagger UI montrant que la réponse attendue pour l'endpoint /products/ est un "array of Product", avec le schéma détaillé du modèle Product.

### Zone de Danger {#zone-de-danger-21}
N'oubliez pas d'importer `List` depuis `typing`. Une erreur commune est d'utiliser `list[Product]` (syntaxe moderne de Python) qui, bien que fonctionnelle dans de nombreux cas, peut être moins compatible avec certains outils et versions antérieures. L'utilisation de `typing.List` est la plus robuste et la plus claire dans le contexte de FastAPI.

---

## Concept 2 : Le Mode ORM (`from_attributes`) {#concept-2-le-mode-orm-from_attributes-21}

### Quoi ? {#quoi-22}
`from_attributes=True` est une option de configuration dans un modèle Pydantic. Elle lui permet de lire les données non pas depuis un dictionnaire (accès par clé, ex: `data['name']`) mais depuis les attributs d'un objet (accès par point, ex: `obj.name`). C'est le pont qui permet à Pydantic de comprendre et de sérialiser des objets provenant d'autres bibliothèques, notamment les ORM (Object-Relational Mappers) comme SQLAlchemy ou Tortoise ORM.

*Note : Dans Pydantic V1 et les anciennes versions de FastAPI, cette option s'appelait `orm_mode = True` dans une classe `Config` interne.*

### Pourquoi ? {#pourquoi-22}
C'est une fonctionnalité qui change la vie pour la propreté du code. Sans elle, vous seriez obligé de convertir manuellement chaque objet de base de données en dictionnaire avant de le retourner :

-   **Avant (fastidieux) :** `return [{"id": item.id, "name": item.name} for item in db_items]`
-   **Avec `from_attributes` (propre) :** `return db_items`

Cela permet de garder votre logique d'endpoint extrêmement simple et de déléguer toute la transformation de données à la couche de sérialisation (Pydantic).

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-22}
On définit une configuration de modèle en utilisant `model_config` et on y place `from_attributes = True`.

**Cas Réel : Renvoyer un objet utilisateur depuis une fausse couche ORM**

```python
from fastapi import FastAPI
from pydantic import BaseModel, ConfigDict

# 1. Simule un objet ORM (ex: de SQLAlchemy)
class DBUser:
    def __init__(self, id, username, email, hashed_password):
        self.id = id
        self.username = username
        self.email = email
        self.hashed_password = hashed_password

# 2. Le modèle Pydantic de réponse
class UserOut(BaseModel):
    # On définit les champs qu'on veut exposer
    id: int
    username: str
    email: str

    # 3. La configuration magique !
    model_config = ConfigDict(from_attributes=True)
    
    # Ancienne syntaxe (Pydantic V1):
    # class Config:
    #     orm_mode = True

app = FastAPI()

@app.get("/users/{user_id}", response_model=UserOut)
async def get_user(user_id: int):
    # Imaginez que cette ligne fait une requête DB: db.query(DBUser).get(user_id)
    db_user = DBUser(id=user_id, username="jdoe", email="jdoe@example.com", hashed_password="a_very_secret_hash")
    
    # On retourne l'objet ORM directement.
    # Pydantic va lire les attributs .id, .username, .email
    # et ignorer .hashed_password car il n'est pas dans UserOut.
    return db_user
```
Le champ `hashed_password` est automatiquement filtré, ce qui est une excellente pratique de sécurité.

### Zone de Danger {#zone-de-danger-23}
L'erreur la plus fréquente est d'oublier `from_attributes=True`. Si vous ne le mettez pas, Pydantic s'attendra à un dictionnaire. En lui passant un objet, il lèvera une `ValidationError` car il ne saura pas comment accéder aux données, et votre API retournera une erreur 500.

---

### 3 Questions Clés {#3-questions-cles-21}
1.  En utilisant `response_model`, comment spécifiez-vous qu'un endpoint doit retourner un tableau d'objets `Item` ?
2.  Quel est le but principal de l'option de configuration `from_attributes=True` (anciennement `orm_mode`) dans un modèle Pydantic ?
3.  Si un objet de votre base de données contient un champ `password_hash` et que votre `response_model` Pydantic ne le contient pas, que se passe-t-il lorsque vous retournez cet objet depuis un endpoint configuré avec `from_attributes=True` ?

### 3 Exercices Progressifs {#3-exercices-progressifs-21}

**Exercice 1 : Liste de Tâches**
Créez une API simple pour une liste de tâches.
-   Définissez un modèle Pydantic `Task` avec les champs `id` (int), `title` (str), et `completed` (bool, avec une valeur par défaut de `False`).
-   Créez un endpoint `GET /tasks` qui utilise `response_model=List[Task]`.
-   La fonction doit retourner une liste de dictionnaires représentant plusieurs tâches.

<details>
<summary>Découvrir la solution commentée</summary>

```python
from typing import List
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Task(BaseModel):
    id: int
    title: str
    completed: bool = False

mock_db_tasks = [
    {"id": 1, "title": "Buy milk", "completed": True},
    {"id": 2, "title": "Learn FastAPI response models"},
    {"id": 3, "title": "Deploy the app", "completed": False},
]

@app.get("/tasks", response_model=List[Task])
async def get_tasks():
    return mock_db_tasks
```
</details>

**Exercice 2 : Structure d'API Imbriquée**
Créez une structure où un auteur a plusieurs livres.
-   Définissez un modèle `Book` avec `title: str` et `published_year: int`.
-   Définissez un modèle `Author` avec `name: str` et `books: List[Book]`.
-   Créez un endpoint `GET /authors/{author_id}` qui retourne un seul objet `Author` (utilisez `response_model=Author`).
-   La fonction doit retourner un dictionnaire représentant un auteur avec une liste de ses livres.

<details>
<summary>Découvrir la solution commentée</summary>

```python
from typing import List
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Book(BaseModel):
    title: str
    published_year: int

class Author(BaseModel):
    name: str
    books: List[Book]

@app.get("/authors/{author_id}", response_model=Author)
async def get_author(author_id: int):
    # En réalité, vous iriez chercher ces données dans une base de données.
    return {
        "name": "George Orwell",
        "books": [
            {"title": "1984", "published_year": 1949},
            {"title": "Animal Farm", "published_year": 1945},
        ]
    }
```
</details>

**Exercice 3 : Mode ORM pour un Catalogue de Voitures**
Simulez la récupération de données depuis un ORM pour un catalogue de voitures.
-   Créez une classe Python simple `DBCar` avec les attributs `id`, `manufacturer`, `model`, `year`, et `internal_code`.
-   Créez un modèle Pydantic `CarPublic` qui expose uniquement `manufacturer`, `model`, et `year`.
-   Configurez `CarPublic` avec `from_attributes=True`.
-   Créez un endpoint `GET /cars/{car_id}` qui utilise `response_model=CarPublic`.
-   La fonction doit créer une instance de `DBCar` et la retourner directement.

<details>
<summary>Découvrir la solution commentée</summary>

```python
from fastapi import FastAPI
from pydantic import BaseModel, ConfigDict

app = FastAPI()

# Simulation d'un modèle ORM
class DBCar:
    def __init__(self, id, manufacturer, model, year, internal_code):
        self.id = id
        self.manufacturer = manufacturer
        self.model = model
        self.year = year
        self.internal_code = internal_code

# Modèle de réponse Pydantic pour l'API publique
class CarPublic(BaseModel):
    manufacturer: str
    model: str
    year: int

    model_config = ConfigDict(from_attributes=True)

@app.get("/cars/{car_id}", response_model=CarPublic)
async def get_car(car_id: int):
    # Simule une requête à la base de données
    car_from_db = DBCar(
        id=car_id,
        manufacturer="Tesla",
        model="Model S",
        year=2022,
        internal_code="TSLA-S-2022-A1"
    )
    # Le champ 'internal_code' sera automatiquement filtré par Pydantic
    return car_from_db
```
</details>