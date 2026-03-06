---
sidebar_label: "JSON Encoders et Custom Encoders"
sidebar_position: 33
difficulty: "confirmé"
---

# JSON Encoders et Custom Encoders {#json-encoders-et-custom-encoders-33}

Par défaut, FastAPI est extrêmement efficace pour convertir vos données en JSON. Il sait comment gérer les types Python de base (chaînes, nombres, booléens), les listes, les dictionnaires, et surtout, les modèles Pydantic. Il le fait via un utilitaire puissant appelé `jsonable_encoder`.

Cependant, vous rencontrerez inévitablement des types de données que l'encodeur JSON standard ne connaît pas, comme les objets `datetime`, les types `Decimal` pour la finance, ou vos propres classes personnalisées. Tenter de retourner ces types directement résultera en une `TypeError`. Ce chapitre vous montre comment étendre les capacités de sérialisation de FastAPI pour gérer n'importe quel type de données.

## Concept 1 : Le `jsonable_encoder` de FastAPI {#concept-1-le-jsonable_encoder-de-fastapi-33}

### Quoi ? {#quoi-33}
Le `jsonable_encoder` est une fonction fournie par FastAPI (`from fastapi.encoders import jsonable_encoder`) qui a pour but de prendre un objet Python (comme un modèle Pydantic, un objet `datetime`, etc.) et de le convertir en une structure de données compatible avec JSON. C'est-à-dire une structure composée uniquement de `dict`, `list`, `str`, `int`, `float`, `bool`, et `None`.

Il parcourt récursivement votre objet de données et convertit chaque élément. Par exemple, il transformera un modèle Pydantic en `dict` et un objet `datetime` en sa représentation `str` au format ISO 8601.

### Pourquoi ? {#pourquoi-33}
Cette fonction est le moteur de sérialisation interne de FastAPI. Lorsque vous retournez un modèle Pydantic depuis une opération de chemin, FastAPI l'appelle en coulisses avant d'envoyer la réponse JSON. Comprendre son fonctionnement est essentiel pour maîtriser la manière dont vos données sont présentées au client.

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-33}
Bien que FastAPI l'utilise implicitement, vous pouvez l'appeler explicitement pour voir son effet.

**Cas Réel : Convertir un modèle Pydantic avant de le sauvegarder**

```python
from datetime import datetime
from pydantic import BaseModel
from fastapi.encoders import jsonable_encoder

class UserActivity(BaseModel):
    username: str
    last_seen: datetime
    actions_today: int

activity = UserActivity(
    username="alex",
    last_seen=datetime(2024, 7, 15, 10, 30, 0),
    actions_today=5
)

# Convertit l'objet Pydantic en un dictionnaire compatible JSON
json_compatible_data = jsonable_encoder(activity)

print(json_compatible_data)
# Output:
# {
#     'username': 'alex',
#     'last_seen': '2024-07-15T10:30:00',  <-- datetime est devenu une chaîne
#     'actions_today': 5
# }
```
Ce dictionnaire `json_compatible_data` peut maintenant être facilement sérialisé en JSON, par exemple pour être stocké dans une base de données comme MongoDB.

### Zone de Danger {#zone-de-danger-33}
Le `jsonable_encoder` est très flexible mais il a un coût. Pour des objets très volumineux ou profondément imbriqués, son exploration récursive peut introduire une surcharge de performance. Dans des scénarios critiques, des méthodes de sérialisation plus directes comme `model.dict()` (de Pydantic) peuvent être plus rapides, bien que moins complètes (par exemple, elles ne gèrent pas les `datetime` par défaut de la même manière).

---

## Concept 2 : Définir un Encodeur Personnalisé Global {#concept-2-definir-un-encodeur-personnalise-global-33}

### Quoi ? {#quoi-34}
Que se passe-t-il si vous utilisez un type que même `jsonable_encoder` ne connaît pas, comme le type `Decimal` pour des calculs financiers précis ? FastAPI lèvera une `TypeError`. Pour résoudre ce problème, vous pouvez fournir un dictionnaire d'encodeurs personnalisés lors de l'initialisation de votre application `FastAPI`.

### Pourquoi ? {#pourquoi-34}
Cela permet de définir, en un seul endroit, comment sérialiser des types de données spécifiques pour l'ensemble de votre application. C'est propre, centralisé et évite de répéter la logique de conversion dans chaque endpoint.

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-34}
On utilise le paramètre `json_encoders` dans le constructeur `FastAPI`. C'est un dictionnaire qui mappe des types à des fonctions d'encodage.

**Cas Réel : Gérer le type `Decimal` pour des prix de produits**

```python
from decimal import Decimal
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI(
    # Déclare un dictionnaire d'encodeurs
    json_encoders={
        # Pour tout objet de type Decimal, utilise cette fonction lambda pour le convertir
        Decimal: lambda v: f"{v:.2f}" # Convertit en chaîne avec 2 décimales
    }
)

class Product(BaseModel):
    name: str
    price: Decimal

@app.get("/products/{product_id}", response_model=Product)
async def get_product(product_id: int):
    # Dans une vraie app, ces données viendraient d'une base de données
    return Product(name="Super Widget", price=Decimal("199.99"))
```
Sans le `json_encoders`, cet endpoint échouerait. Avec lui, la réponse JSON sera :
```json
{
  "name": "Super Widget",
  "price": "199.99"
}
```

### Zone de Danger {#zone-de-danger-35}
Il est généralement plus sûr de sérialiser les types numériques sensibles comme `Decimal` en chaînes de caractères (`"199.99"`) plutôt qu'en nombres à virgule flottante (`199.99`). Cela évite les problèmes d'imprécision inhérents aux `float`. Cependant, vous devez vous assurer que le client (par exemple, une application frontend) est capable de parser cette chaîne pour la traiter comme un nombre.

---

## Concept 3 : Encodeurs Spécifiques au Modèle Pydantic {#concept-3-encodeurs-specifiques-au-modele-pydantic-33}

### Quoi ? {#quoi-35}
Parfois, vous ne voulez pas qu'un encodeur personnalisé s'applique globalement à toute votre application. Vous pourriez vouloir qu'un type de données soit sérialisé différemment selon le modèle dans lequel il se trouve. Pydantic vous permet de définir des encodeurs directement dans la configuration d'un modèle via sa classe interne `Config`.

### Pourquoi ? {#pourquoi-36}
Cela offre un contrôle plus granulaire. Par exemple, pour un panneau d'administration, vous pourriez vouloir des `datetime` au format ISO, mais pour une API publique, vous préférez des timestamps Unix. En attachant l'encodeur au modèle, la logique de sérialisation voyage avec le modèle, le rendant plus autonome et prévisible.

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-36}
On définit une classe `Config` imbriquée dans le modèle Pydantic et on y ajoute un attribut `json_encoders`.

**Cas Réel : Formats de date différents pour une API publique et un log interne**

```python
from datetime import datetime
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

# Modèle pour l'API publique - veut des timestamps
class PublicEvent(BaseModel):
    event_id: str
    timestamp: datetime

    class Config:
        json_encoders = {
            # Cet encodeur ne s'applique qu'au modèle PublicEvent
            datetime: lambda dt: int(dt.timestamp())
        }

# Modèle pour les logs internes - veut le format ISO
class InternalLog(BaseModel):
    log_id: int
    created_at: datetime
    # Pas de Config, donc utilisera l'encodeur par défaut de FastAPI

@app.get("/events/latest", response_model=PublicEvent)
async def get_latest_event():
    return PublicEvent(event_id="evt_123", timestamp=datetime.now())

@app.get("/logs/latest", response_model=InternalLog)
async def get_latest_log():
    return InternalLog(log_id=999, created_at=datetime.now())
```
-   `/events/latest` renverra `{"event_id": "evt_123", "timestamp": 1678886400}` (un entier).
-   `/logs/latest` renverra `{"log_id": 999, "created_at": "2023-03-15T12:00:00.000Z"}` (une chaîne).

### Zone de Danger {#zone-de-danger-37}
**Priorité des encodeurs :** FastAPI respecte une hiérarchie. Si un encodeur est défini dans la `Config` du modèle Pydantic utilisé dans `response_model`, il aura la priorité. Sinon, FastAPI se rabattra sur les `json_encoders` globaux définis dans le constructeur de `FastAPI`. Cette hiérarchie peut prêter à confusion si vous mélangez les deux approches pour le même type.

---

### 3 Questions Clés {#3-questions-cles-33}
1.  Quel est le rôle de la fonction `jsonable_encoder` et quand FastAPI l'utilise-t-elle automatiquement ?
2.  Comment configureriez-vous votre application FastAPI pour sérialiser tous les objets de type `UUID` en chaînes de caractères (`str(uuid)`) ?
3.  Dans quel scénario préféreriez-vous définir un `json_encoder` dans la `Config` d'un modèle Pydantic plutôt que globalement dans l'application ?

### 3 Exercices Progressifs {#3-exercices-progressifs-33}

**Exercice 1 : Sérialiser un Objet Personnalisé**
1.  Créez une simple classe Python `Point` qui n'hérite pas de Pydantic, avec des attributs `x` et `y`.
2.  Créez un endpoint `GET /point` qui retourne une instance de `Point(x=1, y=2)`.
3.  Sans encodeur personnalisé, l'appel échouera. Ajoutez un encodeur global dans `FastAPI()` pour sérialiser la classe `Point` en un dictionnaire `{"x": p.x, "y": p.y}`.

<details>
<summary>Découvrir la solution commentée</summary>

```python
from fastapi import FastAPI

# 1. Classe Python simple
class Point:
    def __init__(self, x: int, y: int):
        self.x = x
        self.y = y

app = FastAPI(
    json_encoders={
        # 3. L'encodeur pour la classe Point
        Point: lambda p: {"x": p.x, "y": p.y}
    }
)

# 2. L'endpoint qui retourne une instance de Point
@app.get("/point")
async def get_point() -> Point:
    return Point(x=1, y=2)
```
</details>

**Exercice 2 : Formater des Objets `date`**
1.  Créez un modèle Pydantic `Appointment` avec un champ `appointment_date` de type `datetime.date`.
2.  Créez un endpoint qui retourne une instance de `Appointment`.
3.  Par défaut, FastAPI formatera la date en `"YYYY-MM-DD"`.
4.  Implémentez un encodeur global pour que tous les objets `datetime.date` soient plutôt formatés en `strftime("%d/%m/%Y")` (ex: `"15/07/2024"`).

<details>
<summary>Découvrir la solution commentée</summary>

```python
import datetime
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI(
    json_encoders={
        # 4. Encodeur pour formater les objets date
        datetime.date: lambda d: d.strftime("%d/%m/%Y")
    }
)

# 1. Modèle Pydantic
class Appointment(BaseModel):
    appointment_date: datetime.date

# 2. Endpoint
@app.get("/appointment", response_model=Appointment)
async def get_appointment():
    return Appointment(appointment_date=datetime.date.today())
```
</details>

**Exercice 3 : Priorité des Encodeurs avec des Ensembles (Sets)**
1.  Créez un modèle Pydantic `TagList` avec un champ `tags` de type `set[str]`.
2.  `jsonable_encoder` convertit les `set` en `list`.
3.  Définissez un encodeur **global** qui convertit les `set` en chaînes de caractères triées et séparées par des virgules (ex: `"alpha,beta,gamma"`).
4.  Définissez un modèle Pydantic `UniqueUserIDs` avec un champ `user_ids` de type `set[int]`. Dans la `Config` de ce modèle, définissez un encodeur qui convertit les `set` en une `list` triée.
5.  Créez deux endpoints, un pour chaque modèle, et vérifiez que le format de sortie est différent, démontrant ainsi la priorité de l'encodeur du modèle.

<details>
<summary>Découvrir la solution commentée</summary>

```python
from fastapi import FastAPI
from pydantic import BaseModel
from typing import Set

app = FastAPI(
    # 3. Encodeur global pour les ensembles
    json_encoders={
        set: lambda s: ",".join(sorted(str(item) for item in s))
    }
)

# 1. Modèle qui utilisera l'encodeur global
class TagList(BaseModel):
    tags: Set[str]

# 4. Modèle avec son propre encodeur prioritaire
class UniqueUserIDs(BaseModel):
    user_ids: Set[int]
    
    class Config:
        json_encoders = {
            # Cet encodeur s'applique uniquement à ce modèle
            set: lambda s: sorted(list(s))
        }

@app.get("/tags", response_model=TagList)
async def get_tags():
    return TagList(tags={"beta", "gamma", "alpha"}) # Devrait retourner une chaîne

@app.get("/users", response_model=UniqueUserIDs)
async def get_user_ids():
    return UniqueUserIDs(user_ids={10, 5, 25}) # Devrait retourner une liste triée
```
</details>