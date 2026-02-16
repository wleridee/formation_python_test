---
sidebar_label: Module `sys` : Interaction avec l'Interpréteur Python
sidebar_position: 27
---

# Chapitre 27 : Module `sys` : Interaction avec l'Interpréteur Python

Arguments de ligne de commande (argv), Chemin de recherche des modules, Sortie standard (stdout, stderr), Informations sur la version

Alors que le module `os` (Chapitre 26) permet de discuter avec le système d'exploitation (Windows, Linux), le module `sys` vous connecte directement aux organes vitaux de l'**interpréteur Python** lui-même.

C'est via `sys` que votre script peut "s'écouter parler" (gérer les entrées/sorties), savoir où il se trouve, comment il a été lancé (arguments), et décider quand il doit mourir (arrêt du programme). C'est un module indispensable pour créer des scripts en ligne de commande (CLI) robustes et des outils d'automatisation.

---

## 1. Arguments de Ligne de Commande (`argv`) {#arguments-de-ligne-de-commande-argv}

### 1. Quoi
La liste `sys.argv` contient tous les arguments passés au script lors de son exécution dans le terminal.
*   `argv` signifie **Argument Vector**.
*   C'est une simple liste de chaînes de caractères (`List[str]`).

### 2. Pourquoi
Pour créer des scripts dynamiques sans modifier le code. Au lieu de coder `file = "data.csv"` en dur, vous passez le nom du fichier à l'exécution : `python process.py data.csv`.

### 3. Comment

#### A. Syntaxe de base

```python
import sys

# Si on lance : python script.py data.txt --verbose

print(sys.argv)
# Résultat : ['script.py', 'data.txt', '--verbose']
```

> ⚠️ **Note Importante** : Le premier élément `sys.argv[0]` est toujours le nom du script lui-même (ou le chemin vers celui-ci). Les "vrais" arguments commencent à l'index 1.

#### B. Cas concret : Un script de copie simple

```python
import sys

# Vérification du nombre d'arguments (Script + Source + Dest = 3)
if len(sys.argv) != 3:
    print("Usage: python copy_tool.py <source> <destination>")
    # On arrête tout si l'usage est incorrect
    sys.exit(1)

source_file = sys.argv[1]
dest_file = sys.argv[2]

print(f"Copie de '{source_file}' vers '{dest_file}'...")
# (Ici on placerait la logique de copie)
```

### 🚨 Limitations de `sys.argv`
Pour des interfaces complexes (options optionnelles, flags `--help`, typage des arguments), `sys.argv` devient vite un enfer de `if/else`.
✅ **Solution** : Utilisez le module **`argparse`** (Chapitre 33) qui est construit par-dessus `sys.argv` mais gère tout le parsing proprement.

---

## 2. Entrées et Sorties Standard (`stdout`, `stderr`) {#entrees-et-sorties-standard}

### 1. Quoi
`sys.stdout` et `sys.stderr` sont des objets fichiers ouverts en permanence qui représentent les canaux de sortie de votre terminal.
*   **stdout** (Standard Output) : Pour le résultat normal du programme.
*   **stderr** (Standard Error) : Pour les messages d'erreur et les logs.

### 2. Pourquoi
Dans les environnements professionnels (serveurs, CI/CD), on sépare les données (stdout) des erreurs (stderr). Cela permet, via le shell, de rediriger les erreurs vers un fichier de log tout en gardant les données propres pour un autre programme (pipe).

### 3. Comment

#### A. Écriture explicite

```python
import sys

# Équivalent à print("Bonjour")
sys.stdout.write("Bonjour\n")

# Pour les erreurs (apparaît souvent en rouge dans les IDE)
sys.stderr.write("Erreur critique !\n")
```

#### B. Redirection (Pipe)
Imaginez un script `process.py`.
Si vous faites `print("Data")` et `sys.stderr.write("Log")`.

Dans le terminal :
`python process.py > resultat.txt 2> erreurs.log`
*   `resultat.txt` contiendra "Data".
*   `erreurs.log` contiendra "Log".

### 4. Zone de Danger
❌ **Mélanger print et stdout.write** :
`print()` ajoute automatiquement un saut de ligne, `write()` non.
De plus, `stdout` est souvent "bufferisé" (mis en mémoire tampon). Si votre programme plante, les derniers messages peuvent ne pas s'afficher.
✅ **Bonne pratique** : En cas de crash imminent, forcez l'affichage avec `sys.stdout.flush()` ou utilisez `print(..., flush=True)`.

---

## 3. Quitter le programme (`exit`) {#quitter-le-programme-exit}

### 1. Quoi
`sys.exit()` permet d'arrêter immédiatement l'exécution du script, où que l'on soit dans le code. On peut lui passer un "code de retour" (exit code).

### 2. Pourquoi
Pour signaler au système qui a lancé le script si tout s'est bien passé ou non.
*   **0** : Succès (Tout est OK).
*   **Non-zéro (1, 2, ...)** : Erreur.

Ceci est crucial pour les chaînes d'intégration continue (CI/CD) : si votre script de test renvoie 1, le build échoue (rouge). S'il renvoie 0, il passe (vert).

### 3. Comment

```python
import sys

def connect_db():
    return False # Simulation échec

if not connect_db():
    sys.stderr.write("Impossible de se connecter à la BDD.\n")
    # Arrêt immédiat avec code d'erreur 1
    sys.exit(1)

print("Cette ligne ne sera jamais exécutée.")
```

---

## 4. Le Chemin des Modules (`path`) {#le-chemin-des-modules-path}

### 1. Quoi
`sys.path` est une liste de chaînes de caractères (chemins de dossiers). Quand vous faites `import mon_module`, Python parcourt cette liste dans l'ordre pour trouver le fichier `mon_module.py`.

### 2. Pourquoi
Comprendre `sys.path` est essentiel pour déboguer les erreurs `ModuleNotFoundError` ou pour importer des modules situés dans des dossiers non standards.

### 3. Comment

#### A. Inspecter le chemin

```python
import sys
from pprint import pprint

print("Python cherche les modules ici :")
pprint(sys.path)
```
*L'ordre typique :*
1. Le dossier du script courant.
2. Les dossiers standards de Python (`/usr/lib/python3.14`).
3. Les dossiers des paquets installés via pip (`site-packages`).

#### B. Modifier le chemin dynamiquement (Hack)
Si vous devez importer un module situé dans un dossier parent ou exotique :

```python
import sys
import os

# Ajout d'un dossier spécifique
sys.path.append("/opt/custom_libs")

# Maintenant, l'import fonctionne
import my_custom_lib
```

### 4. Zone de Danger
❌ **Modifier sys.path n'importe comment** :
Ajouter des chemins dynamiquement rend le code fragile et difficile à comprendre pour les autres développeurs.
✅ **Bonne pratique** : Préférez installer vos modules proprement (setup.py/pip) ou configurer la variable d'environnement `PYTHONPATH` avant de lancer le script.

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-27}

1.  **Que contient `sys.argv[0]` ?**
    Le nom du script en cours d'exécution (ex: `main.py`). Les vrais arguments commencent à l'index 1.

2.  **Quelle est la différence conceptuelle entre `sys.stdout` et `sys.stderr` ?**
    `stdout` est pour la sortie normale du programme (données), `stderr` est réservé aux messages de diagnostic, logs et erreurs.

3.  **Que se passe-t-il si un script se termine par `sys.exit(1)` ?**
    Le programme s'arrête immédiatement et renvoie le code d'erreur 1 au système d'exploitation, signalant un échec.

4.  **Si Python ne trouve pas un module lors d'un import, quelle liste faut-il vérifier ?**
    La liste `sys.path`.

---

## Exercices : {#exercices-27}

### Exercice 1 - Le Validateur de Version {#exercice-1---validateur-version}

🎯 **Objectif** : Utiliser `sys.version_info` et `sys.exit`.

💼 **Mise en situation** : Vous distribuez un script utilisant les nouvelles fonctionnalités de Python 3.14. Vous devez empêcher son exécution sur des versions plus anciennes pour éviter des crashs obscurs.

📝 **Énoncé** :
1.  Vérifiez la version de Python via `sys.version_info`.
2.  Si la version majeure est < 3 ou (majeure est 3 et mineure < 14) :
    - Affichez un message d'erreur sur `stderr` : "Erreur : Python 3.14+ requis (Détecté : X.Y)".
    - Quittez avec le code 1.
3.  Sinon, affichez "Environnement compatible. Démarrage..." et quittez avec le code 0.

📺 **Résultat attendu** :
*Si Python 3.10 :* `Erreur : Python 3.14+ requis (Détecté : 3.10)` (Exit code 1)
*Si Python 3.14 :* `Environnement compatible. Démarrage...` (Exit code 0)

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
import sys

# sys.version_info est un tuple nommé (major, minor, micro, releaselevel, serial)
current_major = sys.version_info.major
current_minor = sys.version_info.minor

print(f"Version détectée : {sys.version}")

# Vérification stricte
if current_major < 3 or (current_major == 3 and current_minor < 14):
    # Écriture sur le canal d'erreur
    sys.stderr.write(f"❌ Erreur : Python 3.14+ requis. (Vous utilisez {current_major}.{current_minor})\n")
    # Signal d'échec au système
    sys.exit(1)

print("✅ Environnement compatible. Démarrage...")
sys.exit(0)
```
</details>

### Exercice 2 - Mini CLI Arithmétique {#exercice-2---mini-cli}

🎯 **Objectif** : Manipuler `sys.argv` et convertir les types.

💼 **Mise en situation** : Créer un petit outil en ligne de commande pour additionner des nombres rapidement.

📝 **Énoncé** :
1.  Le script doit être appelé ainsi : `python calc.py add 5 10 2`.
2.  Si l'opération n'est pas "add", affichez "Opération inconnue" et quittez.
3.  Récupérez tous les arguments après l'opération (indice 2 et plus).
4.  Convertissez-les en `float` et faites la somme.
5.  Gérez le cas où un argument n'est pas un nombre (try-except global ou local).
6.  Affichez le résultat.

📺 **Résultat attendu** :
`python calc.py add 10 20` -> `Total : 30.0`
`python calc.py sub 10` -> `Opération inconnue`

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
import sys

# 1. Validation de base : il faut au moins le nom du script + opération + 1 nombre
if len(sys.argv) < 3:
    print("Usage: python calc.py add <n1> [n2] ...")
    sys.exit(1)

operation = sys.argv[1]

if operation != "add":
    sys.stderr.write(f"Erreur : Opération '{operation}' inconnue.\n")
    sys.exit(1)

# 2. Récupération des nombres (du 3ème élément jusqu'à la fin)
raw_numbers = sys.argv[2:] 
total = 0.0

try:
    for num_str in raw_numbers:
        total += float(num_str)
        
    print(f"Total : {total}")

except ValueError:
    sys.stderr.write("Erreur : Un des arguments n'est pas un nombre valide.\n")
    sys.exit(1)
```
</details>

### Exercice 3 - Le Trieur de Sortie (Stdout vs Stderr) {#exercice-3---trieur-sortie}

🎯 **Objectif** : Comprendre la séparation des flux.

💼 **Mise en situation** : Un script de traitement de données traite une liste. Les données valides doivent être affichées normalement (pour être redirigées vers un fichier CSV), les données invalides doivent être affichées comme erreurs (pour apparaître dans la console sans polluer le CSV).

📝 **Énoncé** :
1.  Liste de données : `["10", "20", "chat", "40", "chien"]`.
2.  Bouclez sur la liste.
3.  Si l'élément est un nombre : écrivez-le sur `sys.stdout` (avec un saut de ligne).
4.  Si c'est du texte : écrivez "Ignoré : [valeur]" sur `sys.stderr`.
5.  Testez le script dans votre terminal en redirigeant la sortie : `python script.py > data.csv`. Vérifiez que les erreurs s'affichent toujours à l'écran mais ne sont pas dans le fichier.

📺 **Résultat attendu** :
*Dans le terminal :*
```text
Ignoré : chat
Ignoré : chien
```
*Dans data.csv :*
```text
10
20
40
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
import sys

data = ["10", "20", "chat", "40", "chien"]

for item in data:
    if item.isdigit():
        # Donnée valide -> stdout (partira dans le fichier si redirection)
        sys.stdout.write(f"{item}\n")
    else:
        # Donnée invalide -> stderr (restera affiché à l'écran)
        sys.stderr.write(f"Ignoré : {item}\n")

# Note : Pas besoin de flush() ici car la fin du script flush automatiquement,
# mais dans une boucle longue, sys.stdout.flush() serait prudent.
```
</details>