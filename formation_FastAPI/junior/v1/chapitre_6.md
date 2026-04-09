---
sidebar_label: "Corps de la requête (Request Body)"
sidebar_position: 6
difficulty: "junior"
---

# Chapitre 6 : Corps de la requête (Request Body) {#chapitre-6-:-corps-de-la-requête-(request-body)}

Pydantic, modèles de données, validation JSON, sérialisation

## Utilisation des modèles Pydantic {#utilisation-des-modèles-pydantic-6}

### 1. Quoi
Le **corps de la requête** (Request Body) est la partie d'une requête HTTP contenant les données envoyées par le client (généralement au format JSON). Dans FastAPI, on utilise des classes héritant de `BaseModel` de la bibliothèque **Pydantic** pour définir la structure attendue de ces données.

### 2. Pourquoi
Pydantic permet de définir des schémas de données stricts. FastAPI utilise ces schémas pour valider automatiquement le JSON entrant, convertir les types, et générer la documentation interactive (Swagger UI).

### 3. Comment
A. **Syntaxe de base** :
```python
from pydantic import BaseModel

# Définition du modèle de données
class Article(BaseModel):
    titre: str
    contenu: str
    est_publie: bool = False # Valeur par défaut
```

B. **Cas concret et robuste** :
```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Article(BaseModel):
    titre: str
    contenu: str
    est_publie: bool = False

@app.post("/articles/")
async def creer_article(article: Article):
    # FastAPI valide que le JSON contient bien les champs requis
    # et les convertit en objet Python 'Article'
    return {"message": "Article créé", "data": article}
```

C. **Exemples pratiques** :
- **Création de ressource** : Envoyer un objet JSON pour créer un nouvel utilisateur.
- **Mise à jour** : Envoyer uniquement les champs modifiés pour mettre à jour un profil.
- **Validation complexe** : Utiliser des types comme `EmailStr` ou des contraintes comme `Field(gt=0)` pour valider des emails ou des nombres positifs.

### 4. Zone de Danger
❌ **À ne pas faire** : Accepter des données brutes (`dict`) sans utiliser de modèle Pydantic, ce qui expose l'application à des données mal formées ou malveillantes.
✅ **Bonne Pratique** : Toujours définir des modèles Pydantic pour chaque endpoint acceptant un corps de requête.

---

## Questions clés (validation des acquis du chapitre) {#questions-clés-(validation-des-acquis-du-chapitre)-6}

- **Quelle bibliothèque FastAPI utilise-t-il pour valider les données du corps de la requête ?**
  *Réponse : Pydantic.*

- **Que se passe-t-il si le client envoie un JSON qui ne correspond pas au modèle Pydantic défini ?**
  *Réponse : FastAPI renvoie automatiquement une erreur 422 (Unprocessable Entity) avec les détails des erreurs de validation.*

- **Pourquoi est-il recommandé d'utiliser des modèles Pydantic plutôt que des dictionnaires bruts ?**
  *Réponse : Pour bénéficier de la validation automatique des types, de la conversion des données et de la génération automatique de la documentation.*

---

## Exercices : {#exercices-:-6}

### Exercice 1 - Création d'un utilisateur {#exercice-1---création-d-un-utilisateur}

🎯 **Objectif** : Créer un modèle simple pour recevoir des données JSON.

💼 **Mise en situation** : Vous devez créer un endpoint pour enregistrer un nouvel utilisateur.

📝 **Énoncé** : Créez un modèle `Utilisateur` avec `nom` (str) et `age` (int). Créez la route `POST /utilisateurs/` qui reçoit cet objet.

📺 **Résultat attendu** : Un POST avec `{"nom": "Alice", "age": 30}` retourne `{"nom": "Alice", "age": 30}`.

💡 **Solution** :
<details><summary>Voir le code complet commenté</summary>

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Utilisateur(BaseModel):
    nom: str
    age: int

@app.post("/utilisateurs/")
async def creer_utilisateur(user: Utilisateur):
    # FastAPI valide automatiquement le JSON entrant selon le modèle
    return user
```
</details>

### Exercice 2 - Modèle avec valeurs par défaut {#exercice-2---modèle-avec-valeurs-par-défaut}

🎯 **Objectif** : Gérer des champs optionnels dans un modèle.

💼 **Mise en situation** : Un utilisateur peut avoir une biographie, mais ce n'est pas obligatoire.

📝 **Énoncé** : Ajoutez un champ `bio` (str, optionnel, défaut `None`) au modèle `Utilisateur` de l'exercice précédent.

📺 **Résultat attendu** : Un POST sans `bio` doit fonctionner et retourner `{"nom": "Alice", "age": 30, "bio": null}`.

💡 **Solution** :
<details><summary>Voir le code complet commenté</summary>

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Utilisateur(BaseModel):
    nom: str
    age: int
    bio: str | None = None # Champ optionnel avec valeur par défaut

@app.post("/utilisateurs/")
async def creer_utilisateur(user: Utilisateur):
    return user
```
</details>

### Exercice 3 - Validation de données {#exercice-3---validation-de-données}

🎯 **Objectif** : Utiliser les contraintes Pydantic.

💼 **Mise en situation** : Vous voulez vous assurer que l'âge est toujours positif.

📝 **Énoncé** : Utilisez `Field` de `pydantic` pour contraindre `age` à être supérieur ou égal à 18.

📺 **Résultat attendu** : Un POST avec `age: 15` doit renvoyer une erreur 422.

💡 **Solution** :
<details><summary>Voir le code complet commenté</summary>

```python
from fastapi import FastAPI
from pydantic import BaseModel, Field

app = FastAPI()

class Utilisateur(BaseModel):
    nom: str
    # On contraint l'âge à être au moins 18
    age: int = Field(ge=18) 
    bio: str | None = None

@app.post("/utilisateurs/")
async def creer_utilisateur(user: Utilisateur):
    return user
```
</details>