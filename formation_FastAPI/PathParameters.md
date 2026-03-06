---
sidebar_label: "Path Parameters : Introduction et Typage"
sidebar_position: 6
difficulty: "junior"
---

# Path Parameters : Introduction et Typage {#path-parameters-introduction-et-typage-6}

Dans le chapitre précédent, nous avons découvert les paramètres de chemin comme un moyen de créer des URL dynamiques. Ce chapitre approfondit ce concept fondamental. Nous allons formaliser leur utilisation, voir comment le typage de données nous protège contre les erreurs et explorer des types plus avancés pour une validation encore plus puissante.

Maîtriser les paramètres de chemin est essentiel, car ils forment la base de la création d'API RESTful où chaque ressource est accessible via une URL unique.

## Concept 1 : Le Pouvoir de la Validation par le Typage {#concept-1-le-pouvoir-de-la-validation-par-le-typage-6}

### Quoi ? {#quoi-6}
Un paramètre de chemin est une variable extraite directement de l'URL. En utilisant les annotations de type (type hints) de Python dans la signature de votre fonction, vous donnez des instructions à FastAPI sur le type de données que vous attendez.

### Pourquoi ? {#pourquoi-6}
L'utilisation explicite des types (`int`, `float`, `bool`, `str`) offre trois avantages majeurs :
1.  **Validation Automatique** : FastAPI rejette les requêtes qui ne correspondent pas au type attendu avec une erreur HTTP 422 claire. Vous n'avez pas à écrire ce code de validation vous-même.
2.  **Conversion de Données** : Si la validation réussit, FastAPI convertit la valeur de l'URL (qui est toujours une chaîne de caractères) dans le type Python que vous avez spécifié. `"123"` devient l'entier `123`.
3.  **Documentation Améliorée** : Votre documentation interactive (`/docs`) indiquera précisément le type de données attendu pour chaque paramètre, rendant votre API plus facile à utiliser pour les autres développeurs.

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-6}
Imaginons une API pour une application météo qui récupère les prévisions pour une localisation donnée en utilisant sa latitude et sa longitude.

```python
from fastapi import FastAPI

app = FastAPI()

# Les types float sont utilisés pour la latitude et la longitude
@app.get("/weather/{latitude}/{longitude}")
def get_weather(latitude: float, longitude: float):
    # Dans une vraie application, vous utiliseriez ces valeurs
    # pour interroger un service météo.
    return {
        "location": {"lat": latitude, "lon": longitude},
        "forecast": "Ensoleillé"
    }
```
-   Si vous appelez `/weather/48.8566/2.3522` (Paris), la fonction recevra `latitude=48.8566` et `longitude=2.3522` en tant que `float`.
-   Si vous appelez `/weather/paris/france`, FastAPI retournera une erreur 422 car `"paris"` ne peut pas être converti en `float`.

### Zone de Danger {#zone-de-danger-6}
Même si le typage est optionnel (`def get_weather(latitude, longitude):` fonctionnerait), ne pas l'utiliser est une mauvaise pratique. Sans type hints, tous les paramètres seront des chaînes de caractères (`str`), et vous perdrez tous les avantages de validation et de conversion automatique de FastAPI. Prenez l'habitude de toujours typer vos paramètres.

---

## Concept 2 : Restreindre les Valeurs avec `Enum` {#concept-2-restreindre-les-valeurs-avec-enum-6}

### Quoi ? {#quoi-7}
Parfois, un paramètre ne doit accepter qu'une liste prédéfinie de valeurs (par exemple, les tailles d'un vêtement : "S", "M", "L", "XL"). Pour ce cas, Python nous offre un outil parfait : les énumérations (`Enum`). FastAPI s'intègre nativement avec les `Enum` pour la validation.

### Pourquoi ? {#pourquoi-7}
Utiliser une `Enum` est bien plus robuste qu'un simple `str`. Cela garantit que seules les valeurs que vous avez explicitement autorisées sont acceptées par l'API. De plus, la documentation interactive générée affichera une liste déroulante avec les choix possibles, ce qui est extrêmement pratique pour les utilisateurs de votre API.

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-7}
Imaginons une API qui retourne des informations sur des types d'utilisateurs spécifiques : `standard`, `admin`, ou `guest`.

```python
from enum import Enum
from fastapi import FastAPI

# 1. On définit une classe qui hérite de str et Enum.
#    Hériter de str est important pour que FastAPI le traite correctement.
class UserType(str, Enum):
    STANDARD = "standard"
    ADMIN = "admin"
    GUEST = "guest"

app = FastAPI()

# 2. On utilise UserType comme type hint.
@app.get("/users/type/{user_type}")
def get_user_type_info(user_type: UserType):
    return {"user_type": user_type, "permissions": f"Permissions for {user_type.value}"}
```
Maintenant :
-   Un appel à `/users/type/admin` fonctionnera. `user_type` sera `UserType.ADMIN`.
-   Un appel à `/users/type/moderator` échouera avec une erreur 422, car `moderator` n'est pas une valeur valide de l'énumération `UserType`.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Documentation Swagger UI montrant une liste déroulante pour le paramètre `user_type`.
> **Alt Text** : Interface Swagger UI avec un champ "user_type" qui est une liste déroulante contenant les options "standard", "admin", et "guest".

### Zone de Danger {#zone-de-danger-8}
N'oubliez pas de faire hériter votre `Enum` de `str` (`class MyEnum(str, Enum):`). Si vous oubliez `str`, la validation fonctionnera, mais la sérialisation en JSON et la documentation pourraient ne pas se comporter comme prévu. La casse est importante : les valeurs dans l'URL doivent correspondre exactement à celles définies dans l'énumération.

---

## Concept 3 : Capturer des Chemins avec `:path` {#concept-3-capturer-des-chemins-avec-path-6}

### Quoi ? {#quoi-9}
Et si votre paramètre de chemin devait lui-même contenir des slashes `/` ? C'est le cas, par exemple, si vous voulez capturer un chemin de fichier comme `dossier/sous_dossier/fichier.txt`. Un paramètre de chemin standard s'arrêterait au premier slash. Pour capturer la totalité du chemin, FastAPI propose une syntaxe spéciale.

### Pourquoi ? {#pourquoi-10}
C'est indispensable pour les API qui doivent interagir avec des ressources dont l'identifiant est hiérarchique, comme des fichiers sur un système de fichiers, des clés dans un stockage d'objets, ou des routes de pages dans un CMS.

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-9}
La syntaxe consiste à ajouter `:path` après le nom du paramètre dans le décorateur. Le type hint de la fonction reste `str`.

**Cas Réel : API de prévisualisation de fichiers**
```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/files/{file_path:path}")
def read_file(file_path: str):
    return {"requested_file_path": file_path}
```
Si vous faites une requête à `/files/home/user/documents/report.pdf` :
-   Le paramètre `file_path` contiendra la chaîne complète : `"home/user/documents/report.pdf"`.

### Zone de Danger {#zone-de-danger-10}
Le convertisseur `:path` est "gourmand" : il capture tout ce qui suit dans l'URL. Pour cette raison, un paramètre de chemin contenant `:path` **doit obligatoirement être le dernier élément du chemin de l'URL**. Une route comme `@app.get("/files/{file_path:path}/details")` ne fonctionnera pas, car `:path` capturera aussi `/details`.

---

### 3 Questions Clés {#3-questions-cles-6}
1.  En dehors de la validation, quel autre avantage majeur apporte l'ajout d'un `type hint` à un paramètre de chemin ?
2.  Dans quel scénario est-il préférable d'utiliser une `Enum` plutôt qu'un `str` pour typer un paramètre de chemin ? Donnez un exemple concret.
3.  Vous devez créer un endpoint qui récupère un article de blog basé sur son URL complète, par exemple `/blog/2023/10/my-awesome-post`. Comment définiriez-vous le paramètre de chemin pour capturer `2023/10/my-awesome-post` ?

### 3 Exercices Progressifs {#3-exercices-progressifs-6}

**Exercice 1 : API de Conversion de Monnaie**
Créez un endpoint `GET` sur `/convert/{amount_eur}/{target_currency}`.
-   `amount_eur` doit être un nombre à virgule (`float`).
-   `target_currency` doit être une chaîne de caractères (`str`).
La fonction doit simuler une conversion (par exemple, en multipliant par un taux fixe) et retourner le montant converti.

<details>
<summary>Découvrir la solution commentée</summary>

```python
from fastapi import FastAPI

app = FastAPI()

# Taux de change fictifs
CONVERSION_RATES = {
    "USD": 1.07,
    "JPY": 160.5,
    "GBP": 0.85
}

@app.get("/convert/{amount_eur}/{target_currency}")
def convert_currency(amount_eur: float, target_currency: str):
    # On met la devise en majuscules pour être plus flexible (usd -> USD)
    currency = target_currency.upper()

    if currency not in CONVERSION_RATES:
        return {"error": f"Currency {currency} not supported."}

    converted_amount = round(amount_eur * CONVERSION_RATES[currency], 2)

    return {
        "source": "EUR",
        "target": currency,
        "amount_eur": amount_eur,
        "converted_amount": converted_amount
    }
```
*Testez avec `http://127.0.0.1:8000/convert/50.75/usd`.*
</details>

**Exercice 2 : API de Statut de Service**
Créez un endpoint `GET` `/services/{service_name}/status` pour surveiller l'état de différents services.
-   Le `service_name` ne peut être que l'un des suivants : `database`, `api`, `payment`. Utilisez une `Enum` pour forcer cette contrainte.
La fonction doit retourner un statut fictif basé sur le nom du service.

<details>
<summary>Découvrir la solution commentée</summary>

```python
from enum import Enum
from fastapi import FastAPI

class ServiceName(str, Enum):
    DATABASE = "database"
    API = "api"
    PAYMENT = "payment"

app = FastAPI()

@app.get("/services/{service_name}/status")
def get_service_status(service_name: ServiceName):
    status = "operational"
    # On peut utiliser une logique différente pour chaque service
    if service_name is ServiceName.PAYMENT:
        status = "degraded_performance"

    return {"service": service_name, "status": status}
```
*Testez avec `/services/database/status` et `/services/payment/status`. Essayez aussi avec `/services/cache/status` pour voir l'erreur de validation.*
</details>

**Exercice 3 : Serveur de Fichiers Statiques**
Créez un endpoint `GET` `/static/{file_path:path}` qui simule le service d'un fichier.
La fonction doit accepter un chemin de fichier, extraire son extension (par exemple, "txt", "jpg", "css") et retourner le chemin complet ainsi que l'extension détectée.

<details>
<summary>Découvrir la solution commentée</summary>

```python
import os
from fastapi import FastAPI

app = FastAPI()

@app.get("/static/{file_path:path}")
def serve_static_file(file_path: str):
    # os.path.splitext est un moyen facile de séparer le nom du fichier et l'extension.
    # On récupère la deuxième partie ([1]) et on enlève le point de début (.[1:]).
    _, extension_with_dot = os.path.splitext(file_path)
    extension = extension_with_dot[1:] if extension_with_dot else "no_extension"

    return {
        "full_path": file_path,
        "file_extension": extension
    }
```
*Testez avec une URL complexe comme `/static/css/themes/dark/main.style.css`. Le résultat devrait être `{"full_path": "css/themes/dark/main.style.css", "file_extension": "css"}`.*
</details>