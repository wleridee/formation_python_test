---
sidebar_label: "Routes dynamiques"
sidebar_position: 5
difficulty: "junior"
---

# Chapitre 5 : Routes dynamiques

Variables d'URL, Convertisseurs de types, Routage flexible

## Les variables dans les URLs {#les-variables-dans-les-urls-5}

### 1. Quoi
Les **routes dynamiques** permettent de capturer des segments d'une URL pour les utiliser comme arguments dans vos fonctions de vue. On utilise la syntaxe `<variable>` dans le décorateur `@app.route()`.

### 2. Pourquoi
Elles permettent de créer des applications génériques. Au lieu de définir une route pour chaque utilisateur ou chaque article, une seule route peut gérer des milliers de ressources différentes.

### 3. Comment
A. **Syntaxe de base** :
```python
@app.route("/article/<article_id>")
def get_article(article_id: str):
    return f"Affichage de l'article {article_id}"
```

B. **Cas concret avec typage** :
```python
from flask import Flask

app = Flask(__name__)

# Utilisation d'un convertisseur pour forcer un entier
@app.route("/produit/<int:id>")
def afficher_produit(id: int):
    # Flask valide automatiquement que 'id' est un entier
    return f"Détails du produit n°{id}"
```

C. **Exemples pratiques** :
- **Recherche** : `@app.route("/search/<query>")` pour capturer les termes de recherche.
- **Catégories** : `@app.route("/category/<category_name>")` pour filtrer des contenus.
- **Dates** : `@app.route("/archive/<int:year>/<int:month>")` pour naviguer dans des archives temporelles.

D. **Convertisseurs disponibles** :

| Convertisseur | Description |
| :--- | :--- |
| `string` | Accepte tout sauf les slashs (défaut) |
| `int` | Accepte les entiers positifs |
| `float` | Accepte les nombres à virgule |
| `path` | Comme string, mais accepte les slashs |

### 4. Zone de Danger
❌ **À ne pas faire** : Faire confiance aux données provenant de l'URL sans validation supplémentaire si elles sont utilisées pour des requêtes en base de données (risque d'injection SQL).
✅ **Bonne Pratique** : Utilisez toujours les convertisseurs de types intégrés et validez les données métier dans votre logique.

### 🚨 Limitations des routes dynamiques {#limitations-des-routes-dynamiques-5}

- **Problèmes concrets** : Une accumulation excessive de routes dynamiques complexes peut rendre le routage difficile à maintenir et à déboguer.
- **Solutions modernes** : Pour des API complexes, envisagez d'utiliser des bibliothèques comme **Marshmallow** pour la validation poussée des schémas de données.
- **Pourquoi l'enseigner** : C'est le socle de la navigation web moderne et indispensable pour comprendre comment Flask traite les requêtes entrantes.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Navigateur affichant une route dynamique avec un entier (ex: /produit/42).
> **Alt Text** : Page web affichant "Détails du produit n°42".

## Questions clés (validation des acquis du chapitre) {#questions-clés-(validation-des-acquis-du-chapitre)-5}

1. **Quelle syntaxe permet de définir une variable dans une route Flask ?**
   Réponse : On utilise les chevrons `<variable_name>` dans la chaîne de l'URL.

2. **Quel est l'intérêt d'utiliser `<int:id>` au lieu de `<id>` ?**
   Réponse : Cela force Flask à valider que le segment d'URL est un entier, sinon il renvoie une erreur 404 automatiquement.

3. **Comment capturer un chemin complet incluant des slashs dans une variable ?**
   Réponse : En utilisant le convertisseur `path` (ex: `<path:subpath>`).

## Exercices : {#exercices-:-5}

### Exercice 1 - Le catalogue de films {#exercice-1---le-catalogue-de-films-5}

- 🎯 **Objectif** : Créer une route dynamique simple.
- 💼 **Mise en situation** : Vous créez un site de critiques de films.
- 📝 **Énoncé** : Créez une route `/film/<nom_film>` qui retourne "Critique du film : [nom_film]".
- 📺 **Résultat attendu** : `/film/inception` affiche "Critique du film : inception".

<details>
<summary>Voir le code complet commenté</summary>

```python
@app.route("/film/<nom_film>")
def critique_film(nom_film: str):
    # On récupère le nom du film depuis l'URL
    return f"Critique du film : {nom_film}"
```
</details>

### Exercice 2 - Calculateur de TVA {#exercice-2---calculateur-de-tva-5}

- 🎯 **Objectif** : Utiliser le convertisseur `float`.
- 💼 **Mise en situation** : Vous développez un outil de facturation rapide.
- 📝 **Énoncé** : Créez une route `/prix/<float:montant>` qui affiche le montant avec une TVA de 20% (montant * 1.2).
- 📺 **Résultat attendu** : `/prix/100.0` affiche "Prix TTC : 120.0".

<details>
<summary>Voir le code complet commenté</summary>

```python
@app.route("/prix/<float:montant>")
def calculer_tva(montant: float):
    # Calcul simple du prix TTC
    ttc = montant * 1.2
    return f"Prix TTC : {ttc}"
```
</details>

### Exercice 3 - Navigation par date {#exercice-3---navigation-par-date-5}

- 🎯 **Objectif** : Utiliser plusieurs variables dynamiques.
- 💼 **Mise en situation** : Vous gérez un blog avec des archives mensuelles.
- 📝 **Énoncé** : Créez une route `/archive/<int:annee>/<int:mois>` qui affiche "Archives de [mois]/[annee]".
- 📺 **Résultat attendu** : `/archive/2023/10` affiche "Archives de 10/2023".

<details>
<summary>Voir le code complet commenté</summary>

```python
@app.route("/archive/<int:annee>/<int:mois>")
def archives(annee: int, mois: int):
    # On récupère deux entiers depuis l'URL
    return f"Archives de {mois}/{annee}"
```
</details>