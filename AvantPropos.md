---
sidebar_label: Avant-propos
sidebar_position: 0
---

# Chapitre 0 : Avant-propos
Introduction, Prérequis, Structure du cours

Bienvenue dans cette formation complète sur **Python 3.12**. Ce cours a été conçu pour vous faire passer du statut de débutant (ou de développeur venant d'un autre langage) à celui de développeur Python moderne, capable de produire du code robuste, testé et prêt pour la production.

## Introduction {#introduction}

### 1. Quoi
Python est un langage de programmation interprété, multiparadigme (orienté objet, impératif, fonctionnel) et dynamiquement typé. Cette formation se concentre spécifiquement sur la version **3.12+**, qui introduit des améliorations significatives en termes de performance, de syntaxe (nouveaux F-strings, mot-clé `type`) et de messages d'erreur.

### 2. Pourquoi
Python domine aujourd'hui plusieurs industries clés :
*   **Web & SaaS** (Django, FastAPI)
*   **Data Science & IA** (Pandas, PyTorch)
*   **Scripting & DevOps** (Ansible, scripts d'automatisation)

Contrairement aux tutoriels obsolètes qui pullulent sur le web (Python 2.7 ou code "spaghetti"), ce cours enseigne le **Python Moderne** :
*   Typage statique strict avec `typing` et `MyPy`.
*   Gestion des chemins orientée objet avec `pathlib` (fini les `os.path.join`).
*   Tests unitaires robustes avec `pytest`.
*   Packaging professionnel avec `poetry`.

### 3. Comment
Nous adoptons une approche **"No Magic"**. Chaque concept est décortiqué. Vous ne copierez-collerez pas de code sans comprendre ce qu'il se passe sous le capot.

#### A. Philosophie du cours
Codez, ne lisez pas seulement. La maîtrise vient de la pratique.

```python
# ❌ Ce que nous ne ferons pas : du code "scripting" fragile
file = open("data.txt", "r")
content = file.read()
# Si ça plante ici, le fichier reste ouvert...

# ✅ Ce que nous ferons : du code robuste et moderne
from pathlib import Path

def read_safe(path: Path) -> str:
    """Lit un fichier de manière sécurisée avec gestion automatique des ressources."""
    # Le Context Manager (with) garantit la fermeture du fichier
    with path.open(encoding="utf-8") as f:
        return f.read()
```

## Prérequis {#prérequis}

### 1. Quoi
Ce cours reprend les bases, mais avance rapidement vers des concepts complexes.

### 2. Pourquoi
Pour ne pas perdre de temps sur des notions non-essentielles tout en garantissant que votre environnement ne soit pas un obstacle.

### 3. Checklist
*   **Matériel** : Un ordinateur (Windows, macOS ou Linux) avec accès administrateur pour installer des paquets.
*   **Système** : Une familiarité basique avec l'utilisation d'un terminal (savoir faire `cd`, `ls` ou `dir`).
*   **Logiciel** : Nous utiliserons **VS Code** comme éditeur de référence (installation détaillée au Chapitre 2).
*   **Anglais** : Le code (noms de variables) sera en anglais, les explications en français. C'est le standard de l'industrie.

### 4. Zone de Danger
*   ❌ **Ne pas utiliser Python 2**. C'est mort et enterré depuis 2020.
*   ❌ **Évitez les IDEs trop magiques au début** (comme PyCharm Community) si vous ne comprenez pas ce qu'ils font pour vous. VS Code offre le bon équilibre.

## Structure du cours {#structure-du-cours}

### 1. Quoi
Le cours est divisé en **62 chapitres** organisés en blocs logiques progressifs.

### 2. Pourquoi
Cette structure permet de construire des fondations solides avant d'attaquer l'architecture logicielle.

### 3. Le Plan de Bataille

| Bloc | Chapitres | Objectif |
| :--- | :--- | :--- |
| **Fondamentaux** | 1 - 8 | Syntaxe, variables, types primitifs et l'interpréteur. |
| **Structures de Données** | 9 - 14 | Listes, Dictionnaires, Tuples, Sets. Manipulation mémoire. |
| **Logique & Fonctions** | 15 - 25 | Découpage du code, portée des variables, gestion des erreurs. |
| **POO (Orienté Objet)** | 26 - 29, 48-49 | Classes, héritage, polymorphisme, protocoles. |
| **Standard Library** | 30 - 46 | Maîtriser les "piles incluses" (`os`, `json`, `datetime`, `re`). |
| **Python Moderne** | 47, 50-52 | Décorateurs, Context Managers, Typage statique, Pattern Matching. |
| **Qualité & Tests** | 53 - 55 | Tests unitaires, Mocks, TDD avec `unittest` et `pytest`. |
| **Concurrence** | 56 - 58 | Threading, Multiprocessing, AsyncIO. |
| **Packaging** | 59 - 61 | Créer un vrai projet distribuable (PyPI) avec `poetry`. |

## Questions clés (validation des acquis du chapitre) {#questions-clés-(validation-des-acquis-du-chapitre)-0}

1.  **Quelle version de Python est ciblée par ce cours ?**
    *   Python 3.12 (et versions ultérieures).
2.  **Quelle est la philosophie principale concernant le code présenté ?**
    *   "No Magic" et typage strict (Modern Python).
3.  **Quel outil de gestion de projet utiliserons-nous plus tard ?**
    *   Poetry (pour le packaging et les environnements virtuels).

## Exercices : {#exercices-:-0}

Puisque nous n'avons pas encore installé Python, ces exercices sont des vérifications de contexte.

### Exercice 1 - Le Mindset "No Magic" {#exercice-1---le-mindset-no-magic}

🎯 **Objectif** : Comprendre la différence entre "Ça marche" et "C'est robuste".

💼 **Mise en situation** : Vous arrivez dans une startup SaaS. Le code précédent plante aléatoirement. On vous demande de le fiabiliser.

📝 **Énoncé** :
Analysez visuellement ces deux approches (sans les exécuter). Laquelle semble la plus explicite et pourquoi ?

*Approche A :*
```python
def calcul(x):
    return x * 1.2  # Ajoute la TVA
```

*Approche B :*
```python
TVA_RATE: float = 1.2

def calculate_price_with_tax(price_ht: float) -> float:
    """Calcule le prix TTC à partir du prix hors-taxe."""
    return price_ht * TVA_RATE
```

💡 **Solution**

<details>
<summary>Voir l'analyse</summary>

**L'Approche B est la gagnante.**

1.  **Nommage** : `calculate_price_with_tax` vs `calcul`. On sait ce que fait la fonction.
2.  **Constante** : `TVA_RATE` explique le chiffre `1.2` (Magic Number). Si la TVA change, on modifie une seule ligne.
3.  **Typage** : `: float` et `-> float` indiquent clairement qu'on attend des nombres, pas des chaînes de caractères.
4.  **Docstring** : Explique l'intention métier.

C'est ce standard que nous viserons tout au long du cours.
</details>

---

### Exercice 2 - La Recherche Documentaire {#exercice-2---la-recherche-documentaire}

🎯 **Objectif** : Apprendre à trouver la source de vérité.

💼 **Mise en situation** : Votre CTO vous demande si une fonctionnalité est disponible dans Python 3.12.

📝 **Énoncé** :
Où devez-vous aller chercher l'information officielle la plus fiable concernant la syntaxe ou la bibliothèque standard ?

1.  StackOverflow
2.  ChatGPT
3.  docs.python.org
4.  Un tutoriel Medium de 2018

💡 **Solution**

<details>
<summary>Voir la réponse</summary>

**Réponse 3 : docs.python.org**

C'est la seule source de vérité absolue.
*   StackOverflow peut être obsolète.
*   ChatGPT peut halluciner des méthodes qui n'existent pas.
*   Les tutoriels vieillissent mal.

**Action recommandée** : Ajoutez [https://docs.python.org/3/](https://docs.python.org/3/) à vos favoris dès maintenant.
</details>

---

### Exercice 3 - Vérification de l'environnement {#exercice-3---vérification-de-l-environnement}

🎯 **Objectif** : S'assurer que votre machine est prête pour le Chapitre 2.

💼 **Mise en situation** : Vous allez installer votre atelier de développement.

📝 **Énoncé** :
Ouvrez votre terminal (Invite de commande sur Windows, Terminal sur Mac/Linux) et vérifiez si Python est déjà installé et quelle version est détectée par défaut. Notez la commande utilisée.

📺 **Résultat attendu** :
Affichage d'une version (ex: `Python 3.10.2`, `Python 2.7.18` ou `Command not found`).

💡 **Solution**

<details>
<summary>Voir la commande</summary>

Essayez les commandes suivantes :

```bash
python --version
# ou
python3 --version
```

*Si vous voyez "Command not found" ou une version 2.x, pas de panique. Le **Chapitre 2** traitera de l'installation propre de Python 3.12 via `pyenv` pour ne pas casser votre système.*
</details>