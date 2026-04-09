---
sidebar_label: "Modèles de paramètres de requête"
sidebar_position: 9
difficulty: "junior"
---

# Chapitre 9 : Modèles de paramètres de requête {#chapitre-9-:-modèles-de-paramètres-de-requête}

Query parameters, Pydantic, Dependency Injection, organisation du code

## Regroupement des paramètres avec Pydantic {#regroupement-des-paramètres-avec-pydantic-9}

### 1. Quoi
Au lieu de déclarer chaque paramètre de requête individuellement dans la signature de la fonction, FastAPI permet de regrouper ces paramètres dans une **classe Pydantic**. On utilise alors l'injection de dépendance `Depends` pour extraire ces valeurs automatiquement.

### 2. Pourquoi
Lorsque vous avez de nombreux paramètres (filtres, pagination, tri), la signature de votre fonction devient illisible. Regrouper ces paramètres dans un modèle améliore la **réutilisabilité**, la **lisibilité** et facilite la maintenance du code.

### 3. Comment
A. **Syntaxe de base** :
```python
from fastapi import FastAPI, Depends
from pydantic import BaseModel

app = FastAPI()

class FiltresRecherche(BaseModel):
    q: str | None = None
    page: int = 1
    taille: int = 10

@app.get("/items/")
async def lire_items(filtres: FiltresRecherche = Depends()):
    # FastAPI extrait automatiquement les paramètres de la requête 
    # et les injecte dans l'instance 'filtres'
    return filtres
```

B. **Cas concret et robuste** :
```python
from fastapi import FastAPI, Depends
from pydantic import BaseModel, Field

app = FastAPI()

class Pagination(BaseModel):
    page: int = Field(1, ge=1)
    taille: int = Field(10, ge=1, le=100)

@app.get("/utilisateurs/")
async def lister_utilisateurs(params: Pagination = Depends()):
    # Utilisation des contraintes Pydantic directement dans le modèle
    return {"page": params.page, "taille": params.taille}
```

C. **Exemples pratiques** :
- **Pagination standardisée** : Créer un modèle `Pagination` réutilisable sur tous les endpoints listant des ressources.
- **Filtres complexes** : Regrouper tous les critères de recherche d'un catalogue dans un modèle `FiltresCatalogue`.
- **Tri** : Inclure les paramètres de tri (`champ`, `ordre`) dans un modèle dédié.

### 4. Zone de Danger
❌ **À ne pas faire** : Créer des modèles de paramètres trop larges qui couvrent des besoins très différents, rendant le modèle difficile à comprendre.
✅ **Bonne Pratique** : Créez des modèles de paramètres spécifiques et cohérents par domaine fonctionnel.

---

## Questions clés (validation des acquis du chapitre) {#questions-clés-(validation-des-acquis-du-chapitre)-9}

- **Quel outil FastAPI permet d'injecter un modèle Pydantic comme paramètres de requête ?**
  *Réponse : L'injection de dépendance `Depends()`.*

- **Pourquoi est-il préférable d'utiliser un modèle Pydantic plutôt que 10 arguments individuels ?**
  *Réponse : Pour améliorer la lisibilité du code, faciliter la réutilisation des paramètres (ex: pagination) et centraliser la validation.*

- **Les contraintes Pydantic (comme `Field(ge=1)`) fonctionnent-elles lorsqu'on utilise `Depends()` ?**
  *Réponse : Oui, FastAPI valide automatiquement les champs du modèle Pydantic lors de l'injection.*

---

## Exercices : {#exercices-:-9}

### Exercice 1 - Modèle de pagination {#exercice-1---modèle-de-pagination}

🎯 **Objectif** : Créer un modèle réutilisable pour la pagination.

💼 **Mise en situation** : Votre API possède plusieurs routes listant des données. Vous voulez standardiser la pagination.

📝 **Énoncé** : Créez un modèle `PaginationParams` avec `page` (int, défaut 1, min 1) et `taille` (int, défaut 20, min 1, max 50). Utilisez-le dans une route `/produits/`.

📺 **Résultat attendu** : `/produits/?page=2&taille=10` retourne `{"page": 2, "taille": 10}`.

💡 **Solution** :
<details><summary>Voir le code complet commenté</summary>

```python
from fastapi import FastAPI, Depends
from pydantic import BaseModel, Field

app = FastAPI()

class PaginationParams(BaseModel):
    # On centralise la logique de pagination
    page: int = Field(1, ge=1)
    taille: int = Field(20, ge=1, le=50)

@app.get("/produits/")
async def lister_produits(params: PaginationParams = Depends()):
    return params
```
</details>

### Exercice 2 - Filtres de recherche {#exercice-2---filtres-de-recherche}

🎯 **Objectif** : Regrouper des filtres métier.

💼 **Mise en situation** : Vous recherchez des employés par nom et par département.

📝 **Énoncé** : Créez un modèle `FiltresEmploye` avec `nom` (str, optionnel) et `departement` (str, optionnel). Utilisez-le dans `/employes/`.

📺 **Résultat attendu** : `/employes/?nom=Alice&departement=IT` retourne `{"nom": "Alice", "departement": "IT"}`.

💡 **Solution** :
<details><summary>Voir le code complet commenté</summary>

```python
from fastapi import FastAPI, Depends
from pydantic import BaseModel

app = FastAPI()

class FiltresEmploye(BaseModel):
    nom: str | None = None
    departement: str | None = None

@app.get("/employes/")
async def lister_employes(filtres: FiltresEmploye = Depends()):
    return filtres
```
</details>

### Exercice 3 - Combinaison de modèles {#exercice-3---combinaison-de-modèles}

🎯 **Objectif** : Combiner plusieurs modèles de paramètres.

💼 **Mise en situation** : Vous voulez filtrer et paginer en même temps.

📝 **Énoncé** : Utilisez les modèles `PaginationParams` et `FiltresEmploye` dans la même route `/recherche-employes/`.

📺 **Résultat attendu** : `/recherche-employes/?nom=Bob&page=2` retourne les deux objets fusionnés ou accessibles via les paramètres.

💡 **Solution** :
<details><summary>Voir le code complet commenté</summary>

```python
from fastapi import FastAPI, Depends
from pydantic import BaseModel, Field

app = FastAPI()

class PaginationParams(BaseModel):
    page: int = Field(1, ge=1)
    taille: int = Field(20, ge=1, le=50)

class FiltresEmploye(BaseModel):
    nom: str | None = None
    departement: str | None = None

@app.get("/recherche-employes/")
async def recherche_employes(
    pagination: PaginationParams = Depends(),
    filtres: FiltresEmploye = Depends()
):
    # On peut injecter plusieurs modèles de paramètres
    return {"pagination": pagination, "filtres": filtres}
```
</details>