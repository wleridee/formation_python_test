---
sidebar_label: "Paramètres de chemin"
sidebar_position: 4
difficulty: "junior"
---

# Chapitre 4 : Paramètres de chemin {#chapitre-4-:-paramètres-de-chemin}

Path parameters, typage dynamique, routage dynamique, validation

## Définition des paramètres de chemin {#définition-des-paramètres-de-chemin-4}

### 1. Quoi
Les **paramètres de chemin** sont des variables dynamiques intégrées directement dans l'URL d'une route. Ils sont définis en utilisant des accolades `{}` dans le chemin de la route et correspondent aux arguments de la fonction associée.

### 2. Pourquoi
Ils permettent de créer des API RESTful où l'URL identifie précisément la ressource manipulée (ex: `/utilisateurs/42` pour l'utilisateur ayant l'identifiant 42), rendant l'API intuitive et structurée.

### 3. Comment
A. **Syntaxe de base** :
```python
from fastapi import FastAPI

app = FastAPI()

# {user_id} est le paramètre de chemin
@app.get("/utilisateurs/{user_id}")
async def lire_utilisateur(user_id: int):
    # FastAPI injecte la valeur de l'URL dans l'argument user_id
    return {"user_id": user_id}
```

B. **Cas concret et robuste** :
```python
# Gestion de plusieurs paramètres
@app.get("/projets/{projet_id}/taches/{tache_id}")
async def lire_tache(projet_id: int, tache_id: int):
    # Validation automatique : si l'un des ID n'est pas un entier, 
    # FastAPI renvoie une erreur 422 automatiquement
    return {"projet": projet_id, "tache": tache_id}
```

C. **Exemples pratiques** :
- **Identifiants uniques** : `/commandes/{commande_id}` pour accéder à une commande précise.
- **Hiérarchie de ressources** : `/categories/{cat_nom}/produits/{prod_id}` pour naviguer dans un catalogue.
- **Fichiers** : `/fichiers/{chemin_fichier:path}` pour autoriser le passage de chemins complexes contenant des `/`.

### 4. Zone de Danger
❌ **À ne pas faire** : Utiliser des noms de paramètres ambigus ou trop longs dans l'URL.
✅ **Bonne Pratique** : Utiliser des noms de paramètres courts, explicites et cohérents avec votre modèle de données.

---

## Questions clés (validation des acquis du chapitre) {#questions-clés-(validation-des-acquis-du-chapitre)-4}

- **Comment déclare-t-on un paramètre de chemin dans une route FastAPI ?**
  *Réponse : En entourant le nom de la variable par des accolades `{}` dans la chaîne de caractères du chemin de la route.*

- **Que se passe-t-il si le type déclaré dans la fonction ne correspond pas à la valeur passée dans l'URL ?**
  *Réponse : FastAPI tente de convertir la valeur. Si la conversion échoue, il renvoie une erreur 422 (Unprocessable Entity).*

- **Est-il possible d'avoir plusieurs paramètres de chemin dans une même URL ?**
  *Réponse : Oui, il suffit de les déclarer successivement dans le chemin (ex: `/a/{id1}/b/{id2}`).*

---

## Exercices : {#exercices-:-4}

### Exercice 1 - Identifiant utilisateur {#exercice-1---identifiant-utilisateur}

🎯 **Objectif** : Créer une route dynamique simple.

💼 **Mise en situation** : Vous gérez une base d'utilisateurs. Vous devez créer une route pour récupérer le profil d'un utilisateur par son ID.

📝 **Énoncé** : Créez une route `/profil/{user_id}` qui retourne l'ID reçu.

📺 **Résultat attendu** : Une requête sur `/profil/123` doit retourner `{"user_id": 123}`.

💡 **Solution** :
<details><summary>Voir le code complet commenté</summary>

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/profil/{user_id}")
async def get_profil(user_id: int):
    # Le type int force la validation de l'ID
    return {"user_id": user_id}
```
</details>

### Exercice 2 - Hiérarchie de ressources {#exercice-2---hiérarchie-de-ressources}

🎯 **Objectif** : Manipuler plusieurs paramètres de chemin.

💼 **Mise en situation** : Dans une bibliothèque, chaque livre appartient à une étagère.

📝 **Énoncé** : Créez une route `/etagere/{nom_etagere}/livre/{livre_id}`.

📺 **Résultat attendu** : `/etagere/science/livre/5` doit retourner `{"etagere": "science", "livre_id": 5}`.

💡 **Solution** :
<details><summary>Voir le code complet commenté</summary>

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/etagere/{nom_etagere}/livre/{livre_id}")
async def get_livre(nom_etagere: str, livre_id: int):
    # nom_etagere est typé en str, livre_id en int
    return {"etagere": nom_etagere, "livre_id": livre_id}
```
</details>

### Exercice 3 - Chemin complexe {#exercice-3---chemin-complexe}

🎯 **Objectif** : Utiliser le convertisseur `:path`.

💼 **Mise en situation** : Vous voulez passer un chemin de fichier complet dans une URL.

📝 **Énoncé** : Créez une route `/fichiers/{file_path:path}`.

📺 **Résultat attendu** : `/fichiers/dossier/sous-dossier/image.png` doit retourner `{"file_path": "dossier/sous-dossier/image.png"}`.

💡 **Solution** :
<details><summary>Voir le code complet commenté</summary>

```python
from fastapi import FastAPI

app = FastAPI()

# Le convertisseur :path permet de capturer les '/' dans la variable
@app.get("/fichiers/{file_path:path}")
async def get_fichier(file_path: str):
    return {"file_path": file_path}
```
</details>