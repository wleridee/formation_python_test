---
sidebar_label: Packaging Python : Création de Distributions
sidebar_position: 47
---

# Chapitre 47 : Packaging Python : Création de Distributions

Structure de projet, setup.py / pyproject.toml, Build system (setuptools, Poetry), Wheel et sdist

Jusqu'à présent, nous avons écrit des scripts ou des modules pour nous-mêmes. Mais comment transformer un dossier de code en une bibliothèque installable par n'importe qui avec un simple `pip install mon-outil` ?

C'est là qu'intervient le **Packaging**. En 2026, l'écosystème Python s'est standardisé autour de formats modernes. Fini le temps du `setup.py` impératif et complexe ; place à la configuration déclarative avec `pyproject.toml` et aux standards PEP 517/518.

Ce chapitre vous apprend à structurer votre projet professionnellement et à générer les artefacts prêts à être distribués.

---

## 1. La Structure de Projet Standard (Src Layout) {#structure-de-projet-src-layout}

### 1. Quoi
La manière dont vous organisez vos dossiers détermine si votre code sera empaquetable ou non. Il existe deux écoles, mais une seule est recommandée pour les projets modernes robustes : le **Src Layout**.

### 2. Pourquoi
Mettre le code source dans un sous-dossier `src/` (au lieu de la racine) force les tests à s'exécuter sur la version *installée* du package, et non sur les fichiers locaux. Cela évite le fameux "Ça marche chez moi (car j'importe le fichier local) mais pas chez l'utilisateur".

### 3. Comment

#### A. Arborescence type

Voici à quoi ressemble un projet Python professionnel prêt à être packagé :

```text
mon_projet/
├── pyproject.toml       # 👈 Le cœur de la config (remplace setup.py)
├── README.md            # Description pour PyPI
├── .gitignore           # Fichiers à ignorer par git
├── src/                 # 👈 Dossier source isolé
│   └── mon_package/     # Le nom réel du package à importer
│       ├── __init__.py
│       ├── core.py
│       └── utils.py
└── tests/               # Dossier de tests (hors du package)
    ├── __init__.py
    └── test_core.py
```

### 4. Zone de Danger
❌ **Le Flat Layout (Mélange à la racine)** :
```text
mon_projet/
├── mon_package/  # Directement à la racine
├── setup.py
└── tests/
```
Bien que courant, cela cause souvent des erreurs d'import subtiles lors des tests ou du déploiement.

---

## 2. La Configuration Moderne : `pyproject.toml` {#configuration-pyproject-toml}

### 1. Quoi
Le fichier `pyproject.toml` est le standard officiel pour définir les métadonnées de votre projet (nom, version, auteurs) et les outils de construction (build system). Il remplace `setup.py`, `requirements.txt` (pour les dépendances abstraites) et souvent `pytest.ini`.

### 2. Pourquoi
Il est **déclaratif** (on décrit *ce que* l'on veut, pas *comment* le faire en Python) et centralise la configuration. Il permet de changer d'outil de build (Setuptools, Flit, Hatch, Poetry) sans changer la structure du projet.

### 3. Comment

#### A. Exemple complet avec Setuptools
Même si Poetry est populaire (voir chapitre suivant), `setuptools` reste le backend standard universel.

```toml
# pyproject.toml

# 1. Définition du système de build
[build-system]
requires = ["setuptools>=61.0"]
build-backend = "setuptools.build_meta"

# 2. Métadonnées du projet
[project]
name = "analytics-saas-lib"
version = "0.1.0"
description = "Une librairie pour calculer des métriques SaaS"
readme = "README.md"
requires-python = ">=3.10"
license = {file = "LICENSE"}
authors = [
  {name = "Jean Dupont", email = "jean@example.com"}
]
keywords = ["saas", "analytics", "metrics"]

# 3. Dépendances requises pour que le package fonctionne
dependencies = [
  "pandas>=2.0.0",
  "requests>=2.28.0"
]

# 4. URLs utiles (Affichées sur PyPI)
[project.urls]
"Homepage" = "https://github.com/monorg/analytics-saas-lib"
"Bug Tracker" = "https://github.com/monorg/analytics-saas-lib/issues"
```

### 🚨 Limitations de `setup.py`
Le fichier `setup.py` existe toujours pour des cas très avancés (compilation C complexe), mais il n'est plus recommandé pour la configuration standard. Ne l'utilisez que si `pyproject.toml` ne suffit pas.

---

## 3. Artefacts : Wheel (.whl) vs Sdist (.tar.gz) {#artefacts-wheel-sdist}

### 1. Quoi
Une fois le projet configuré, nous devons le "construire" (build) pour créer des fichiers distribuables.
*   **sdist (Source Distribution)** : Une archive `.tar.gz` contenant le code source brut.
*   **Wheel (Binary Distribution)** : Une archive `.whl` pré-construite, prête à être dézippée dans le dossier `site-packages` de l'utilisateur.

### 2. Pourquoi
*   **Wheel** : Installation ultra-rapide (pas de build à l'installation). C'est le format préféré de pip.
*   **sdist** : Format de secours. Si pip ne trouve pas de Wheel compatible avec la plateforme de l'utilisateur, il télécharge la sdist et compile une Wheel localement.

### 3. Comment

#### A. Générer les artefacts

N'utilisez plus `python setup.py sdist bdist_wheel` (déprécié). Utilisez l'outil standard `build`.

1. **Installer l'outil de build :**
   ```bash
   pip install build
   ```

2. **Lancer le build (à la racine où se trouve pyproject.toml) :**
   ```bash
   python -m build
   ```

3. **Résultat :**
   Un dossier `dist/` est créé :
   ```text
   dist/
   ├── analytics_saas_lib-0.1.0-py3-none-any.whl  (Le Wheel)
   └── analytics_saas_lib-0.1.0.tar.gz            (La sdist)
   ```

---

## 4. Build Systems : Setuptools vs Poetry vs Autres {#build-systems-comparaison}

### 1. Quoi
Le "Build Backend" est le moteur qui lit `pyproject.toml` et crée les fichiers `.whl`.
*   **Setuptools** : Le standard historique. Robuste, mais verbeux.
*   **Poetry** : (Voir Chapitre 48) Outil tout-en-un (gestion de dépendances + build + publish). Très populaire pour les applications.
*   **Hatch / Flit** : Alternatives modernes et légères.

### 2. Pourquoi
Le choix dépend de vos besoins. Pour une bibliothèque pure Python simple, `Flit` est génial. Pour une application complexe avec verrouillage de versions, `Poetry` excelle. Pour la compatibilité maximale, `Setuptools`.

### 3. D. Tableau comparatif

| Caractéristique | Setuptools (via pyproject.toml) | Poetry | Flit |
| :--- | :--- | :--- | :--- |
| **Usage** | Standard historique | Tout-en-un (Dev & Ops) | Minimaliste |
| **Complexité** | Moyenne | Élevée (apprend une nouvelle CLI) | Faible |
| **Gestion venv** | Non (manuel) | Oui (automatique) | Non |
| **Lock file** | Non (besoin de pip-tools) | Oui (`poetry.lock`) | Non |
| **Popularité** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |

> *Note : Ce chapitre se concentre sur les standards (Setuptools/Build). Le chapitre suivant détaillera Poetry.*

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-47}

1.  **Quel fichier remplace `setup.py` pour la configuration du packaging ?**
    `pyproject.toml`.

2.  **Pourquoi privilégier le "Src Layout" (`src/mon_package`) ?**
    Pour éviter les erreurs d'importation où les tests chargent le dossier local au lieu du package installé, garantissant ainsi des tests plus fiables.

3.  **Quelle est la différence entre un Wheel (.whl) et une Sdist (.tar.gz) ?**
    Le Wheel est un package pré-construit prêt à l'emploi (rapide). La Sdist contient les sources et nécessite une étape de construction lors de l'installation.

4.  **Quelle commande moderne génère les fichiers de distribution ?**
    `python -m build`.

---

## Exercices : {#exercices-47}

### Exercice 1 - Restructuration Pro {#exercice-1-restructuration-pro}

🎯 **Objectif** : Transformer un script "plat" en structure de package standard.

💼 **Mise en situation** : Vous avez développé un script `invoice_generator.py` pour votre startup. Vous voulez maintenant en faire une librairie interne propre.

📝 **Énoncé** :
1.  Créez une arborescence de dossiers vide respectant le **Src Layout**.
2.  Placez un fichier Python factice dans `src/invoicelib/__init__.py`.
3.  Créez le fichier `pyproject.toml` minimaliste configuré avec `setuptools`.
4.  Vérifiez la structure avec la commande `tree` (ou l'explorateur).

📺 **Résultat attendu** :
```text
mon_projet/
├── pyproject.toml
└── src/
    └── invoicelib/
        └── __init__.py
```

<details>
<summary>💡 Voir le code du pyproject.toml</summary>

```toml
[build-system]
requires = ["setuptools>=61.0"]
build-backend = "setuptools.build_meta"

[project]
name = "invoicelib"
version = "0.0.1"
authors = [{name="Moi", email="moi@startup.com"}]
description = "Générateur de factures"
readme = "README.md"
requires-python = ">=3.9"
```
</details>

### Exercice 2 - Dépendances et Métadonnées {#exercice-2-dependances-metadonnees}

🎯 **Objectif** : Définir correctement les dépendances d'un projet.

💼 **Mise en situation** : Votre librairie `invoicelib` a besoin de `reportlab` pour générer des PDF et de `arrow` pour gérer les dates.

📝 **Énoncé** :
1.  Reprenez le `pyproject.toml` de l'exercice 1.
2.  Ajoutez la section `dependencies`.
3.  Spécifiez `reportlab` (version 3.6 ou plus) et `arrow` (n'importe quelle version).
4.  Ajoutez une URL vers le dépôt GitHub (fictif).

📺 **Résultat attendu** :
Le fichier doit contenir les clés `dependencies` et `project.urls`.

<details>
<summary>💡 Voir la solution</summary>

```toml
# ... début du fichier ...

dependencies = [
    "reportlab>=3.6",
    "arrow"
]

[project.urls]
"Source" = "https://github.com/mastartup/invoicelib"
```
</details>

### Exercice 3 - Le Build Final {#exercice-3-le-build-final}

🎯 **Objectif** : Générer les fichiers `.whl` et `.tar.gz`.

💼 **Mise en situation** : Tout est prêt. Vous devez livrer le binaire à l'équipe DevOps.

📝 **Énoncé** :
1.  Assurez-vous d'avoir installé l'outil de build : `pip install build`.
2.  Placez-vous à la racine du projet (au niveau du `pyproject.toml`).
3.  Lancez la commande de build.
4.  Listez le contenu du dossier `dist/` généré.

📺 **Résultat attendu** :
```bash
$ python -m build
...
Successfully built invoicelib-0.0.1.tar.gz and invoicelib-0.0.1-py3-none-any.whl

$ ls dist/
invoicelib-0.0.1-py3-none-any.whl
invoicelib-0.0.1.tar.gz
```

<details>
<summary>💡 Voir les commandes</summary>

```bash
# 1. Installation de l'outil
pip install build

# 2. Construction
python -m build

# 3. Vérification (Sur Linux/Mac)
ls -l dist/
# Sur Windows (PowerShell)
# ls dist/
```
</details>