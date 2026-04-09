---
sidebar_label: "Routage de base"
sidebar_position: 4
difficulty: "junior"
---

# Chapitre 4 : Routage de base

Décorateurs, Routage, Méthodes HTTP, Paramètres d'URL

## Le système de routage {#le-système-de-routage-4}

### 1. Quoi
Le **routage** est le mécanisme qui lie une URL spécifique à une fonction Python, appelée **vue**. Dans Flask, cela se fait principalement via le décorateur `@app.route()`.

### 2. Pourquoi
Il permet de structurer votre application en différentes pages ou endpoints API, rendant le code modulaire et l'application navigable.

### 3. Comment
A. **Syntaxe de base** :
```python
@app.route("/accueil")
def index():
    return "Bienvenue sur la page d'accueil"
```

B. **Cas concret** :
```python
from flask import Flask

app = Flask(__name__)

# Route simple
@app.route("/")
def home():
    return "Page d'accueil"

# Route avec paramètre dynamique
@app.route("/user/<username>")
def show_user(username: str):
    # On retourne le nom d'utilisateur passé dans l'URL
    return f"Profil de {username}"
```

C. **Exemples pratiques** :
- **Route statique** : `@app.route("/about")` pour une page d'information fixe.
- **Route dynamique** : `@app.route("/post/<int:post_id>")` pour afficher un article spécifique par son ID.
- **Méthodes HTTP** : `@app.route("/login", methods=["GET", "POST"])` pour gérer à la fois l'affichage du formulaire et la soumission des données.

D. **Tableau comparatif des types de paramètres** :

| Type | Description | Exemple |
| :--- | :--- | :--- |
| `string` | Chaîne de caractères (défaut) | `/user/alice` |
| `int` | Nombre entier | `/post/42` |
| `float` | Nombre à virgule | `/price/19.99` |

### 4. Zone de Danger
❌ **À ne pas faire** : Créer des routes avec des noms ambigus ou des paramètres non typés quand le type est connu.
✅ **Bonne Pratique** : Utilisez toujours le typage des paramètres (ex: `<int:id>`) pour valider automatiquement les entrées utilisateur.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Navigateur affichant une route dynamique (ex: /user/jean).
> **Alt Text** : Page web affichant "Profil de jean".

## Questions clés (validation des acquis du chapitre) {#questions-clés-(validation-des-acquis-du-chapitre)-4}

1. **Quel décorateur utilise-t-on pour définir une route dans Flask ?**
   Réponse : Le décorateur `@app.route()`.

2. **Comment définir un paramètre entier dans une URL ?**
   Réponse : En utilisant la syntaxe `<int:nom_du_parametre>`.

3. **Comment autoriser plusieurs méthodes HTTP sur une même route ?**
   Réponse : En passant une liste de méthodes au paramètre `methods` du décorateur (ex: `methods=["GET", "POST"]`).

## Exercices : {#exercices-:-4}

### Exercice 1 - La route personnalisée {#exercice-1---la-route-personnalisée-4}

- 🎯 **Objectif** : Créer une route simple.
- 💼 **Mise en situation** : Vous devez ajouter une page "Contact" à votre site.
- 📝 **Énoncé** : Créez une route `/contact` qui retourne le texte "Contactez-nous à contact@example.com".
- 📺 **Résultat attendu** : L'URL `/contact` affiche le texte demandé.

<details>
<summary>Découvrir la solution commentée</summary>

```python
@app.route("/contact")
def contact():
    # Retourne une chaîne simple pour la page contact
    return "Contactez-nous à contact@example.com"
```
</details>

### Exercice 2 - Profil utilisateur dynamique {#exercice-2---profil-utilisateur-dynamique-4}

- 🎯 **Objectif** : Utiliser des paramètres dynamiques.
- 💼 **Mise en situation** : Vous construisez un réseau social et chaque utilisateur a sa propre page.
- 📝 **Énoncé** : Créez une route `/profile/<username>` qui affiche "Bienvenue sur le profil de [username]".
- 📺 **Résultat attendu** : `/profile/bob` affiche "Bienvenue sur le profil de bob".

<details>
<summary>Découvrir la solution commentée</summary>

```python
@app.route("/profile/<username>")
def profile(username: str):
    # Le paramètre username est injecté dynamiquement dans la chaîne
    return f"Bienvenue sur le profil de {username}"
```
</details>

### Exercice 3 - Gestion des IDs produits {#exercice-3---gestion-des-ids-produits-4}

- 🎯 **Objectif** : Utiliser le typage des paramètres.
- 💼 **Mise en situation** : Vous gérez un catalogue de produits où chaque produit est identifié par un entier.
- 📝 **Énoncé** : Créez une route `/product/<int:product_id>` qui affiche "Affichage du produit numéro [product_id]". Si l'utilisateur entre quelque chose qui n'est pas un entier, Flask doit renvoyer une erreur 404.
- 📺 **Résultat attendu** : `/product/123` affiche "Affichage du produit numéro 123". `/product/abc` renvoie une erreur 404.

<details>
<summary>Découvrir la solution commentée</summary>

```python
@app.route("/product/<int:product_id>")
def product(product_id: int):
    # Le typage <int:...> garantit que product_id est un entier
    return f"Affichage du produit numéro {product_id}"
```
</details>