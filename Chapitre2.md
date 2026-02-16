---
sidebar_label: Installation et environnement de développement
sidebar_position: 2
---

# Chapitre 2 : Installation et environnement de développement

Installation du runtime (pyenv), Gestionnaire de paquets (pip), Environnements virtuels (venv), Configuration VS Code, Hello World

Avoir un environnement de développement propre et reproductible est la première étape pour devenir un développeur professionnel. Installer Python "n'importe comment" (via l'installateur Windows par défaut ou via le gestionnaire de paquets système Linux) peut mener à des conflits de versions infernaux.

Ce chapitre vous guide vers une installation moderne, isolée et robuste de Python 3.14.

---

## 1. Installation du Runtime avec `pyenv` {#installation-du-runtime-avec-pyenv}

### 1. Quoi
**pyenv** est un outil qui permet d'installer et de gérer plusieurs versions de Python sur la même machine. Il ne remplace pas le Python système (utilisé par votre OS), mais installe des versions indépendantes dans votre dossier utilisateur.

### 2. Pourquoi
*   **Isolation** : Ne jamais casser son système d'exploitation en modifiant le Python système.
*   **Flexibilité** : Passer de Python 3.12 à 3.14 en une commande selon les besoins du projet.
*   **Permissions** : Installer des paquets sans avoir besoin de `sudo` ou de droits d'administrateur.

### 3. Comment

#### A. Installation de pyenv

*   **MacOS / Linux** :
    ```bash
    curl https://pyenv.run | bash
    ```
    *Suivez ensuite les instructions affichées pour ajouter pyenv à votre `.bashrc` ou `.zshrc`.*

*   **Windows** :
    Utilisez le fork `pyenv-win` via PowerShell :
    ```powershell
    Invoke-WebRequest -UseBasicParsing -Uri "https://raw.githubusercontent.com/pyenv-win/pyenv-win/master/pyenv-win/install-pyenv-win.ps1" -OutFile "./install-pyenv-win.ps1"; &"./install-pyenv-win.ps1"
    ```

#### B. Installation de Python 3.14

Une fois pyenv installé et configuré :

```bash
# Lister les versions disponibles
pyenv install --list

# Installer Python 3.14
pyenv install 3.14.0

# Définir cette version comme globale pour votre utilisateur
pyenv global 3.14.0

# Vérifier
python --version
# Doit afficher : Python 3.14.0
```

### 4. Zone de Danger

❌ **À ne pas faire** :
*   Télécharger l'installateur `.exe` ou `.pkg` depuis python.org si vous comptez gérer plusieurs projets. C'est acceptable pour un débutant absolu, mais `pyenv` est la voie royale.
*   Utiliser `sudo pip install` sur Linux/Mac. Cela installe des paquets dans le système global, ce qui est dangereux.

✅ **Bonne Pratique** :
*   Toujours vérifier `which python` (ou `Get-Command python` sur Windows) pour s'assurer qu'on utilise bien la version de pyenv (le chemin doit ressembler à `~/.pyenv/shims/python`).

---

## 2. Environnements Virtuels (`venv`) {#environnements-virtuels-venv}

### 1. Quoi
Un **environnement virtuel** est un dossier isolé contenant une copie de l'interpréteur Python et un dossier pour les bibliothèques tierces. Chaque projet doit avoir son propre environnement virtuel.

### 2. Pourquoi
Imaginez deux projets : le Projet A a besoin de `Django 4.0` et le Projet B de `Django 5.0`. Si vous installez Django globalement, vous ne pouvez en avoir qu'une seule version. Les environnements virtuels résolvent ce conflit.

### 3. Comment

Dans le terminal, naviguez vers le dossier de votre projet :

```bash
# 1. Créer l'environnement virtuel (dossier .venv)
python -m venv .venv

# 2. Activer l'environnement
# Sur MacOS/Linux :
source .venv/bin/activate

# Sur Windows (PowerShell) :
.venv\Scripts\Activate
```

Une fois activé, votre invite de commande affichera souvent `(.venv)`.

### 4. Zone de Danger

❌ **À ne pas faire** :
*   Commettre le dossier `.venv` dans Git. Ajoutez-le toujours au fichier `.gitignore`.
*   Renommer ou déplacer le dossier `.venv` après sa création. Les chemins sont "hardcodés" à l'intérieur. Si vous déplacez le projet, supprimez et recréez le venv.

---

## 3. Gestionnaire de Paquets (`pip`) {#gestionnaire-de-paquets-pip}

### 1. Quoi
**pip** (Pip Installs Packages) est l'outil standard pour installer des bibliothèques depuis le PyPI (Python Package Index). Il est installé par défaut avec Python.

### 2. Pourquoi
Pour ajouter des fonctionnalités à votre code (ex: requêtes HTTP avec `requests`, analyse de données avec `pandas`) sans réinventer la roue.

### 3. Comment

```bash
# (Assurez-vous que votre venv est activé)

# Installer un paquet
pip install requests

# Installer une version précise
pip install requests==2.31.0

# Mettre à jour pip lui-même
pip install --upgrade pip

# Lister les paquets installés
pip list
```

Pour partager votre projet, vous devez lister vos dépendances dans un fichier :

```bash
# Générer le fichier requirements.txt
pip freeze > requirements.txt

# Installer les dépendances depuis ce fichier (sur une autre machine)
pip install -r requirements.txt
```

### 🚨 Limitations de `pip`
`pip` gère mal les dépendances complexes et les conflits. Pour des projets professionnels, on préfère aujourd'hui des outils comme **Poetry** ou **uv** (voir Chapitre 48), mais `pip` reste la base indispensable à connaître.

---

## 4. Configuration de VS Code {#configuration-de-vs-code}

### 1. Quoi
**Visual Studio Code (VS Code)** est l'éditeur de code le plus populaire pour Python. Il est léger, extensible et gratuit.

### 2. Pourquoi
Avec la bonne extension, VS Code offre :
*   L'autocomplétion intelligente (IntelliSense).
*   Le débogage pas à pas.
*   La détection automatique des environnements virtuels.
*   Le linting (analyse statique des erreurs).

### 3. Comment

1.  **Installation** : Téléchargez et installez VS Code depuis [code.visualstudio.com](https://code.visualstudio.com/).
2.  **Extensions** : Installez l'extension officielle **Python** de Microsoft (id: `ms-python.python`).
3.  **Sélection de l'interpréteur** :
    *   Ouvrez votre dossier de projet dans VS Code.
    *   Ouvrez la palette de commandes (`Ctrl+Shift+P` ou `Cmd+Shift+P`).
    *   Tapez "Python: Select Interpreter".
    *   Choisissez l'interpréteur situé dans votre dossier `.venv` (recommandé) ou votre version pyenv globale.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : La palette de sélection d'interpréteur dans VS Code.
> **Annotation** : Mettre en évidence l'interpréteur marqué comme "('.venv': venv)".
> **Alt Text suggéré** : Sélection de l'interpréteur Python virtuel dans VS Code.

---

## 5. Hello World : Votre premier programme {#hello-world-votre-premier-programme}

### 1. Quoi
La tradition veut que le premier programme affiche simplement "Bonjour le monde". C'est le test ultime pour vérifier que toute la chaîne (Python, Venv, IDE) fonctionne.

### 2. Comment

1.  Créez un fichier `main.py`.
2.  Écrivez le code suivant :

```python
import sys

def main():
    # Affiche la version de Python utilisée pour confirmer l'environnement
    print(f"Exécution avec Python {sys.version}")
    print("Hello, World!")

if __name__ == "__main__":
    main()
```

3.  Exécutez-le via le terminal intégré de VS Code : `python main.py`

### 3. Analyse
Si le terminal affiche `Hello, World!` et la version `3.14.x`, félicitations : votre environnement de développement est opérationnel.

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-2}

1.  **Pourquoi est-il risqué d'installer des paquets avec `pip install` sans environnement virtuel activé ?**
    Cela installe les paquets dans l'environnement Python global (ou utilisateur global), ce qui peut créer des conflits de versions entre différents projets et potentiellement casser des outils système dépendants de Python.

2.  **Quelle est la commande pour créer un environnement virtuel nommé `.venv` ?**
    `python -m venv .venv` (Windows/Mac/Linux).

3.  **À quoi sert le fichier `requirements.txt` généré par `pip freeze` ?**
    Il sert à lister toutes les bibliothèques installées et leurs versions exactes, permettant à un autre développeur (ou à un serveur de déploiement) de réinstaller le même environnement avec `pip install -r requirements.txt`.