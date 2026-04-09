---
sidebar_label: "Construction d'URLs avec url_for"
sidebar_position: 6
difficulty: "junior"
---

# Chapitre 6 : Construction d'URLs avec url_for

url_for, Routage dynamique, Génération d'URLs, Maintenance

## La fonction url_for {#la-fonction-url_for-6}

### 1. Quoi
`url_for` est une fonction utilitaire de Flask qui génère dynamiquement l'URL correspondant à une fonction de vue donnée.

### 2. Pourquoi
Coder les URLs en dur (ex: `href="/user/1"`) est risqué : si vous changez la structure de vos routes, tous vos liens deviennent invalides. `url_for` permet de lier les URLs aux noms des fonctions, garantissant que les liens restent valides même si l'URL change.

### 3. Comment
A. **Syntaxe de base** :
```python
from flask import url_for

# Génère l'URL pour la fonction nommée 'index'
url = url_for("index")
```

B. **Cas concret avec paramètres** :
```python
@app.route("/user/<username>")
def profile(username: str):
    return f"Profil de {username}"

# Génération d'un lien vers le profil de 'alice'
# Résultat : "/user/alice"
lien = url_for("profile", username="alice")
```

C. **Exemples pratiques** :
- **Navigation** : Créer des menus de navigation qui s'adaptent automatiquement.
- **Redirection** : Rediriger un utilisateur vers une autre page après une action (ex: `redirect(url_for("login"))`).
- **Ressources** : Lier des fichiers statiques (CSS, images) avec `url_for("static", filename="style.css")`.

D. **Tableau comparatif** :

| Approche | Avantages | Inconvénients |
| :--- | :--- | :--- |
| **URL en dur** | Simple à écrire | Fragile, difficile à maintenir |
| **url_for** | Robuste, flexible, centralisé | Nécessite de connaître le nom de la fonction |

### 4. Zone de Danger
❌ **À ne pas faire** : Utiliser des chaînes de caractères pour construire des URLs manuellement dans vos templates ou votre code.
✅ **Bonne Pratique** : Utilisez systématiquement `url_for` pour toute référence à une route interne.

## Questions clés (validation des acquis du chapitre) {#questions-clés-(validation-des-acquis-du-chapitre)-6}

1. **Quel est l'avantage principal de `url_for` par rapport à une URL écrite en dur ?**
   Réponse : Il permet de modifier les URLs dans les décorateurs sans avoir à mettre à jour tous les liens dans le code ou les templates.

2. **Comment passer un paramètre dynamique à `url_for` ?**
   Réponse : En le passant comme argument nommé (ex: `url_for("profile", username="bob")`).

3. **Comment `url_for` gère-t-il les fichiers statiques ?**
   Réponse : En utilisant le nom de point de terminaison spécial `"static"` et le paramètre `filename`.

## Exercices : {#exercices-:-6}

### Exercice 1 - Lien vers l'accueil {#exercice-1---lien-vers-l-accueil-6}

- 🎯 **Objectif** : Utiliser `url_for` pour une route simple.
- 💼 **Mise en situation** : Vous voulez créer un lien vers votre page d'accueil.
- 📝 **Énoncé** : Créez une route `/` nommée `home` et une route `/info` qui retourne un lien vers `home` en utilisant `url_for`.
- 📺 **Résultat attendu** : La page `/info` affiche le chemin `/`.

<details>
<summary>Voir le code complet commenté</summary>

```python
from flask import Flask, url_for

app = Flask(__name__)

@app.route("/")
def home():
    return "Accueil"

@app.route("/info")
def info():
    # Génère l'URL pour la fonction home
    return f"Lien vers l'accueil : {url_for('home')}"
```
</details>

### Exercice 2 - Génération de profil {#exercice-2---génération-de-profil-6}

- 🎯 **Objectif** : Passer des arguments à `url_for`.
- 💼 **Mise en situation** : Vous créez une liste d'utilisateurs.
- 📝 **Énoncé** : Créez une route `/user/<username>` et une route `/list` qui retourne deux liens générés avec `url_for` pour les utilisateurs "alice" et "bob".
- 📺 **Résultat attendu** : `/list` affiche les URLs `/user/alice` et `/user/bob`.

<details>
<summary>Voir le code complet commenté</summary>

```python
@app.route("/user/<username>")
def profile(username: str):
    return f"Profil de {username}"

@app.route("/list")
def list_users():
    # Génération dynamique des URLs pour deux utilisateurs différents
    url_alice = url_for("profile", username="alice")
    url_bob = url_for("profile", username="bob")
    return f"Alice: {url_alice}, Bob: {url_bob}"
```
</details>

### Exercice 3 - Lien vers un fichier statique {#exercice-3---lien-vers-un-fichier-statique-6}

- 🎯 **Objectif** : Lier une ressource statique.
- 💼 **Mise en situation** : Vous devez charger une feuille de style.
- 📝 **Énoncé** : Créez une route `/style` qui retourne l'URL générée pour un fichier nommé `main.css` situé dans le dossier `static`.
- 📺 **Résultat attendu** : `/style` affiche `/static/main.css`.

<details>
<summary>Voir le code complet commenté</summary>

```python
@app.route("/style")
def get_style():
    # Le point de terminaison 'static' est réservé par Flask
    return url_for("static", filename="main.css")
```
</details>