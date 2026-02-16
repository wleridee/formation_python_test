---
sidebar_label: Packaging Python : Gestion de Dépendances avec Poetry
sidebar_position: 48
---

# Chapitre 48 : Packaging Python : Gestion de Dépendances avec Poetry

Installation de Poetry, Création de projet, Ajout/Suppression de dépendances, Lock file

Dans le chapitre précédent, nous avons vu comment structurer un projet et utiliser `setuptools`. Cependant, gérer manuellement les dépendances, les environnements virtuels et la publication peut devenir laborieux.

**Poetry** est l'outil tout-en-un qui modernise le développement Python. Il remplace `setup.py`, `requirements.txt`, `pip`, `venv` et `twine` par une seule interface en ligne de commande cohérente. C'est l'équivalent de `npm` (Node.js) ou `cargo` (Rust) pour Python.

---

## 1. Installation et Initialisation de Projet {#installation-et-initialisation}

### 1. Quoi
Poetry est un outil CLI (Command Line Interface) qui doit être installé globalement sur votre machine (pas dans un environnement virtuel de projet). Il permet ensuite de créer des structures de projets prêtes à l'emploi.

### 2. Pourquoi
Pour démarrer un projet avec une architecture saine (Src Layout), une configuration `pyproject.toml` valide et un environnement virtuel isolé automatique, sans taper 10 commandes différentes.

### 3. Comment

#### A. Installation (Recommandée via pipx)
En 2026, la bonne pratique pour installer des outils Python globaux est `pipx`.

```bash
# Installe poetry de manière isolée sur votre système
pipx install poetry

# Vérification
poetry --version
```

#### B. Créer un nouveau projet
```bash
poetry new mon-super-projet
```
Cela génère :
```text
mon-super-projet/
├── pyproject.toml      # La configuration
├── README.md
├── src/
│   └── mon_super_projet/
│       └── __init__.py
└── tests/
    └── __init__.py
```

#### C. Initialiser Poetry dans un projet existant
Si vous avez déjà du code :
```bash
cd mon-vieux-projet
poetry init
# Répondez aux questions interactives pour générer le pyproject.toml
```

---

## 2. Gestion des Dépendances (Add / Remove) {#gestion-dependances}

### 1. Quoi
Au lieu d'éditer manuellement un fichier texte ou de faire `pip install X` puis `pip freeze > requirements.txt`, vous "commandez" à Poetry d'ajouter une librairie.

### 2. Pourquoi
Poetry vérifie **immédiatement** la compatibilité des versions (résolution de dépendances). Si la librairie A veut `numpy<1.20` et la librairie B veut `numpy>1.22`, Poetry vous avertira du conflit *avant* de casser votre environnement.

### 3. Comment

#### A. Ajouter une dépendance principale
```bash
poetry add requests
```
Cette commande :
1.  Télécharge `requests`.
2.  L'installe dans l'environnement virtuel du projet (créé automatiquement s'il n'existe pas).
3.  Ajoute la ligne dans `pyproject.toml`.
4.  Verrouille la version exacte dans `poetry.lock`.

#### B. Ajouter une dépendance de développement
Outils nécessaires pour coder/tester, mais pas pour exécuter l'app en production (ex: pytest, linters).

```bash
poetry add --group dev pytest
```

#### C. Voir le résultat dans `pyproject.toml`
```toml
[tool.poetry.dependencies]
python = "^3.14"
requests = "^2.31.0"

[tool.poetry.group.dev.dependencies]
pytest = "^8.0.0"
```

### 4. Zone de Danger
❌ **Ne jamais installer avec `pip` dans un projet Poetry** : Si vous faites `pip install pandas` dans le terminal, Poetry ne sera pas au courant. Au prochain déploiement, `pandas` manquera. Passez toujours par `poetry add`.

---

## 3. Le Fichier `poetry.lock` : La Garantie de Reproductibilité {#fichier-lock}

### 1. Quoi
Le fichier `poetry.lock` est un fichier généré automatiquement qui contient l'arbre exact de toutes les dépendances (y compris les sous-dépendances) avec leurs versions précises (hash cryptographique inclus).

### 2. Pourquoi
C'est la solution au problème : **"Ça marche sur ma machine mais pas en prod"**.
*   `pyproject.toml` dit : "Je veux `requests` (peu importe la version mineure tant qu'elle est compatible)".
*   `poetry.lock` dit : "J'installe `requests` version 2.31.0 et `urllib3` version 2.0.7 exactement".

### 3. Comment

*   **En développement (Local)** : Quand vous faites `poetry add`, le lockfile se met à jour.
*   **En production / CI** : On utilise le lockfile pour installer l'environnement exact.

```bash
# Installe EXACTEMENT les versions du fichier .lock
poetry install
```

#### D. Tableau : Install vs Update

| Commande | Action sur `pyproject.toml` | Action sur `poetry.lock` | Cas d'usage |
| :--- | :--- | :--- | :--- |
| `poetry install` | Lit le fichier | **Lit** le fichier (Respect strict) | Déploiement, nouveau collègue qui clone le repo |
| `poetry update` | Lit le fichier | **Réécrit** le fichier (Mise à jour) | Maintenance, monter de version les dépendances |

### 🚨 Limitations de Poetry
Le "Solver" de dépendances de Poetry est très strict. Parfois, il peut mettre du temps à trouver une solution si vous avez beaucoup de dépendances avec des contraintes de versions conflictuelles.

---

## 4. Exécuter le code (Environnements Virtuels) {#executer-code}

### 1. Quoi
Poetry gère les environnements virtuels (`venv`) de manière transparente. Vous n'avez pas besoin de faire `source .venv/bin/activate`.

### 2. Pourquoi
Pour garantir que vos scripts s'exécutent bien avec les librairies du projet, et non celles de votre système global.

### 3. Comment

**Option A : Préfixe `poetry run` (Ponctuel)**
```bash
poetry run python src/main.py
poetry run pytest
```

**Option B : Entrer dans le shell (Session)**
```bash
poetry shell
# Vous êtes maintenant dans le venv
python src/main.py
exit
```

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Terminal montrant l'exécution de `poetry add requests`
> **Annotation** : Montrez les étapes "Resolving dependencies", "Writing lock file" et l'installation.
> **Alt Text suggéré** : Interface CLI de Poetry ajoutant une dépendance et résolvant l'arbre de dépendances.

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-48}

1.  **Pourquoi faut-il commiter le fichier `poetry.lock` dans Git ?**
    Pour garantir que tous les développeurs et le serveur de production utilisent *exactement* les mêmes versions de dépendances, au bit près.

2.  **Quelle est la différence entre `poetry add` et `pip install` ?**
    `poetry add` modifie les fichiers de configuration (`pyproject.toml`, `poetry.lock`) et gère la compatibilité, alors que `pip install` installe juste le paquet sans laisser de trace pérenne dans la config du projet.

3.  **Comment installer uniquement les dépendances nécessaires pour lancer l'application (sans les outils de test) ?**
    `poetry install --without dev` (ou `--only main`).

4.  **Si je supprime une ligne dans `pyproject.toml`, la dépendance est-elle désinstallée ?**
    Non, pas automatiquement. Il faut lancer `poetry remove <package>` ou `poetry install --sync` pour nettoyer l'environnement.

---

## Exercices : {#exercices-48}

### Exercice 1 - Initialisation d'un projet Data {#exercice-1-init-data}

🎯 **Objectif** : Créer un projet propre avec Poetry.

💼 **Mise en situation** : Vous démarrez un projet d'analyse de données "DataViz".

📝 **Énoncé** :
1.  Utilisez `poetry new dataviz-app`.
2.  Entrez dans le dossier.
3.  Vérifiez le contenu du `pyproject.toml` généré.
4.  Installez l'environnement initial avec `poetry install`.

📺 **Résultat attendu** :
Un dossier `.venv` est créé (parfois caché selon la config) ou activé, et Poetry vous indique que le projet est prêt.

<details>
<summary>💡 Voir les commandes commentées</summary>

```bash
# 1. Création
poetry new dataviz-app

# 2. Navigation
cd dataviz-app

# 3. Vérification (optionnel)
cat pyproject.toml

# 4. Premier install (crée le venv)
poetry install
```
</details>

### Exercice 2 - Ajout de dépendances sélectif {#exercice-2-add-deps}

🎯 **Objectif** : Distinguer dépendances de Prod et de Dev.

💼 **Mise en situation** : Votre app a besoin de `pandas` pour fonctionner, et de `black` (formateur de code) uniquement pour les développeurs.

📝 **Énoncé** :
1.  Ajoutez `pandas` comme dépendance principale.
2.  Ajoutez `black` comme dépendance de développement.
3.  Ouvrez `pyproject.toml` pour vérifier les sections `[tool.poetry.dependencies]` et `[tool.poetry.group.dev.dependencies]`.

📺 **Résultat attendu** :
```toml
# Extrait du pyproject.toml
[tool.poetry.dependencies]
python = "^3.14"
pandas = "^2.2.0"  # Version exemple

[tool.poetry.group.dev.dependencies]
black = "^24.0.0"
```

<details>
<summary>💡 Voir la solution</summary>

```bash
poetry add pandas
poetry add --group dev black
```
</details>

### Exercice 3 - Le script d'exécution {#exercice-3-poetry-run}

🎯 **Objectif** : Exécuter un script utilisant une librairie installée.

💼 **Mise en situation** : Vous voulez vérifier que `pandas` est bien accessible.

📝 **Énoncé** :
1.  Créez un fichier `src/dataviz_app/main.py`.
2.  Écrivez : `import pandas as pd; print(f"Pandas version: {pd.__version__}")`.
3.  Essayez de lancer `python src/dataviz_app/main.py` (ça devrait échouer si vous n'êtes pas dans le venv).
4.  Lancez avec `poetry run python ...` (ça doit réussir).

📺 **Résultat attendu** :
```bash
$ python src/dataviz_app/main.py
ModuleNotFoundError: No module named 'pandas'

$ poetry run python src/dataviz_app/main.py
Pandas version: 2.2.0
```

<details>
<summary>💡 Voir la solution</summary>

```python
# src/dataviz_app/main.py
import pandas as pd
print(f"Pandas version: {pd.__version__}")
```

```bash
# Dans le terminal
poetry run python src/dataviz_app/main.py
```
</details>