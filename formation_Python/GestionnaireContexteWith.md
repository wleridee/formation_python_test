---
sidebar_label: "Gestionnaire de Contexte: Le Mot-clé 'with'"
sidebar_position: 22
difficulty: "confirmé"
---

# Gestionnaire de Contexte: Le Mot-clé 'with' {#gestionnaire-contexte-with-22}

Vous avez déjà utilisé l'instruction `with` pour manipuler des fichiers. C'est la manière la plus propre et la plus sûre de gérer des ressources qui doivent être ouvertes puis fermées, comme des fichiers, des connexions à une base de données ou des verrous (`locks`) en programmation concurrente.

Ce mécanisme est rendu possible par le **protocole de gestion de contexte**. Dans ce chapitre, nous allons non seulement approfondir son fonctionnement, mais aussi apprendre à créer nos propres gestionnaires de contexte pour rendre notre code plus robuste et plus lisible.

## 1. Le Problème que `with` Résout {#probleme-resolu-par-with-22}

### Quoi
Un gestionnaire de contexte est un objet qui définit un contexte d'exécution temporaire pour un bloc de code. Il est responsable d'effectuer des actions à l'entrée (`setup`) et à la sortie (`teardown`) de ce bloc.

### Pourquoi
Le principal avantage est la **garantie du nettoyage**. Imaginez que vous gérez une ressource manuellement avec un bloc `try...finally`.

```mermaid
graph TD
    A[Ouvrir la ressource (ex: fichier)] --> B{Bloc `try` : Utiliser la ressource};
    B -- Erreur --> C{Bloc `finally` : Nettoyer/Fermer la ressource};
    B -- Pas d'erreur --> C;
    C --> D[Fin];
```

**Sans `with`, la gestion de fichier ressemble à ça :**
```python
f = open('mon_fichier.txt', 'w')
try:
    # Opérations potentiellement risquées
    f.write('Hello, world!')
finally:
    # Ce bloc s'exécute TOUJOURS, qu'il y ait eu une erreur ou non
    print("Fermeture du fichier.")
    f.close()
```
Ce code est verbeux et il est facile d'oublier le bloc `finally`. L'instruction `with` encapsule cette logique de manière élégante.

### Comment
L'instruction `with` est un sucre syntaxique pour ce bloc `try...finally`.

```python
# La version pythonique et sûre
with open('mon_fichier.txt', 'w') as f:
    f.write('Hello, world!')
# La fermeture est automatique et garantie à la sortie de ce bloc
```

### Zone de Danger
*   **Fuites de ressources** : Si vous n'utilisez ni `with` ni `try...finally`, une exception inattendue peut empêcher l'exécution de l'instruction de nettoyage (comme `f.close()`). Dans une application qui tourne longtemps, ces ressources "oubliées" s'accumulent et peuvent faire planter le système.

---

## 2. Créer son Propre Gestionnaire de Contexte : La Méthode par Classe {#creer-contexte-classe-22}

### Quoi
Tout objet qui implémente deux méthodes spéciales, `__enter__` et `__exit__`, peut être utilisé comme un gestionnaire de contexte.

-   `__enter__(self)` : Exécutée au début du bloc `with`. La valeur qu'elle retourne est assignée à la variable après le `as`.
-   `__exit__(self, exc_type, exc_value, traceback)` : Exécutée à la fin du bloc `with`. Elle reçoit des informations sur l'exception si une s'est produite, ce qui lui permet de gérer le nettoyage différemment en cas d'erreur.

### Pourquoi
Cela permet d'encapsuler la logique de `setup` et de `teardown` au sein même de vos propres objets. C'est une technique puissante pour rendre vos classes plus faciles et plus sûres à utiliser.

### Comment
**Cas Réel : Créer un gestionnaire de contexte `Timer` qui mesure le temps d'exécution d'un bloc de code.**
```python
import time

class Timer:
    def __enter__(self):
        """Action à l'entrée : enregistrer le temps de départ."""
        print("Début du chronomètre...")
        self.start_time = time.time()
        # On ne retourne rien de spécial, donc pas besoin de 'as'
        return self

    def __exit__(self, exc_type, exc_value, traceback):
        """Action à la sortie : calculer et afficher le temps écoulé."""
        self.end_time = time.time()
        duree = self.end_time - self.start_time
        print(f"Temps écoulé : {duree:.4f} secondes.")
        # Si aucune exception n'est survenue, les 3 arguments seront None
        if exc_type is not None:
            print(f"Le bloc s'est terminé avec une exception de type : {exc_type.__name__}")
        
        # On ne retourne rien (ou False), donc si une exception a eu lieu, elle est propagée.
        return False

# Utilisation
with Timer():
    # Simulons une tâche qui prend du temps
    time.sleep(1.5)

print("\n--- Utilisation avec une erreur ---")
try:
    with Timer():
        time.sleep(0.5)
        result = 10 / 0 # Provoque une ZeroDivisionError
except ZeroDivisionError:
    print("L'exception a bien été capturée à l'extérieur du bloc 'with'.")
```

### Zone de Danger
*   **Suppression d'exceptions** : Si la méthode `__exit__` retourne `True`, elle indique à Python que l'exception a été gérée et ne doit pas être propagée à l'extérieur du bloc `with`. C'est un comportement très spécifique et potentiellement dangereux, car il peut masquer des bugs. Utilisez-le avec une extrême prudence.

---

## 3. La Voie Élégante : Le Décorateur `@contextmanager` {#contexte-contextlib-22}

### Quoi
Le module `contextlib` de la bibliothèque standard fournit un décorateur, `@contextmanager`, qui permet de créer un gestionnaire de contexte à partir d'une simple fonction génératrice.

### Pourquoi
C'est souvent plus simple et plus lisible que de créer une classe entière. La logique de `setup`, le `yield`, et le `teardown` sont réunis dans une seule fonction.

### Comment
Une fonction décorée avec `@contextmanager` doit :
1.  Contenir du code de `setup` avant le `yield`.
2.  Utiliser `yield` une seule fois. La valeur "yieldée" sera celle assignée à la variable `as`.
3.  Contenir le code de `teardown` après le `yield`.
4.  Idéalement, enrober le `yield` dans un bloc `try...finally` pour garantir le nettoyage.

**Cas Réel : Ré-implémenter notre `Timer` avec `@contextmanager`.**
```python
from contextlib import contextmanager
import time

@contextmanager
def timer_func():
    """Un gestionnaire de contexte basé sur un générateur pour chronométrer du code."""
    start_time = time.time()
    print("Début du chronomètre (version @contextmanager)...")
    try:
        # yield suspend la fonction et exécute le bloc 'with'
        yield
    finally:
        # Ce code est exécuté après le bloc 'with', même en cas d'erreur
        end_time = time.time()
        duree = end_time - start_time
        print(f"Temps écoulé : {duree:.4f} secondes.")

# Utilisation
with timer_func():
    time.sleep(1.2)
```

### Zone de Danger
*   **Oublier `try...finally`** : Si vous n'utilisez pas de bloc `try...finally` autour du `yield`, et qu'une exception se produit dans le bloc `with`, le code de nettoyage situé après le `yield` ne sera **jamais** exécuté. C'est la même problématique de fuite de ressources que nous cherchions à éviter.

---

## Validation des Acquis {#validation-22}

### 3 Questions Clés
1.  Quelles sont les deux méthodes qu'une classe doit implémenter pour fonctionner comme un gestionnaire de contexte ? Quel est le rôle de chacune ?
2.  Dans un gestionnaire de contexte basé sur une classe, que se passe-t-il si la méthode `__exit__` retourne `True` ?
3.  Quel est le principal avantage d'utiliser le décorateur `@contextmanager` par rapport à l'approche par classe ?

### 3 Exercices Progressifs

#### Exercice 1 : Un Contexte de "Silence"
Créez un gestionnaire de contexte `MuteExceptions` (basé sur une classe) qui empêche la propagation de certains types d'exceptions spécifiés.
Il devra prendre en paramètre lors de son initialisation une liste de types d'exceptions à "silencier".

```python
# Objectif d'utilisation
with MuteExceptions([ValueError, TypeError]):
    print("On entre dans le bloc.")
    x = int("hello") # Ceci lève un ValueError
    print("Cette ligne ne sera jamais atteinte.")

print("Le programme continue car l'exception a été gérée.")
```

<details>
<summary>Découvrir la solution commentée</summary>

```python
class MuteExceptions:
    def __init__(self, exceptions_to_mute):
        # On stocke les types d'exceptions à ignorer
        self.exceptions_to_mute = tuple(exceptions_to_mute) # Un tuple est plus sûr

    def __enter__(self):
        # Pas de setup particulier à faire
        pass

    def __exit__(self, exc_type, exc_value, traceback):
        # C'est ici que la magie opère.
        # exc_type sera None s'il n'y a pas d'erreur.
        if exc_type is not None:
            # On vérifie si le type de l'exception levée est dans notre liste
            if issubclass(exc_type, self.exceptions_to_mute):
                print(f"Exception {exc_type.__name__} capturée et silencée.")
                # Retourner True supprime l'exception !
                return True
        # Si ce n'est pas une exception à ignorer, on ne retourne rien (équivalent à False)
        # et l'exception est propagée normalement.

# Test
print("--- Test 1: Silencier un ValueError ---")
with MuteExceptions([ValueError, TypeError]):
    print("On entre dans le bloc.")
    x = int("hello") # Ceci lève un ValueError
    print("Cette ligne ne sera jamais atteinte.")
print("Le programme continue car l'exception a été gérée.")

print("\n--- Test 2: Ne pas silencier un ZeroDivisionError ---")
try:
    with MuteExceptions([ValueError, TypeError]):
        result = 10 / 0
except ZeroDivisionError as e:
    print(f"L'exception '{e}' n'a pas été silencée, ce qui est correct.")
```
</details>

#### Exercice 2 : Gestionnaire de Fichier Temporaire
En utilisant `@contextmanager`, créez un gestionnaire de contexte `temp_file(content)` qui :
1.  Prend une chaîne de caractères `content` en argument.
2.  Crée un fichier temporaire sur le disque avec un nom unique.
3.  Écrit le `content` dans ce fichier.
4.  Le `yield` retourne le chemin complet du fichier créé.
5.  À la sortie du bloc `with`, le fichier est supprimé, même en cas d'erreur.

*Indice : les modules `os` et `tempfile` pourraient être utiles (`tempfile.NamedTemporaryFile` est une piste, mais essayez de le faire manuellement avec `os` pour bien pratiquer).*

<details>
<summary>Découvrir la solution commentée</summary>

```python
from contextlib import contextmanager
import os
import uuid # Pour générer un nom de fichier unique

@contextmanager
def temp_file(content):
    """
    Crée un fichier temporaire avec un contenu, fournit son chemin,
    et le supprime à la fin.
    """
    # 1-3. Setup : créer le fichier et y écrire
    filename = f"temp_{uuid.uuid4()}.txt"
    try:
        with open(filename, 'w', encoding='utf-8') as f:
            f.write(content)
        
        # 4. Yield : on fournit le chemin au bloc 'with'
        print(f"Fichier temporaire '{filename}' créé.")
        yield filename
        
    finally:
        # 5. Teardown : on supprime le fichier
        if os.path.exists(filename):
            os.remove(filename)
            print(f"Fichier temporaire '{filename}' supprimé.")

# Utilisation
with temp_file("Ceci est un test.") as path:
    print(f"Le bloc 'with' a accès au chemin : {path}")
    # On peut vérifier que le fichier existe et lire son contenu
    with open(path, 'r') as f:
        print(f"Contenu lu depuis le bloc : '{f.read()}'")

print("Le fichier a dû être supprimé après le bloc.")
```
</details>

#### Exercice 3 : Mesurer la Consommation Mémoire
En vous inspirant du `Timer`, créez un gestionnaire de contexte `MemoryMonitor` (avec une classe ou `@contextmanager`, au choix) qui mesure et affiche la quantité de mémoire (en Mo) utilisée par l'exécution d'un bloc de code.

*Indice : vous aurez besoin du package `psutil`. Installez-le avec `pip install psutil`. Le code pour obtenir la consommation mémoire du processus actuel est : `psutil.Process(os.getpid()).memory_info().rss` (donne des octets).*

<details>
<summary>Découvrir la solution commentée</summary>

```python
# D'abord, installez psutil : pip install psutil
import os
import psutil

class MemoryMonitor:
    def __enter__(self):
        self.process = psutil.Process(os.getpid())
        self.mem_before = self.process.memory_info().rss / (1024 * 1024) # en Mo
        print(f"Mémoire avant : {self.mem_before:.2f} Mo")
        return self

    def __exit__(self, exc_type, exc_value, traceback):
        mem_after = self.process.memory_info().rss / (1024 * 1024) # en Mo
        print(f"Mémoire après : {mem_after:.2f} Mo")
        print(f"Mémoire utilisée par le bloc : {mem_after - self.mem_before:.2f} Mo")
        return False

# Utilisation
print("--- Test de consommation mémoire ---")
with MemoryMonitor():
    # Créons une grosse liste pour consommer de la mémoire
    # 10 millions d'entiers
    big_list = list(range(10_000_000))
    # Cette liste sera supprimée par le garbage collector après le bloc 'with'
    
# L'utilisation de la mémoire devrait redescendre après,
# mais le moniteur nous montre le pic d'utilisation pendant le bloc.
```
</details>