---
sidebar_label: Module `pathlib` : Gestion des Chemins de Fichiers Moderne
sidebar_position: 24
---

# Chapitre 24 : Module `pathlib` : Gestion des Chemins de Fichiers Moderne

Objets Path, Opérations sur les chemins, Chemins absolus/relatifs, Création de fichiers/répertoires

Pendant des années, les développeurs Python ont lutté avec le module `os.path`, manipulant des chemins de fichiers comme de simples chaînes de caractères. C'était source d'erreurs (séparateurs `/` vs `\`), verbeux et peu intuitif.

Depuis Python 3.4 (et mature en 3.14), `pathlib` a révolutionné cette gestion en traitant les chemins non plus comme du texte, mais comme des **objets**. C'est aujourd'hui la méthode standard, robuste et orientée objet pour interagir avec le système de fichiers.

---

## 1. L'Objet Path : La fin des chaînes de caractères {#l-objet-path}

### 1. Quoi
La classe `Path` est le cœur de ce module. Elle représente un chemin (fichier ou dossier) sur votre disque dur, indépendamment du système d'exploitation (Windows, macOS, Linux).

### 2. Pourquoi
*   **Portabilité** : Windows utilise `\` et Linux `/`. `pathlib` gère cela automatiquement.
*   **Lisibilité** : Fini les concaténations de chaînes hasardeuses.
*   **Puissance** : L'objet possède des méthodes intégrées pour se manipuler lui-même.

### 3. Comment

#### A. Importation et Création

```python
from pathlib import Path

# Création d'un objet Path (ne crée pas le fichier sur le disque, juste la représentation)
p = Path("data") / "2026" / "report.txt"

print(p) 
# Sur Windows : data\2026\report.txt
# Sur Linux/Mac : data/2026/report.txt
```

#### B. L'opérateur `/` (La "Killer Feature")
Au lieu d'utiliser `os.path.join()`, vous pouvez utiliser l'opérateur de division `/` pour construire des chemins intuitivement.

```python
folder = Path.home() # Dossier utilisateur (ex: /home/alice)
subfolder = "Documents"
filename = "todo.txt"

# Construction intuitive
full_path = folder / subfolder / filename

print(full_path)
# /home/alice/Documents/todo.txt
```

### 4. Zone de Danger
❌ **Concaténation de chaînes** :
`Path("dossier") + "/" + "fichier"` est une mauvaise pratique. Cela retourne au problème des séparateurs d'OS.
✅ **Utilisez toujours** l'opérateur `/` ou `Path.joinpath()`.

---

## 2. Analyser et Vérifier un Chemin {#analyser-et-verifier}

### 1. Quoi
Un objet `Path` permet de décomposer un chemin pour en extraire le nom, l'extension, ou le parent, et de vérifier son état réel sur le disque.

### 2. Pourquoi
Pour éviter de manipuler des index de chaînes (`string[:-3]`) pour trouver une extension, ce qui est fragile (ex: `archive.tar.gz`).

### 3. Comment

#### A. Dissection d'un chemin

```python
file_path = Path("/usr/bin/python3.14.exe")

print(file_path.name)   # python3.14.exe
print(file_path.stem)   # python3.14 (sans extension finale)
print(file_path.suffix) # .exe
print(file_path.parent) # /usr/bin
```

#### B. Vérification d'existence (Méthodes booléennes)

```python
p = Path("config.json")

if p.exists():
    if p.is_file():
        print("C'est un fichier.")
    elif p.is_dir():
        print("C'est un dossier.")
else:
    print("Le chemin n'existe pas.")
```

---

## 3. Lecture, Écriture et Création {#lecture-ecriture-creation}

### 1. Quoi
`pathlib` ne sert pas qu'à manipuler des noms. Il peut aussi créer des dossiers, des fichiers vides, et même lire/écrire du texte rapidement sans passer par `open()`.

### 2. Pourquoi
Simplifier les scripts d'automatisation (scripting). Pour des opérations simples, cela réduit 3 lignes de code en 1 seule.

### 3. Comment

#### A. Créer des dossiers (`mkdir`)

```python
folder = Path("backups/janvier/semaine1")

# parents=True : crée les dossiers intermédiaires (backups, janvier...) si besoin
# exist_ok=True : ne plante pas si le dossier existe déjà
folder.mkdir(parents=True, exist_ok=True)
```

#### B. Lire et Écrire (Raccourcis)

```python
readme = Path("README.md")

# Écriture rapide (ouvre, écrit, ferme)
readme.write_text("# Mon Projet\nDescription...", encoding="utf-8")

# Lecture rapide
content = readme.read_text(encoding="utf-8")
print(content)
```

#### C. Rechercher des fichiers (`glob`)
Pour lister tous les fichiers `.py` du dossier courant :

```python
current_dir = Path(".")
# glob retourne un générateur d'objets Path
for py_file in current_dir.glob("*.py"):
    print(f"Script trouvé : {py_file.name}")
```

---

## 4. Chemins Absolus vs Relatifs {#absolus-vs-relatifs}

### 1. Quoi
*   **Relatif** : Part de l'endroit où le script est exécuté (ex: `images/logo.png`).
*   **Absolu** : Part de la racine du disque (ex: `C:\Projets\App\images\logo.png`).

### 2. Pourquoi
Les chemins relatifs sont pratiques pour le développement mais peuvent casser si le script est appelé depuis un autre dossier. Convertir en absolu (`resolve`) permet de sécuriser les accès.

### 3. Comment

```python
# Chemin relatif
rel_path = Path("logs/error.log")

# Conversion en absolu (résout aussi les liens symboliques et les "..")
abs_path = rel_path.resolve()

print(f"Le fichier sera cherché ici : {abs_path}")
```

### 🚨 Limitations de `resolve()`
Sur Windows, `resolve()` peut parfois échouer si le fichier n'existe pas (selon les versions de Python), bien que ce comportement se soit amélioré en 3.10+. Pour être sûr, travaillez souvent avec `Path(__file__).parent` pour ancrer vos chemins par rapport au script lui-même.

```python
# Astuce Pro : Ancrer le chemin par rapport au fichier de script
BASE_DIR = Path(__file__).resolve().parent
DATA_FILE = BASE_DIR / "data.csv" # Toujours correct, peu importe d'où on lance le script
```

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-24}

1.  **Comment concaténer deux parties de chemin avec `pathlib` ?**
    En utilisant l'opérateur de division `/` (ex: `Path("dossier") / "fichier.txt"`).

2.  **Quelle méthode utiliser pour créer un dossier et tous ses parents manquants ?**
    `.mkdir(parents=True, exist_ok=True)`.

3.  **Comment récupérer l'extension d'un fichier (ex: `.txt`) depuis un objet Path ?**
    Via l'attribut `.suffix`.

4.  **Quel est l'avantage de `read_text()` par rapport à `open()` ?**
    C'est un raccourci syntaxique qui gère l'ouverture, la lecture et la fermeture en une seule instruction. Idéal pour les petits fichiers.

---

## Exercices : {#exercices-24}

### Exercice 1 - L'Architecte de Dossiers {#exercice-1---architecte}

🎯 **Objectif** : Utiliser `mkdir` avec paramètres pour structurer un projet.

💼 **Mise en situation** : Vous initialisez un nouveau projet Data Science. Vous devez créer une structure standardisée automatiquement.

📝 **Énoncé** :
1.  Créez un dossier racine nommé `mon_projet_data`.
2.  À l'intérieur, créez les sous-dossiers : `data/raw`, `data/processed`, `notebooks`, `models`.
3.  Utilisez une boucle pour éviter de répéter le code.
4.  Dans `data/raw`, créez un fichier vide `.keep` (utilisez `.touch()`) pour que git conserve le dossier.

📺 **Résultat attendu** :
Dossiers créés sur le disque. Pas de sortie console sauf confirmation.

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
from pathlib import Path

# Définition de la racine
root = Path("mon_projet_data")

# Liste des dossiers à créer
folders = [
    root / "data" / "raw",
    root / "data" / "processed",
    root / "notebooks",
    root / "models"
]

for folder in folders:
    # Création récursive, ne plante pas si existe déjà
    folder.mkdir(parents=True, exist_ok=True)
    print(f"✅ Dossier créé : {folder}")

# Création du fichier vide .keep
keep_file = root / "data" / "raw" / ".keep"
keep_file.touch()
print(f"✅ Fichier créé : {keep_file}")
```
</details>

### Exercice 2 - Le Nettoyeur d'Extensions (Glob & Rename) {#exercice-2---nettoyeur}

🎯 **Objectif** : Itérer sur des fichiers (`glob`) et les renommer (`rename`).

💼 **Mise en situation** : Un bug a généré des fichiers de logs avec l'extension `.txt` au lieu de `.log`. Vous devez corriger cela en masse.

📝 **Énoncé** :
1.  Créez un dossier `logs_temp` et ajoutez-y 3 fichiers : `error.txt`, `info.txt`, `warn.txt`.
2.  Parcourez tous les fichiers `.txt` de ce dossier.
3.  Pour chaque fichier, changez son extension en `.log` (astuce : utilisez `.with_suffix(".log")` pour générer le nouveau nom cible).
4.  Renommez le fichier sur le disque.
5.  Affichez l'ancien et le nouveau nom.

📺 **Résultat attendu** :
```text
Renommé : logs_temp/error.txt -> logs_temp/error.log
Renommé : logs_temp/info.txt -> logs_temp/info.log
...
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
from pathlib import Path

# Setup : Création de l'environnement de test
work_dir = Path("logs_temp")
work_dir.mkdir(exist_ok=True)
(work_dir / "error.txt").touch()
(work_dir / "info.txt").touch()
(work_dir / "warn.txt").touch()

# Traitement
# On itère sur tous les .txt
for old_path in work_dir.glob("*.txt"):
    # On calcule le nouveau chemin (objet Path)
    # .with_suffix remplace l'extension existante par la nouvelle
    new_path = old_path.with_suffix(".log")
    
    # Action sur le système de fichiers
    old_path.rename(new_path)
    
    print(f"Renommé : {old_path.name} -> {new_path.name}")
```
</details>

### Exercice 3 - L'Inventaire Intelligent (Attributs de fichiers) {#exercice-3---inventaire}

🎯 **Objectif** : Analyser les métadonnées (`stat`) et filtrer.

💼 **Mise en situation** : Vous devez lister tous les fichiers "lourds" (> 1Ko) d'un répertoire pour faire du nettoyage.

📝 **Énoncé** :
1.  Créez un dossier `documents` avec quelques fichiers (utilisez `write_text` pour en remplir un avec beaucoup de texte pour qu'il dépasse 10 octets, et un autre vide).
2.  Parcourez le dossier.
3.  Ignorez les sous-dossiers (utilisez `.is_file()`).
4.  Pour chaque fichier, récupérez sa taille via `.stat().st_size`.
5.  Affichez le nom et la taille seulement si la taille > 10 octets.

📺 **Résultat attendu** :
```text
big_file.txt : 500 octets
(small_file.txt n'apparaît pas)
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
from pathlib import Path

p = Path("documents")
p.mkdir(exist_ok=True)

# Création des fichiers
(p / "small.txt").write_text("coucou") # 6 octets
(p / "big.txt").write_text("Texte " * 100) # 600 octets

print(f"--- Analyse de {p} ---")

for entry in p.iterdir(): # iterdir liste tout (fichiers et dossiers)
    if entry.is_file():
        # Récupération des infos système du fichier
        size = entry.stat().st_size
        
        if size > 10:
            print(f"Fichier lourd détecté : {entry.name} ({size} octets)")
```
</details>