---
sidebar_label: Introduction à Python
sidebar_position: 1
---

# Chapitre 1 : Introduction à Python

Historique, Philosophie, Écosystème, Cas d'usage

Avant d'écrire votre première ligne de code, il est essentiel de comprendre d'où vient Python, pourquoi il domine le paysage technologique actuel (de l'IA au développement Web) et quelle est la philosophie qui guide sa conception.

Ce chapitre pose les bases culturelles et techniques nécessaires pour devenir un développeur Python efficace.

---

## 1. Historique et Évolution {#historique-et-evolution}

### 1. Quoi
**Python** est un langage de programmation interprété, multi-paradigme et multiplateforme. Il a été créé par **Guido van Rossum** à la fin des années 80 aux Pays-Bas (au CWI) et publié pour la première fois en **1991**.

Contrairement à une idée reçue, son nom ne vient pas du serpent, mais de la troupe de comiques britanniques **Monty Python**, dont le créateur était fan.

L'histoire du langage est marquée par une transition majeure : le passage de **Python 2 à Python 3** en 2008. Cette mise à jour a cassé la rétrocompatibilité pour nettoyer le langage de ses défauts de jeunesse. Depuis 2020, Python 2 est officiellement obsolète (EOL).

### 2. Pourquoi
Python a été conçu pour être un successeur du langage ABC, capable de gérer les exceptions et d'interagir avec le système d'exploitation Amoeba. L'objectif principal de Guido van Rossum était la **productivité du développeur** et la **lisibilité du code**. Il voulait un langage aussi puissant que C mais aussi simple que le Shell.

### 3. Comment
Python est géré par la **Python Software Foundation (PSF)**, une organisation à but non lucratif. Le développement se fait via des **PEP** (Python Enhancement Proposals), des documents de conception ouverts à la communauté. La version ciblée par ce cours, **Python 3.14**, continue d'apporter des améliorations de performance (JIT compiler expérimental, suppression du GIL optionnelle) et de syntaxe.

### 4. Zone de Danger

❌ **À ne pas faire** :
*   Utiliser Python 2 pour de nouveaux projets. C'est un risque de sécurité majeur.
*   Penser que Python est un langage "lent" par nature. S'il est plus lent que C en exécution pure, il est infiniment plus rapide en temps de développement, et délègue les calculs lourds à des bibliothèques C (comme NumPy).

✅ **Bonne Pratique** :
*   Toujours vérifier la version de Python installée (`python --version`) et viser les versions stables récentes (3.12+).

---

## 2. Philosophie : Le Zen de Python {#philosophie-le-zen-de-python}

### 1. Quoi
Python possède une "âme", résumée dans un texte appelé **Le Zen de Python** (PEP 20). C'est une collection de 19 aphorismes qui guident l'écriture du code "Pythonique".

Quelques principes clés :
*   **"Beautiful is better than ugly."** (Le beau est mieux que le laid).
*   **"Explicit is better than implicit."** (L'explicite est mieux que l'implicite).
*   **"Simple is better than complex."** (Le simple est mieux que le complexe).
*   **"Readability counts."** (La lisibilité compte).

### 2. Pourquoi
Le code est lu beaucoup plus souvent qu'il n'est écrit. En imposant un style clair et épuré (notamment via l'indentation obligatoire), Python facilite la maintenance et la collaboration. Contrairement à Perl ("There is more than one way to do it"), Python préfère qu'il n'y ait, de préférence, qu'une seule façon évidente de faire les choses.

### 3. Comment
L'indentation (les espaces en début de ligne) n'est pas stylistique en Python, elle est **syntaxique**. Elle définit les blocs de code (là où d'autres langages utilisent des accolades `{}`).

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Terminal affichant le résultat de `import this`.
> **Annotation** : Surligner les phrases "Explicit is better than implicit" et "Readability counts".
> **Alt Text suggéré** : Le Zen de Python affiché dans un terminal.

### 4. Zone de Danger

❌ **À ne pas faire** :
*   Écrire des "one-liners" complexes (tout mettre sur une seule ligne) juste pour frimer. Ce n'est pas "Pythonique".
*   Mélanger les tabulations et les espaces pour l'indentation (erreur classique `IndentationError`).

---

## 3. Écosystème et "Batteries Included" {#ecosysteme-et-batteries-included}

### 1. Quoi
Python est célèbre pour son approche **"Batteries Included"**. Cela signifie que l'installation standard de Python vient avec une bibliothèque standard immense et robuste, permettant de faire presque tout (manipuler des fichiers, accéder au réseau, gérer des bases de données SQLite, parser du JSON, etc.) sans rien installer de plus.

En plus de cela, il existe le **PyPI (Python Package Index)**, un dépôt tiers contenant des centaines de milliers de paquets installables via l'outil `pip`.

### 2. Pourquoi
Cette richesse permet un **Time-to-Market** imbattable. Vous n'avez pas besoin de réinventer la roue. Si vous avez un problème, il existe probablement une bibliothèque Python pour le résoudre.

### 3. Comment
L'écosystème s'appuie sur des outils de gestion modernes :
*   **pip** : L'installateur de paquets standard.
*   **venv** : Le créateur d'environnements virtuels (pour isoler les projets).
*   **Poetry / uv** : Des gestionnaires de dépendances modernes (que nous verrons plus tard).

### 4. Zone de Danger

❌ **À ne pas faire** :
*   Installer des paquets globalement sur votre machine système (risque de casser les outils système qui dépendent de Python). Utilisez toujours des **environnements virtuels**.

---

## 4. Cas d'Usage et Limitations {#cas-d-usage-et-limitations}

### 1. Quoi
Python est un langage généraliste ("Glue Language"), mais il excelle particulièrement dans certains domaines.

**Domaines de prédilection :**
1.  **Data Science & IA** : Pandas, NumPy, Scikit-learn, PyTorch, TensorFlow. C'est le standard mondial.
2.  **Développement Web (Backend)** : Django, FastAPI, Flask.
3.  **Automatisation & Scripting** : Remplacer des scripts Bash/PowerShell complexes.
4.  **DevOps / Infrastructure** : Ansible, scripts cloud.
5.  **Éducation** : Premier langage enseigné dans de nombreuses universités.

### 2. Pourquoi
Sa simplicité syntaxique permet aux scientifiques et aux mathématiciens de l'utiliser sans être des experts en génie logiciel. Sa capacité à s'interfacer avec du code C/C++ permet d'avoir la facilité de Python avec la performance du bas niveau.

### 3. Comment
On choisit Python quand la vitesse de développement et la maintenabilité sont prioritaires sur la vitesse d'exécution brute pure, ou quand on peut déléguer le calcul lourd à des bibliothèques optimisées.

### 🚨 Limitations de Python
Il est honnête de reconnaître où Python est moins adapté :
*   **Applications Mobiles (Android/iOS)** : Possible (Kivy, BeeWare) mais l'expérience n'est pas native. Swift/Kotlin ou Flutter sont préférables.
*   **Développement Front-end** : Python ne tourne pas nativement dans le navigateur (bien que PyScript/WebAssembly change la donne, JS/TS reste roi).
*   **Systèmes Temps Réel Critique** : Le "Garbage Collector" et le typage dynamique peuvent introduire des latences imprévisibles.

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-1}

1.  **Quelle est la différence majeure de philosophie entre Python et un langage comme Perl ou C++ ?**
    Python privilégie la lisibilité et l'unicité de la solution ("One obvious way to do it"), tandis que d'autres langages offrent souvent de multiples façons complexes d'atteindre le même résultat. L'indentation syntaxique en est la preuve la plus visible.

2.  **Pourquoi est-il crucial d'utiliser des environnements virtuels (venv) ?**
    Pour éviter le "Dependency Hell". Les environnements virtuels isolent les bibliothèques d'un projet spécifique par rapport au système global et aux autres projets, évitant les conflits de versions.

3.  **Que signifie "Batteries Included" ?**
    Cela signifie que la bibliothèque standard de Python est suffisamment riche pour permettre de développer des applications complexes (serveur web, gestion de fichiers, protocoles réseaux) sans avoir besoin de télécharger des paquets externes immédiatement.