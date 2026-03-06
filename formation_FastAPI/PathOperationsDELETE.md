---
sidebar_label: "Path Operations : Requêtes DELETE pour la Suppression"
sidebar_position: 12
difficulty: "junior"
---

# Path Operations : Requêtes DELETE pour la Suppression {#path-operations-requetes-delete-pour-la-suppression-12}

Nous avons couvert la création (`POST`), la lecture (`GET`), et la mise à jour (`PUT`) de ressources. La dernière pièce du puzzle des opérations CRUD (Create, Read, Update, Delete) est la suppression. Pour cela, la méthode HTTP standard est `DELETE`.

Une requête `DELETE` est envoyée à l'URL exacte de la ressource que l'on souhaite supprimer. C'est une opération directe et puissante, qui, comme son nom l'indique, est généralement irréversible. FastAPI fournit le décorateur `@app.delete()` pour gérer ces requêtes de manière simple et sécurisée.

## Concept 1 : Le Décorateur `@app.delete()` et sa Sémantique {#concept-1-le-decorateur-appdelete-et-sa-semantique-12}

### Quoi ? {#quoi-12}
Le décorateur `@app.delete("/items/{item_id}")` déclare une opération de chemin qui répond aux requêtes HTTP `DELETE`. Il est utilisé pour les endpoints dont le seul but est de supprimer une ressource spécifique, identifiée par un ou plusieurs paramètres de chemin.

### Pourquoi ? {#pourquoi-12}
Utiliser `DELETE` est la manière sémantiquement correcte et attendue de supprimer une ressource dans une API REST.
-   **Spécificité :** L'URL cible directement la ressource à supprimer (ex: `/users/123`), rendant l'intention de la requête sans équivoque.
-   **Idempotence :** Tout comme `PUT`, `DELETE` est une opération idempotente. Supprimer une ressource une fois a pour résultat sa suppression. La supprimer une deuxième fois (ou plus) ne change pas l'état du système (la ressource est toujours "non existante"). Le serveur peut retourner une erreur 404, mais l'état final est le même.
-   **Absence de corps :** En général, une requête `DELETE` n'a pas de corps (Request Body). Toutes les informations nécessaires sont dans l'URL.

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-12}
La syntaxe est très simple : on décore une fonction avec `@app.delete()` et on lui passe le paramètre de chemin qui identifie la ressource.

**Cas Réel : Supprimer un article d'une base de données**
Imaginons une simple base de données d'articles sous forme de dictionnaire. Nous voulons un endpoint pour supprimer un article par son ID.

```python
from fastapi import FastAPI, HTTPException

app = FastAPI()

fake_items_db = {
    1: {"name": "Article A"},
    2: {"name": "Article B"},
    3: {"name": "Article C"},
}

@app.delete("/items/{item_id}")
async def delete_item(item_id: int):
    if item_id not in fake_items_db:
        # Si l'item n'existe pas, on ne peut pas le supprimer.
        raise HTTPException(status_code=404, detail="Item not found")
    
    # On retire l'item de notre "DB"
    deleted_item = fake_items_db.pop(item_id)
    
    return {"message": "Item deleted successfully", "deleted_item": deleted_item}
```

### Zone de Danger {#zone-de-danger-12}
**Les suppressions sont souvent définitives !** Dans une application réelle, la suppression d'une ressource de la base de données est une action destructive. Il est absolument crucial de protéger les endpoints `DELETE` par une authentification et une autorisation robustes pour s'assurer que seul un utilisateur habilité peut supprimer une ressource.

---

## Concept 2 : Gérer les Codes de Statut de Réponse {#concept-2-gerer-les-codes-de-statut-de-reponse-12}

### Quoi ? {#quoi-13}
Que doit retourner un endpoint `DELETE` en cas de succès ? Il y a deux approches communes, chacune avec son propre code de statut HTTP :
1.  `200 OK` avec le corps de la ressource supprimée.
2.  `204 No Content` avec un corps de réponse vide.

### Pourquoi ? {#pourquoi-13}
Le choix dépend du besoin du client de l'API :
-   **Retourner la ressource (200 OK) :** C'est utile si le client a besoin d'afficher un message de confirmation ("Vous avez supprimé l'article 'Article B'") ou de réaliser une action de type "undo" (annuler). C'est l'approche que nous avons utilisée dans le premier exemple.
-   **Ne rien retourner (204 No Content) :** C'est la méthode la plus pure et la plus courante. Le client demande une suppression, le serveur la réalise. Si le serveur répond `204`, cela signifie "Opération réussie. Je n'ai rien d'autre à ajouter." C'est efficace car aucune donnée n'est transférée dans la réponse.

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-13}
Pour renvoyer un statut `204`, vous pouvez le spécifier directement dans le décorateur et ne rien retourner (`None`) dans la fonction.

**Cas Réel : Suppression d'une note avec une réponse 204**

```python
from fastapi import FastAPI, HTTPException, status

app = FastAPI()

fake_notes_db = {1: "Note 1", 2: "Note 2"}

# On spécifie le code de statut de succès par défaut pour cet endpoint
@app.delete("/notes/{note_id}", status_code=status.HTTP_204_NO_CONTENT)
async def delete_note(note_id: int):
    if note_id not in fake_notes_db:
        raise HTTPException(status_code=404, detail="Note not found")
    
    del fake_notes_db[note_id]
    
    # On retourne None, FastAPI se charge de créer une réponse vide
    # avec le statut 204.
    return None
```
En définissant `status_code=status.HTTP_204_NO_CONTENT`, vous rendez votre API plus explicite et conforme aux standards.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Documentation Swagger UI pour le endpoint `DELETE /notes/{note_id}`.
> **Alt Text** : Interface Swagger UI montrant le endpoint DELETE avec son paramètre `note_id`. La section "Responses" indique "204 No Content" comme réponse de succès.

### Zone de Danger {#zone-de-danger-14}
Si vous déclarez un `status_code=204` mais que votre fonction retourne des données (par exemple, un dictionnaire), FastAPI sera assez intelligent pour ne pas inclure de corps dans la réponse, car le protocole HTTP l'interdit pour ce statut. Cependant, cela peut prêter à confusion. La bonne pratique est de s'assurer que votre fonction retourne `None` ou `Response(status_code=204)` si vous visez un statut `204`.

---

### 3 Questions Clés {#3-questions-cles-12}
1.  Quel est le rôle du décorateur `@app.delete()` dans FastAPI ?
2.  Expliquez pourquoi une opération `DELETE` est considérée comme idempotente.
3.  Quel est le code de statut HTTP le plus approprié pour une opération `DELETE` réussie qui n'a pas besoin de renvoyer d'informations au client, et pourquoi ?

### 3 Exercices Progressifs {#3-exercices-progressifs-12}

**Exercice 1 : Suppression Simple d'un Commentaire**
Créez un endpoint `DELETE /comments/{comment_id}`.
-   Utilisez un dictionnaire comme fausse base de données de commentaires.
-   L'endpoint doit supprimer le commentaire identifié par `comment_id`.
-   S'il n'existe pas, levez une `HTTPException` 404.
-   En cas de succès, retournez un simple message de confirmation comme `{"status": "Comment deleted"}`.

<details>
<summary>Découvrir la solution commentée</summary>

```python
from fastapi import FastAPI, HTTPException

app = FastAPI()

fake_comments_db = {
    101: "C'est un super commentaire.",
    102: "Je ne suis pas d'accord.",
    103: "Intéressant.",
}

@app.delete("/comments/{comment_id}")
async def delete_comment(comment_id: int):
    if comment_id not in fake_comments_db:
        raise HTTPException(status_code=404, detail="Comment not found")
    
    # pop() est utile car il retire l'élément et le retourne,
    # même si nous ne l'utilisons pas ici.
    fake_comments_db.pop(comment_id)
    
    return {"status": "Comment deleted"}
```
</details>

**Exercice 2 : Désinscription d'un Utilisateur (avec retour de données)**
Créez un endpoint `DELETE /users/{username}` pour désinscrire un utilisateur.
-   La ressource est identifiée par son `username` (une chaîne de caractères).
-   Si l'utilisateur n'existe pas, levez une erreur 404.
-   En cas de succès, supprimez l'utilisateur et retournez les données de l'utilisateur qui vient d'être supprimé (par exemple, son nom d'utilisateur et son email).

<details>
<summary>Découvrir la solution commentée</summary>

```python
from fastapi import FastAPI, HTTPException

app = FastAPI()

fake_users_db = {
    "alice": {"username": "alice", "email": "alice@example.com"},
    "bob": {"username": "bob", "email": "bob@example.com"},
}

@app.delete("/users/{username}")
async def unregister_user(username: str):
    if username not in fake_users_db:
        raise HTTPException(status_code=404, detail="User not found")
        
    unregistered_user = fake_users_db.pop(username)
    
    # On retourne les informations de l'utilisateur supprimé
    return unregistered_user
```
</details>

**Exercice 3 : Annuler une Réservation (avec réponse 204)**
Créez un endpoint `DELETE /reservations/{reservation_code}`.
-   `reservation_code` est une chaîne de caractères (ex: "CONF-XYZ123").
-   Si le code de réservation n'existe pas, levez une erreur 404.
-   Si la suppression réussit, la réponse doit être un `204 No Content`.

<details>
<summary>Découvrir la solution commentée</summary>

```python
from fastapi import FastAPI, HTTPException, status, Response

app = FastAPI()

fake_reservations_db = {
    "CONF-XYZ123": {"guest_name": "Charlie", "room": "101"},
    "CONF-ABC456": {"guest_name": "Diana", "room": "202"},
}

# On peut définir le code de statut dans le décorateur...
@app.delete("/reservations/{reservation_code}", status_code=status.HTTP_204_NO_CONTENT)
async def cancel_reservation(reservation_code: str):
    if reservation_code not in fake_reservations_db:
        raise HTTPException(status_code=404, detail="Reservation not found")
    
    del fake_reservations_db[reservation_code]
    
    # ...et retourner None.
    return None

# Une autre approche consiste à retourner un objet Response directement.
@app.delete("/reservations-alt/{reservation_code}")
async def cancel_reservation_alt(reservation_code: str):
    if reservation_code not in fake_reservations_db:
        raise HTTPException(status_code=404, detail="Reservation not found")
    
    del fake_reservations_db[reservation_code]
    
    return Response(status_code=status.HTTP_204_NO_CONTENT)
```
*Les deux approches sont valides. La première est souvent considérée comme plus "FastAPI-esque" car elle est plus déclarative.*
</details>