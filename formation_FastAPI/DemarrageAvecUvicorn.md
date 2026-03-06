---
sidebar_label: "Démarrage de l'Application avec Uvicorn"
sidebar_position: 4
difficulty: "junior"
---

# Démarrage de l'Application avec Uvicorn {#demarrage-de-lapplication-avec-uvicorn-4}

Nous avons écrit notre première API, mais le code Python seul ne fait rien. Il ne sait pas écouter les requêtes HTTP venant d'un navigateur ou d'un autre service. Pour que notre application FastAPI devienne un serveur web fonctionnel, nous avons besoin d'un intermédiaire : un serveur **ASGI**. C'est là qu'intervient **Uvicorn**.

Ce chapitre est entièrement dédié à la commande `uvicorn`, l'outil que vous utiliserez constamment pendant le développement pour lancer, tester et déboguer votre application.

## Le Rôle du Serveur ASGI {#le-role-du-serveur-asgi-4}

Avant de détailler la commande, visualisons où se situe Uvicorn dans notre architecture.

```mermaid
graph TD
    Client[Navigateur / Client API] -- "1. Requête HTTP (GET /)" --> Uvicorn
    Uvicorn -- "2. Traduit la requête pour Python" --> FastAPI
    subgraph "Votre Code"
        FastAPI[Instance FastAPI 'app'] -- "3. Trouve la bonne fonction" --> MaFonction[def ma_fonction():]
        MaFonction -- "4. Retourne un dict" --> FastAPI
    end
    FastAPI -- "5. Formate la réponse" --> Uvicorn
    Uvicorn -- "6. Envoie la réponse HTTP" --> Client

    style Uvicorn fill:#FFC107,stroke:#333,stroke-width:2px
```

Uvicorn est le pont entre le monde extérieur (le réseau) et notre application FastAPI. Il écoute les requêtes entrantes, les transmet à FastAPI dans un format standardisé (la spécification ASGI), puis reprend la réponse de FastAPI pour l'envoyer au client.

## La Commande `uvicorn` Décortiquée {#la-commande-uvicorn-decortiquee-4}

### Quoi ? {#quoi-4}
`uvicorn` est une application en ligne de commande qui lance un serveur web ASGI. Son rôle principal est de charger votre application FastAPI en mémoire et de la rendre accessible via une adresse IP et un port. La commande que nous avons déjà utilisée est un excellent point de départ : `uvicorn main:app --reload`.

### Pourquoi ? {#pourquoi-4}
Sans Uvicorn, votre fichier `main.py` n'est qu'un script Python. Si vous l'exécutez avec `python main.py`, il se terminera immédiatement sans rien faire. Uvicorn met en place une boucle d'événements qui écoute en permanence les nouvelles connexions réseau, permettant à votre application de traiter des milliers de requêtes sans que vous ayez à gérer cette complexité.

### Comment (Syntaxe + Cas Réel) ? {#comment-syntaxe--cas-reel-4}

La syntaxe de base est `uvicorn <module>:<instance> [options]`. Analysons chaque partie :

1.  **`main:app`** : C'est la partie la plus importante.
    *   `main` : Fait référence au fichier Python qui contient votre application, ici `main.py`. Uvicorn est assez intelligent pour comprendre que `main` correspond à `main.py`.
    *   `app` : Fait référence à l'objet **à l'intérieur** du fichier `main.py` qui est l'instance de l'application ASGI. Dans notre code, c'est la ligne `app = FastAPI()`. Si vous aviez nommé votre variable `api = FastAPI()`, la commande serait `uvicorn main:api`.

2.  **`--reload`** : L'option magique pour le développement.
    *   Quand cette option est activée, Uvicorn surveille les modifications de vos fichiers Python. Dès que vous enregistrez un changement, le serveur redémarre automatiquement avec le nouveau code. Cela vous évite d'avoir à arrêter et relancer manuellement le serveur à chaque modification, ce qui accélère considérablement le cycle de développement.

    > 📸 **CAPTURE D'ÉCRAN REQUISE**
    > **Sujet** : Terminal montrant Uvicorn qui se recharge après une modification de fichier.
    > **Alt Text** : Logs Uvicorn affichant "Application startup complete." suivi de "Detected file change in 'main.py'. Reloading...", puis à nouveau "Application startup complete.".

3.  **Changer le port avec `--port`**
    *   Par défaut, Uvicorn démarre sur le port `8000`. Si ce port est déjà utilisé par une autre application, vous pouvez en choisir un autre.
    *   **Exemple** : Pour lancer l'application sur le port 5000.
        ```bash
        uvicorn main:app --port 5000
        ```
        Votre API sera alors accessible sur `http://127.0.0.1:5000`.

4.  **Changer l'hôte avec `--host`**
    *   Par défaut, Uvicorn n'est accessible que depuis votre propre machine (`127.0.0.1` ou `localhost`). Si vous voulez rendre votre API accessible à d'autres appareils sur votre réseau local (par exemple, pour la tester depuis votre téléphone), vous devez utiliser l'hôte `0.0.0.0`.
    *   **Exemple** : Rendre l'API accessible sur le réseau local.
        ```bash
        uvicorn main:app --host 0.0.0.0
        ```
        Uvicorn affichera une URL comme `http://0.0.0.0:8000`. Vous devrez remplacer `0.0.0.0` par l'adresse IP locale de votre machine (ex: `192.168.1.10`) pour y accéder depuis un autre appareil.

### Zone de Danger {#zone-de-danger-4}
*   **Oublier d'activer l'environnement virtuel** : Si vous exécutez `uvicorn ...` sans avoir activé votre `venv`, votre terminal vous dira `command not found: uvicorn`, car l'exécutable n'est pas dans le `PATH` de votre système global.
*   **Utiliser `--reload` en production** : Le rechargement automatique consomme des ressources supplémentaires pour surveiller les fichiers. C'est parfait pour le développement, mais il ne faut **jamais** l'utiliser sur un serveur de production. En production, on lance le serveur une seule fois, de manière plus robuste (souvent avec Gunicorn comme gestionnaire de processus pour Uvicorn, ce que nous verrons dans les chapitres avancés).
*   **Erreur de syntaxe `ModuleNotFoundError`** : Si vous faites une faute de frappe, par exemple `uvicorn mian:app`, Uvicorn ne trouvera pas le fichier `mian.py` et lèvera une erreur. Vérifiez toujours que le nom du module et de l'instance sont corrects.

---

## Conclusion et Validation {#conclusion-et-validation-4}

Vous savez maintenant comment donner vie à votre application FastAPI grâce à Uvicorn. Maîtriser sa commande de lancement et ses options de base (`--reload`, `--port`, `--host`) est une compétence fondamentale que vous utiliserez tous les jours.

### 3 Questions Clés {#3-questions-cles-4}

1.  Dans la commande `uvicorn my_api:server_instance`, à quoi correspondent `my_api` et `server_instance` ?
2.  Pourquoi l'option `--reload` est-elle si utile pendant le développement mais déconseillée en production ?
3.  Quelle commande utiliseriez-vous pour démarrer votre application (fichier `main.py`, instance `app`) sur le port `8888` et la rendre visible sur votre réseau local ?

### 3 Exercices Progressifs {#3-exercices-progressifs-4}

Pour ces exercices, utilisez le fichier `main.py` des chapitres précédents. Assurez-vous d'arrêter le serveur Uvicorn (`Ctrl+C` dans le terminal) entre chaque exercice.

**Exercice 1 : Changer de Port**
Lancez votre application sur le port `9999`. Ouvrez votre navigateur et vérifiez qu'elle est bien accessible sur `http://127.0.0.1:9999` et qu'elle n'est plus accessible sur le port par défaut `8000`.

<details>
<summary>Découvrir la solution commentée</summary>

```bash
# 1. Lancez cette commande dans votre terminal
uvicorn main:app --port 9999 --reload

# 2. Ouvrez votre navigateur et visitez http://127.0.0.1:9999.
#    Vous devriez voir {"Hello":"World"}.

# 3. Essayez de visiter http://127.0.0.1:8000.
#    Le navigateur devrait afficher une erreur "Ce site est inaccessible".
```
</details>

**Exercice 2 : Renommer l'instance de l'application**
1.  Dans votre fichier `main.py`, changez la ligne `app = FastAPI()` en `api_instance = FastAPI()`.
2.  N'oubliez pas de changer également les décorateurs (ex: `@app.get` devient `@api_instance.get`).
3.  Trouvez et exécutez la commande `uvicorn` correcte pour lancer cette application modifiée.

<details>
<summary>Découvrir la solution commentée</summary>

```python
# main.py (modifié)
from fastapi import FastAPI

# L'instance a été renommée ici
api_instance = FastAPI()

# Le décorateur doit utiliser le nouveau nom
@api_instance.get("/")
def read_root():
    return {"Hello": "World"}

# ... (modifiez les autres décorateurs si vous en avez)
```

**Commande à exécuter :**
```bash
# On spécifie le nouveau nom de l'instance après les deux-points
uvicorn main:api_instance --reload
```
</details>

**Exercice 3 : Changer la Structure du Projet**
1.  Dans votre dossier `formation-fastapi`, créez un nouveau dossier nommé `source`.
2.  Déplacez votre fichier `main.py` à l'intérieur de `source`.
3.  Depuis le dossier racine (`formation-fastapi`), exécutez la commande `uvicorn` qui permet de lancer l'application qui se trouve maintenant dans `source/main.py`.

*Indice : La notation des modules en Python utilise le point (`.`).*

<details>
<summary>Découvrir la solution commentée</summary>

Votre structure de fichiers devrait ressembler à ceci :
```
formation-fastapi/
├── source/
│   └── main.py
└── venv/
```

**Commande à exécuter depuis le dossier `formation-fastapi` :**
```bash
# On indique à uvicorn le chemin du module en utilisant la notation Python
# "source.main" correspond au fichier source/main.py
uvicorn source.main:app --reload
```
*Note : Cette façon de structurer le code est très courante dans les projets plus grands pour séparer le code source des autres fichiers (tests, documentation, etc.).*
</details>