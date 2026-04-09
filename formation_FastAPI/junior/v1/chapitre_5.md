---
sidebar_label: "Paramètres de requête"
sidebar_position: 5
difficulty: "junior"
---

# Chapitre 5 : Paramètres de requête {#chapitre-5-:-paramètres-de-requête}

Query parameters, filtrage, tri, pagination, paramètres optionnels

## Gestion des paramètres de requête {#gestion-des-paramètres-de-requête-5}

### 1. Quoi
Les **paramètres de requête** (query parameters) sont les éléments ajoutés après le `?` dans une URL (ex: `/items?skip=0&limit=10`). Dans FastAPI, tout argument de fonction qui n'est pas déclaré dans le chemin de la route est automatiquement interprété comme un paramètre de requête.

### 2. Pourquoi
Ils permettent de rendre vos endpoints flexibles sans multiplier les routes. Ils sont indispensables pour implémenter des fonctionnalités de **filtrage**, de **tri** ou de **pagination** sur des listes de ressources.

### 3. Comment
A. **Syntaxe de base** :
```python
from fastapi import FastAPI

app = FastAPI()

# 'skip' et 'limit' sont des paramètres de requête
@app.get("/items/")
async def lire_items(skip: int = 0, limit: int = 10):
    # On définit des valeurs par défaut pour rendre les paramètres optionnels
    return {"skip": skip, "limit": limit}
```

B. **Cas concret et robuste** :
```python
# Utilisation pour filtrage et tri
@app.get("/produits/")
async def lister_produits(categorie: str | None = None, trier_par: str = "nom"):
    # Si 'categorie' est omis, il vaut None
    # Si 'trier_par' est omis, il vaut "nom" par défaut
    return {"categorie": categorie, "trier_par": trier_par}
```

C. **Exemples pratiques** :
- **Pagination** : `?page=1&taille=20` pour limiter le volume de données retourné.
- **Filtrage** : `?statut=actif&type=service` pour restreindre les résultats.
- **Recherche** : `?q=recherche_utilisateur` pour filtrer par mots-clés.

### 4. Zone de Danger
❌ **À ne pas faire** : Utiliser des paramètres de requête pour transmettre des données sensibles (mots de passe, tokens) car ils apparaissent dans les logs du serveur et de l'historique du navigateur.
✅ **Bonne Pratique** : Utiliser les paramètres de requête uniquement pour le filtrage, le tri ou la pagination. Pour les données sensibles, utilisez le corps de la requête (POST) ou les headers (Authorization).

---

## Questions clés (validation des acquis du chapitre) {#questions-clés-(validation-des-acquis-du-chapitre)-5}

- **Comment FastAPI distingue-t-il un paramètre de chemin d'un paramètre de requête ?**
  *Réponse : Si le nom de l'argument est présent dans le chemin de la route (entre `{}`), c'est un paramètre de chemin. Sinon, c'est un paramètre de requête.*

- **Comment rendre un paramètre de requête optionnel ?**
  *Réponse : En lui attribuant une valeur par défaut dans la signature de la fonction (ex: `arg: str = "defaut"` ou `arg: str | None = None`).*

- **Pourquoi ne faut-il pas passer de données sensibles via les paramètres de requête ?**
  *Réponse : Parce qu'ils sont visibles dans les logs, l'historique du navigateur et les outils de monitoring, ce qui pose un risque de sécurité.*

---

## Exercices : {#exercices-:-5}

### Exercice 1 - Pagination simple {#exercice-1---pagination-simple}

🎯 **Objectif** : Implémenter une pagination basique.

💼 **Mise en situation** : Vous avez une API qui liste des articles. Vous voulez permettre au client de demander un nombre précis d'articles.

📝 **Énoncé** : Créez une route `/articles/` avec deux paramètres optionnels : `page` (int, défaut 1) et `taille` (int, défaut 5).

📺 **Résultat attendu** : `/articles/?page=2&taille=10` doit retourner `{"page": 2, "taille": 10}`.

💡 **Solution** :
<details><summary>Voir le code complet commenté</summary>

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/articles/")
async def lister_articles(page: int = 1, taille: int = 5):
    # On définit des valeurs par défaut pour assurer une pagination stable
    return {"page": page, "taille": taille}
```
</details>

### Exercice 2 - Filtrage par statut {#exercice-2---filtrage-par-statut}

🎯 **Objectif** : Filtrer une liste de ressources.

💼 **Mise en situation** : Vous gérez des tâches. L'utilisateur veut voir uniquement les tâches "terminées" ou "en_cours".

📝 **Énoncé** : Créez une route `/taches/` avec un paramètre `statut` (str, optionnel).

📺 **Résultat attendu** : `/taches/?statut=terminee` doit retourner `{"statut": "terminee"}`.

💡 **Solution** :
<details><summary>Voir le code complet commenté</summary>

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/taches/")
async def lister_taches(statut: str | None = None):
    # Le type | None permet d'indiquer que le filtre est optionnel
    return {"statut": statut}
```
</details>

### Exercice 3 - Recherche multi-critères {#exercice-3---recherche-multi-critères}

🎯 **Objectif** : Combiner plusieurs paramètres.

💼 **Mise en situation** : Vous avez un moteur de recherche de produits.

📝 **Énoncé** : Créez une route `/recherche/` avec `q` (str, optionnel), `min_prix` (float, optionnel) et `max_prix` (float, optionnel).

📺 **Résultat attendu** : `/recherche/?q=ordinateur&min_prix=500` doit retourner `{"q": "ordinateur", "min_prix": 500.0, "max_prix": None}`.

💡 **Solution** :
<details><summary>Voir le code complet commenté</summary>

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/recherche/")
async def rechercher_produits(
    q: str | None = None, 
    min_prix: float | None = None, 
    max_prix: float | None = None
):
    # On combine plusieurs filtres optionnels pour une recherche avancée
    return {"q": q, "min_prix": min_prix, "max_prix": max_prix}
```
</details>