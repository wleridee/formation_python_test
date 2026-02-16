Voici le contenu complet du chapitre 1, rédigé selon les instructions et structuré pour une introduction théorique solide avant la pratique.

```markdown
---
sidebar_label: Introduction à Python
sidebar_position: 1
---

# Chapitre 1 : Introduction à Python

Historique, Philosophie (Zen of Python), Cas d'usage

Bienvenue dans cette formation complète sur **Python 3.12**. Avant d'écrire votre première ligne de code, il est crucial de comprendre *pourquoi* ce langage domine l'industrie actuelle, de la start-up agile aux géants de la Tech, et quelles sont les règles implicites qui régissent son écosystème.

## 1. Historique et Contexte {#historique-et-contexte}

### 1. Quoi {#quoi-historique}
Python est un langage de programmation **interprété**, **multi-paradigme** (impératif, orienté objet, fonctionnel) et à **typage dynamique fort**.

Créé par **Guido van Rossum** à la fin des années 80, sa première version publique date de 1991. Contrairement à une idée reçue, son nom ne vient pas du reptile, mais de la troupe d'humoristes britanniques *Monty Python*.

### 2. Pourquoi {#pourquoi-historique}
Python a été conçu pour combler le fossé entre :
1.  Le **Shell/Bash** : excellent pour l'automatisation simple mais difficile à maintenir pour de gros scripts.
2.  Le **C/C++** : performant mais verbeux et complexe (gestion manuelle de la mémoire).

L'objectif était de créer un langage où la **lisibilité du code** est prioritaire, permettant aux développeurs d'exprimer des concepts en moins de lignes de code qu'en C++ ou Java.

### 3. Comment {#comment-historique}
L'histoire de Python est marquée par une transition majeure : le passage de la version 2 à la version 3.
- **Python 2** (Legacy) : Fin de vie officielle le 1er janvier 2020. Il ne doit plus être utilisé.
- **Python 3** (Moderne) : La version actuelle. Elle a corrigé des défauts de conception majeurs (notamment sur la gestion de l'Unicode), quitte à casser la rétrocompatibilité.

Dans ce cours, nous utiliserons **Python 3.12**, qui apporte des améliorations significatives :
- Des messages d'erreur beaucoup plus explicites (idéal pour l'apprentissage).
- Des gains de performances (optimisations internes).
- Une syntaxe enrichie pour le typage statique optionnel.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Terminal affichant l'exécution de `python --version`
> **Annotation** : Montrez une sortie type `Python 3.12.x`.
> **Alt Text suggéré** : Terminal de commande montrant la vérification de la version de Python installée.

---

## 2. La Philosophie : Le Zen de Python {#la-philosophie-le-zen-de-python}

### 1. Quoi {#quoi-philosophie}
Python n'est pas qu'une syntaxe, c'est une culture. Cette philosophie est résumée dans un texte appelé le **Zen de Python** (PEP 20), écrit par Tim Peters. Il s'agit de 19 aphorismes qui guident le design du langage et la manière dont nous devons coder.

### 2. Pourquoi {#pourquoi-philosophie}
Adopter le "Zen" permet d'écrire du code dit **Pythonic**. Un code Pythonic est idiomatique, naturel pour le langage, et immédiatement compréhensible par un autre développeur Python, qu'il soit junior ou senior.

### 3. Comment {#comment-philosophie}
Vous pouvez lire ces principes directement dans votre interpréteur Python en tapant :

```python
import this
```

Voici les 3 principes les plus critiques pour votre apprentissage :

#### A. "Explicit is better than implicit"
Ne faites pas de "magie". Le code doit montrer clairement ce qu'il fait.
*   ❌ **Mauvais** : Une fonction qui charge une base de données sans qu'on lui demande.
*   ✅ **Bon** : Une fonction qui prend la connexion à la base de données en argument explicite.

#### B. "Readability counts" (La lisibilité compte)
Le code est lu beaucoup plus souvent qu'il n'est écrit.
*   ❌ **Mauvais** : `x = a if b else c` (si imbriqué de manière complexe).
*   ✅ **Bon** : Un bloc `if/else` clair, même s'il prend 4 lignes au lieu d'une.

#### C. "There should be one-- and preferably only one --obvious way to do it"
Contrairement au Perl ou au Ruby qui offrent mille façons de faire la même chose, Python encourage une approche standardisée. Cela facilite la maintenance des projets collaboratifs.

### 4. Zone de Danger {#zone-de-danger-philosophie}

#### ❌ À ne pas faire
- Essayer d'écrire du Java ou du C en Python (par exemple, abuser des getters/setters ou des boucles `for` basées sur des index au lieu d'itérer directement sur les objets).
- Ignorer les conventions de style (PEP 8), que nous verrons plus tard.

#### ✅ Bonne Pratique
- Toujours se demander : "Si je relis ce code dans 6 mois, est-ce que je comprendrai l'intention immédiatement ?"

---

## 3. Cas d'usage et Écosystème {#cas-dusage-et-ecosysteme}

### 1. Quoi {#quoi-usage}
Python est souvent qualifié de **"Glue Language"** (langage colle) ou de couteau suisse. Grâce à sa bibliothèque standard gigantesque ("Batteries included") et à son gestionnaire de paquets (PyPI), il excelle dans presque tous les domaines sauf le bas niveau et le mobile natif.

### 2. Pourquoi {#pourquoi-usage}
Sa popularité explose car il est devenu le standard *de facto* dans deux domaines majeurs de la décennie : la **Data Science/IA** et le **Développement Backend** moderne.

### 3. Comment {#comment-usage}
Voici les principaux domaines d'application en 2026 :

| Domaine | Outils / Frameworks Clés | Description |
| :--- | :--- | :--- |
| **Développement Web (Backend)** | **FastAPI**, Django, Flask | Création d'API REST performantes et d'applications SaaS complètes. |
| **Data Science & IA** | Pandas, NumPy, Scikit-learn | Manipulation de données massives, statistiques et Machine Learning classique. |
| **Deep Learning** | **PyTorch**, TensorFlow | Réseaux de neurones, LLM (Large Language Models) et vision par ordinateur. |
| **Scripting & DevOps** | Ansible, Scripts Python natifs | Automatisation de serveurs, pipelines CI/CD, manipulation de fichiers. |
| **Web Scraping** | Scrapy, BeautifulSoup, Playwright | Extraction de données depuis des sites web. |

### 🚨 Limitations de Python {#limitations-de-python}

Bien que puissant, Python a des limites techniques claires qu'il faut connaître :

1.  **Vitesse d'exécution brute** : En tant que langage interprété, Python est plus lent que C, C++ ou Rust.
    *   *Nuance* : Pour la Data Science, ce n'est pas grave car Python appelle des bibliothèques écrites en C/C++ (comme NumPy). Le code Python n'est que le "chef d'orchestre".
2.  **Développement Mobile** : Python est très faible pour créer des applications natives iOS ou Android (bien que des outils comme Kivy ou BeeWare existent, ils ne sont pas des standards industriels).
3.  **Consommation Mémoire** : Les objets Python consomment plus de mémoire que des types primitifs en C.
4.  **Le GIL (Global Interpreter Lock)** :
    *   C'est un mécanisme interne qui empêche l'interpréteur Python standard (CPython) d'exécuter plusieurs threads natifs simultanément sur un seul cœur CPU.
    *   *Conséquence* : Le multithreading en Python n'accélère pas les tâches lourdes en calcul (CPU-bound). Pour cela, on utilise le **Multiprocessing** (que nous verrons au chapitre 47).

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-1}

1.  **Quelle est la différence fondamentale entre Python 2 et Python 3 ?**
    *   Python 3 est la version actuelle et activement maintenue. Python 2 est obsolète depuis 2020 et n'est pas rétrocompatible avec le 3.

2.  **Que signifie "Batteries included" pour Python ?**
    *   Cela signifie que Python est livré avec une bibliothèque standard très riche, permettant de faire beaucoup de choses (manipuler des fichiers, accéder au réseau, gérer des dates) sans rien installer de plus.

3.  **Pourquoi dit-on que Python est un langage interprété ?**
    *   Le code source n'est pas compilé en code machine binaire avant exécution. Il est lu et exécuté ligne par ligne par un interpréteur (le programme `python`).

4.  **Dans quel cas ne devriez-vous PAS choisir Python ?**
    *   Pour des systèmes temps réel critiques (pilotes de périphériques), des moteurs de jeux vidéo 3D haute performance (le cœur du moteur), ou des applications mobiles natives.
```