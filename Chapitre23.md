---
sidebar_label: Lecture et Écriture de Fichiers Texte
sidebar_position: 23
---

# Chapitre 23 : Lecture et Écriture de Fichiers Texte

Ouverture et fermeture, Modes de lecture/écriture, Contexte manager (with open), Lecture ligne par ligne

Les données ne servent à rien si elles disparaissent à l'arrêt du programme. Pour conserver des informations (persistance) ou traiter des données externes (logs, configurations, exports), votre programme doit interagir avec le système de fichiers.

En Python, manipuler des fichiers est une opération courante mais critique : un fichier mal fermé peut corrompre des données ou bloquer des ressources système. Heureusement, Python offre des outils élégants pour gérer cela en toute sécurité.

---

## 1. Le Context Manager `with open` {#le-context-manager-with-open}

### 1. Quoi
La fonction `open()` permet d'accéder à un fichier. Le mot-clé `with` crée un **contexte** d'exécution qui garantit que le fichier sera automatiquement fermé dès que le bloc de code est terminé, même en cas d'erreur.

### 2. Pourquoi
*   **Sécurité** : Évite les fuites de mémoire et les fichiers verrouillés si vous oubliez `close()`.
*   **Propreté** : Rend le code plus lisible en supprimant la gestion manuelle de la fermeture (`try...finally`).
*   **Gestion des encodages** : Permet de gérer correctement les accents (UTF-8).

### 3. Comment

#### A. Syntaxe de base (La bonne façon)

```python
# Syntaxe : with open(chemim, mode, encoding) as variable:
file_path = "message.txt"

# Le fichier est ouvert au début du bloc
with open(file_path, "w", encoding="utf-8") as file:
    file.write("Bonjour Python !")
    # Traitement...

# ICI : Le fichier est automatiquement fermé, même si le code ci-dessus a planté.
print("Fichier fermé ? ", file.closed) # True
```

#### B. Syntaxe ancienne (À éviter)
```python
f = open("message.txt", "w")
try:
    f.write("Bonjour")
finally:
    f.close() # Verbeux et risque d'oubli
```

### 4. Zone de Danger
❌ **Oublier l'encodage** :
Sur Windows, l'encodage par défaut peut être `cp1252`. Si vous lisez un fichier créé sous Linux/Mac (`utf-8`), les accents vont casser (ex: `Ã©` au lieu de `é`).
✅ **Toujours spécifier** `encoding="utf-8"`.

---

## 2. Les Modes d'Ouverture (`r`, `w`, `a`) {#les-modes-douverture}

### 1. Quoi
Le deuxième argument de `open()` dicte ce que vous avez le droit de faire avec le fichier.

### 2. Pourquoi
Pour protéger les données. Si vous ouvrez un fichier important en mode "écriture" (`w`), son contenu est immédiatement effacé.

### 3. Comment

#### A. Tableau Comparatif

| Mode | Signification | Comportement si fichier existe | Comportement si fichier absent | Curseur |
| :--- | :--- | :--- | :--- | :--- |
| `'r'` | **Read** (Lecture seule) | Lit le contenu | ❌ Erreur `FileNotFoundError` | Début |
| `'w'` | **Write** (Écriture) | ⚠️ **Efface tout** (Écrase) | ✅ Crée le fichier | Début |
| `'a'` | **Append** (Ajout) | Conserve le contenu | ✅ Crée le fichier | Fin |
| `'x'` | **Exclusive** (Création) | ❌ Erreur `FileExistsError` | ✅ Crée le fichier | Début |

#### B. Exemple concret

```python
# 1. Création (et écrasement si existant)
with open("log.txt", "w", encoding="utf-8") as f:
    f.write("Démarrage du système.\n") # \n est nécessaire pour le saut de ligne

# 2. Ajout (sans effacer)
with open("log.txt", "a", encoding="utf-8") as f:
    f.write("Nouvelle entrée utilisateur.\n")

# 3. Lecture
with open("log.txt", "r", encoding="utf-8") as f:
    content = f.read()
    print(content)
```

### 🚨 Limitations de `'r+'`
Le mode `'r+'` permet de lire et écrire, mais il est complexe à gérer (le curseur se déplace et on risque d'écraser des bouts de texte au milieu). Préférez lire tout le fichier, modifier en mémoire, et réécrire, ou utiliser `'a'`.

---

## 3. Lecture Efficace (Mémoire et Performance) {#lecture-efficace}

### 1. Quoi
Il existe plusieurs méthodes pour extraire le texte : tout d'un coup (`read`), ligne par ligne (`readline`), ou via une itération directe.

### 2. Pourquoi
Si vous ouvrez un fichier de 10 Go avec `.read()`, vous saturez votre RAM et votre programme plante. L'itération ligne par ligne est la méthode la plus performante (lazy loading).

### 3. Comment

#### A. Lecture complète (Petits fichiers uniquement)
```python
with open("config.txt", "r", encoding="utf-8") as f:
    # Charge TOUT le fichier en mémoire dans une seule string
    data = f.read() 
```

#### B. Lecture ligne par ligne (Recommandé pour gros fichiers)
L'objet fichier est un **itérateur**.

```python
with open("big_data.txt", "r", encoding="utf-8") as f:
    # Ne charge qu'une seule ligne à la fois en mémoire
    for line in f:
        # .strip() retire le saut de ligne (\n) à la fin
        print(f"Traitement de : {line.strip()}")
```

#### C. Lecture en liste
```python
with open("todos.txt", "r", encoding="utf-8") as f:
    lines = f.readlines() # Retourne une liste ['tache1\n', 'tache2\n']
    # Attention à la consommation mémoire !
```

### 4. Zone de Danger
❌ **Lire un fichier binaire comme du texte** :
Pour les images, PDF, ou exécutables, utilisez le mode `'rb'` (Read Binary) ou `'wb'` (Write Binary) et n'utilisez pas `encoding`.

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-23}

1.  **Pourquoi utiliser `with open(...)` plutôt que simplement `open(...)` ?**
    Pour garantir la fermeture automatique du fichier, libérant ainsi les ressources système même en cas d'erreur.

2.  **Quelle est la différence entre le mode `'w'` et le mode `'a'` ?**
    `'w'` (Write) écrase le fichier existant (contenu perdu). `'a'` (Append) écrit à la suite du contenu existant (contenu préservé).

3.  **Que se passe-t-il si on tente d'ouvrir en mode `'r'` un fichier qui n'existe pas ?**
    Python lève une exception `FileNotFoundError`.

4.  **Pourquoi faut-il souvent utiliser `.strip()` lors de la lecture ligne par ligne ?**
    Car chaque ligne lue contient le caractère invisible de saut de ligne `\n` à la fin, qu'il est souvent préférable de retirer pour le traitement.

---

## Exercices : {#exercices-23}

### Exercice 1 - Le Journal de Bord (Mode Append) {#exercice-1---journal-de-bord}

🎯 **Objectif** : Écrire dans un fichier sans écraser l'historique.

💼 **Mise en situation** : Vous créez un petit système de logs pour une startup. Chaque fois que le script tourne, il doit ajouter une ligne avec un message dans `journal.txt`.

📝 **Énoncé** :
1.  Créez une liste de messages : `["Démarrage serveur", "Erreur connexion", "Arrêt maintenance"]`.
2.  Parcourez cette liste.
3.  Pour chaque message, ouvrez le fichier `journal.txt` en mode ajout (`append`).
4.  Écrivez le message suivi d'un saut de ligne `\n`.
5.  Vérifiez le contenu du fichier (soit en l'ouvrant manuellement, soit avec un script de lecture).

📺 **Résultat attendu (dans journal.txt)** :
```text
Démarrage serveur
Erreur connexion
Arrêt maintenance
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
logs = ["Démarrage serveur", "Erreur connexion", "Arrêt maintenance"]

# On ouvre le fichier pour chaque log (simulation d'événements séparés dans le temps)
# En production, on garderait le fichier ouvert si les logs arrivent en rafale.
for entry in logs:
    # Mode 'a' pour ajouter à la fin
    with open("journal.txt", "a", encoding="utf-8") as f:
        # f.write ne met pas de saut de ligne automatique, il faut l'ajouter
        f.write(f"{entry}\n")

print("Écriture terminée.")

# Vérification lecture
with open("journal.txt", "r", encoding="utf-8") as f:
    print("--- Contenu du fichier ---")
    print(f.read())
```
</details>

### Exercice 2 - Analyseur de Ventes (Lecture itérative) {#exercice-2---analyseur-de-ventes}

🎯 **Objectif** : Lire un fichier ligne par ligne et convertir les données.

💼 **Mise en situation** : Vous recevez un export CSV brut des ventes de la journée (`ventes.txt`). Chaque ligne est au format `Produit,Prix`. Vous devez calculer le chiffre d'affaires total.

📝 **Énoncé** :
1.  Créez un fichier `ventes.txt` contenant :
    ```text
    Laptop,1200
    Souris,25
    Clavier,45
    Ecran,300
    ```
2.  Écrivez un script Python qui initialise `total = 0`.
3.  Ouvrez le fichier en lecture.
4.  Pour chaque ligne :
    - Séparez le nom et le prix (méthode `.split(',')`).
    - Convertissez le prix en entier (`int()`).
    - Ajoutez le prix au total.
5.  Affichez le chiffre d'affaires final.

📺 **Résultat attendu** :
```text
Vente : Laptop à 1200€
Vente : Souris à 25€
Vente : Clavier à 45€
Vente : Ecran à 300€
Chiffre d'affaires total : 1570€
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
# Préparation du fichier (pour que l'exercice soit autonome)
data = "Laptop,1200\nSouris,25\nClavier,45\nEcran,300"
with open("ventes.txt", "w", encoding="utf-8") as f:
    f.write(data)

# Début de l'exercice
total_revenue = 0

try:
    with open("ventes.txt", "r", encoding="utf-8") as f:
        for line in f:
            # Nettoyage des espaces/sauts de ligne
            clean_line = line.strip()
            if not clean_line: continue # Sauter les lignes vides

            # Découpage "Produit,Prix" -> ["Produit", "Prix"]
            parts = clean_line.split(',')
            
            # Extraction et conversion
            product = parts[0]
            price = int(parts[1])
            
            print(f"Vente : {product} à {price}€")
            total_revenue += price

    print(f"Chiffre d'affaires total : {total_revenue}€")

except FileNotFoundError:
    print("Erreur : Le fichier ventes.txt est introuvable.")
except ValueError:
    print("Erreur : Une ligne du fichier est mal formatée (prix non numérique).")
```
</details>

### Exercice 3 - Le Nettoyeur de Fichiers (Read & Write) {#exercice-3---nettoyeur-de-fichiers}

🎯 **Objectif** : Lire un fichier source et écrire le résultat filtré dans un fichier cible.

💼 **Mise en situation** : Un outil génère une liste d'emails, mais certaines lignes contiennent le mot "SPAM". Vous devez créer une version propre de la liste.

📝 **Énoncé** :
1.  Créez `raw_emails.txt` avec des adresses valides et des lignes contenant "SPAM".
2.  Ouvrez `raw_emails.txt` en lecture.
3.  Ouvrez `clean_emails.txt` en écriture (`w`).
4.  Parcourez le fichier source. Si la ligne ne contient pas "SPAM", écrivez-la dans le fichier cible.
5.  Affichez "Nettoyage terminé. X lignes supprimées."

📺 **Résultat attendu** :
```text
Nettoyage terminé. 2 lignes supprimées.
```
*(Le fichier `clean_emails.txt` ne doit contenir que les adresses valides)*

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
# Setup des données sales
content = """alice@gmail.com
bob@yahoo.fr
SPAM: Gagnez un iPhone !!!
charlie@protonmail.com
SPAM: Héritage du prince
david@hotmail.com"""

with open("raw_emails.txt", "w", encoding="utf-8") as f:
    f.write(content)

# Traitement
spam_count = 0

# On peut ouvrir deux fichiers dans le même 'with' !
with open("raw_emails.txt", "r", encoding="utf-8") as source, \
     open("clean_emails.txt", "w", encoding="utf-8") as destination:
    
    for line in source:
        # Vérification insensible à la casse
        if "SPAM" in line:
            spam_count += 1
            # On n'écrit PAS dans la destination
        else:
            # On copie la ligne valide
            destination.write(line)

print(f"Nettoyage terminé. {spam_count} lignes supprimées.")
```
</details>