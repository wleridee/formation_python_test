---
sidebar_label: Concurrence : Threads (threading)
sidebar_position: 42
---

# Chapitre 42 : Concurrence : Threads (threading)

Introduction aux threads, Création de threads, Verrous (locks), Problèmes de concurrence

Dans un programme classique, les instructions s'exécutent les unes après les autres. Si l'une d'elles prend du temps (télécharger un fichier, attendre une réponse API), tout le programme se fige.
Le **multithreading** permet de lancer plusieurs tâches en "parallèle" au sein du même processus. C'est l'outil idéal pour garder une application réactive pendant qu'elle effectue des opérations d'Entrée/Sortie (I/O).

---

## 1. Concepts de Base et le Module `threading` {#concepts-de-base-threading}

### 1. Quoi
Un **Thread** (fil d'exécution) est la plus petite unité de traitement qu'un système d'exploitation peut planifier.
Le module `threading` de la bibliothèque standard permet de créer et gérer ces threads en Python.

### 2. Pourquoi
Pour exécuter des tâches bloquantes (I/O Bound) en arrière-plan sans geler le programme principal.
*Exemple :* Une interface graphique ne doit pas se figer pendant que l'application sauvegarde un fichier.

### 3. Comment

#### A. Syntaxe de base

```python
import threading
import time

def worker(name):
    print(f"🧵 Thread {name} démarre")
    time.sleep(2) # Simule une tâche longue (I/O)
    print(f"✅ Thread {name} terminé")

# Création des threads
t1 = threading.Thread(target=worker, args=("A",))
t2 = threading.Thread(target=worker, args=("B",))

# Démarrage
t1.start()
t2.start()

print("🚀 Programme principal continue...")

# Attendre la fin des threads (optionnel mais recommandé ici)
t1.join()
t2.join()

print("🏁 Tout est fini")
```

**Sortie typique :**
```text
🧵 Thread A démarre
🧵 Thread B démarre
🚀 Programme principal continue...
(2 secondes de pause)
✅ Thread A terminé
✅ Thread B terminé
🏁 Tout est fini
```

### 🚨 Limitations : Le GIL (Global Interpreter Lock)
Python (CPython) possède un verrou global (GIL) qui empêche deux threads Python d'exécuter du bytecode **simultanément** sur le même cœur CPU.
*   **Conséquence** : Le multithreading en Python n'accélère PAS les calculs purs (CPU Bound).
*   **Usage idéal** : Opérations I/O (Réseau, Disque, Base de données) où le CPU attend de toute façon.

---

## 2. La Course aux Données (Race Conditions) {#race-conditions}

### 1. Quoi
Une **Race Condition** survient quand plusieurs threads accèdent et modifient une ressource partagée (variable, fichier) en même temps, sans coordination. Le résultat final dépend de l'ordre aléatoire d'exécution.

### 2. Pourquoi
C'est la source de bugs les plus difficiles à reproduire. Une opération comme `compteur += 1` n'est pas atomique (elle se décompose en lecture, addition, écriture). Si un thread est interrompu au milieu, les données sont corrompues.

### 3. Comment

#### A. Exemple de bug (Race Condition)

```python
import threading

compteur = 0

def incrementer():
    global compteur
    for _ in range(1000000):
        compteur += 1

t1 = threading.Thread(target=incrementer)
t2 = threading.Thread(target=incrementer)

t1.start()
t2.start()
t1.join()
t2.join()

# On attendrait 2 000 000, mais on aura souvent moins (ex: 1 458 921)
print(f"Valeur finale : {compteur}") 
```

---

## 3. Synchronisation avec les Verrous (Locks) {#synchronisation-locks}

### 1. Quoi
Un **Lock** (verrou) est un mécanisme qui permet à un seul thread d'accéder à une section de code critique à la fois.
Si un thread A possède le verrou, le thread B doit attendre que A le relâche pour continuer.

### 2. Pourquoi
Pour garantir l'intégrité des données partagées en rendant les opérations atomiques.

### 3. Comment

#### A. Utilisation de `Lock` avec `with`

```python
import threading

compteur = 0
verrou = threading.Lock() # Création du verrou

def incrementer_safe():
    global compteur
    for _ in range(1000000):
        # Acquisition automatique du verrou
        with verrou:
            compteur += 1 
        # Relâchement automatique à la fin du bloc with

t1 = threading.Thread(target=incrementer_safe)
t2 = threading.Thread(target=incrementer_safe)

t1.start()
t2.start()
t1.join()
t2.join()

print(f"Valeur finale sécurisée : {compteur}") # Toujours 2 000 000
```

### 4. Zone de Danger
❌ **Deadlock (Interblocage)** : Si le thread A attend le verrou B, et que le thread B attend le verrou A, le programme se fige pour l'éternité.
✅ **Bonne pratique** : Utilisez toujours `with lock:` pour garantir que le verrou est relâché même en cas d'exception.

---

## 4. `ThreadPoolExecutor` : La gestion moderne {#threadpoolexecutor}

### 1. Quoi
Une abstraction de haut niveau fournie par le module `concurrent.futures`. Elle gère automatiquement un pool (groupe) de threads réutilisables.

### 2. Pourquoi
Plus simple et plus sûr que de gérer manuellement `start()` et `join()`. Permet de récupérer facilement les valeurs de retour des fonctions exécutées dans les threads.

### 3. Comment

```python
from concurrent.futures import ThreadPoolExecutor
import time

def telecharger_page(url):
    time.sleep(1) # Simule latence réseau
    return f"Contenu de {url}"

urls = ["google.com", "python.org", "github.com"]

# Création d'un pool de 3 workers
with ThreadPoolExecutor(max_workers=3) as executor:
    # Mappe la fonction sur la liste d'URLs
    resultats = executor.map(telecharger_page, urls)

for res in resultats:
    print(res)
```

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-42}

1.  **Pourquoi le multithreading n'accélère-t-il pas un calcul mathématique intensif en Python ?**
    À cause du **GIL (Global Interpreter Lock)** qui empêche l'exécution simultanée de bytecode Python sur plusieurs cœurs. Seul un thread Python tourne à la fois.

2.  **Quand faut-il utiliser des threads en Python ?**
    Pour les tâches **I/O Bound** : requêtes réseau, lecture/écriture disque, attente utilisateur.

3.  **Quel est le risque principal lors du partage de variables entre threads ?**
    Les **Race Conditions** (accès concurrent non protégé) qui corrompent les données.

4.  **Comment éviter un Deadlock ?**
    Toujours acquérir les verrous dans le même ordre, utiliser des timeouts, ou préférer des architectures sans état partagé (comme les queues).

---

## Exercices : {#exercices-42}

### Exercice 1 - Le Téléchargeur Parallèle {#exercice-1-telechargeur-parallele}

🎯 **Objectif** : Comparer l'exécution séquentielle vs multithreadée.

💼 **Mise en situation** : Vous devez vérifier le statut de 5 sites web. En séquentiel, c'est lent.

📝 **Énoncé** :
1.  Créez une fonction `check_site(url)` qui simule une requête avec `time.sleep(1)` et affiche "Checked [url]".
2.  Liste d'URLs : `["site1", "site2", "site3", "site4", "site5"]`.
3.  Mesurez le temps total pour traiter la liste de manière séquentielle (boucle `for`).
4.  Mesurez le temps total en utilisant `ThreadPoolExecutor` avec 5 workers.

📺 **Résultat attendu** :
```text
Séquentiel : environ 5 secondes.
Parallèle : environ 1 seconde.
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
import time
from concurrent.futures import ThreadPoolExecutor

urls = [f"site{i}" for i in range(1, 6)]

def check_site(url):
    time.sleep(1) # Simulation I/O
    return f"Checked {url}"

# --- Version Séquentielle ---
start = time.perf_counter()
for url in urls:
    check_site(url)
end = time.perf_counter()
print(f"Séquentiel : {end - start:.2f} secondes")

# --- Version Threadée ---
start = time.perf_counter()
with ThreadPoolExecutor(max_workers=5) as executor:
    executor.map(check_site, urls)
end = time.perf_counter()
print(f"Parallèle : {end - start:.2f} secondes")
```
</details>

### Exercice 2 - Le Compte Bancaire Partagé (Lock) {#exercice-2-compte-bancaire-lock}

🎯 **Objectif** : Protéger une ressource critique.

💼 **Mise en situation** : Un compte commun est utilisé par deux personnes (threads) simultanément. L'une dépose, l'autre retire.

📝 **Énoncé** :
1.  Classe `BankAccount` avec un solde `balance = 100`.
2.  Méthode `modify(amount)` qui lit le solde, fait un `sleep(0.001)` (pour provoquer la race condition), modifie, et sauvegarde.
3.  Lancez 100 threads qui font `+10` et 100 threads qui font `-10`.
4.  Sans Lock : le solde final sera faux.
5.  Ajoutez un `threading.Lock` pour corriger.

📺 **Résultat attendu** :
```text
Sans Lock : Solde final aléatoire (ex: 80, 120...)
Avec Lock : Solde final exactement 100.
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
import threading
import time

class BankAccount:
    def __init__(self):
        self.balance = 100
        self.lock = threading.Lock() # Le gardien

    def modify(self, amount):
        # Section Critique protégée
        with self.lock:
            local_copy = self.balance
            time.sleep(0.001) # Le piège à race condition
            local_copy += amount
            self.balance = local_copy

account = BankAccount()
threads = []

# 100 dépôts et 100 retraits
for _ in range(100):
    t_add = threading.Thread(target=account.modify, args=(10,))
    t_sub = threading.Thread(target=account.modify, args=(-10,))
    threads.extend([t_add, t_sub])
    t_add.start()
    t_sub.start()

for t in threads:
    t.join()

print(f"Solde final : {account.balance}")
```
</details>

### Exercice 3 - Producteur / Consommateur avec Queue {#exercice-3-producteur-consommateur}

🎯 **Objectif** : Communication inter-thread sûre.

💼 **Mise en situation** : Un thread génère des commandes (Producteur), un autre les traite (Consommateur). Ils ne doivent pas se marcher dessus.

📝 **Énoncé** :
1.  Utilisez `queue.Queue` (qui est thread-safe par défaut, pas besoin de Lock manuel).
2.  `producer` : Ajoute 5 entiers dans la queue avec un petit délai.
3.  `consumer` : Récupère les entiers avec `q.get()` et affiche "Traitement de X". S'arrête quand il reçoit `None` (signal de fin).
4.  Lancez les deux threads.

📺 **Résultat attendu** :
```text
Produit 1
Consommé 1
Produit 2
Consommé 2...
Terminé.
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
import threading
import queue
import time

def producer(q):
    for i in range(5):
        time.sleep(0.5)
        print(f"🏭 Production commande {i}")
        q.put(i)
    # Signal de fin (Sentinel value)
    q.put(None) 
    print("🏭 Production terminée")

def consumer(q):
    while True:
        item = q.get() # Bloque si vide
        if item is None:
            break # On arrête la boucle
        print(f"🛒 Traitement commande {item}")
        time.sleep(1) # Simulation traitement
    print("🛒 Consommateur fermé")

ma_queue = queue.Queue()

t1 = threading.Thread(target=producer, args=(ma_queue,))
t2 = threading.Thread(target=consumer, args=(ma_queue,))

t1.start()
t2.start()

t1.join()
t2.join()
```
</details>