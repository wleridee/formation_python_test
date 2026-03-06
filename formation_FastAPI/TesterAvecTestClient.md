---
sidebar_label: "Tester une Application FastAPI avec TestClient"
sidebar_position: 32
difficulty: "confirmé"
---

# Tester une Application FastAPI avec TestClient {#tester-une-application-fastapi-avec-testclient-32}

Écrire du code est une chose ; s'assurer qu'il fonctionne comme prévu et continue de fonctionner après des modifications en est une autre. Les tests automatisés sont une pierre angulaire du développement logiciel moderne. FastAPI rend les tests d'API simples, rapides et robustes grâce à son `TestClient`.

Le `TestClient` est basé sur la bibliothèque `httpx` et interagit avec votre application FastAPI directement en mémoire, sans avoir besoin d'un serveur réseau en cours d'exécution. C'est ce qui le rend incroyablement rapide.

**Prérequis :** Vous devez installer `pytest` et `httpx`.
```bash
pip install pytest httpx
```

## Concept 1 : Le `TestClient` et la Structure de Base des Tests {#concept-1-le-testclient-et-la-structure-de-base-des-tests-32}

### Quoi ? {#quoi-32}
Le `TestClient` est une classe qui prend votre instance d'application FastAPI comme argument et vous fournit une interface pour lui envoyer des requêtes HTTP programmatiques (comme `get`, `post`, `put`, etc.). Chaque appel de méthode sur le client retourne un objet de réponse qui contient le statut, les en-têtes et le corps de la réponse.

### Pourquoi ? {#pourquoi-32}
-   **Rapidité :** Les tests s'exécutent sans la surcharge d'un réseau, ce qui les rend des ordres de grandeur plus rapides que les tests de bout en bout.
-   **Isolation :** Chaque test peut s'exécuter dans un environnement contrôlé et isolé.
-   **Intégration :** Il s'intègre parfaitement avec des frameworks de test comme `pytest` pour l'organisation des tests, les assertions et les fixtures.
-   **Fiabilité :** Évite les problèmes liés à la configuration du réseau ou aux ports occupés.

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-32}
1.  **Créez votre application `main.py` :**
    ```python
    # main.py
    from fastapi import FastAPI

    app = FastAPI()

    @app.get("/")
    def read_root():
        return {"message": "Hello World"}
    ```

2.  **Créez votre fichier de test `test_main.py` :**
    ```python
    # test_main.py
    from fastapi.testclient import TestClient
    from .main import app  # Importe l'objet app depuis main.py

    # Instancie le TestClient
    client = TestClient(app)

    # Définit une fonction de test (doit commencer par "test_")
    def test_read_root():
        # Envoie une requête GET à l'URL "/"
        response = client.get("/")
        
        # Vérifie que le code de statut est 200 OK
        assert response.status_code == 200
        
        # Vérifie que le corps de la réponse est le JSON attendu
        assert response.json() == {"message": "Hello World"}
    ```

3.  **Lancez les tests depuis votre terminal :**
    ```bash
    pytest
    ```
    Pytest découvrira et exécutera automatiquement le test.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Sortie du terminal après avoir exécuté `pytest`.
> **Alt Text** : Capture d'écran montrant une sortie de pytest avec "1 passed in X.XXs", indiquant que le test a réussi.

### Zone de Danger {#zone-de-danger-32}
Il est crucial de séparer votre instance `app` du code qui lance le serveur. N'incluez jamais `uvicorn.run(app, ...)` dans le même fichier `main.py` que vous importez pour les tests. `TestClient` a besoin d'importer l'objet `app`, pas de démarrer un serveur Uvicorn.

---

## Concept 2 : Tester les Paramètres, En-têtes et Corps de Requête {#concept-2-tester-les-parametres-en-tetes-et-corps-de-requete-32}

### Quoi ? {#quoi-33}
Les applications réelles ont des endpoints qui acceptent des paramètres de chemin et de requête, qui attendent des en-têtes spécifiques (comme pour l'authentification) ou qui reçoivent un corps de requête (JSON). Le `TestClient` fournit des arguments pour simuler tous ces cas.

### Pourquoi ? {#pourquoi-33}
Tester ces interactions est essentiel pour valider :
-   La logique métier qui dépend des entrées utilisateur.
-   La validation des données via Pydantic.
-   Les mécanismes de sécurité comme la vérification des clés d'API ou des tokens JWT.

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-33}
Considérons une application plus complexe :
```python
# main.py
from fastapi import FastAPI, Header, HTTPException
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    price: float

app = FastAPI()

@app.get("/items/{item_id}")
def read_item(item_id: int, q: str | None = None):
    return {"item_id": item_id, "q": q}

@app.post("/items/")
def create_item(item: Item, x_token: str = Header(...)):
    if x_token != "fake-super-secret-token":
        raise HTTPException(status_code=400, detail="X-Token header invalid")
    return {"item": item, "x_token": x_token}
```

**Fichier de test `test_main.py` :**
```python
# test_main.py
from fastapi.testclient import TestClient
from .main import app

client = TestClient(app)

def test_read_item_with_query_param():
    response = client.get("/items/42?q=testquery")
    assert response.status_code == 200
    assert response.json() == {"item_id": 42, "q": "testquery"}

def test_create_item_success():
    response = client.post(
        "/items/",
        headers={"X-Token": "fake-super-secret-token"},
        json={"name": "Super Gadget", "price": 99.99},
    )
    assert response.status_code == 200
    assert response.json() == {
        "item": {"name": "Super Gadget", "price": 99.99},
        "x_token": "fake-super-secret-token",
    }

def test_create_item_invalid_token():
    response = client.post(
        "/items/",
        headers={"X-Token": "invalid-token"},
        json={"name": "Gadget", "price": 10.0},
    )
    assert response.status_code == 400

def test_create_item_invalid_payload():
    # Envoie un corps de requête qui ne respecte pas le modèle Pydantic
    response = client.post(
        "/items/",
        headers={"X-Token": "fake-super-secret-token"},
        json={"name": "Incomplete"}, # "price" est manquant
    )
    assert response.status_code == 422 # Erreur de validation
```

### Zone de Danger {#zone-de-danger-34}
Pour envoyer un corps de requête JSON, utilisez toujours le paramètre `json=...`. N'utilisez pas `data=...` avec une chaîne JSON encodée manuellement. L'utilisation de `json=` garantit que `TestClient` (via `httpx`) définit automatiquement l'en-tête `Content-Type: application/json` et gère correctement la sérialisation, ce qui est crucial pour que FastAPI décode le corps de la requête.

---

## Concept 3 : Surcharger les Dépendances (`dependency_overrides`) {#concept-3-surcharger-les-dependances-dependency_overrides-32}

### Quoi ? {#quoi-34}
C'est la fonctionnalité la plus puissante pour les tests dans FastAPI. `app.dependency_overrides` est un dictionnaire qui vous permet de remplacer une fonction de dépendance par une autre fonction pendant la durée d'un test.

### Pourquoi ? {#pourquoi-35}
-   **Isolation des services externes :** Vous pouvez remplacer une dépendance qui se connecte à une base de données, à un service de cache (Redis) ou à une API tierce par une version "mock" qui renvoie des données contrôlées. Cela rend vos tests indépendants, rapides et déterministes.
-   **Simulation de scénarios :** Vous pouvez facilement simuler différents états, comme un utilisateur authentifié, un utilisateur administrateur, ou une base de données vide, en faisant en sorte que votre dépendance surchargée renvoie la valeur appropriée.

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-35}
**Cas Réel : Tester un endpoint protégé en simulant un utilisateur authentifié**

1.  **`main.py` avec une dépendance d'authentification :**
    ```python
    # main.py
    from fastapi import FastAPI, Depends

    # Dépendance factice
    async def get_current_user():
        # Dans une vraie app, cela lirait un token et retournerait un user de la DB
        return {"username": "realuser"}

    app = FastAPI()

    @app.get("/users/me")
    def read_current_user(current_user: dict = Depends(get_current_user)):
        return current_user
    ```

2.  **`test_main.py` avec une surcharge de dépendance :**
    ```python
    # test_main.py
    from fastapi.testclient import TestClient
    from .main import app, get_current_user

    # Fonction de dépendance de remplacement
    async def override_get_current_user():
        return {"username": "fakeuser"}

    # Applique la surcharge
    app.dependency_overrides[get_current_user] = override_get_current_user

    client = TestClient(app)

    def test_read_current_user():
        response = client.get("/users/me")
        assert response.status_code == 200
        assert response.json() == {"username": "fakeuser"}

    # N'oubliez pas de nettoyer les surcharges si vous avez d'autres tests
    # app.dependency_overrides = {}
    ```

### Zone de Danger {#zone-de-danger-36}
Les surcharges de dépendances sont globales à l'objet `app`. Si vous ne les nettoyez pas après un test, elles peuvent "fuir" et affecter d'autres tests de manière inattendue. La meilleure pratique consiste à utiliser une fixture `pytest` pour appliquer et nettoyer automatiquement la surcharge avant et après chaque test, garantissant ainsi une isolation parfaite.

---

### 3 Questions Clés {#3-questions-cles-32}
1.  Le `TestClient` de FastAPI a-t-il besoin de lancer un serveur web sur un port pour fonctionner ? Pourquoi est-ce un avantage ?
2.  Quelle est la différence entre passer des données avec le paramètre `json=` et `data=` dans une requête `POST` avec `TestClient` ?
3.  Quel est le principal cas d'utilisation de `app.dependency_overrides` et pourquoi est-ce crucial pour des tests robustes ?

### 3 Exercices Progressifs {#3-exercices-progressifs-32}

**Exercice 1 : Tester Plusieurs Endpoints Simples**
Créez une application FastAPI avec deux endpoints : `GET /` qui retourne `{"status": "ok"}` et `GET /info` qui retourne `{"app_name": "My API"}`. Écrivez deux fonctions de test distinctes dans un fichier `test_app.py`, une pour chaque endpoint, et vérifiez leurs réponses.

<details>
<summary>Découvrir la solution commentée</summary>

**`app.py`**
```python
from fastapi import FastAPI
app = FastAPI()
@app.get("/")
def root(): return {"status": "ok"}
@app.get("/info")
def info(): return {"app_name": "My API"}
```
**`test_app.py`**
```python
from fastapi.testclient import TestClient
from .app import app
client = TestClient(app)

def test_root():
    response = client.get("/")
    assert response.status_code == 200
    assert response.json() == {"status": "ok"}

def test_info():
    response = client.get("/info")
    assert response.status_code == 200
    assert response.json() == {"app_name": "My API"}
```
</details>

**Exercice 2 : Tester la Validation d'Erreurs (422)**
Créez un endpoint `POST /notes` qui accepte un corps JSON avec un champ `text` (chaîne, longueur min 3) et un champ `is_done` (booléen). Écrivez deux tests :
1.  Un test "happy path" qui envoie une note valide et vérifie la réponse 200.
2.  Un test qui envoie une note avec un champ `text` trop court (`"ab"`) et vérifie que le code de statut est bien 422 (Unprocessable Entity).

<details>
<summary>Découvrir la solution commentée</summary>

**`app.py`**
```python
from fastapi import FastAPI
from pydantic import BaseModel, Field
app = FastAPI()

class Note(BaseModel):
    text: str = Field(..., min_length=3)
    is_done: bool
    
@app.post("/notes")
def create_note(note: Note):
    return note
```
**`test_app.py`**
```python
from fastapi.testclient import TestClient
from .app import app
client = TestClient(app)

def test_create_note_success():
    response = client.post("/notes", json={"text": "My valid note", "is_done": False})
    assert response.status_code == 200
    assert response.json() == {"text": "My valid note", "is_done": False}

def test_create_note_invalid_text():
    response = client.post("/notes", json={"text": "ab", "is_done": True})
    assert response.status_code == 422
```
</details>

**Exercice 3 : Tester avec une Dépendance Surchargée (Fixture Pytest)**
Créez un endpoint `GET /settings` qui dépend d'une fonction `get_settings` pour retourner des configurations. Écrivez un test qui surcharge `get_settings` pour retourner des valeurs de test spécifiques en utilisant une fixture `pytest` pour une gestion propre.

<details>
<summary>Découvrir la solution commentée</summary>

**`app.py`**
```python
from fastapi import FastAPI, Depends
app = FastAPI()

class Settings:
    def __init__(self, db_url: str = "prod_db_url", api_key: str = "prod_key"):
        self.db_url = db_url
        self.api_key = api_key

def get_settings():
    return Settings()

@app.get("/settings")
def get_app_settings(settings: Settings = Depends(get_settings)):
    return settings
```
**`test_app.py`**
```python
import pytest
from fastapi.testclient import TestClient
from .app import app, get_settings, Settings

# Fixture pytest pour gérer la surcharge
@pytest.fixture
def test_client_with_override():
    # Dépendance de remplacement
    def override_get_settings():
        return Settings(db_url="test_db_url", api_key="test_key")

    # Applique la surcharge
    app.dependency_overrides[get_settings] = override_get_settings
    
    # Crée le client et le fournit au test
    client = TestClient(app)
    yield client
    
    # Nettoie la surcharge après l'exécution du test
    app.dependency_overrides.clear()

# Le test utilise la fixture en l'acceptant comme argument
def test_get_settings_with_override(test_client_with_override):
    response = test_client_with_override.get("/settings")
    assert response.status_code == 200
    assert response.json() == {"db_url": "test_db_url", "api_key": "test_key"}
```
</details>