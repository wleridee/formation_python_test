---
sidebar_label: "Installation et premier serveur"
sidebar_position: 1
difficulty: "junior"
---

# Chapitre 1 : Installation et premier serveur {#chapitre-1-:-installation-et-premier-serveur}

Environnement virtuel, pip, FastAPI, Uvicorn, serveur ASGI

## Configuration de l'environnement {#configuration-de-l-environnement-1}

### 1. Quoi
La création d'un **environnement virtuel** est une pratique standard en Python pour isoler les dépendances d'un projet spécifique du reste du système.

### 2. Pourquoi
Cela évite les conflits de versions entre différents projets sur une même machine et garantit que votre application FastAPI dispose exactement des bibliothèques nécessaires pour fonctionner.

### 3. Comment
A. **Création et activation** :
```bash
# Création du dossier d'environnement
python -m venv venv

# Activation (Linux/macOS)
source venv/bin/activate

# Activation (Windows)
.\venv\Scripts\activate
```

B. **Installation des dépendances** :
```bash
# Installation de FastAPI et du serveur ASGI Uvicorn
pip install fastapi "uvicorn[standard]"
```

### 4. Zone de Danger
❌ **À ne pas faire** : Installer FastAPI globalement sur votre machine avec `pip install` sans environnement virtuel.
✅ **Bonne Pratique** : Toujours utiliser un environnement virtuel et générer un fichier `requirements.txt` (`pip freeze > requirements.txt`) pour assurer la reproductibilité.

---

## Création du premier serveur {#création-du-premier-serveur-1}

### 1. Quoi
Un serveur FastAPI est une instance de la classe `FastAPI` qui définit les routes et la logique de traitement des requêtes. **Uvicorn** est le serveur **ASGI** (Asynchronous Server Gateway Interface) qui permet d'exécuter cette application.

### 2. Pourquoi
FastAPI utilise les capacités asynchrones de Python pour gérer un grand nombre de connexions simultanées avec une latence minimale.

### 3. Comment
A. **Code minimal** :
```python
# main.py
from fastapi import FastAPI

# Initialisation de l'application
app = FastAPI()

# Définition d'une route racine
@app.get("/")
async def root():
    # Retourne un dictionnaire qui sera converti en JSON
    return {"message": "Hello World"}
```

B. **Lancement du serveur** :
```bash
# Lancement avec rechargement automatique pour le développement
uvicorn main:app --reload
```

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Terminal affichant le démarrage réussi d'Uvicorn avec l'URL http://127.0.0.1:8000.
> **Alt Text** : Terminal montrant "Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)".

### 4. Zone de Danger
❌ **À ne pas faire** : Utiliser `--reload` en environnement de production (cela consomme des ressources inutilement).
✅ **Bonne Pratique** : Utiliser un serveur de production comme Gunicorn avec des workers Uvicorn pour le déploiement.

---

## Questions clés (validation des acquis du chapitre) {#questions-clés-(validation-des-acquis-du-chapitre)-1}

- **Pourquoi est-il recommandé d'utiliser un environnement virtuel pour un projet FastAPI ?**
  *Réponse : Pour isoler les dépendances du projet et éviter les conflits de versions entre différents projets Python.*

- **Quel est le rôle d'Uvicorn dans l'architecture FastAPI ?**
  *Réponse : Uvicorn est un serveur ASGI qui exécute l'application FastAPI et gère la communication entre le client et le framework.*

- **Que signifie l'option `--reload` lors du lancement d'Uvicorn ?**
  *Réponse : Elle permet de redémarrer automatiquement le serveur à chaque modification du code source, facilitant ainsi le développement.*