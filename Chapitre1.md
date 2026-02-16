---
sidebar_label: Introduction à Python
sidebar_position: 1
---

# Chapitre 1 : Introduction à Python

Historique et philosophie, Caractéristiques du langage, Écosystème et cas d'usage

## Historique et philosophie {#historique-et-philosophie}

### 1. Quoi
Python est un langage de programmation **interprété**, **orienté objet** et de **haut niveau**. Il a été créé par **Guido van Rossum** et publié pour la première fois en 1991.

Contrairement à une idée reçue, son nom ne vient pas du reptile, mais de la troupe de comiques britanniques *Monty Python*. Python est un projet **Open Source** géré par la Python Software Foundation (PSF).

La philosophie de Python repose sur des principes de clarté et de simplicité, résumés dans le "Zen de Python".

### 2. Pourquoi
Dans le monde du développement, le temps humain coûte plus cher que le temps machine. Python a été conçu pour maximiser la **productivité du développeur**. 

Il privilégie la lisibilité du code : un programme Python bien écrit se lit presque comme de l'anglais. Cela facilite énormément la maintenance et la collaboration, qui sont les défis majeurs des projets logiciels modernes.

### 3. Comment

#### A. Le Zen de Python
Ces principes sont intégrés au langage. Vous pouvez les lire en exécutant cette commande dans un interpréteur Python :

```python
import this
# Affiche les 19 aphorismes du Zen de Python
```

Voici les 3 plus importants à retenir pour ce cours :
1. **Beautiful is better than ugly.** (Le beau est préférable au laid)
2. **Explicit is better than implicit.** (L'explicite est préférable à l'implicite - pas de "magie" cachée)
3. **Readability counts.** (La lisibilité compte)

#### B. Exemple de lisibilité
Comparaison conceptuelle pour afficher "Bonjour" 5 fois :

*Style C++/Java (Verbeux) :*
```java
for (int i = 0; i < 5; i++) {
    System.out.println("Bonjour");
}
```

*Style Python (Explicite et concis) :*
```python
for _ in range(5):
    print("Bonjour")
```

### 4. Zone de Danger
❌ **À ne pas faire** : Écrire du Python comme on écrit du C ou du Java. On appelle cela ne pas être "Pythonic".
✅ **Bonne Pratique** : Suivre la **PEP 8**, qui est le guide de style officiel du code Python (indentation de 4 espaces, noms de variables en `snake_case`, etc.).

---

## Caractéristiques du langage {#caractéristiques-du-langage}

### 1. Quoi
Python possède une combinaison unique de caractéristiques techniques :
*   **Interprété** : Le code est lu et exécuté ligne par ligne par l'interpréteur (CPython généralement), sans compilation préalable en langage machine.
*   **Typage Dynamique mais Fort** : 
    *   *Dynamique* : Le type d'une variable est déterminé à l'exécution (`x` peut être un entier puis une chaîne).
    *   *Fort* : Le langage ne convertit pas implicitement les types de manière incohérente (pas d'addition `1 + "1"`).
*   **Multi-paradigme** : Supporte la programmation impérative, orientée objet et fonctionnelle.
*   **Gestion automatique de la mémoire** : Utilise un *Garbage Collector* pour nettoyer la mémoire inutilisée.

### 2. Pourquoi
Cette flexibilité permet de prototyper extrêmement vite. Là où il faut 2 jours pour configurer un projet C++, il faut 2 heures en Python.

De plus, Python est **portable** : le même code source tourne sur Windows, macOS et Linux sans modification (tant qu'il n'utilise pas de fonctions spécifiques à l'OS).

### 3. Comment

#### A. Typage dynamique vs fort
```python
# Typage dynamique : x change de type
x = 10      # x est un int
x = "Hello" # x est maintenant un str

# Typage fort : Sécurité
a = 10
b = "5"
# print(a + b)  # ❌ TypeError: unsupported operand type(s) for +: 'int' and 'str'
print(a + int(b)) # ✅ 15 (Conversion explicite requise)
```

#### B. Nouveautés Python 3.12+ (Modernité)
Python évolue constamment. La version 3.12 (et ultérieures) met l'accent sur :
*   **Messages d'erreur améliorés** : L'interpréteur vous suggère des corrections ("Did you mean...?").
*   **Performance** : Optimisations internes (inlining des compréhensions).
*   **Typage graduel** : Ajout d'annotations de types pour les gros projets (ex: `name: str = "Alice"`), bien que l'exécution reste dynamique.

### 🚨 Limitations de Python
*   **Vitesse d'exécution brute** : Python est plus lent que C ou Rust car il est interprété.
*   **GIL (Global Interpreter Lock)** : Un mécanisme interne qui empêche plusieurs threads Python de s'exécuter simultanément sur un seul cœur CPU (bien que cela soit en cours d'assouplissement dans les versions très récentes comme 3.13+).
*   **Mobile** : Python n'est pas natif sur Android/iOS (contrairement à Swift/Kotlin).

---

## Écosystème et cas d'usage {#écosystème-et-cas-d-usage}

### 1. Quoi
La plus grande force de Python n'est pas le langage lui-même, mais son **écosystème**. Il suit la philosophie "Batteries Included" (Batteries incluses) : la bibliothèque standard est immense.

De plus, le dépôt central **PyPI** (Python Package Index) héberge des centaines de milliers de bibliothèques tierces.

### 2. Pourquoi
Dans une startup ou une grande entreprise, vous ne voulez pas réinventer la roue. Pour presque n'importe quel problème (envoyer un email, analyser un fichier Excel, créer une IA), il existe déjà une bibliothèque Python éprouvée.

### 3. Comment : Les domaines rois

| Domaine | Bibliothèques/Frameworks Célèbres | Cas d'usage Réel |
| :--- | :--- | :--- |
| **Développement Web** | **Django**, **FastAPI**, Flask | Backend de Instagram, Pinterest, APIs Netflix. |
| **Data Science & IA** | **Pandas**, NumPy, **PyTorch**, TensorFlow, Scikit-learn | Algorithmes de recommandation Spotify, ChatGPT. |
| **Scripting & DevOps** | Ansible, Scripts système (`os`, `sys`) | Automatisation de déploiement de serveurs AWS. |
| **Finance** | Pandas, SciPy | Analyse de risques, Trading algorithmique. |

### 4. Zone de Danger
❌ **À ne pas faire** : Installer toutes les bibliothèques globalement sur votre machine. Cela crée des conflits de versions (le "Dependency Hell").
✅ **Bonne Pratique** : Utiliser toujours des **environnements virtuels** (`venv`) pour isoler les dépendances de chaque projet. (Ce sera le sujet du Chapitre 2).

---

## Questions clés (validation des acquis du chapitre) {#questions-clés-(validation-des-acquis-du-chapitre)-1}

1.  **Python est-il un langage compilé ou interprété ?**
    *   Interprété (le code est exécuté directement par un interpréteur, pas compilé en binaire machine au préalable).

2.  **Que signifie "Typage fort" en Python ?**
    *   Cela signifie que Python ne convertit pas implicitement les données si cela risque de créer une ambiguïté ou une erreur (impossible d'additionner un texte et un nombre sans conversion explicite).

3.  **Quelle est la commande pour lire la philosophie de Python (The Zen of Python) ?**
    *   `import this`

4.  **Citez deux domaines où Python est leader mondial.**
    *   Data Science/IA et Développement Web (Backend).

5.  **Pourquoi dit-on que Python est "Batteries Included" ?**
    *   Car il est livré avec une bibliothèque standard très riche permettant d'effectuer de nombreuses tâches (réseau, fichiers, etc.) sans rien installer de plus.