---
sidebar_label: Context Managers et le mot-clé `with`
sidebar_position: 53
---

# Chapitre 53 : Context Managers et le mot-clé `with`

Protocoles __enter__ et __exit__, Utilisation de `with`, Création de context managers, Décorateur @contextmanager

Vous avez déjà utilisé `with open(...)` pour lire des fichiers. C'est magique : le fichier se ferme tout seul, même en cas d'erreur. Cette magie s'appelle un **Context Manager**.

Dans ce chapitre, nous allons lever le capot pour comprendre comment cela fonctionne et comment créer vos propres gestionnaires de contexte pour gérer des connexions bases de données, des verrous (locks) ou des sessions API de manière élégante et sécurisée.

---

## 1. Le mot-clé `with` et la gestion de ressources {#le-mot-cle-with}

### 1. Quoi
Le mot-clé `with` crée un **contexte d'exécution** temporaire. Il garantit que des opérations de nettoyage (fermeture de fichier, libération de mémoire, fin de transaction) sont *toujours* effectuées, que le code réussisse ou plante.

### 2. Pourquoi
Sans `with`, nous sommes obligés d'utiliser des blocs `try...finally` verbeux pour éviter les fuites de ressources.

### 3. Comment

#### A. Syntaxe de base (Comparaison)

❌ **Sans Context Manager (L'approche risquée)**
```python
file = open("data.txt", "w")
try:
    file.write("Hello")
    # Si une erreur arrive ici, le fichier reste ouvert !
    raise ValueError("Oups")
finally:
    # On doit penser à fermer manuellement
    file.close() 
```

✅ **Avec Context Manager (L'approche Pythonique)**
```python
# Le fichier sera fermé automatiquement à la sortie du bloc, quoi qu'il arrive
with open("data.txt", "w") as file:
    file.write("Hello")
    # raise ValueError("Oups") -> Le fichier sera quand même fermé
```

#### B. Gestion multiple (Python 3.10+)
On peut ouvrir plusieurs ressources simultanément de manière lisible.

```python
# Copie de fichier ligne par ligne
with (
    open("source.txt", "r") as source,
    open("destination.txt", "w") as destination
):
    for line in source:
        destination.write(line.upper())
```

### 4. Zone de Danger
❌ **Oublier le `as` quand c'est nécessaire** : Si vous faites juste `with open(...)`, l'objet fichier est créé puis fermé, mais vous n'avez pas de variable pour écrire dedans.
✅ **Bonne Pratique** : Utilisez `with` pour *toutes* les ressources qui nécessitent une libération explicite (fichiers, sockets réseau, connexions DB, Threads Locks).

---

## 2. Le Protocole `__enter__` et `__exit__` (Classes) {#protocole-enter-exit}

### 1. Quoi
Pour qu'un objet soit compatible avec `with`, il doit implémenter deux méthodes magiques :
*   `__enter__` : Exécuté au début du bloc `with`. Retourne souvent `self`.
*   `__exit__` : Exécuté à la fin du bloc. Gère le nettoyage et éventuellement les exceptions.

### 2. Pourquoi
Cela permet de créer vos propres objets sécurisés, comme un connecteur de base de données maison.

### 3. Comment

#### A. Structure d'une classe Context Manager
```python
class DatabaseConnection:
    def __init__(self, db_name: str):
        self.db_name = db_name

    def __enter__(self):
        print(f"🔌 Connexion à {self.db_name}...")
        # On retourne l'objet qui sera assigné à la variable après 'as'
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        print(f"👋 Déconnexion de {self.db_name}.")
        # Si exc_type n'est pas None, une erreur est survenue dans le bloc
        if exc_type:
            print(f"🚨 Erreur interceptée : {exc_val}")
        # Retourner False propage l'erreur, True l'étouffe (rare)
        return False

    def query(self, sql: str):
        print(f"Exécution de : {sql}")

# Utilisation
with DatabaseConnection("UserDB") as db:
    db.query("SELECT * FROM users")
    # En sortant, __exit__ est appelé automatiquement
```

#### B. Gestion des Exceptions dans `__exit__`
Les arguments `exc_type`, `exc_val`, `exc_tb` contiennent les infos sur l'erreur si le bloc plante.
*   Si tout va bien : ils valent `None`.
*   Si vous retournez `True`, l'erreur est considérée comme "gérée" et le programme continue.
*   Si vous retournez `False` (ou rien), l'erreur remonte (crash normal).

### 4. Zone de Danger
❌ **Ne pas retourner `True` silencieusement** : Si vous retournez `True` dans `__exit__` sans loguer l'erreur, vous masquez des bugs critiques. Faites-le uniquement si vous voulez explicitement ignorer certaines erreurs connues.

---

## 3. Le décorateur `@contextmanager` (Générateurs) {#decorateur-contextmanager}

### 1. Quoi
Le module `contextlib` fournit un décorateur `@contextmanager` qui permet de créer un context manager à partir d'une simple fonction génératrice, sans créer de classe entière.

### 2. Pourquoi
C'est beaucoup plus concis et lisible pour des cas simples (ex: un timer, changer temporairement une variable d'environnement).

### 3. Comment

#### A. Syntaxe avec `yield`
Le code avant `yield` équivaut à `__enter__`. Le code après `yield` équivaut à `__exit__`.

```python
from contextlib import contextmanager
import os

@contextmanager
def temporary_env_var(key: str, value: str):
    # --- Phase __enter__ ---
    old_value = os.environ.get(key)
    os.environ[key] = value
    print(f"Variable {key} définie à {value}")
    
    try:
        yield  # Rend la main au bloc 'with'
    finally:
        # --- Phase __exit__ ---
        # Le 'finally' garantit l'exécution même si le bloc 'with' plante
        if old_value is None:
            del os.environ[key]
        else:
            os.environ[key] = old_value
        print(f"Variable {key} restaurée")

# Utilisation
with temporary_env_var("API_MODE", "DEBUG"):
    print(f"Mode actuel : {os.environ.get('API_MODE')}")

print(f"Mode après : {os.environ.get('API_MODE')}")
```

#### B. Cas Concret : Timer de performance
```python
import time
from contextlib import contextmanager

@contextmanager
def timer(label: str):
    start = time.perf_counter()
    try:
        yield
    finally:
        end = time.perf_counter()
        print(f"{label}: {end - start:.4f} sec")

with timer("Calcul complexe"):
    sum(range(10_000_000))
# Sortie : Calcul complexe: 0.1234 sec
```

### 🚨 Limitations de `@contextmanager`
*   C'est moins flexible qu'une classe si vous avez besoin de stocker beaucoup d'état interne.
*   Pensez OBLIGATOIREMENT au bloc `try...finally` autour du `yield`, sinon le code de nettoyage ne s'exécutera pas en cas d'erreur dans le bloc `with`.

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-53}

1.  **Pourquoi préférer `with open(...)` à `file = open(...)` ?**
    Pour garantir la fermeture du fichier même si une exception survient pendant l'écriture/lecture.

2.  **Quelles sont les deux méthodes magiques requises pour créer une classe compatible avec `with` ?**
    `__enter__` et `__exit__`.

3.  **Que se passe-t-il si la méthode `__exit__` retourne `True` ?**
    L'exception survenue dans le bloc `with` est supprimée (catchée) et l'exécution du programme continue après le bloc.

4.  **Dans un générateur décoré par `@contextmanager`, quel mot-clé sépare la logique d'initialisation de la logique de nettoyage ?**
    Le mot-clé `yield`.

---

## Exercices : {#exercices-53}

### Exercice 1 - Le Gestionnaire de Dossier Temporaire {#exercice-1-cd-temporaire}

🎯 **Objectif** : Utiliser `@contextmanager` pour modifier l'état global temporairement.

💼 **Mise en situation** : Vous devez exécuter un script qui génère des fichiers dans le dossier courant. Vous voulez changer de dossier le temps de l'opération, puis revenir au dossier initial automatiquement.

📝 **Énoncé** :
1.  Importez `os` et `contextlib`.
2.  Créez un context manager `change_dir(destination: str)`.
3.  Il doit sauvegarder `os.getcwd()` (dossier actuel).
4.  Il doit faire `os.chdir(destination)`.
5.  Il doit restaurer le dossier initial dans le bloc `finally`.
6.  Testez-le avec un bloc `with`.

📺 **Résultat attendu** :
```text
Dossier avant: /home/user/projet
Dossier pendant: /tmp
Dossier après: /home/user/projet
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
import os
from contextlib import contextmanager

@contextmanager
def change_dir(destination: str):
    original_dir = os.getcwd()
    try:
        print(f"➡️ Déplacement vers {destination}")
        os.chdir(destination)
        yield
    finally:
        print(f"⬅️ Retour vers {original_dir}")
        os.chdir(original_dir)

# Test (assurez-vous que /tmp existe, ou utilisez un autre dossier)
print(f"Dossier avant: {os.getcwd()}")

with change_dir("/tmp"):
    print(f"Dossier pendant: {os.getcwd()}")
    # Simulation d'erreur potentielle
    # raise ValueError("Crash test") 

print(f"Dossier après: {os.getcwd()}")
```
</details>

### Exercice 2 - La Balise HTML Automatique {#exercice-2-html-tag}

🎯 **Objectif** : Créer une Classe Context Manager pour du formatage.

💼 **Mise en situation** : Vous générez du HTML simple. Vous voulez éviter d'oublier de fermer vos balises `<div>` ou `<p>`.

📝 **Énoncé** :
1.  Créez une classe `HtmlTag`.
2.  Le constructeur prend le nom de la balise (ex: "div").
3.  `__enter__` imprime `<tag>`.
4.  `__exit__` imprime `</tag>`.
5.  Nistez (imbriquez) deux blocs `with` pour voir le résultat indenté (l'indentation visuelle n'est pas requise, juste l'ordre des balises).

📺 **Résultat attendu** :
```text
<div>
<p>
Contenu
</p>
</div>
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
class HtmlTag:
    def __init__(self, tag_name: str):
        self.tag_name = tag_name

    def __enter__(self):
        print(f"<{self.tag_name}>")
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        print(f"</{self.tag_name}>")

# Utilisation imbriquée
with HtmlTag("div"):
    with HtmlTag("p"):
        print("Contenu important")
```
</details>

### Exercice 3 - Le Connecteur "Crash-Proof" {#exercice-3-connecteur-crash-proof}

🎯 **Objectif** : Gérer les exceptions dans `__exit__`.

💼 **Mise en situation** : Un système de logs critique doit absolument écrire "FIN DE SESSION" même si le programme plante au milieu. De plus, il doit intercepter les erreurs `ValueError` mais laisser passer les autres.

📝 **Énoncé** :
1.  Créez une classe `SessionLogger`.
2.  Dans `__exit__`, vérifiez `exc_type`.
3.  Si l'erreur est `ValueError`, affichez "Erreur de valeur ignorée" et retournez `True`.
4.  Sinon, affichez "Erreur critique !" et retournez `False`.
5.  Testez avec une `ValueError` (le programme ne doit pas planter) et une `TypeError` (le programme doit planter).

📺 **Résultat attendu** :
Test 1 : Le script continue.
Test 2 : Le script affiche la traceback de l'erreur.

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
class SessionLogger:
    def __enter__(self):
        print("--- DÉBUT SESSION ---")
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        print("--- FIN SESSION ---")
        
        if exc_type is ValueError:
            print(f"🛡️ J'ai intercepté une ValueError: {exc_val}")
            return True # On étouffe l'erreur
        
        if exc_type:
            print("⚠️ Erreur critique non gérée, ça va planter...")
            return False # On laisse l'erreur remonter

# Test 1 : ValueError (Gérée)
print("Test 1:")
with SessionLogger():
    raise ValueError("Valeur invalide")
print("Le script continue après Test 1.\n")

# Test 2 : TypeError (Non gérée)
print("Test 2:")
with SessionLogger():
    raise TypeError("Type invalide")
# Ce print ne sera jamais atteint
print("Jamais affiché")
```
</details>