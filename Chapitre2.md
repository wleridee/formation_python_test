---
sidebar_label: Installation et environnement de développement
sidebar_position: 2
---

# Chapitre 2 : Installation et environnement de développement

Concepts clés : pyenv, pip, VS Code & Extensions, Hello World

## Gestion des versions avec Pyenv (vs System Python) {#gestion-des-versions-avec-pyenv-vs-system-python}

### 1. Quoi
**Pyenv** est un outil en ligne de commande qui permet de gérer plusieurs versions de Python sur une même machine. Il intercepte les commandes Python et les redirige vers la version spécifique définie pour votre projet ou votre utilisateur.

### 2. Pourquoi
Sur macOS et Linux, une version de Python est souvent préinstallée ("System Python") car le système d'exploitation en a besoin pour fonctionner. **Il ne faut jamais toucher à cette version système** (pas d'installation de paquets, pas de mises à jour manuelles), sous peine de casser des fonctionnalités de l'OS.

De plus, différents projets peuvent nécessiter différentes versions de Python (3.10 pour un vieux projet, 3.12 pour un nouveau). Pyenv permet de passer de l'une à l'autre sans conflit.

### 3. Comment

#### macOS et Linux (Recommandé : Pyenv)
L'installation standard se fait via le script automatique :

```bash
curl https://pyenv.run | bash
```
*Note : Après installation, suivez les instructions affichées dans le terminal pour ajouter pyenv à votre fichier de configuration shell (.bashrc, .zshrc).*

Commandes utiles :
- `pyenv install 3.12.0` : Installe une version spécifique.
- `pyenv global 3.12.0` : Définit la version par défaut pour l'utilisateur.
- `pyenv local 3.12.0` : Définit la version uniquement pour le dossier actuel (crée un fichier `.python-version`).

#### Windows
Sur Windows, **pyenv** n'est pas supporté nativement. Deux options s'offrent à vous :
1.  **Pour débuter (Simple)** : Utilisez l'installateur officiel de [python.org](https://www.python.org/downloads/windows/).
    *   ⚠️ **IMPORTANT** : Cochez la case **"Add Python to PATH"** lors de l'installation.
2.  **Pour les utilisateurs avancés** : Utilisez le portage `pyenv-win` ou travaillez sous WSL2 (Windows Subsystem for Linux) pour utiliser le vrai `pyenv`.

### 4. Zone de Danger
❌ **À ne pas faire** : Utiliser `sudo pip install` sur macOS ou Linux. Cela installe des paquets dans le Python du système avec des droits d'administrateur, ce qui crée des risques de sécurité et de stabilité majeurs.

✅ **Bonne Pratique** : Toujours utiliser une version de Python installée par l'utilisateur (via pyenv ou l'installateur Windows) et non celle du système.

---

## Le gestionnaire de paquets : Pip {#le-gestionnaire-de-paquets-pip}

### 1. Quoi
**Pip** (Package Installer for Python) est l'outil standard pour installer et gérer des bibliothèques logicielles écrites en Python. Il se connecte au **PyPI** (Python Package Index), le dépôt officiel des paquets tiers.

### 2. Pourquoi
Python est livré avec une bibliothèque standard riche ("batteries included"), mais la force de Python réside dans son écosystème communautaire (Django pour le web, Pandas pour la data, etc.). Pip est la porte d'entrée vers cet écosystème.

### 3. Comment
Pip est installé automatiquement avec les versions modernes de Python (à partir de 3.4).

```bash
# Vérifier la version
pip --version

# Installer un paquet
pip install nom_du_paquet

# Lister les paquets installés
pip list
```

### 4. Zone de Danger
❌ **À ne pas faire** : Installer des paquets globalement sans environnement virtuel (voir Chapitre 29). Cela peut créer des conflits de versions entre vos projets.

---

## Éditeur de code : VS Code & Extensions {#editeur-de-code-vs-code-extensions}

### 1. Quoi
**Visual Studio Code (VS Code)** est un éditeur de code source léger mais puissant, devenu le standard de facto pour le développement Python moderne. Il est extensible via un système de plugins.

### 2. Pourquoi
Il offre un équilibre parfait entre légèreté et fonctionnalités : coloration syntaxique, auto-complétion intelligente (IntelliSense), débogage intégré et support natif de Git.

### 3. Comment

1.  Téléchargez et installez VS Code depuis [code.visualstudio.com](https://code.visualstudio.com/).
2.  Ouvrez VS Code, allez dans l'onglet **Extensions** (carrés sur la gauche) ou faites `Ctrl+Shift+X`.
3.  Recherchez et installez l'extension **Python** officielle de Microsoft (ID : `ms-python.python`).

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Interface des extensions VS Code montrant l'extension Python officielle.
> **Annotation** : Mettre en évidence l'extension "Python" de Microsoft et le bouton "Install".
> **Alt Text suggéré** : Installation de l'extension Python pour VS Code.

Cette extension installera automatiquement d'autres outils utiles comme **Pylance** (pour l'analyse de code performante) et le **Python Debugger**.

### 4. Zone de Danger
❌ **À ne pas faire** : Installer des dizaines d'extensions non maintenues ou redondantes. L'extension officielle suffit pour 99% des besoins initiaux.

✅ **Bonne Pratique** : Vérifiez toujours en bas à droite de VS Code quel interpréteur Python est sélectionné. Il doit correspondre à la version que vous avez installée (ex: 3.12.x).

---

## Hello World : Le premier script {#hello-world-le-premier-script}

### 1. Quoi
Le "Hello World" est le programme le plus simple possible. Il sert à valider que tout votre environnement (interpréteur, éditeur, terminal) est correctement configuré.

### 2. Pourquoi
Si ce script ne fonctionne pas, rien de plus complexe ne fonctionnera. C'est le test de fumée ("smoke test") de votre installation.

### 3. Comment

1.  Créez un dossier pour votre projet et ouvrez-le avec VS Code.
2.  Créez un fichier nommé `main.py`.
3.  Ajoutez le code suivant :

```python
print("Hello Python 3.12 !")
```

4.  Ouvrez le terminal intégré dans VS Code (`Ctrl+ù` ou menu *Terminal > New Terminal*).
5.  Exécutez le script :

```bash
python main.py
# Sur certains systèmes Linux/Mac anciens, il faut parfois taper python3 :
# python3 main.py
```

### 4. Zone de Danger
❌ **Erreur classique** : Taper simplement `python` dans le terminal sans nom de fichier. Cela ouvre le **REPL** (l'interpréteur interactif), où vous pouvez taper du code ligne par ligne, mais ce n'est pas comme cela qu'on lance un fichier `.py`. Pour quitter le REPL, tapez `exit()`.

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-2}

**1. Pourquoi ne faut-il pas utiliser le Python préinstallé sur macOS ou Linux pour développer ?**
Il est réservé au système d'exploitation. Le modifier ou mettre à jour ses paquets peut rendre le système instable ou casser des outils système.

**2. Quelle case est critique de cocher lors de l'installation de Python sur Windows ?**
La case **"Add Python to PATH"**. Sans elle, la commande `python` ne sera pas reconnue dans le terminal.

**3. Quelle est l'extension VS Code indispensable pour le développement Python ?**
L'extension **Python** développée par Microsoft (`ms-python.python`).

**4. Quelle est la différence entre exécuter `python` et `python main.py` ?**
`python` ouvre le mode interactif (REPL) pour tester des commandes en direct. `python main.py` demande à l'interpréteur de lire et d'exécuter le contenu du fichier `main.py`.