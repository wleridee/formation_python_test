---
sidebar_label: "Manipulation de Fichiers: Lecture et Écriture (Textes)"
sidebar_position: 21
difficulty: "confirmé"
---

# Manipulation de Fichiers: Lecture et Écriture (Textes) {#manipulation-fichiers-21}

La plupart des applications informatiques ont besoin d'interagir avec des fichiers, que ce soit pour lire des données de configuration, traiter des datasets, ou enregistrer des logs. Savoir manipuler des fichiers est une compétence fondamentale en Python.

Nous nous concentrerons ici sur les fichiers texte, qui sont lisibles par l'homme. La manipulation de fichiers binaires (images, exécutables) suit une logique similaire mais avec des modes d'ouverture différents.

## 1. La Bonne Pratique : Le Gestionnaire de Contexte `with` {#gestionnaire-contexte-with-21}

### Quoi
Pour ouvrir un fichier, on utilise la fonction `open()`. Cependant, la manière la plus sûre et la plus recommandée de le faire est d'utiliser l'instruction `with`. C'est ce qu'on appelle un **gestionnaire de contexte**.

```python
with open('mon_fichier.txt', 'r') as f:
    # Le code qui manipule le fichier 'f' va ici
    contenu = f.read()

# Une fois sorti du bloc 'with', le fichier est automatiquement fermé.
```

```mermaid
graph TD
    A[Début du bloc `with open(...)`] --> B{Le fichier s'ouvre};
    B --> C[Opérations de lecture/écriture sur le fichier];
    C --> D{Fin du bloc `with`};
    D -- Normalement --> E[Le fichier est automatiquement fermé];
    C -- Une erreur se produit --> E;
```

### Pourquoi
Le gestionnaire de contexte `with` garantit que le fichier sera **automatiquement et correctement fermé**, même si une erreur se produit à l'intérieur du bloc. Oublier de fermer un fichier peut entraîner des fuites de ressources, voire la corruption de données. La syntaxe `with` élimine ce risque.

### Comment
La fonction `open()` prend principalement deux arguments :
1.  Le **chemin** vers le fichier.
2.  Le **mode** d'ouverture. Les principaux modes pour les fichiers texte sont :
    -   `'r'` : **Read** (Lecture). C'est le mode par défaut. Le fichier doit exister, sinon une `FileNotFoundError` est levée.
    -   `'w'` : **Write** (Écriture). Crée le fichier s'il n'existe pas. S'il existe, **son contenu est écrasé**.
    -   `'a'` : **Append** (Ajout). Crée le fichier s'il n'existe pas. S'il existe, les nouvelles données sont ajoutées à la fin.

**Cas Réel : Lire le contenu d'un fichier de configuration.**
Supposons un fichier `config.txt` :
```text
# Fichier de configuration
user=admin
host=localhost
```

```python
try:
    with open('config.txt', 'r', encoding='utf-8') as config_file:
        # .read() lit tout le contenu du fichier dans une seule chaîne de caractères
        contenu_total = config_file.read()
        print("--- Contenu avec .read() ---")
        print(contenu_total)

    with open('config.txt', 'r', encoding='utf-8') as config_file:
        # .readlines() lit toutes les lignes et les retourne dans une liste de chaînes
        lignes = config_file.readlines()
        print("\n--- Contenu avec .readlines() ---")
        print(lignes) # Remarquez les '\n' à la fin de chaque ligne
        
except FileNotFoundError:
    print("Erreur : Le fichier 'config.txt' n'a pas été trouvé.")
```

### Zone de Danger
*   **Encodage** : Ne pas spécifier l'encodage avec `encoding='utf-8'` est une source fréquente de bugs. Votre code pourrait fonctionner sur votre machine mais planter sur une autre (ou avec des fichiers contenant des accents). Prenez l'habitude de toujours le spécifier pour les fichiers texte.
*   **Fichiers volumineux** : Utiliser `.read()` ou `.readlines()` sur un fichier de plusieurs gigaoctets chargera tout son contenu en mémoire, ce qui peut faire planter votre programme. Pour les gros fichiers, il est préférable de lire ligne par ligne en itérant directement sur l'objet fichier.

---

## 2. Écrire et Ajouter du Contenu {#ecrire-ajouter-contenu-21}

### Quoi
Les modes `'w'` (write) et `'a'` (append) permettent d'écrire dans des fichiers en utilisant principalement la méthode `.write()`.

### Pourquoi
Le choix entre `'w'` et `'a'` est crucial et dépend de l'objectif :
-   Utilisez `'w'` quand vous voulez générer un fichier de rapport de zéro ou sauvegarder un nouvel état qui remplace l'ancien.
-   Utilisez `'a'` pour des fichiers de log, où chaque nouvelle information doit s'ajouter aux précédentes sans les effacer.

### Comment
**Cas Réel : Créer un fichier de log.**
```python
import datetime

def log_event(message):
    """Ajoute un message avec un horodatage au fichier journal.log."""
    
    # On récupère l'heure actuelle et on la formate
    timestamp = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    log_line = f"[{timestamp}] - {message}\n" # On ajoute manuellement le retour à la ligne
    
    # On ouvre le fichier en mode 'append' ('a')
    with open('journal.log', 'a', encoding='utf-8') as log_file:
        log_file.write(log_line)
        
# Simulons quelques événements
log_event("Démarrage de l'application.")
log_event("Connexion de l'utilisateur 'admin'.")
log_event("Erreur critique: base de données indisponible.")
```
Après l'exécution de ce script, un fichier `journal.log` sera créé (ou complété) avec les trois lignes de log.

### Zone de Danger
*   **Le mode `'w'` est destructeur !** Une erreur fréquente est d'ouvrir un fichier en mode `'w'` en pensant l'ouvrir en lecture. L'acte même d'ouvrir le fichier en mode `'w'` vide son contenu instantanément. Soyez extrêmement prudent.
*   **`.write()` n'ajoute pas de retour à la ligne** : Contrairement à la fonction `print()`, vous devez explicitement ajouter le caractère de nouvelle ligne `\n` dans la chaîne que vous écrivez si vous voulez que les écritures successives apparaissent sur des lignes différentes.

---

## 3. Gestion des Chemins de Fichiers (Module `os`) {#gestion-chemins-os-21}

### Quoi
Jusqu'à présent, nous avons utilisé des chemins relatifs (`'config.txt'`). Pour construire des chemins plus complexes de manière portable (qui fonctionnent sur Windows, macOS et Linux), il est recommandé d'utiliser le module `os`.

### Pourquoi
Les systèmes d'exploitation n'utilisent pas le même séparateur de dossiers (Windows utilise `\` tandis que macOS/Linux utilisent `/`). Coder en dur un chemin avec `/` ou `\` rendra votre script non portable. `os.path.join()` résout ce problème en utilisant le bon séparateur pour le système sur lequel le script s'exécute.

### Comment
**Cas Réel : Écrire un rapport dans un sous-dossier `rapports`.**
```python
import os

# Créer le dossier 'rapports' s'il n'existe pas
nom_dossier = "rapports"
if not os.path.exists(nom_dossier):
    os.makedirs(nom_dossier)

# Construire le chemin du fichier de manière portable
nom_fichier = "rapport_2023_10.txt"
chemin_complet = os.path.join(nom_dossier, nom_fichier)

print(f"Le chemin construit est : {chemin_complet}")

# Écrire dans le fichier en utilisant ce chemin
with open(chemin_complet, 'w', encoding='utf-8') as f:
    f.write("Ceci est le rapport du mois d'octobre.\n")
    f.write("Ventes : 150 unités.\n")
```

---

## Validation des Acquis {#validation-21}

### 3 Questions Clés
1.  Pourquoi la syntaxe `with open(...)` est-elle considérée comme la meilleure pratique pour manipuler des fichiers en Python ?
2.  Quelle est la différence fondamentale entre ouvrir un fichier en mode `'w'` et en mode `'a'` ?
3.  Si vous lisez un fichier très volumineux, quelle est la méthode à privilégier pour éviter de saturer la mémoire de l'ordinateur ?

### 3 Exercices Progressifs

#### Exercice 1 : Compteur de Mots
Créez un script qui lit un fichier `texte.txt`, compte le nombre total de mots qu'il contient, et affiche le résultat. (Considérez qu'un mot est une séquence de caractères séparée par un espace).

<details>
<summary>Découvrir la solution commentée</summary>

```python
# Créez d'abord un fichier 'texte.txt' avec du contenu, par exemple :
# "Python est un langage de programmation
# populaire et puissant."

try:
    with open('texte.txt', 'r', encoding='utf-8') as f:
        # On lit tout le contenu du fichier
        contenu = f.read()
        
        # La méthode split() sans argument sépare la chaîne par les espaces
        mots = contenu.split()
        
        # On compte le nombre d'éléments dans la liste résultante
        nombre_de_mots = len(mots)
        
        print(f"Le fichier contient {nombre_de_mots} mots.")

except FileNotFoundError:
    print("Erreur : Le fichier 'texte.txt' est introuvable. Veuillez le créer.")
```
</details>

#### Exercice 2 : Copie de Fichier
Écrivez une fonction `copier_fichier(source, destination)` qui lit le contenu d'un fichier `source` et l'écrit dans un nouveau fichier `destination`. La fonction devra gérer le cas où le fichier source n'existe pas.

<details>
<summary>Découvrir la solution commentée</summary>

```python
def copier_fichier(source, destination):
    """
    Copie le contenu du fichier source vers le fichier destination.
    """
    try:
        # On ouvre le fichier source en lecture
        with open(source, 'r', encoding='utf-8') as f_source:
            contenu = f_source.read()
        
        # On ouvre (ou crée) le fichier destination en écriture
        with open(destination, 'w', encoding='utf-8') as f_dest:
            f_dest.write(contenu)
            
        print(f"Le fichier '{source}' a été copié avec succès dans '{destination}'.")
        
    except FileNotFoundError:
        print(f"Erreur : Le fichier source '{source}' n'a pas été trouvé.")
    except Exception as e:
        print(f"Une erreur inattendue est survenue : {e}")

# Exemple d'utilisation
# (Assurez-vous d'avoir un fichier 'mon_document.txt' à copier)
copier_fichier('mon_document.txt', 'copie_document.txt')
copier_fichier('fichier_inexistant.txt', 'autre_copie.txt')
```
</details>

#### Exercice 3 : Filtrage de Lignes
Vous avez un fichier `data.csv` contenant des données au format `nom,age,ville`. Écrivez un script qui lit ce fichier et crée un nouveau fichier `data_filtree.csv` contenant uniquement les lignes où l'âge est supérieur ou égal à 30.

**`data.csv` (exemple) :**
```csv
Alice,25,Paris
Bob,35,Lyon
Charlie,30,Marseille
David,22,Paris
```

<details>
<summary>Découvrir la solution commentée</summary>

```python
# Fichier d'entrée : data.csv
# Fichier de sortie : data_filtree.csv

try:
    # On ouvre les deux fichiers en même temps
    with open('data.csv', 'r', encoding='utf-8') as f_in, \
         open('data_filtree.csv', 'w', encoding='utf-8') as f_out:
        
        # On itère sur le fichier d'entrée ligne par ligne (efficace en mémoire)
        for ligne in f_in:
            # On nettoie la ligne et on la sépare par la virgule
            elements = ligne.strip().split(',')
            
            # On vérifie qu'on a bien 3 éléments pour éviter les erreurs
            if len(elements) == 3:
                nom, age_str, ville = elements
                
                # On essaie de convertir l'âge en entier
                try:
                    age = int(age_str)
                    
                    # C'est ici qu'on applique notre filtre
                    if age >= 30:
                        # Si le filtre passe, on écrit la ligne originale dans le fichier de sortie
                        f_out.write(ligne)
                except ValueError:
                    # Si l'âge n'est pas un nombre, on ignore la ligne
                    print(f"Ligne ignorée (âge non valide) : {ligne.strip()}")

    print("Le fichier 'data_filtree.csv' a été créé avec succès.")

except FileNotFoundError:
    print("Erreur : Le fichier 'data.csv' est introuvable.")

# Le fichier data_filtree.csv contiendra :
# Bob,35,Lyon
# Charlie,30,Marseille
```
</details>