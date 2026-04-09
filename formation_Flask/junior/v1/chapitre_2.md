---
sidebar_label: "Application minimale"
sidebar_position: 2
difficulty: "junior"
---

# Chapitre 2 : Application minimale

Instance Flask, Décorateur de route, Serveur de développement

## Création de l'instance Flask {#création-de-l-instance-flask-2}

### 1. Quoi
L'**instance Flask** est l'objet central de votre application. Elle gère la configuration, les routes et le cycle de vie de l'application web.

### 2. Pourquoi
Cet objet sert de point d'entrée unique pour enregistrer les composants de votre application (vues, modèles, erreurs).

### 3. Comment
A. **Syntaxe de base** :
```python
from flask import Flask

# Création de l'instance de l'application
app = Flask(__name__)
```

B. **Configuration** :
Le paramètre `__name__` permet à Flask de localiser les ressources (templates, fichiers statiques) à partir du répertoire où se trouve votre fichier.

### 4. Zone de Danger
❌ **À ne pas faire** : Créer plusieurs instances de `Flask` dans le même processus sans une architecture spécifique (Blueprints).
✅ **Bonne Pratique** : Une seule instance `app` par application.

## Routage et exécution {#routage-et-exécution-2}

### 1. Quoi
Le **routage** consiste à associer une URL (ex: `/`) à une fonction Python (appelée "vue"). Le **serveur de développement** est un serveur HTTP intégré permettant de tester l'application localement.

### 2. Pourquoi
C'est le mécanisme fondamental qui permet à votre application de répondre aux requêtes HTTP entrantes.

### 3. Comment
A. **Syntaxe de base** :
```python
@app.route("/")
def hello_world():
    return "Hello, World!"

if __name__ == "__main__":
    # Lancement du serveur en mode développement
    app.run(debug=True)
```

B. **Vérification** :
Lancez la commande `python app.py` dans votre terminal. Vous verrez une sortie indiquant que le serveur tourne sur `http://127.0.0.1:5000`.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Terminal affichant le démarrage du serveur Flask et navigateur affichant "Hello, World!".
> **Alt Text** : Terminal montrant "Running on http://127.0.0.1:5000" et navigateur affichant le texte de réponse.

C. **Tableau comparatif** :

| Mode | Utilité | Sécurité |
| :--- | :--- | :--- |
| `debug=True` | Développement (rechargement auto) | Faible (expose des erreurs) |
| `debug=False` | Production | Élevée (masque les détails) |

### 4. Zone de Danger
❌ **À ne pas faire** : Utiliser `debug=True` en production. Cela permet l'exécution de code arbitraire via le débogueur interactif.
✅ **Bonne Pratique** : Utilisez toujours des variables d'environnement pour gérer la configuration de production.

## Questions clés (validation des acquis du chapitre) {#questions-clés-(validation-des-acquis-du-chapitre)-2}

1. **Quel est le rôle du paramètre `__name__` lors de l'instanciation de `Flask` ?**
   Réponse : Il aide Flask à déterminer le chemin racine du projet pour localiser les ressources.

2. **À quoi sert le décorateur `@app.route("/")` ?**
   Réponse : Il associe l'URL racine `/` à la fonction définie juste en dessous.

3. **Pourquoi ne faut-il jamais utiliser `debug=True` en production ?**
   Réponse : Parce qu'il expose des informations sensibles sur le code et permet potentiellement l'exécution de code à distance via le débogueur.