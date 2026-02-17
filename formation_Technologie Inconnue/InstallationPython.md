---
sidebar_label: "Installation de Python et Configuration de l'Environnement"
sidebar_position: 2
difficulty: "junior"
---

# Installation de Python et Configuration de l'Environnement {#installation-configuration-2}

Pour développer en Python, votre ordinateur a besoin de deux éléments essentiels : un **Interpréteur** (le moteur qui comprend et exécute le code) et un **Éditeur de code** (l'outil où vous écrivez le code). Ce chapitre vous guide pas à pas dans la mise en place d'un environnement de développement professionnel, identique à celui utilisé en entreprise.

## Architecture de votre Environnement de Développement {#architecture-environnement-2}

Avant d'installer les outils, il est important de visualiser comment ils interagissent. Contrairement à un logiciel classique (comme Word ou Excel), un environnement de développement est composé de plusieurs briques logicielles connectées.

```mermaid
graph TD
    User[Vous / Développeur] -->|Écrit du code .py| IDE[VS Code (Éditeur)]
    IDE -->|Analyse & Autocomplétion| Ext[Extension Python]
    Ext -->|Exécute & Débogue| Python[Interpréteur Python 3.x]
    Python -->|Traduit en binaire| OS[Système d'Exploitation]
    
    style IDE fill:#e1f5fe,stroke:#01579b,stroke-width:2px
    style Python fill:#fff9c4,stroke:#fbc02d,stroke-width:2px
    style OS fill:#e0e0e0,stroke:#616161,stroke-width:2px
```

## Étape 1 : Installation de l'Interpréteur Python {#installation-python-2}

L'installation varie selon votre système d'exploitation. Suivez la section correspondante.

### Option A : Windows

1.  Rendez-vous sur le site officiel : **python.org/downloads**.
2.  Téléchargez la dernière version stable (bouton jaune "Download Python 3.x.x").
3.  Lancez l'installateur.
4.  **TRES IMPORTANT** : Cochez la case **"Add Python to PATH"** en bas de la fenêtre avant de cliquer sur "Install Now". Si vous oubliez cette étape, vous ne pourrez pas lancer Python depuis votre terminal.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Fenêtre d'installation de Python sur Windows.
> **Alt Text** : Installateur Python Windows avec la case "Add Python to PATH" entourée en rouge pour insister sur son importance.

### Option B : macOS

MacOS est souvent livré avec une version ancienne de Python. Il est impératif d'installer la dernière version.

1.  Téléchargez l'installateur macOS 64-bit universal2 depuis **python.org**.
2.  Suivez les étapes classiques d'installation (Suivant, Suivant, Installer).
3.  Une fois terminé, un dossier s'ouvrira avec un script `Install Certificates.command`. Double-cliquez dessus pour gérer les certificats SSL (nécessaire pour télécharger des paquets plus tard).

### Option C : Linux (Ubuntu/Debian)

La plupart des distributions Linux incluent Python par défaut. Cependant, pour avoir la dernière version et les outils de développement :

```bash
sudo apt update
sudo apt install python3 python3-pip python3-venv
```

## Étape 2 : Installation de l'IDE (VS Code) {#installation-vscode-2}

Bien que vous puissiez écrire du Python dans le Bloc-notes, un IDE (Integrated Development Environment) vous offre la coloration syntaxique, la détection d'erreurs en temps réel et l'exécution intégrée. Nous utiliserons **Visual Studio Code (VS Code)**, le standard actuel du marché.

1.  Téléchargez et installez VS Code depuis **code.visualstudio.com**.
2.  Lancez VS Code.
3.  Dans la barre latérale gauche, cliquez sur l'icône des extensions (les quatre carrés).
4.  Recherchez "Python".
5.  Installez l'extension officielle développée par **Microsoft**.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Interface de VS Code montrant l'onglet Extensions.
> **Alt Text** : Recherche et installation de l'extension Python officielle de Microsoft dans VS Code.

## Étape 3 : Validation de l'Environnement {#validation-environnement-2}

Il est crucial de vérifier que tout communique correctement avant de commencer à coder.

### Vérification du Terminal

Ouvrez votre terminal (Invite de commandes ou PowerShell sur Windows, Terminal sur Mac/Linux) et tapez :

```bash
python --version
# Si cela ne fonctionne pas sur Mac/Linux, essayez :
python3 --version
```

Vous devez voir une réponse du type `Python 3.10.x` (ou supérieur). Si vous voyez une erreur "command not found", l'installation a échoué (probablement l'oubli du "Add to PATH" sur Windows).

### Votre Premier "Hello World"

Vérifions maintenant que l'interpréteur peut exécuter du code.
Dans votre terminal, tapez simplement `python` (ou `python3`) pour entrer dans le mode interactif (REPL) :

```python
>>> print("Environnement Valide !")
Environnement Valide !
```

Si le texte s'affiche, félicitations ! Votre environnement est prêt pour la production. Tapez `exit()` pour quitter.