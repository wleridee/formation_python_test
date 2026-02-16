---
sidebar_label: Déploiement : Publication sur PyPI
sidebar_position: 49
---

# Chapitre 49 : Déploiement : Publication sur PyPI

Compte PyPI, Utilisation de twine, Gestion des versions, Bonnes pratiques

Vous avez structuré votre projet, écrit vos tests et généré vos artefacts (`.whl` et `.tar.gz`). Il est temps de partager votre création avec le reste du monde (ou avec vos collègues).

**PyPI** (Python Package Index) est le dépôt officiel des paquets Python. C'est grâce à lui que `pip install requests` fonctionne. Ce chapitre vous guide à travers le processus délicat de la publication, en passant par l'environnement de test pour éviter les erreurs publiques.

---

## 1. PyPI et TestPyPI : L'App Store de Python {#pypi-et-testpypi}

### 1. Quoi
*   **PyPI (Real)** : Le dépôt de production (`pypi.org`). Tout ce qui y est publié est public, immuable et installable par tout le monde.
*   **TestPyPI** : Un clone de PyPI (`test.pypi.org`) destiné au "bac à sable". Il est nettoyé régulièrement et sert à tester la procédure d'upload et l'installation sans polluer le vrai index.

### 2. Pourquoi
Une fois une version publiée sur PyPI (ex: `1.0.0`), **vous ne pouvez plus jamais la modifier ni la réutiliser**, même si vous la supprimez. Si vous faites une erreur de packaging sur la prod, vous devez "brûler" un numéro de version (passer à `1.0.1`). TestPyPI sert à éviter ce gaspillage.

### 3. Comment

#### A. Création des comptes
Il faut créer **deux** comptes distincts (les bases de données ne sont pas liées) :
1.  Allez sur [test.pypi.org/account/register](https://test.pypi.org/account/register/) et créez un compte.
2.  Allez sur [pypi.org/account/register](https://pypi.org/account/register/) pour le compte réel.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Page de configuration des API Tokens sur PyPI
> **Annotation** : Entourez le bouton "Add API Token".
> **Alt Text suggéré** : Interface utilisateur de PyPI montrant la section Account Settings et la création de token.

#### B. Authentification Moderne (API Token)
L'authentification par mot de passe est obsolète pour l'upload. Vous devez générer un **API Token**.
1.  Dans vos paramètres de compte PyPI (et TestPyPI), section "API tokens".
2.  Créez un token (Scope: "Entire account" pour le premier upload).
3.  Gardez ce token (commençant par `pypi-`) précieusement.

---

## 2. Gestion des Versions (SemVer) {#gestion-des-versions-semver}

### 1. Quoi
Le "Semantic Versioning" (SemVer) est une convention universelle `MAJEUR.MINEUR.PATCH` (ex: `2.14.3`).

### 2. Pourquoi
C'est un contrat de confiance avec vos utilisateurs :
*   **MAJEUR** : Changements incompatibles (Breaking changes). Le code de l'utilisateur risque de casser.
*   **MINEUR** : Nouvelles fonctionnalités rétro-compatibles.
*   **PATCH** : Corrections de bugs rétro-compatibles.

### 3. Comment

Dans `pyproject.toml` :

```toml
[project]
name = "mon-super-package"
# On incrémente ce numéro AVANT chaque publication
version = "1.0.2" 
```

Si vous utilisez **Poetry**, la commande est simplifiée :
```bash
poetry version patch   # 1.0.2 -> 1.0.3
poetry version minor   # 1.0.3 -> 1.1.0
poetry version major   # 1.1.0 -> 2.0.0
```

---

## 3. Publication Standard avec `twine` {#publication-avec-twine}

### 1. Quoi
`twine` est l'outil officiel pour sécuriser l'upload des fichiers sur PyPI. Contrairement à l'ancienne méthode (`python setup.py upload`), Twine fait l'upload via HTTPS vérifié et permet de pré-vérifier les métadonnées.

### 2. Pourquoi
Pour s'assurer que la description du paquet (souvent le README) est bien formatée et que l'upload est sécurisé.

### 3. Comment

#### A. Installation
```bash
pip install twine
```

#### B. Vérification des artefacts
Avant d'envoyer, vérifiez que votre build (fait au chapitre 47) est sain :
```bash
twine check dist/*
# Doit afficher : PASSED
```

#### C. Upload vers TestPyPI (Le Répétition)
```bash
twine upload --repository testpypi dist/*
```
*   **Username** : `__token__` (littéralement cette chaîne de caractères).
*   **Password** : Votre token API complet (`pypi-AgE...`).

Une fois fait, essayez d'installer votre package dans un venv vierge :
```bash
pip install --index-url https://test.pypi.org/simple/ --no-deps mon-super-package
```

#### D. Upload vers PyPI (La Production)
Si tout fonctionne :
```bash
twine upload dist/*
```
(Utilisez le token de production cette fois).

---

## 4. Publication Simplifiée avec Poetry {#publication-avec-poetry}

### 1. Quoi
Si vous avez suivi le chapitre 48 et utilisez Poetry, vous n'avez pas besoin de `twine` explicitement. Poetry gère le build et le publish en interne.

### 2. Pourquoi
Moins d'outils, configuration centralisée, moins d'erreurs manuelles.

### 3. Comment

#### A. Configuration du Token
Dites à Poetry d'utiliser votre token (une seule fois par machine) :
```bash
poetry config pypi-token.pypi pypi-AgE...
```

#### B. Build et Publish
```bash
poetry build
poetry publish
```

Pour TestPyPI avec Poetry :
```bash
poetry config repositories.testpypi https://test.pypi.org/legacy/
poetry config pypi-token.testpypi pypi-AgE... # (Token TestPyPI)
poetry publish -r testpypi
```

---

## 5. Zone de Danger et Bonnes Pratiques {#zone-de-danger}

### ❌ À ne pas faire
*   **Ne jamais commiter vos Tokens API** dans GitHub/GitLab. Utilisez des variables d'environnement (`TWINE_PASSWORD`) dans vos pipelines CI/CD.
*   **Ne pas oublier le `.gitignore`** : Assurez-vous que les fichiers `.pyc`, les dossiers `__pycache__`, et vos fichiers de secrets (`.env`) ne sont pas inclus dans le package source (`sdist`). Vérifiez le contenu du `.tar.gz` avant upload.

### ✅ Bonne Pratique : Yanking
Si vous publiez une version `1.0.5` qui contient un bug critique :
1.  Ne supprimez pas la release (cela casse les builds des gens qui l'ont déjà installée).
2.  Utilisez la fonction **"Yank"** sur l'interface web de PyPI. Cela marque la version comme "dépréciée" : pip ne l'installera plus par défaut, mais elle reste accessible si spécifiquement demandée.

### 🚨 Limitations de PyPI
*   **Immutabilité des noms** : Une fois que vous avez pris le nom `super-lib`, il est à vous. Si vous le supprimez, le nom peut être bloqué pour éviter le "typosquatting".
*   **Taille limite** : PyPI n'est pas fait pour héberger des modèles AI de 2Go. Il y a des limites de taille (généralement < 100 Mo).

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-49}

1.  **Pourquoi est-il conseillé d'utiliser TestPyPI avant le vrai PyPI ?**
    Car la publication sur PyPI est définitive. On ne peut pas écraser une version existante. TestPyPI permet de valider le processus sans "brûler" de version.

2.  **Quel nom d'utilisateur doit-on utiliser avec un Token API ?**
    `__token__`.

3.  **Qu'est-ce que Semantic Versioning ?**
    Une convention `Majeur.Mineur.Patch` pour communiquer l'impact des changements (Breaking vs Feature vs Fix).

4.  **Si je publie une version buggée, dois-je la supprimer ?**
    Non, il vaut mieux la "Yanker" via l'interface PyPI et publier immédiatement une version corrective supérieure.

---

## Exercices : {#exercices-49}

### Exercice 1 - Premier Upload sur TestPyPI {#exercice-1-upload-testpypi}

🎯 **Objectif** : Configurer la chaîne de publication sans risque.

💼 **Mise en situation** : Votre librairie de calcul de TVA est prête. Votre chef veut valider qu'elle s'installe bien via `pip` avant de la donner aux clients.

📝 **Énoncé** :
1.  Créez un compte sur TestPyPI.
2.  Générez un API Token.
3.  Prenez un projet existant (celui du chapitre 47 ou 48).
4.  Modifiez le nom dans `pyproject.toml` pour qu'il soit unique (ex: `tva-lib-[votre-pseudo]`).
5.  Construisez (`python -m build` ou `poetry build`) et uploadez sur TestPyPI.

📺 **Résultat attendu** :
Le terminal affiche :
```text
Uploading distributions to https://test.pypi.org/legacy/
View at:
https://test.pypi.org/project/tva-lib-votre-pseudo/0.1.0/
```

<details>
<summary>💡 Voir les étapes clés</summary>

```bash
# Avec Twine
pip install twine build
python -m build
twine upload --repository testpypi dist/*
# Username: __token__
# Password: pypi-Ag...
```
</details>

### Exercice 2 - Gestion de Version SemVer {#exercice-2-semver}

🎯 **Objectif** : Appliquer la logique SemVer.

💼 **Mise en situation** :
*   Version actuelle : `1.2.4`
*   Cas A : Vous corrigez une faute d'orthographe dans un log.
*   Cas B : Vous ajoutez une fonction `calculate_ttc_batch`.
*   Cas C : Vous renommez la fonction `calculate` en `calc` (ce qui casse le code existant).

📝 **Énoncé** :
Déterminez le prochain numéro de version pour chaque cas.

📺 **Résultat attendu** :
*   Cas A : `1.2.5` (Patch)
*   Cas B : `1.3.0` (Mineur)
*   Cas C : `2.0.0` (Majeur)

<details>
<summary>💡 Explication</summary>
Le versioning sémantique est strict : si on casse la compatibilité descendante, on DOIT changer le chiffre Majeur, même pour un petit renommage.
</details>

### Exercice 3 - Le fichier `.pypirc` (Automatisation) {#exercice-3-pypirc}

🎯 **Objectif** : Ne plus taper son mot de passe.

💼 **Mise en situation** : Vous publiez tous les jours. Copier-coller le token est fastidieux.

📝 **Énoncé** :
1.  Créez un fichier `.pypirc` dans votre dossier utilisateur (`~/.pypirc` sur Linux/Mac ou `%USERPROFILE%/.pypirc` sur Windows).
2.  Configurez-le pour TestPyPI.
3.  Lancez un upload `twine` sans entrer de mot de passe.

📺 **Résultat attendu** :
`twine upload -r testpypi dist/*` se lance immédiatement.

<details>
<summary>💡 Voir le contenu du fichier</summary>

```ini
[distutils]
index-servers =
    testpypi

[testpypi]
repository = https://test.pypi.org/legacy/
username = __token__
password = pypi-AgE... (votre token complet)
```
</details>