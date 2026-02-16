---
sidebar_label: Concurrence : Processus (multiprocessing)
sidebar_position: 43
---

# Chapitre 43 : Concurrence : Processus (multiprocessing)

Différence threads/processus, Création de processus, Communication inter-processus, Pool de processus

Dans le chapitre précédent, nous avons vu que les **Threads** sont excellents pour gérer les attentes (Entrées/Sorties), mais incapables d'utiliser plusieurs cœurs de votre processeur simultanément à cause du GIL (Global Interpreter Lock).

Pour faire du calcul intensif (traitement d'image, machine learning, analyse de données) et utiliser toute la puissance de votre machine, il faut sortir de l'interpréteur Python unique. Le module `multiprocessing` permet de créer des **Processus** indépendants, chacun ayant son propre interpréteur Python, sa propre mémoire et... son propre GIL !

---

## 1. Processus vs Threads : Le Grand Duel {#processus-vs-threads}

### 1. Quoi
*   **Processus** : Une instance indépendante d'un programme. Il possède son propre espace mémoire isolé. Créer un processus est "lourd" pour le système.
*   **Thread** : Une unité d'exécution *au sein* d'un processus. Il partage la mémoire du processus parent. Créer un thread est "léger".

### 2. Pourquoi
En Python, `multiprocessing` est la seule façon native de contourner le GIL pour paralléliser des tâches **CPU-bound** (qui consomment du processeur).

### 3. Comment

#### D. Tableau comparatif

| Caractéristique | Threads (`threading`) | Processus (`multiprocessing`) |
| :--- | :--- | :--- |
| **Mémoire** | Partagée (Danger Race Conditions) | Isolée (Sécurisé par défaut) |
| **Cœurs CPU** | 1 seul (à cause du GIL) | Tous les cœurs disponibles |
| **Coût de démarrage** | Faible | Élevé (doit copier l'environnement) |
| **Communication** | Facile (variables partagées) | Complexe (Queue, Pipe, IPC) |
| **Cas d'usage** | I/O (Réseau, Disque) | CPU (Calculs mathématiques) |

---

## 2. Création de Processus {#creation-de-processus}

### 1. Quoi
L'API de `multiprocessing` imite celle de `threading`. On crée un objet `Process` au lieu de `Thread`.

### 2. Pourquoi
Pour lancer une tâche lourde sur un cœur séparé sans bloquer le processus principal.

### 3. Comment

#### A. Syntaxe de base

```python
import multiprocessing
import time

def cpu_heavy_task(name):
    print(f"🔥 Processus {name} démarre sur le CPU")
    # Simulation de calcul intensif
    result = sum(i*i for i in range(10**7))
    print(f"✅ Processus {name} terminé. Résultat: {result}")

if __name__ == "__main__":
    # ⚠️ IMPORTANT : En multiprocessing, le code de lancement 
    # DOIT être protégé par if __name__ == "__main__" sous Windows/macOS.
    
    p1 = multiprocessing.Process(target=cpu_heavy_task, args=("A",))
    p2 = multiprocessing.Process(target=cpu_heavy_task, args=("B",))

    p1.start()
    p2.start()

    p1.join()
    p2.join()
    print("🏁 Tout est fini")
```

### 4. Zone de Danger
❌ **Oublier le bloc `if __name__ == "__main__"`** : Sur Windows et macOS, Python doit réimporter le script principal pour lancer le nouveau processus. Si le code de création n'est pas protégé, vous créez une boucle infinie de création de processus ("Fork bomb") qui peut planter votre machine.

---

## 3. Pool de Processus : La Puissance Industrielle {#pool-de-processus}

### 1. Quoi
L'objet `Pool` permet de gérer un ensemble fixe de processus ouvriers (workers) et de leur distribuer des tâches automatiquement.

### 2. Pourquoi
Pour ne pas saturer le système. Si vous avez 4 cœurs et 100 tâches, lancer 100 processus tuerait la machine. Un Pool de 4 processus traitera les 100 tâches par lots, de manière optimale.

### 3. Comment

#### A. Utilisation de `Pool` avec `map`

```python
from multiprocessing import Pool
import os

def square(x):
    # Affiche l'ID du processus système (PID) pour prouver le parallélisme
    return f"Input: {x}, Carré: {x*x}, PID: {os.getpid()}"

if __name__ == "__main__":
    # Par défaut, utilise autant de processus que de cœurs CPU
    with Pool() as pool:
        numbers = [1, 2, 3, 4, 5, 6, 7, 8]
        
        # Distribue le travail et rassemble les résultats (comme le map() classique)
        results = pool.map(square, numbers)

    for res in results:
        print(res)
```

---

## 4. Communication Inter-Processus (IPC) {#communication-inter-processus}

### 1. Quoi
Comme la mémoire n'est pas partagée, on ne peut pas utiliser une variable globale pour communiquer. Il faut utiliser des canaux explicites comme `Queue` ou `Pipe`.

### 2. Pourquoi
Pour échanger des données entre le processus parent et les enfants, ou entre enfants.

### 3. Comment

#### A. Utilisation de `Queue`
Attention : Ce n'est pas `queue.Queue` (threads), mais `multiprocessing.Queue`.

```python
from multiprocessing import Process, Queue

def producer(q):
    q.put("Donnée importante")
    print("📤 Producteur a envoyé la donnée")

def consumer(q):
    data = q.get() # Bloquant jusqu'à réception
    print(f"📥 Consommateur a reçu : {data}")

if __name__ == "__main__":
    q = Queue()
    
    p1 = Process(target=producer, args=(q,))
    p2 = Process(target=consumer, args=(q,))
    
    p1.start()
    p2.start()
    
    p1.join()
    p2.join()
```

### 🚨 Limitations de IPC
Transférer des objets entre processus implique de les **sérialiser** (pickle) et de les désérialiser. Si vous transférez des gigaoctets de données, le coût de copie peut annuler le gain de performance du parallélisme.

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-43}

1.  **Pourquoi `multiprocessing` est-il nécessaire pour les calculs intensifs en Python ?**
    Car il contourne le GIL en créant des processus distincts, permettant l'utilisation réelle de plusieurs cœurs CPU simultanément.

2.  **Pourquoi doit-on protéger le code principal avec `if __name__ == "__main__":` ?**
    Pour éviter que chaque nouveau processus ne relance récursivement la création de nouveaux processus lors de l'importation du script (particulièrement sur Windows/macOS).

3.  **Quelle est la différence majeure concernant la mémoire entre `threading` et `multiprocessing` ?**
    Les threads partagent la mémoire. Les processus ont chacun leur propre mémoire isolée.

4.  **Est-il plus rapide de lancer 100 processus pour 100 petites tâches ?**
    Non. Le coût de création d'un processus est élevé (overhead). Mieux vaut utiliser un `Pool` (ex: 4 workers) pour traiter les 100 tâches.

---

## Exercices : {#exercices-43}

### Exercice 1 - La Course aux Nombres Premiers {#exercice-1-nombres-premiers}

🎯 **Objectif** : Comparer la performance Séquentielle vs Multiprocessing.

💼 **Mise en situation** : Vous devez trouver le nombre de nombres premiers dans une large plage. C'est du pur calcul CPU.

📝 **Énoncé** :
1.  Créez une fonction `is_prime(n)` (algorithme naïf ok).
2.  Créez une fonction `count_primes(numbers)` qui compte les premiers dans une liste.
3.  Générez une liste de 100 000 nombres aléatoires.
4.  Mesurez le temps pour traiter tout en séquentiel.
5.  Mesurez le temps en divisant la liste en 4 morceaux et en utilisant 4 processus (`Pool`).

📺 **Résultat attendu** :
```text
Séquentiel : ~4.2 secondes
Parallèle (4 cœurs) : ~1.3 secondes (Gain significatif !)
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
import time
from multiprocessing import Pool

def is_prime(n):
    if n < 2: return False
    if n == 2: return True
    if n % 2 == 0: return False
    for i in range(3, int(n**0.5) + 1, 2):
        if n % i == 0:
            return False
    return True

def count_primes_in_chunk(chunk):
    return sum(1 for n in chunk if is_prime(n))

if __name__ == "__main__":
    # Génération des données
    data = list(range(100_000, 300_000))
    
    # --- SÉQUENTIEL ---
    start = time.perf_counter()
    res_seq = count_primes_in_chunk(data)
    end = time.perf_counter()
    print(f"Séquentiel : {end - start:.2f}s (Trouvé: {res_seq})")

    # --- PARALLÈLE ---
    start = time.perf_counter()
    # On découpe en 4 parts
    chunk_size = len(data) // 4
    chunks = [data[i:i + chunk_size] for i in range(0, len(data), chunk_size)]
    
    with Pool(processes=4) as pool:
        results = pool.map(count_primes_in_chunk, chunks)
        total = sum(results)
        
    end = time.perf_counter()
    print(f"Parallèle : {end - start:.2f}s (Trouvé: {total})")
```
</details>

### Exercice 2 - Traitement d'Images en Masse {#exercice-2-traitement-images}

🎯 **Objectif** : Utiliser `Pool` pour une tâche réaliste.

💼 **Mise en situation** : Une startup de e-commerce vous demande de redimensionner ("thumbnail") des milliers d'images produit téléchargées.

📝 **Énoncé** :
1.  Simulez une fonction `process_image(img_name)` qui dort 0.5s (simulation traitement lourd) et renvoie `f"{img_name}_thumb.jpg"`.
2.  Liste de 10 noms d'images.
3.  Utilisez `Pool().imap_unordered` pour traiter les images et afficher le résultat dès qu'une image est prête (pas besoin d'attendre la fin de tout le lot).

📺 **Résultat attendu** :
```text
Image 1 traitée...
Image 2 traitée...
(Les résultats apparaissent par paquets de N cœurs)
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
import time
import os
from multiprocessing import Pool

def process_image(img_name):
    # Simulation d'un traitement lourd (resize, filter...)
    time.sleep(0.5)
    return f"{img_name} -> {img_name}_thumb.jpg (PID: {os.getpid()})"

if __name__ == "__main__":
    images = [f"img_{i}.jpg" for i in range(10)]
    
    print("Début du traitement...")
    
    with Pool() as pool:
        # imap_unordered est idéal pour afficher la progression en temps réel
        # Il retourne un itérateur
        for result in pool.imap_unordered(process_image, images):
            print(f"✅ {result}")
            
    print("Traitement terminé.")
```
</details>

### Exercice 3 - Variables Partagées (Mémoire Partagée) {#exercice-3-memoire-partagee}

🎯 **Objectif** : Manipuler des données communes entre processus (Avancé).

💼 **Mise en situation** : Plusieurs processus doivent mettre à jour un compteur global atomique.

📝 **Énoncé** :
1.  Utilisez `multiprocessing.Value` pour créer un entier partagé initialisé à 0.
2.  Créez une fonction `increment(counter)` qui ajoute 100 au compteur.
3.  Lancez 10 processus qui exécutent tous cette fonction.
4.  Affichez la valeur finale. Notez l'utilisation nécessaire d'un verrou (lock) qui est souvent inclus ou géré manuellement.

📺 **Résultat attendu** :
```text
Valeur finale attendue : 1000
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
from multiprocessing import Process, Value, Lock

def increment(counter, lock):
    for _ in range(100):
        with lock:
            counter.value += 1

if __name__ == "__main__":
    # 'i' pour integer, 0 valeur initiale
    shared_counter = Value('i', 0)
    lock = Lock()
    
    processes = []
    for _ in range(10):
        p = Process(target=increment, args=(shared_counter, lock))
        processes.append(p)
        p.start()
        
    for p in processes:
        p.join()
        
    print(f"Valeur finale du compteur partagé : {shared_counter.value}")
```
</details>