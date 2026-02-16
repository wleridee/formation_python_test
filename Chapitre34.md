---
sidebar_label: Module `sqlite3` : Bases de Données Embarquées
sidebar_position: 34
---

# Chapitre 34 : Module `sqlite3` : Bases de Données Embarquées

Connexion à la base, Exécution de requêtes (SQL), Fetch de résultats, Transactions

Jusqu'à présent, nous avons stocké nos données dans des variables ou des fichiers texte. Mais pour des applications plus robustes (CRM, catalogue produit, gestion d'utilisateurs), il est indispensable d'utiliser une base de données relationnelle.

`sqlite3` est un module intégré à Python qui fournit une interface pour la base de données SQLite. C'est une base de données "sans serveur" (embarquée) : toute la base tient dans un seul fichier sur votre disque (`.db` ou `.sqlite`). Elle est utilisée par des géants comme Firefox, Chrome, Android et iOS. C'est l'outil parfait pour prototyper ou pour des applications locales sans configuration complexe.

---

## 1. Connexion et Curseurs {#connexion-et-curseurs}

### 1. Quoi
Pour interagir avec la base, il faut deux objets :
1.  **Connection** : Représente le lien avec le fichier de base de données.
2.  **Cursor** : Un "pointeur" qui permet d'exécuter des requêtes SQL et de parcourir les résultats.

### 2. Pourquoi
Contrairement à la lecture d'un fichier texte où tout est chargé en mémoire, le curseur permet de traiter les données ligne par ligne, optimisant ainsi les ressources.

### 3. Comment

#### A. Syntaxe de base

```python
import sqlite3

# 1. Connexion au fichier (il est créé s'il n'existe pas)
# Utiliser ":memory:" pour une base temporaire en RAM
conn = sqlite3.connect('database.db')

# 2. Création du curseur
cursor = conn.cursor()

# 3. Opérations...
print("Base de données connectée !")

# 4. Fermeture (Très important !)
conn.close()
```

#### B. La méthode robuste : Context Manager
Comme pour les fichiers, utilisez `with` pour garantir la fermeture automatique de la connexion, même en cas d'erreur.

```python
import sqlite3
from contextlib import closing

db_path = "mon_entreprise.db"

# closing() est nécessaire car sqlite3.connect() ne supporte pas 
# __exit__ (fermeture) nativement avant Python 3.12+ de manière intuitive
with sqlite3.connect(db_path) as conn:
    # La connexion est gérée ici
    cursor = conn.cursor()
    cursor.execute("SELECT sqlite_version();")
    version = cursor.fetchone()
    print(f"Version SQLite : {version[0]}")
    
# Ici la connexion est automatiquement fermée (si supporté par la version)
# ou au moins committée.
```
*Note : Pour être parfaitement sûr de la fermeture en Python < 3.12, on utilise souvent `conn.close()` explicitement ou `contextlib.closing`.*

### 4. Zone de Danger
❌ **Ne pas fermer la connexion** : Risque de verrouillage du fichier `.db`, empêchant d'autres processus d'y accéder.

---

## 2. Exécution de Requêtes et SQL Injection {#execution-requetes-sql-injection}

### 1. Quoi
La méthode `cursor.execute(sql, parameters)` envoie des commandes SQL (CREATE, INSERT, UPDATE, DELETE, SELECT) au moteur de base de données.

### 2. Pourquoi
Le SQL est le langage standard pour manipuler les données. Cependant, **concaténer des chaînes** pour créer des requêtes est la faille de sécurité n°1 du web (Injection SQL). `sqlite3` gère cela via des **placeholders** (`?`).

### 3. Comment

#### A. Création de table (DDL)

```python
create_table_sql = """
CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    username TEXT NOT NULL,
    email TEXT UNIQUE,
    age INTEGER
);
"""
cursor.execute(create_table_sql)
```

#### B. Insertion sécurisée (DML)
❌ **INTERDIT** (Faille de sécurité) :
```python
user_input = "admin"
# Si user_input vaut "admin'; DROP TABLE users; --", vous perdez tout.
cursor.execute(f"INSERT INTO users (username) VALUES ('{user_input}')") 
```

✅ **OBLIGATOIRE** (Paramètres nommés ou positionnels) :
```python
new_user = ("Alice", "alice@example.com", 30)

# Le '?' est un placeholder. SQLite se charge de l'échappement.
cursor.execute("INSERT INTO users (username, email, age) VALUES (?, ?, ?)", new_user)

# Pour insérer plusieurs lignes d'un coup : executemany
users_list = [
    ("Bob", "bob@mail.com", 25),
    ("Charlie", "charlie@mail.com", 40)
]
cursor.executemany("INSERT INTO users (username, email, age) VALUES (?, ?, ?)", users_list)
```

---

## 3. Transactions (Commit et Rollback) {#transactions-commit-rollback}

### 1. Quoi
Une transaction est une unité de travail atomique. Soit **toutes** les opérations réussissent (`commit`), soit **aucune** ne s'applique (`rollback`).

### 2. Pourquoi
Imaginez un virement bancaire : on débite A, puis on crédite B. Si le script plante entre les deux, l'argent disparaît ! Les transactions garantissent l'intégrité des données.

### 3. Comment
Par défaut, `sqlite3` ouvre une transaction. Vous devez valider explicitement avec `conn.commit()`.

```python
try:
    cursor.execute("UPDATE comptes SET solde = solde - 100 WHERE id = 1")
    cursor.execute("UPDATE comptes SET solde = solde + 100 WHERE id = 2")
    
    # Si on arrive ici sans erreur, on valide tout
    conn.commit() 
    print("Virement effectué.")
    
except Exception as e:
    # En cas d'erreur, on annule tout ce qui a été fait depuis le début de la transaction
    conn.rollback()
    print(f"Erreur, transaction annulée : {e}")
```

### 🚨 Limitations de SQLite
SQLite ne gère pas bien les accès concurrents **en écriture**. Si deux processus essaient d'écrire en même temps, l'un d'eux recevra une erreur `database is locked`. Pour des applications web à fort trafic, préférez PostgreSQL ou MySQL.

---

## 4. Récupération des Données (Fetch) {#recuperation-donnees-fetch}

### 1. Quoi
Une fois une requête `SELECT` exécutée, les résultats sont dans le curseur. On les extrait avec `fetchone()`, `fetchall()` ou `fetchmany()`.

### 2. Pourquoi
On n'a pas toujours besoin de tout charger. `fetchone` est idéal pour vérifier une existence. `fetchall` récupère tout dans une liste (attention à la RAM !).

### 3. Comment

```python
cursor.execute("SELECT * FROM users WHERE age > ?", (20,))

# Option 1 : Itérer directement sur le curseur (Efficace en mémoire)
for row in cursor:
    # row est un tuple (id, username, email, age)
    print(f"User: {row[1]}, Email: {row[2]}")

# Option 2 : Tout récupérer (Attention si 1 million de lignes)
# rows = cursor.fetchall()

# Option 3 : Récupérer un seul (le premier trouvé)
# row = cursor.fetchone()
# if row: ...
```

#### Astuce Pro : `sqlite3.Row`
Par défaut, les résultats sont des tuples (`row[0]`). Pour accéder aux colonnes par nom (`row['email']`), configurez la `row_factory`.

```python
conn.row_factory = sqlite3.Row
cursor = conn.cursor()
cursor.execute("SELECT * FROM users")
first_user = cursor.fetchone()

print(first_user['email']) # Plus lisible que first_user[2] !
```

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-34}

1.  **Pourquoi est-il dangereux d'utiliser des f-strings pour construire des requêtes SQL ?**
    Cela expose l'application aux injections SQL, permettant à un utilisateur malveillant de manipuler ou détruire la base de données.

2.  **Quelle est la différence entre `execute` et `executemany` ?**
    `execute` lance une requête une seule fois. `executemany` lance la même requête plusieurs fois avec une liste de paramètres différents, ce qui est beaucoup plus optimisé.

3.  **Que se passe-t-il si on ferme la connexion sans faire de `commit()` après un `INSERT` ?**
    Les changements sont perdus (rollback implicite ou non persistance selon la configuration), car ils n'ont pas été validés.

4.  **Comment accéder aux champs par nom de colonne au lieu de l'index numérique ?**
    En définissant `conn.row_factory = sqlite3.Row` avant de créer le curseur.

---

## Exercices : {#exercices-34}

### Exercice 1 - Gestionnaire de Tâches (CRUD) {#exercice-1-todo-crud}

🎯 **Objectif** : Créer une table, insérer et lire des données.

💼 **Mise en situation** : Vous créez le backend d'une application de ToDo List.

📝 **Énoncé** :
1.  Créez un fichier `todo.db`.
2.  Créez une table `tasks` avec : `id` (auto), `title` (texte), `done` (entier 0 ou 1, défaut 0).
3.  Insérez 3 tâches : "Faire les courses", "Apprendre Python", "Dormir".
4.  Affichez toutes les tâches non terminées (`done = 0`).

📺 **Résultat attendu** :
```text
1 - Faire les courses (À faire)
2 - Apprendre Python (À faire)
3 - Dormir (À faire)
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
import sqlite3

# Connexion
conn = sqlite3.connect('todo.db')
cursor = conn.cursor()

# 1. Création de la table
cursor.execute("""
    CREATE TABLE IF NOT EXISTS tasks (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        title TEXT NOT NULL,
        done INTEGER DEFAULT 0
    )
""")

# 2. Insertion des données
tasks = [
    ("Faire les courses",),
    ("Apprendre Python",),
    ("Dormir",)
]
cursor.executemany("INSERT INTO tasks (title) VALUES (?)", tasks)
conn.commit() # Validation importante !

# 3. Lecture
cursor.execute("SELECT * FROM tasks WHERE done = 0")
for row in cursor:
    print(f"{row[0]} - {row[1]} (À faire)")

conn.close()
```
</details>

### Exercice 2 - Mise à Jour Sécurisée {#exercice-2-update-secure}

🎯 **Objectif** : Utiliser `UPDATE` avec des paramètres sécurisés et `commit`.

💼 **Mise en situation** : L'utilisateur a coché la tâche "Apprendre Python" comme terminée. Vous devez mettre à jour la base.

📝 **Énoncé** :
1.  Connectez-vous à `todo.db` (créée à l'exercice 1).
2.  Demandez à l'utilisateur (via `input` ou variable fixe) le TITRE exact de la tâche à terminer.
3.  Exécutez une requête `UPDATE` sécurisée (placeholder `?`) pour passer `done` à 1.
4.  Vérifiez le changement en affichant la tâche modifiée.

📺 **Résultat attendu** :
```text
Tâche 'Apprendre Python' marquée comme terminée.
État en base : (2, 'Apprendre Python', 1)
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
import sqlite3

target_task = "Apprendre Python"

with sqlite3.connect('todo.db') as conn:
    cursor = conn.cursor()
    
    # UPDATE sécurisé
    # On met done=1 là où le titre correspond
    cursor.execute("UPDATE tasks SET done = 1 WHERE title = ?", (target_task,))
    
    if cursor.rowcount > 0:
        print(f"Tâche '{target_task}' marquée comme terminée.")
        conn.commit() # Sauvegarde
    else:
        print("Tâche non trouvée.")

    # Vérification
    cursor.execute("SELECT * FROM tasks WHERE title = ?", (target_task,))
    print(f"État en base : {cursor.fetchone()}")
```
</details>

### Exercice 3 - Import CSV vers SQLite {#exercice-3-csv-to-sqlite}

🎯 **Objectif** : Combiner lecture de fichier et insertion BDD.

💼 **Mise en situation** : Vous recevez un export de produits en CSV et devez l'importer dans la base de données pour le site e-commerce.

📝 **Énoncé** :
1.  Créez un fichier `products.csv` avec le contenu :
    ```csv
    Laptop,999.99
    Souris,25.50
    Clavier,45.00
    ```
2.  Créez un script qui :
    - Lit ce CSV.
    - Crée une table `products` (`name`, `price`).
    - Insère toutes les lignes en une seule transaction.
3.  Affichez le nombre total de produits insérés.

📺 **Résultat attendu** :
```text
Importation terminée. 3 produits ajoutés.
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
import sqlite3
import csv
import os

# Création du CSV factice pour l'exercice
csv_content = "Laptop,999.99\nSouris,25.50\nClavier,45.00"
with open("products.csv", "w") as f:
    f.write(csv_content)

# Script d'import
with sqlite3.connect('ecommerce.db') as conn:
    cursor = conn.cursor()
    
    # Création table
    cursor.execute("""
        CREATE TABLE IF NOT EXISTS products (
            id INTEGER PRIMARY KEY,
            name TEXT,
            price REAL
        )
    """)
    
    # Lecture CSV
    with open("products.csv", "r") as f:
        reader = csv.reader(f)
        # On transforme le reader en liste de tuples pour executemany
        data_to_insert = [(row[0], float(row[1])) for row in reader]
        
    # Insertion de masse
    cursor.executemany("INSERT INTO products (name, price) VALUES (?, ?)", data_to_insert)
    
    # Le commit est automatique à la fin du bloc 'with' si pas d'erreur, 
    # mais le mettre explicite est une bonne pratique pour la clarté.
    conn.commit()
    
    print(f"Importation terminée. {cursor.rowcount} produits ajoutés.")

# Nettoyage
os.remove("products.csv")
```
</details>