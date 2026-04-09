---
sidebar_label: "Validation numérique"
sidebar_position: 8
difficulty: "junior"
---

# Chapitre 8 : Validation numérique {#chapitre-8-:-validation-numérique}

Query, Path, Field, contraintes numériques, bornes, multiples

## Contraintes sur les nombres {#contraintes-sur-les-nombres-8}

### 1. Quoi
La **validation numérique** dans FastAPI permet de restreindre les valeurs acceptées pour les paramètres de type entier (`int`) ou flottant (`float`). En utilisant `Query`, `Path` ou `Field`, on peut définir des bornes inférieures, supérieures, ou même des multiples.

### 2. Pourquoi
Dans un contexte métier, les données numériques ont souvent des limites logiques (ex: un âge ne peut pas être négatif, une note doit être comprise entre 0 et 20). La validation automatique évite de polluer votre logique métier avec des vérifications répétitives.

### 3. Comment
A. **Syntaxe de base** :
```python
from fastapi import FastAPI, Query

app = FastAPI()

# gt: greater than, ge: greater than or equal
# lt: less than, le: less than or equal
@app.get("/items/")
async def lire_items(page: int = Query(1, ge=1), limite: int = Query(10, le=100)):
    return {"page": page, "limite": limite}
```

B. **Cas concret et robuste** :
```python
from fastapi import FastAPI, Path

app = FastAPI()

@app.get("/temperature/{valeur}")
async def verifier_temp(valeur: float = Path(..., ge=-50.0, le=150.0)):
    # On s'assure que la température est dans une plage physique réaliste
    return {"temperature": valeur}
```

C. **Exemples pratiques** :
- **Pagination** : `Query(ge=1)` pour éviter les pages négatives.
- **Prix** : `Query(gt=0)` pour s'assurer qu'un prix est strictement positif.
- **Pourcentage** : `Query(ge=0, le=100)` pour valider une valeur de progression.

### 4. Zone de Danger
❌ **À ne pas faire** : Oublier de définir des bornes pour des paramètres numériques critiques, ce qui peut entraîner des erreurs de calcul ou des comportements inattendus en base de données.
✅ **Bonne Pratique** : Soyez toujours aussi restrictif que possible sur les plages de valeurs acceptées.

---

## Questions clés (validation des acquis du chapitre) {#questions-clés-(validation-des-acquis-du-chapitre)-8}

- **Quelle est la différence entre `ge` et `gt` dans les contraintes numériques ?**
  *Réponse : `ge` signifie "Greater or Equal" (supérieur ou égal), tandis que `gt` signifie "Greater Than" (strictement supérieur).*

- **Comment limiter une valeur numérique à un maximum de 50 ?**
  *Réponse : En utilisant le paramètre `le=50` (Less or Equal) dans `Query`, `Path` ou `Field`.*

- **Peut-on combiner plusieurs contraintes numériques sur le même paramètre ?**
  *Réponse : Oui, par exemple `Query(ge=0, le=100)` pour restreindre un nombre entre 0 et 100 inclus.*

---

## Exercices : {#exercices-:-8}

### Exercice 1 - Âge valide {#exercice-1---âge-valide}

🎯 **Objectif** : Valider un âge.

💼 **Mise en situation** : Vous créez une API pour un site de jeux vidéo. L'utilisateur doit avoir au moins 13 ans.

📝 **Énoncé** : Créez une route `/inscription/` avec un paramètre `age` (int) qui doit être supérieur ou égal à 13.

📺 **Résultat attendu** : `/inscription/?age=12` doit renvoyer une erreur 422.

💡 **Solution** :
<details><summary>Voir le code complet commenté</summary>

```python
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/inscription/")
async def inscription(age: int = Query(..., ge=13)):
    # ge=13 garantit que l'utilisateur a l'âge minimum requis
    return {"age": age}
```
</details>

### Exercice 2 - Note d'évaluation {#exercice-2---note-d-évaluation}

🎯 **Objectif** : Valider une note comprise dans un intervalle.

💼 **Mise en situation** : Les utilisateurs peuvent noter des produits de 1 à 5 étoiles.

📝 **Énoncé** : Créez une route `/note/{etoiles}` où `etoiles` (int) doit être compris entre 1 et 5 inclus.

📺 **Résultat attendu** : `/note/6` doit renvoyer une erreur 422.

💡 **Solution** :
<details><summary>Voir le code complet commenté</summary>

```python
from fastapi import FastAPI, Path

app = FastAPI()

@app.get("/note/{etoiles}")
async def noter_produit(etoiles: int = Path(..., ge=1, le=5)):
    # ge=1 et le=5 restreignent la note à l'intervalle [1, 5]
    return {"note": etoiles}
```
</details>

### Exercice 3 - Prix promotionnel {#exercice-3---prix-promotionnel}

🎯 **Objectif** : Valider un prix flottant.

💼 **Mise en situation** : Vous appliquez une réduction. Le prix final doit être positif.

📝 **Énoncé** : Créez une route `/prix/` avec un paramètre `montant` (float) qui doit être strictement supérieur à 0.

📺 **Résultat attendu** : `/prix/?montant=0` doit renvoyer une erreur 422.

💡 **Solution** :
<details><summary>Voir le code complet commenté</summary>

```python
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/prix/")
async def set_prix(montant: float = Query(..., gt=0)):
    # gt=0 garantit que le montant est strictement positif
    return {"montant": montant}
```
</details>