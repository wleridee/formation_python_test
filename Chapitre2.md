---
sidebar_label: Installation et environnement de développement
sidebar_position: 2
---

# Chapitre 2 : Installation et environnement de développement

Concepts clés : pyenv (gestion de versions), pip et venv (dépendances), VS Code (éditeur), Hello World

Pour coder en Python efficacement et sans frustration, il est crucial de mettre en place un environnement de développement sain dès le départ. Ce chapitre vous guide pas à pas pour installer Python correctement (sans casser votre système d'exploitation) et configurer un éditeur moderne.

## pyenv (Gestion de versions) {#pyenv-gestion-de-versions}

### 1. Quoi
**pyenv** est un outil en ligne de commande qui permet d'installer et de gérer plusieurs versions de Python sur une même machine. Il permet de basculer facilement entre, par exemple, Python 3.8 pour un projet existant et Python 3.12 pour un nouveau projet.

### 2. Pourquoi
Le "System Python" (la version préinstallée sur macOS ou Linux) est utilisé par le système d'exploitation. Si vous installez des paquets dessus ou tentez de le mettre à jour, vous risquez de casser votre OS. **pyenv** isole vos installations Python de celles du système.

### 3. Comment

#### A. Installation (macOS / Linux)
Sous macOS et Linux, l'installation se fait généralement via un script automatique ou Homebrew.

```bash
# Installation recommandée (Linux / macOS)
curl https://pyenv.run | bash

# Ensuite, suivez les instructions affichées pour ajouter pyenv à votre .bashrc ou .zshrc
# Exemple pour Zsh (macOS par défaut) :
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
echo '[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
echo 'eval "$(pyenv init -)"' >> ~/.zshrc
```

#### B. Installation (Windows)
Windows nécessite une version spécifique appelée `pyenv-win`. Utilisez PowerShell :

```powershell
# Dans PowerShell (en tant qu'Administrateur si nécessaire)
Invoke-WebRequest -UseBasicParsing -Uri "https://raw.githubusercontent.com/pyenv-win/pyenv-win/master/pyenv-win/install-pyenv-win.ps1" -OutFile "./install-pyenv-win.ps1"; &"./install-pyenv-win.ps1"
```

#### C. Utilisation de base
Une fois installé, voici les commandes essentielles :

```bash
# Lister les versions disponibles à l'installation
pyenv install --list

# Installer Python 3.12
pyenv install 3.12.2

# Définir Python 3.12 comme version globale par défaut
pyenv global 3.12.2

# Vérifier la version active
python --version
```

### 4. Zone de Danger
❌ **Ne faites jamais** `sudo pip install ...`. Cela installe des paquets pour tout le système, avec des risques de conflits majeurs.
✅ **Utilisez toujours** `pyenv` pour gérer l'interpréteur, et `venv` (voir ci-dessous) pour les paquets.

---

## pip et venv (Dépendances) {#pip-et-venv-dependances}

### 1. Quoi
*   **pip** est le gestionnaire de paquets standard de Python (l'équivalent de `npm` pour Node.js).
*   **venv** est un module intégré pour créer des "environnements virtuels". Un environnement virtuel est un dossier isolé contenant une version de Python et des bibliothèques spécifiques à un projet.

### 2. Pourquoi
Imaginez le Projet A qui a besoin de `pandas` version 1.0 et le Projet B qui a besoin de `pandas` version 2.0. Si vous installez tout au même endroit, c'est le conflit assuré. `venv` crée une bulle étanche pour chaque projet.

### 3. Comment

#### A. Création et activation
Placez-vous dans le dossier de votre projet :

```bash
# 1. Créer l'environnement virtuel (nommé .venv par convention)
python -m venv .venv

# 2. Activer l'environnement
# Sur macOS / Linux :
source .venv/bin/activate

# Sur Windows (PowerShell) :
.venv\Scripts\Activate
```

Une fois activé, votre terminal affichera `(.venv)` au début de la ligne de commande.

#### B. Gestion des paquets avec pip
```bash
# Installer une librairie (dans l'environnement activé)
pip install requests

# Lister les paquets installés
pip list

# Figer les dépendances dans un fichier pour le partage
pip freeze > requirements.txt

# Installer depuis un fichier
pip install -r requirements.txt
```

### 🚨 Limitations de pip
`pip` ne gère pas nativement la résolution de conflits complexes aussi bien que des outils plus modernes comme **Poetry** ou **uv** (que nous verrons au chapitre 53). Pour débuter, `pip` + `venv` est la fondation indispensable à maîtriser.

---

## VS Code (Éditeur) {#vs-code-editeur}

### 1. Quoi
Visual Studio Code (VS Code) est un éditeur de code léger, gratuit et open-source, devenu le standard de facto pour le développement Python grâce à son écosystème d'extensions.

### 2. Pourquoi
Il offre l'IntelliSense (autocomplétion intelligente), le débogage intégré, et une excellente gestion des environnements virtuels Python sans configuration lourde.

### 3. Comment

#### A. Installation
Téléchargez et installez VS Code depuis [code.visualstudio.com](https://code.visualstudio.com/).

#### B. Configuration Python
1. Ouvrez VS Code.
2. Allez dans l'onglet **Extensions** (carrés à gauche ou `Ctrl+Shift+X`).
3. Cherchez et installez l'extension **Python** (publiée par Microsoft).

#### C. Sélectionner l'interpréteur
C'est l'étape la plus importante. Quand vous ouvrez un projet Python :
1. Ouvrez un fichier `.py`.
2. Regardez en bas à droite de la fenêtre, ou appuyez sur `Ctrl+Shift+P` (ou `Cmd+Shift+P` sur Mac) et tapez **"Python: Select Interpreter"**.
3. Sélectionnez l'interpréteur correspondant à votre environnement virtuel (`.venv`) ou à votre version pyenv (`3.12.x`).

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : La palette de commande VS Code montrant "Python: Select Interpreter".
> **Annotation** : Mettre en surbrillance l'option montrant un chemin vers `.venv` ou `pyenv`.
> **Alt Text suggéré** : Sélection de l'interpréteur Python dans VS Code via la palette de commandes.

### 4. Zone de Danger
❌ Ne codez pas sans sélectionner d'interpréteur. VS Code ne pourra pas vous aider pour l'autocomplétion et soulignera tout en rouge.
✅ Vérifiez toujours en bas à droite que vous êtes sur la bonne version de Python.

---

## Hello World {#hello-world}

### 1. Quoi
Le programme "Hello World" est traditionnellement le premier programme écrit pour vérifier que le langage et l'environnement sont correctement installés.

### 2. Pourquoi
Si vous parvenez à afficher du texte, cela valide toute la chaîne : OS -> Pyenv -> Python -> VS Code -> Terminal.

### 3. Comment

#### A. Création du fichier
Créez un fichier nommé `main.py` et écrivez :

```python
# main.py
def dire_bonjour(nom: str) -> None:
    """Affiche un message de bienvenue."""
    print(f"Bonjour, {nom} ! Ton environnement Python 3.12 est prêt 🚀")

if __name__ == "__main__":
    dire_bonjour("Développeur")
```

#### B. Exécution
Ouvrez le terminal intégré de VS Code (`Ctrl+ù` ou Terminal > New Terminal) et lancez :

```bash
python main.py
```

Vous devriez voir : `Bonjour, Développeur ! Ton environnement Python 3.12 est prêt 🚀`

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-validation-des-acquis-du-chapitre-2}

**1. Pourquoi ne faut-il jamais utiliser le "System Python" pour vos projets de développement ?**
Car toute modification ou installation de paquet sur le System Python risque de corrompre des fonctionnalités vitales du système d'exploitation.

**2. Quelle est la différence entre `pyenv` et `venv` ?**
`pyenv` gère les **versions** de Python (ex: 3.10 vs 3.12) installées sur la machine, tandis que `venv` crée des environnements isolés pour gérer les **paquets/librairies** d'un projet spécifique.

**3. Comment savoir si votre environnement virtuel est actif dans le terminal ?**
Le nom de l'environnement (par exemple `(.venv)`) apparaît entre parenthèses au tout début de l'invite de commande (prompt).

**4. Que faire si VS Code ne souligne pas les erreurs ou ne propose pas l'autocomplétion ?**
Il faut vérifier qu'un interpréteur Python est bien sélectionné (via `Python: Select Interpreter`) et que l'extension Python de Microsoft est installée.

---

## Exercices : {#exercices-:-2}

⚠️ *Note : S'agissant d'un chapitre d'installation, les exercices sont des étapes de vérification.*

### Exercice 1 - La Preuve de Version {#exercice-1---la-preuve-de-version}
🎯 **Objectif** : Valider l'installation de pyenv et Python 3.12.
💼 **Mise en situation** : Vous intégrez une startup SaaS. Le CTO vous demande de configurer votre machine avec la dernière version stable.
📝 **Énoncé** : Installez Python 3.12.x avec pyenv. Créez un dossier `startup-config`. Dans ce dossier, forcez l'utilisation de Python 3.12 (via `pyenv local`). Affichez la version.
📺 **Résultat attendu** :
```bash
$ python --version
Python 3.12.2 (ou version supérieure)
```

### Exercice 2 - L'Isolation {#exercice-2---l-isolation}
🎯 **Objectif** : Créer et activer un environnement virtuel.
💼 **Mise en situation** : Vous devez tester une librairie sans polluer votre installation principale.
📝 **Énoncé** : Dans le dossier `startup-config`, créez un venv nommé `.venv`. Activez-le. Vérifiez où se trouve l'exécutable python.
📺 **Résultat attendu** :
```bash
$ which python  # ou 'Get-Command python' sur Windows PowerShell
.../startup-config/.venv/bin/python
```

### Exercice 3 - Le Premier Script Sécurisé {#exercice-3---le-premier-script-securise}
🎯 **Objectif** : Configurer VS Code et lancer un script.
💼 **Mise en situation** : Écrivez un script qui vérifie que vous n'êtes PAS sur une version legacy (Python 2).
📝 **Énoncé** : Créez `check.py` dans VS Code. Le script doit importer `sys` et afficher `sys.version`. Lancez-le via le bouton "Run" de VS Code.
📺 **Résultat attendu** :
L'affichage de la version 3.12.x dans le terminal intégré.
💡 **Solution** :
<details>
<summary>Voir le code complet commenté</summary>

```python
import sys

# sys.version contient les détails de la version de l'interpréteur actif
print("Version active :")
print(sys.version)

if sys.version_info.major < 3:
    print("❌ Attention : Vous utilisez Python 2 !")
else:
    print("✅ Parfait : Vous utilisez Python 3")
```
</details>