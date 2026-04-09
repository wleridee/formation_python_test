---
sidebar_label: "Validation de chaînes"
sidebar_position: 7
difficulty: "junior"
---

# Chapitre 7 : Validation de chaînes {#chapitre-7-:-validation-de-chaînes}

Query, Path, Field, expressions régulières, contraintes de longueur

## Utilisation de Query et Path pour la validation {#utilisation-de-query-et-path-pour-la-validation-7}

### 1. Quoi
FastAPI fournit les classes `Query` et `Path` pour ajouter des métadonnées et des contraintes de validation aux paramètres de requête et de chemin. Elles permettent de définir des règles comme la longueur minimale/maximale ou des motifs via des **expressions régulières** (Regex).

### 2. Pourquoi
La validation native de Python est limitée. `Query` et `Path` permettent de garantir que les données entrantes respectent des règles métier strictes (ex: un nom d'utilisateur doit faire entre 3 et 20 caractères et ne contenir que des lettres) avant même que votre logique métier ne soit exécutée.

### 3. Comment
A. **Syntaxe de base** :
```python
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def lire_items(q: str = Query(..., min_length=3, max_length=50)):
    # '...' signifie que le paramètre est obligatoire
    return {"q": q}
```

B. **Cas concret avec Regex** :
```python
from fastapi import FastAPI, Path

app = FastAPI()

@app.get("/utilisateurs/{user_id}")
async def lire_utilisateur(
    user_id: int = Path(..., gt=0), # Doit être strictement supérieur à 0
    nom: str = Query(..., regex="^[a-zA-Z]+$") # Doit être uniquement des lettres
):
    return {"user_id": user_id, "nom": nom}
```

C. **Exemples pratiques** :
- **Recherche** : `Query(min_length=3)` pour éviter les recherches trop larges.
- **Identifiants** : `Path(ge=1)` pour s'assurer qu'un ID est positif.
- **Formatage** : `Query(regex="^item_.*")` pour forcer un préfixe sur une chaîne.

### 4. Zone de Danger
❌ **À ne pas faire** : Utiliser des Regex trop complexes qui peuvent impacter les performances ou être difficiles à maintenir.
✅ **Bonne Pratique** : Garder les validations simples et documenter les contraintes via le paramètre `description` de `Query` ou `Path`.

---

## Questions clés (validation des acquis du chapitre) {#questions-clés-(validation-des-acquis-du-chapitre)-7}

- **Quelle est la différence entre `Query` et `Path` dans FastAPI ?**
  *Réponse : `Query` est utilisé pour valider les paramètres de requête (après le `?`), tandis que `Path` est utilisé pour valider les paramètres intégrés dans le chemin de l'URL.*

- **Que signifie l'argument `...` (Ellipsis) dans `Query(...)` ?**
  *Réponse : Il indique que le paramètre est obligatoire, même s'il possède des contraintes de validation.*

- **Comment forcer une chaîne de caractères à respecter un format spécifique (ex: lettres uniquement) ?**
  *Réponse : En utilisant le paramètre `regex` dans `Query` ou `Path` avec une expression régulière appropriée.*

---

## Exercices : {#exercices-:-7}

### Exercice 1 - Validation de longueur {#exercice-1---validation-de-longueur}

🎯 **Objectif** : Restreindre la taille d'une recherche.

💼 **Mise en situation** : Vous voulez éviter que les utilisateurs fassent des recherches avec un seul caractère, ce qui est trop coûteux pour votre base de données.

📝 **Énoncé** : Créez une route `/recherche/` avec un paramètre `q` (str) qui doit faire au moins 3 caractères et au plus 20.

📺 **Résultat attendu** : `/recherche/?q=ab` doit renvoyer une erreur 422.

💡 **Solution** :
<details><summary>Voir le code complet commenté</summary>

```python
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/recherche/")
async def recherche(q: str = Query(..., min_length=3, max_length=20)):
    # On impose une longueur minimale et maximale pour optimiser la recherche
    return {"q": q}
```
</details>

### Exercice 2 - Identifiant positif {#exercice-2---identifiant-positif}

🎯 **Objectif** : Valider un paramètre de chemin.

💼 **Mise en situation** : Vous manipulez des ressources identifiées par des entiers positifs.

📝 **Énoncé** : Créez une route `/produit/{prod_id}` où `prod_id` doit être supérieur ou égal à 1.

📺 **Résultat attendu** : `/produit/0` doit renvoyer une erreur 422.

💡 **Solution** :
<details><summary>Voir le code complet commenté</summary>

```python
from fastapi import FastAPI, Path

app = FastAPI()

@app.get("/produit/{prod_id}")
async def get_produit(prod_id: int = Path(..., ge=1)):
    # ge=1 signifie "Greater or Equal to 1"
    return {"prod_id": prod_id}
```
</details>

### Exercice 3 - Formatage via Regex {#exercice-3---formatage-via-regex}

🎯 **Objectif** : Forcer un format de code produit.

💼 **Mise en situation** : Les codes produits doivent toujours commencer par "PROD-" suivi de chiffres.

📝 **Énoncé** : Créez une route `/code/{code}` où `code` doit respecter le pattern `^PROD-[0-9]+$`.

📺 **Résultat attendu** : `/code/PROD-123` fonctionne, `/code/TEST-123` renvoie 422.

💡 **Solution** :
<details><summary>Voir le code complet commenté</summary>

```python
from fastapi import FastAPI, Path

app = FastAPI()

@app.get("/code/{code}")
async def get_code(code: str = Path(..., regex="^PROD-[0-9]+$")):
    # Le regex garantit que le format est strictement respecté
    return {"code": code}
```
</details>