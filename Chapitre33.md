---
sidebar_label: Module `argparse` : Création d'Interfaces CLI
sidebar_position: 33
---

# Chapitre 33 : Module `argparse` : Création d'Interfaces CLI

Parser des arguments, Ajouter des arguments (add_argument), Arguments positionnels, Arguments optionnels

Créer des outils en ligne de commande (CLI - Command Line Interface) est une compétence fondamentale pour tout développeur Python. Que ce soit pour automatiser une tâche, configurer un serveur ou traiter des fichiers, votre script doit être pilotable sans modifier le code source.

Le module `argparse` de la bibliothèque standard est l'outil de référence pour construire ces interfaces. Il gère automatiquement l'aide (`--help`), valide les types des entrées et fournit des messages d'erreur clairs.

---

## 1. Création du Parser et Analyse (`ArgumentParser`) {#creation-du-parser}

### 1. Quoi
L'objet `ArgumentParser` est le chef d'orchestre. Il contient toutes les informations nécessaires pour convertir la ligne de commande (une liste de chaînes de caractères) en objets Python utilisables.

### 2. Pourquoi
Pour ne pas avoir à analyser manuellement `sys.argv` (la liste brute des arguments), ce qui est fastidieux et source d'erreurs. `argparse` offre une documentation automatique et une gestion standardisée.

### 3. Comment

#### A. Syntaxe de base

```python
import argparse

# 1. Création du parser avec une description
parser = argparse.ArgumentParser(description="Mon super outil CLI.")

# 2. Analyse des arguments (vide pour l'instant)
args = parser.parse_args()

# 3. Utilisation
print("Le script a démarré.")
```

Si vous lancez ce script avec l'option `-h` ou `--help`, `argparse` génère automatiquement ceci :
```text
$ python script.py --help
usage: script.py [-h]

Mon super outil CLI.

options:
  -h, --help  show this help message and exit
```

---

## 2. Arguments Positionnels (`add_argument`) {#arguments-positionnels}

### 1. Quoi
Ce sont les arguments **obligatoires** dont la signification dépend de leur ordre (leur position). Par exemple, dans `cp source.txt dest.txt`, `source.txt` est le premier argument positionnel.

### 2. Pourquoi
Indispensable pour les entrées principales du script (ex: le fichier à traiter, l'URL à scanner) sans lesquelles le script ne peut pas fonctionner.

### 3. Comment

#### A. Syntaxe

```python
import argparse

parser = argparse.ArgumentParser()

# Le nom sans tiret (-) définit un argument positionnel
parser.add_argument("filename", help="Le fichier à traiter")

args = parser.parse_args()

print(f"Traitement du fichier : {args.filename}")
```

#### B. Cas concret : Typage automatique
`argparse` peut convertir automatiquement l'entrée (qui est toujours une `str` par défaut) vers `int`, `float`, etc.

```python
import argparse

parser = argparse.ArgumentParser()
parser.add_argument("x", type=int, help="Le nombre à mettre au carré")

args = parser.parse_args()

# args.x est déjà un int, pas besoin de conversion manuelle
print(f"{args.x}^2 = {args.x ** 2}")
```

### 4. Zone de Danger
❌ **Ordre rigide** : L'utilisateur DOIT fournir les arguments dans l'ordre exact. Si vous avez 3 arguments positionnels, l'expérience utilisateur peut devenir confuse.
✅ **Bonne pratique** : Limitez les arguments positionnels à 1 ou 2 essentiels. Pour le reste, utilisez des arguments optionnels.

---

## 3. Arguments Optionnels (Flags) {#arguments-optionnels}

### 1. Quoi
Ce sont des arguments précédés de `-` (court) ou `--` (long). Ils ne sont pas obligatoires (sauf mention contraire) et peuvent être placés n'importe où dans la commande.

### 2. Pourquoi
Pour modifier le comportement par défaut du script : activer un mode verbeux, changer un port, spécifier un dossier de sortie alternatif.

### 3. Comment

#### A. Syntaxe de base

```python
import argparse

parser = argparse.ArgumentParser()

# Les tirets définissent un argument optionnel
# 'store_true' signifie : si l'argument est présent, la variable vaut True, sinon False
parser.add_argument("-v", "--verbose", action="store_true", help="Activer les logs détaillés")

# Argument optionnel avec valeur (ex: --port 8080)
# 'default' définit la valeur si l'utilisateur ne précise rien
parser.add_argument("--port", type=int, default=5000, help="Port du serveur (défaut: 5000)")

args = parser.parse_args()

if args.verbose:
    print("Mode verbeux activé !")

print(f"Démarrage sur le port {args.port}")
```

#### B. Choix restreints (`choices`)
Vous pouvez forcer l'utilisateur à choisir parmi une liste de valeurs valides.

```python
parser.add_argument("--env", choices=["dev", "prod", "test"], default="dev")
```

Si l'utilisateur tape `--env staging`, `argparse` affichera une erreur claire et arrêtera le script :
`error: argument --env: invalid choice: 'staging' (choose from 'dev', 'prod', 'test')`

---

## 4. Gestion Avancée et Bonnes Pratiques {#gestion-avancee}

### 1. Groupes Mutuellement Exclusifs
Parfois, deux options sont incompatibles (ex: on ne peut pas être en mode "silencieux" ET "verbeux" en même temps).

```python
group = parser.add_mutually_exclusive_group()
group.add_argument("-v", "--verbose", action="store_true")
group.add_argument("-q", "--quiet", action="store_true")
```

### 2. Documentation et Métadonnées
Un bon CLI doit être auto-documenté.

```python
parser = argparse.ArgumentParser(
    prog="MonOutil",
    description="Compresseur d'images ultra-rapide",
    epilog="Pour plus d'infos, visitez notre wiki."
)
```

### 3. Typage de fichier (`FileType`)
`argparse` peut ouvrir les fichiers pour vous.

```python
# Ouvre automatiquement le fichier en lecture ('r')
# Si le fichier n'existe pas, argparse lève l'erreur avant même que votre code ne s'exécute
parser.add_argument('input_file', type=argparse.FileType('r'))
```

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-33}

1.  **Quelle est la différence syntaxique entre un argument positionnel et optionnel dans `add_argument` ?**
    Les arguments optionnels commencent par `-` ou `--` (ex: `--output`). Les positionnels n'ont pas de préfixe (ex: `filename`).

2.  **Comment faire en sorte qu'un argument booléen (`flag`) soit `True` s'il est présent et `False` sinon ?**
    En utilisant le paramètre `action="store_true"`.

3.  **À quoi sert le paramètre `type` dans `add_argument` ?**
    Il permet de convertir automatiquement la chaîne de caractères reçue en ligne de commande vers un type Python (int, float, etc.) et de valider l'entrée.

4.  **Comment définir une valeur par défaut pour un argument optionnel ?**
    Avec le paramètre `default=...` (ex: `default=8080`).

---

## Exercices : {#exercices-33}

### Exercice 1 - La Calculatrice CLI {#exercice-1---calculatrice-cli}

🎯 **Objectif** : Manipuler arguments positionnels, typage et choix.

💼 **Mise en situation** : Vous devez créer un petit utilitaire rapide pour faire des opérations mathématiques depuis le terminal.

📝 **Énoncé** :
1.  Créez un script qui accepte 3 arguments :
    - `x` : un nombre (float).
    - `y` : un nombre (float).
    - `--op` : l'opération à effectuer, qui peut être "add", "sub", "mul". (Défaut : "add").
2.  Effectuez le calcul et affichez le résultat.
3.  Gérez l'affichage de l'aide automatique.

📺 **Résultat attendu** :
```bash
$ python script.py 10 5 --op mul
Resultat : 50.0

$ python script.py 10 5
Resultat : 15.0  (car add par défaut)
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
import argparse

# 1. Création du parser
parser = argparse.ArgumentParser(description="Calculatrice CLI simple.")

# 2. Ajout des arguments
parser.add_argument("x", type=float, help="Premier nombre")
parser.add_argument("y", type=float, help="Deuxième nombre")

# Argument optionnel avec choix restreints
parser.add_argument(
    "--op", 
    choices=["add", "sub", "mul"], 
    default="add",
    help="Opération à effectuer (add, sub, mul)"
)

# 3. Parsing
args = parser.parse_args()

# 4. Logique métier
result = 0.0
if args.op == "add":
    result = args.x + args.y
elif args.op == "sub":
    result = args.x - args.y
elif args.op == "mul":
    result = args.x * args.y

print(f"Resultat : {result}")
```
</details>

### Exercice 2 - Le Générateur de Mots de Passe {#exercice-2---generateur-mdp}

🎯 **Objectif** : Gérer des flags booléens (`store_true`) et des entiers.

💼 **Mise en situation** : L'équipe sysadmin a besoin d'un outil pour générer des mots de passe aléatoires sécurisés selon des critères variables.

📝 **Énoncé** :
1.  Créez un script `genpass.py`.
2.  Argument `length` (optionnel) : longueur du mot de passe (int), défaut 12.
3.  Flag `--numbers` : si présent, inclure des chiffres.
4.  Flag `--symbols` : si présent, inclure des symboles spéciaux.
5.  Générez et affichez le mot de passe (utilisez le module `random` et `string`).

📺 **Résultat attendu** :
```bash
$ python genpass.py --length 8 --numbers
aB4k9z1m

$ python genpass.py
abcDefghijKl (lettres seules par défaut)
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
import argparse
import random
import string

# 1. Configuration
parser = argparse.ArgumentParser(description="Générateur de mot de passe.")

# Argument --length avec une valeur par défaut
parser.add_argument("--length", type=int, default=12, help="Longueur du mot de passe")

# Flags booléens
parser.add_argument("--numbers", action="store_true", help="Inclure des chiffres")
parser.add_argument("--symbols", action="store_true", help="Inclure des symboles")

args = parser.parse_args()

# 2. Construction de l'alphabet
chars = string.ascii_letters # Toujours inclus

if args.numbers:
    chars += string.digits
if args.symbols:
    chars += string.punctuation

# 3. Génération
# On choisit 'length' caractères au hasard
password = "".join(random.choices(chars, k=args.length))

print(f"Mot de passe généré : {password}")
```
</details>

### Exercice 3 - Le Copieur de Fichier Sécurisé {#exercice-3---copieur-secure}

🎯 **Objectif** : Utiliser `FileType` et gérer des exceptions.

💼 **Mise en situation** : Un script de backup qui lit un fichier source et l'écrit ailleurs, en passant tout le contenu en majuscules (pour l'exercice).

📝 **Énoncé** :
1.  Argument `source` : fichier en lecture (`'r'`).
2.  Argument `dest` : fichier en écriture (`'w'`).
3.  Lisez le contenu de `source`, convertissez en majuscules, écrivez dans `dest`.
4.  Affichez un message de succès.
5.  Testez en appelant le script avec un fichier source inexistant pour voir l'erreur gérée par argparse.

📺 **Résultat attendu** :
```bash
$ python copy.py input.txt output.txt
Copie terminée avec succès.

$ python copy.py inconnu.txt output.txt
usage: copy.py [-h] source dest
copy.py: error: argument source: can't open 'inconnu.txt': [Errno 2] No such file or directory
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
import argparse

parser = argparse.ArgumentParser(description="Copieur de fichier MAJUSCULE.")

# argparse va essayer d'ouvrir les fichiers
# Si ça échoue, il quitte le script proprement avec un message d'erreur standard
parser.add_argument("source", type=argparse.FileType('r'), help="Fichier source")
parser.add_argument("dest", type=argparse.FileType('w'), help="Fichier destination")

args = parser.parse_args()

try:
    # Lecture
    content = args.source.read()
    
    # Traitement
    upper_content = content.upper()
    
    # Écriture
    args.dest.write(upper_content)
    
    print("Copie terminée avec succès.")
    
finally:
    # Bonne pratique : fermer les handles ouverts par argparse
    # Note : argparse ne les ferme pas automatiquement
    args.source.close()
    args.dest.close()
```
</details>