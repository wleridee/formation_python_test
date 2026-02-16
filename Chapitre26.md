---
sidebar_label: Module `os` : Interaction avec le Système d'Exploitation
sidebar_position: 26
---

# Chapitre 26 : Module `os` : Interaction avec le Système d'Exploitation

Variables d'environnement, Opérations sur les fichiers/répertoires, Exécution de commandes, Information sur le processus

Le module `os` est le "couteau suisse" historique de Python pour dialoguer avec le système d'exploitation (Windows, Linux, macOS). Avant l'arrivée de modules spécialisés comme `pathlib` (fichiers) ou `subprocess` (commandes), `os` faisait tout.

Aujourd'hui, il reste incontournable pour certaines tâches de bas niveau : gérer les variables d'environnement (sécurité), obtenir des infos sur le processeur (parallélisme) ou naviguer dans le système de fichiers de manière portable.

---

## 1. Variables d'Environnement : Sécurité et Config {#variables-environnement}

### 1. Quoi
Les variables d'environnement sont des paires clé-valeur gérées par le système d'exploitation, en dehors de votre code Python. Exemples courants : `PATH`, `USER`, ou des clés secrètes comme `DB_PASSWORD`.

### 2. Pourquoi
*   **Sécurité** : Ne jamais stocker de mots de passe en dur dans le code source (`git`).
*   **Configuration** : Changer le comportement de l'app (Dev vs Prod) sans modifier le code.

### 3. Comment

#### A. Lecture (`environ` et `getenv`)

```python
import os

# 1. Accès direct via os.environ (Dictionnaire)
# ⚠️ Lève une KeyError si la variable n'existe pas
try:
    user = os.environ["USER"] # Ou 'USERNAME' sur Windows
    print(f"Utilisateur système : {user}")
except KeyError:
    print("Variable USER non définie.")

# 2. Accès sécurisé via os.getenv (Recommandé)
# Renvoie None (ou une valeur par défaut) si la clé manque
db_host = os.getenv("DB_HOST", "localhost")
print(f"Hôte de la base de données : {db_host}")
```

#### B. Écriture (Temporaire)
Vous pouvez modifier les variables d'environnement pour la durée de votre script. Cela n'affecte pas le système de manière permanente.

```python
# Définit une variable pour les processus enfants ou la suite du script
os.environ["API_KEY"] = "12345-SECRET"
```

### 4. Zone de Danger
❌ **Committer des secrets** :
Ne faites jamais `API_KEY = "12345"` dans votre fichier `.py`. Utilisez `os.getenv("API_KEY")` et définissez la valeur dans votre système ou un fichier `.env` (non versionné).

---

## 2. Navigation et Manipulation de Fichiers {#navigation-et-fichiers}

### 1. Quoi
Le module `os` permet de se déplacer dans les dossiers (`cd`), de lister les fichiers (`ls`) et de les manipuler (`rm`, `mkdir`).

### 2. Pourquoi
Bien que `pathlib` soit plus moderne pour manipuler des *chemins*, `os` reste souvent utilisé pour changer le répertoire de travail courant (CWD - Current Working Directory) ou dans des bases de code existantes.

### 3. Comment

#### A. Navigation

```python
import os

# Où suis-je ? (pwd)
current_dir = os.getcwd()
print(f"Dossier courant : {current_dir}")

# Changer de dossier (cd)
os.chdir("..") # Remonte d'un niveau
print(f"Nouveau dossier : {os.getcwd()}")
```

#### B. Actions sur les fichiers/dossiers

```python
import os

# Créer un dossier
# exist_ok n'existe pas ici, il faut gérer l'erreur manuellement (contrairement à pathlib)
try:
    os.mkdir("temp_logs")
except FileExistsError:
    print("Le dossier existe déjà.")

# Renommer
os.rename("temp_logs", "logs_archive")

# Supprimer un fichier
# os.remove("fichier.txt")

# Supprimer un dossier VIDE
# os.rmdir("logs_archive")
```

### 🚨 Limitations de `os.path` vs `pathlib`
Le sous-module `os.path` contient des fonctions comme `os.path.join("a", "b")`.
✅ **Conseil 2026** : Préférez `pathlib` (Chapitre 24) pour la manipulation de chemins. Utilisez `os` pour les opérations bas niveau (permissions, liens symboliques, variables d'env).

---

## 3. Informations sur le Processus et le Système {#infos-systeme}

### 1. Quoi
Obtenir des détails sur le programme en cours d'exécution (PID) ou la machine (CPU).

### 2. Pourquoi
*   **PID (Process ID)** : Utile pour les logs ou pour tuer un processus bloqué.
*   **CPU Count** : Indispensable pour calibrer le parallélisme (combien de threads lancer ?).

### 3. Comment

```python
import os

# ID du processus courant
pid = os.getpid()
print(f"Mon PID est : {pid}")

# Nombre de cœurs logiques (CPU)
cpu_count = os.cpu_count()
print(f"Cœurs disponibles : {cpu_count}")

# Séparateur de dossiers selon l'OS ('/' ou '\')
print(f"Séparateur de chemin : {os.sep}")
```

---

## 4. Exécution de Commandes Système {#execution-commandes}

### 1. Quoi
Lancer une commande externe (comme si vous étiez dans le terminal) depuis Python.

### 2. Pourquoi
Automatiser des tâches système : lancer une sauvegarde, un script shell, ou un convertisseur d'images.

### 3. Comment

#### A. `os.system()` (Approche basique)
Exécute la commande et renvoie le code de sortie (0 = succès).

```python
import os

# Lance un ping (commande différente selon OS)
# Windows : ping -n 1 google.com
# Linux/Mac : ping -c 1 google.com
command = "ping -c 1 google.com" if os.name != 'nt' else "ping -n 1 google.com"

exit_code = os.system(command)

if exit_code == 0:
    print("✅ Commande réussie")
else:
    print("❌ Échec de la commande")
```

### 4. Zone de Danger
❌ **Injection de commande** :
```python
file = "file; rm -rf /" # 😱 DANGER
os.system(f"cat {file}")
```
Si vous passez des entrées utilisateur à `os.system`, un attaquant peut exécuter n'importe quoi.

✅ **Bonne Pratique** :
`os.system` est vieillissant. Pour des besoins complexes (récupérer la sortie texte, gérer les erreurs, sécurité), on utilise le module **`subprocess`** (hors programme de ce chapitre mais à connaître de nom).

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-26}

1.  **Quelle est la différence entre `os.environ["KEY"]` et `os.getenv("KEY")` ?**
    L'accès par dictionnaire `[]` lève une erreur `KeyError` si la clé n'existe pas, alors que `getenv()` renvoie `None` (ou une valeur par défaut) sans planter.

2.  **Pourquoi `os.cpu_count()` est-il utile ?**
    Il permet d'adapter dynamiquement la charge de travail (nombre de processus/threads) à la puissance de la machine hôte.

3.  **Quelle fonction permet de connaître le répertoire de travail actuel ?**
    `os.getcwd()` (Get Current Working Directory).

4.  **Peut-on supprimer un dossier non vide avec `os.rmdir()` ?**
    Non, `os.rmdir()` ne supprime que des dossiers vides. Pour supprimer un dossier et son contenu, il faut utiliser `shutil.rmtree()` (module `shutil`).

---

## Exercices : {#exercices-26}

### Exercice 1 - Le Chargeur de Configuration Sécurisé {#exercice-1-config-secure}

🎯 **Objectif** : Utiliser `os.getenv` pour configurer une application.

💼 **Mise en situation** : Vous développez un connecteur de base de données. Il doit fonctionner sur votre PC (local) et sur le serveur de prod sans changer le code.

📝 **Énoncé** :
1.  Créez un script qui tente de récupérer la variable d'environnement `APP_MODE`.
2.  Si elle n'est pas définie, elle vaut "DEV" par défaut.
3.  Tentez de récupérer `DB_PASSWORD`.
4.  Si le mode est "PROD" et que le mot de passe est vide/absent, levez une `EnvironmentError` avec un message d'alerte.
5.  Sinon, affichez "Démarrage en mode [MODE]...".

📺 **Résultat attendu** :
*Sans variable définie :* `Démarrage en mode DEV...`
*Avec APP_MODE="PROD" (et sans mot de passe) :* `Erreur : Mot de passe BDD requis en PROD`

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
import os

# Simulation : Décommentez la ligne suivante pour tester le crash en PROD
# os.environ["APP_MODE"] = "PROD"

def start_app():
    # Récupération avec valeur par défaut
    mode = os.getenv("APP_MODE", "DEV")
    
    # Récupération stricte (None si absent)
    password = os.getenv("DB_PASSWORD")

    if mode == "PROD" and not password:
        # On interdit le démarrage en prod sans mot de passe
        raise EnvironmentError("CRITIQUE : La variable DB_PASSWORD est manquante pour la PROD !")

    print(f"🚀 Démarrage de l'application en mode {mode}")
    if password:
        print("Connexion BDD sécurisée établie.")
    else:
        print("Utilisation de la BDD locale (sans mot de passe).")

try:
    start_app()
except EnvironmentError as e:
    print(e)
```
</details>

### Exercice 2 - L'Explorateur de Dossiers {#exercice-2-explorateur}

🎯 **Objectif** : Utiliser `os.getcwd`, `os.listdir` et `os.path`.

💼 **Mise en situation** : Vous devez générer un rapport rapide listant tous les fichiers Python du dossier courant.

📝 **Énoncé** :
1.  Affichez le dossier courant.
2.  Listez tous les éléments du dossier (`os.listdir`).
3.  Parcourez la liste.
4.  Si l'élément se termine par `.py` :
    - Affichez "Script trouvé : [Nom]".

*(Note : Même si `pathlib` est mieux pour cela, faites-le avec `os` pour pratiquer).*

📺 **Résultat attendu** :
```text
Exploration de : /home/user/python-course
Script trouvé : main.py
Script trouvé : utils.py
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
import os

# Récupération du chemin actuel
cwd = os.getcwd()
print(f"Exploration de : {cwd}")

# Liste brute des noms de fichiers/dossiers (chaînes de caractères)
entries = os.listdir(cwd)

for entry in entries:
    # Vérification simple sur la chaîne de caractères
    if entry.endswith(".py"):
        print(f"Script trouvé : {entry}")
```
</details>

### Exercice 3 - Nettoyage Système Multi-plateforme {#exercice-3-nettoyage}

🎯 **Objectif** : Combiner `os.system` (ou logique OS) et variables d'environnement.

💼 **Mise en situation** : Vous voulez effacer l'écran du terminal pour rendre l'affichage plus propre au lancement de votre programme. La commande dépend de l'OS (`cls` sur Windows, `clear` sur Unix).

📝 **Énoncé** :
1.  Détectez le système d'exploitation via `os.name` (`nt` pour Windows, `posix` pour Linux/Mac).
2.  Définissez la commande appropriée (`cls` ou `clear`).
3.  Exécutez la commande via `os.system`.
4.  Ensuite, affichez le nombre de CPU et le nom de l'utilisateur (`os.getlogin()` ou variable d'env) au centre de l'écran.

📺 **Résultat attendu** :
Le terminal s'efface, puis affiche les infos système.

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
import os

def clean_and_report():
    # 1. Détection de l'OS
    if os.name == 'nt':
        cmd = 'cls' # Windows
    else:
        cmd = 'clear' # Linux / Mac / Unix

    # 2. Exécution de la commande système
    os.system(cmd)

    # 3. Récupération des infos
    cpus = os.cpu_count()
    
    # os.getlogin() peut échouer dans certains environnements (ex: docker/cron)
    # On utilise une solution de repli avec les variables d'env
    user = os.getenv('USER') or os.getenv('USERNAME') or "Inconnu"

    print("--- RAPPORT SYSTÈME ---")
    print(f"Utilisateur : {user}")
    print(f"Puissance   : {cpus} cœurs logiques")

clean_and_report()
```
</details>