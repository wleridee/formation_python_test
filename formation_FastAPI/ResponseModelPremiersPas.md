---
sidebar_label: "Modèles de Réponse (Response Model) : Premiers Pas"
sidebar_position: 15
difficulty: "junior"
---

# Modèles de Réponse (Response Model) : Premiers Pas {#modeles-de-reponse-response-model--premiers-pas-15}

Jusqu'à présent, nos endpoints retournaient des dictionnaires Python, et FastAPI se chargeait de les convertir en JSON. C'est simple, mais cela présente deux risques majeurs :
1.  **Fuite de données sensibles :** On peut accidentellement retourner des informations qui ne devraient jamais quitter le serveur, comme des mots de passe hachés, des clés secrètes ou des données personnelles.
2.  **Incohérence de la réponse :** La structure de la réponse peut changer involontairement au fil des modifications du code, cassant les applications clientes qui consomment votre API.

Pour résoudre ces problèmes, FastAPI propose un outil puissant : le `response_model`. Il s'agit d'un modèle Pydantic qui définit la structure exacte des données que votre endpoint est **autorisé** à renvoyer.

```mermaid
graph TD
    subgraph "Logique de votre Fonction"
        A[Objet Python complet<br/>(ex: avec mot de passe)]
    end
    subgraph "Moteur FastAPI"
        B{Décorateur avec<br/>response_model=UserOut}
        C[Filtrage & Validation]
    end
    subgraph Client
        D[Réponse JSON propre<br/>(ex: sans mot de passe)]
    end

    A --> B
    B --> C
    C -- "Ne garde que les champs de UserOut" --> D
```

## Concept 1 : Déclarer un `response_model` {#concept-1-declarer-un-response-model-15}

### Quoi ? {#quoi-15}
Le `response_model` est un paramètre du décorateur de votre opération de chemin (`@app.get`, `@app.post`, etc.). Vous lui assignez une classe de modèle Pydantic, et FastAPI s'assurera que la réponse de votre fonction est filtrée et validée pour correspondre à la structure de ce modèle avant d'être envoyée au client.

### Pourquoi ? {#pourquoi-15}
1.  **Sécurité (Filtrage) :** C'est le cas d'usage le plus important. Vous définissez une "vue publique" de vos données et vous vous assurez que seules ces informations sont exposées.
2.  **Robustesse (Validation) :** FastAPI validera que les données que vous retournez sont du bon type. Si votre fonction essaie de retourner `price: "cher"` au lieu de `price: 100.0`, FastAPI lèvera une erreur côté serveur, vous signalant un bug dans votre code avant qu'il n'atteigne le client.
3.  **Documentation :** La documentation OpenAPI (Swagger UI) affichera le schéma exact de la réponse attendue, rendant votre API beaucoup plus facile à utiliser pour les autres développeurs.

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-15}
On définit deux modèles Pydantic : un pour les données internes (ou les données en entrée) et un autre, plus restrictif, pour la sortie.

**Cas Réel : Créer un utilisateur et retourner des données publiques**
Lors de la création d'un utilisateur, le client envoie un mot de passe. Mais lorsque l'API confirme la création, elle ne doit JAMAIS renvoyer ce mot de passe.

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

# Modèle pour les données en entrée (avec mot de passe)
class UserIn(BaseModel):
    username: str
    password: str
    email: str

# Modèle pour les données en sortie (SANS mot de passe)
class UserOut(BaseModel):
    username: str
    email: str

@app.post("/users/", response_model=UserOut)
async def create_user(user: UserIn):
    # Dans une vraie application, vous hasheriez le mot de passe
    # et enregistreriez l'utilisateur en base de données.
    
    # Ici, nous retournons simplement le dictionnaire de l'objet reçu.
    # FastAPI va intercepter ce dictionnaire...
    user_data = user.dict()
    print(f"Données internes avant filtrage: {user_data}")
    
    # ...et le filtrer pour ne garder que les champs de UserOut.
    return user_data
```
Si vous envoyez une requête `POST` avec `username`, `password`, et `email`, la réponse JSON ne contiendra que `username` et `email`. Le champ `password` aura été automatiquement retiré par FastAPI.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Documentation Swagger UI pour le endpoint POST /users/.
> **Alt Text** : Section "Responses" dans Swagger UI, montrant clairement un schéma de réponse qui inclut "username" et "email", mais pas "password".

### Zone de Danger {#zone-de-danger-15}
Si votre fonction retourne des données qui ne peuvent pas être converties vers le `response_model` (par exemple, un champ requis dans le `response_model` est manquant dans les données que vous retournez), FastAPI lèvera une erreur `500 Internal Server Error`. C'est une bonne chose : cela signifie que votre API ne respecte pas son propre contrat, et vous avez un bug à corriger.

---

## Concept 2 : Utiliser des Listes dans le `response_model` {#concept-2-utiliser-des-listes-dans-le-response-model-15}

### Quoi ? {#quoi-16}
Il est très fréquent qu'un endpoint retourne une liste de ressources (ex: `GET /items/` doit retourner tous les items). Vous pouvez définir cela dans le `response_model` en utilisant les types standards de Python, comme `list[MonModele]`.

### Pourquoi ? {#pourquoi-16}
Cela étend tous les bénéfices du `response_model` (filtrage, validation, documentation) à des collections de données. Chaque élément de la liste sera filtré et validé individuellement, garantissant une réponse cohérente et sécurisée pour l'ensemble de la collection.

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-16}
On importe `list` depuis le module `typing` (pour Python < 3.9, on utilise `List`) et on l'utilise pour "envelopper" notre modèle Pydantic.

**Cas Réel : Lister des produits sans révéler le coût d'achat**

```python
from fastapi import FastAPI
from pydantic import BaseModel
from typing import list # Pour Python 3.9+

app = FastAPI()

class ProductDB(BaseModel):
    name: str
    sale_price: float
    cost_price: float  # Coût d'achat, doit rester secret
    supplier_id: int   # ID interne, doit rester secret

class ProductPublic(BaseModel):
    name: str
    sale_price: float

# La "base de données" interne
fake_products_db = [
    {"name": "Laptop", "sale_price": 1200.0, "cost_price": 750.0, "supplier_id": 101},
    {"name": "Clavier", "sale_price": 75.5, "cost_price": 30.0, "supplier_id": 102},
]

@app.get("/products/", response_model=list[ProductPublic])
async def get_products():
    # Notre fonction retourne la liste complète des données internes.
    # FastAPI va parcourir chaque dictionnaire de la liste
    # et le filtrer selon le modèle ProductPublic.
    return fake_products_db
```
La réponse finale sera une liste JSON où chaque objet n'aura que les clés `name` et `sale_price`.

### Zone de Danger {#zone-de-danger-17}
Assurez-vous que votre fonction retourne bien une **liste** (ou un itérable). Si vous avez `response_model=list[Item]` mais que votre fonction retourne un seul objet `Item` (ou un dictionnaire), FastAPI lèvera une erreur de validation. Le type de données retourné doit correspondre à la structure globale du `response_model`.

---

### 3 Questions Clés {#3-questions-cles-15}
1.  Citez les trois avantages principaux de l'utilisation de `response_model`.
2.  Vous avez un modèle `User` interne avec les champs `id`, `name`, et `password_hash`. Comment définiriez-vous un `response_model` pour exposer publiquement uniquement l'`id` et le `name` ?
3.  Si un endpoint est censé retourner une liste de livres, quelle serait la syntaxe correcte pour le paramètre `response_model` en utilisant un modèle Pydantic nommé `Book` ?

### 3 Exercices Progressifs {#3-exercices-progressifs-15}

**Exercice 1 : Profil Utilisateur Public**
Créez un endpoint `GET /users/me`. En interne, simulez un objet utilisateur contenant `username`, `email`, `full_name`, et `api_key`. Créez un `response_model` qui ne retourne que `username` et `full_name`.

<details>
<summary>Découvrir la solution commentée</summary>

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

# Le modèle pour la réponse publique
class UserProfile(BaseModel):
    username: str
    full_name: str

# Simulation des données complètes de l'utilisateur connecté
current_user_data = {
    "username": "jdoe",
    "email": "john.doe@example.com",
    "full_name": "John Doe",
    "api_key": "SECRET_KEY_12345"
}

@app.get("/users/me", response_model=UserProfile)
async def get_current_user_profile():
    # On retourne toutes les données, FastAPI se charge du filtrage.
    return current_user_data
```
</details>

**Exercice 2 : Liste de Tâches Simplifiée**
Créez un endpoint `GET /todos` qui retourne une liste de tâches. Chaque tâche dans votre "DB" interne est un dictionnaire avec `id`, `title`, `completed`, et `owner_id`. L'API ne doit exposer publiquement que `id`, `title`, et `completed`.

<details>
<summary>Découvrir la solution commentée</summary>

```python
from fastapi import FastAPI
from pydantic import BaseModel
from typing import list

app = FastAPI()

class TodoPublic(BaseModel):
    id: int
    title: str
    completed: bool

fake_todos_db = [
    {"id": 1, "title": "Acheter du lait", "completed": False, "owner_id": 101},
    {"id": 2, "title": "Appeler le support", "completed": True, "owner_id": 101},
    {"id": 3, "title": "Finir le projet", "completed": False, "owner_id": 102},
]

@app.get("/todos", response_model=list[TodoPublic])
async def get_all_todos():
    return fake_todos_db
```
</details>

**Exercice 3 : Création et Retour d'un Article**
Créez un endpoint `POST /articles`.
-   Il doit accepter en entrée un modèle `ArticleIn` avec `title` et `content`.
-   En interne, votre fonction doit simuler la création de l'article en ajoutant un `id` et une `publication_date`.
-   L'endpoint doit utiliser un `response_model` nommé `ArticleOut` qui retourne `id`, `title`, `content` et `publication_date`.

<details>
<summary>Découvrir la solution commentée</summary>

```python
import datetime
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

# Modèle pour les données d'entrée
class ArticleIn(BaseModel):
    title: str
    content: str

# Modèle pour les données de sortie
class ArticleOut(BaseModel):
    id: int
    title: str
    content: str
    publication_date: datetime.date

@app.post("/articles", response_model=ArticleOut)
async def create_article(article: ArticleIn):
    # Simuler la création en base de données
    new_article_data = article.dict()
    new_article_data["id"] = 123  # ID simulé
    new_article_data["publication_date"] = datetime.date.today()
    
    # Retourner le dictionnaire complet.
    # FastAPI va le valider et le convertir en JSON selon ArticleOut.
    return new_article_data
```
*Cet exercice montre que le `response_model` sert aussi à garantir que les données générées par le serveur (comme l'ID et la date) sont bien incluses dans la réponse finale.*
</details>