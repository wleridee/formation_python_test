---
sidebar_label: Asynchrone : asyncio et await
sidebar_position: 44
---

# Chapitre 44 : Asynchrone : `asyncio` et `await`

Programmation asynchrone, Mots-clés async/await, Event loop, Tâches asynchrones

Le code asynchrone a révolutionné la façon dont nous écrivons des applications web et réseau performantes en Python. Contrairement aux threads (concurrence préemptive gérée par l'OS), l'asynchrone est une **concurrence coopérative** gérée par Python lui-même via une "boucle d'événements" (Event Loop).

C'est la technologie derrière des frameworks ultra-rapides comme FastAPI. Maîtriser `async` et `await` est indispensable pour un développeur Python moderne.

---

## 1. Concepts : Async, Await et Coroutines {#concepts-async-await}

### 1. Quoi
*   **Synchrone (Bloquant)** : "Je lance une tâche, j'attends qu'elle finisse avant de passer à la suite."
*   **Asynchrone (Non-bloquant)** : "Je lance une tâche, je fais autre chose en attendant, et je reviens quand elle a fini."
*   **Coroutine** : Une fonction spéciale définie avec `async def`. Elle peut être mise en pause et reprise.

### 2. Pourquoi
Pour gérer des milliers de connexions réseau simultanées (WebSockets, requêtes HTTP, Chatbots) sur **un seul thread**, sans la lourdeur mémoire des threads système.

### 3. Comment

#### A. Syntaxe de base : La Transformation

**❌ Version Synchrone (Classique)**
```python
import time

def dire_bonjour():
    print("Bonjour")
    time.sleep(1) # Bloque TOUT le programme pendant 1s
    print("Au revoir")

dire_bonjour()
```

**✅ Version Asynchrone (Moderne)**
```python
import asyncio

# 1. async def définit une coroutine
async def dire_bonjour_async():
    print("Bonjour")
    # 2. await "rend la main" à l'event loop pendant l'attente
    await asyncio.sleep(1) 
    print("Au revoir")

# 3. Lancement via asyncio.run (le point d'entrée)
if __name__ == "__main__":
    asyncio.run(dire_bonjour_async())
```

### 4. Zone de Danger
❌ **Ne jamais utiliser `time.sleep()` dans du code async !** Cela bloquerait la boucle d'événement et donc *toutes* les autres tâches. Utilisez toujours `await asyncio.sleep()`.

---

## 2. La Boucle d'Événements (Event Loop) {#event-loop}

### 1. Quoi
C'est le chef d'orchestre. C'est une boucle infinie (`while True`) qui tourne en arrière-plan, vérifie si des tâches sont terminées, et exécute les callbacks correspondants. `asyncio.run()` crée et gère cette boucle pour vous.

### 2. Pourquoi
C'est ce mécanisme qui permet à un seul thread de gérer plusieurs tâches "en même temps" (entrelacées). Quand une tâche attend (ex: requête DB), la boucle passe instantanément à une autre tâche.

### 3. Comment

#### A. Lancer plusieurs tâches en parallèle (Gather)

```python
import asyncio

async def fetch_data(id_data):
    print(f"⏳ Début chargement {id_data}...")
    await asyncio.sleep(1) # Simule I/O (Réseau/Disque)
    print(f"✅ Fin chargement {id_data}")
    return {"id": id_data, "data": "X"}

async def main():
    print("🚀 Démarrage du batch")
    
    # asyncio.gather lance tout en "parallèle"
    # Le temps total sera ~1s, pas 3s !
    resultats = await asyncio.gather(
        fetch_data(1),
        fetch_data(2),
        fetch_data(3)
    )
    
    print(f"📦 Résultats reçus : {resultats}")

if __name__ == "__main__":
    asyncio.run(main())
```

---

## 3. Gestion des Tâches (Tasks) {#gestion-des-taches}

### 1. Quoi
Une `Task` est une coroutine enveloppée qui est planifiée pour s'exécuter "bientôt" dans la boucle d'événements. Cela permet de lancer une tâche en arrière-plan sans l'attendre immédiatement.

### 2. Pourquoi
Pour faire du "Fire and Forget" ou préparer des traitements pendant qu'on fait autre chose.

### 3. Comment

```python
import asyncio

async def tache_fond():
    print("   🔧 Tâche de fond : En cours...")
    await asyncio.sleep(2)
    print("   🔧 Tâche de fond : Terminée !")
    return "OK"

async def main():
    # 1. Créer la tâche (elle est planifiée immédiatement)
    task = asyncio.create_task(tache_fond())
    
    print("👉 Le main continue son travail...")
    await asyncio.sleep(1)
    print("👉 Le main a fini son travail, on attend la tâche de fond...")
    
    # 2. Attendre le résultat final
    res = await task
    print(f"👉 Résultat final : {res}")

if __name__ == "__main__":
    asyncio.run(main())
```

#### D. Comparatif : Threading vs Asyncio

| Critère | Threading | Asyncio |
| :--- | :--- | :--- |
| **Modèle** | Préemptif (L'OS décide quand changer) | Coopératif (Le code décide avec `await`) |
| **Contexte Switch** | Coûteux (Ressources système) | Très léger (Fonction Python) |
| **Race Conditions** | Fréquentes (Locks nécessaires) | Rares (Code séquentiel entre les `await`) |
| **Code** | Ressemble au séquentiel | "Contaminant" (Tout doit être async) |
| **Idéal pour** | I/O Bloquant (librairies anciennes) | I/O Massif (WebSockets, HTTP/2) |

---

## 4. Timeout et Annulation {#timeout-et-annulation}

### 1. Quoi
La capacité d'interrompre une tâche si elle prend trop de temps, chose très difficile à faire proprement avec des threads.

### 2. Pourquoi
Pour la résilience. Si une API externe ne répond pas en 5 secondes, on ne veut pas bloquer l'utilisateur indéfiniment.

### 3. Comment

```python
import asyncio

async def requete_lente():
    try:
        print("📡 Envoi requête...")
        await asyncio.sleep(10) # Simule un serveur lent
        return "Réponse 200 OK"
    except asyncio.CancelledError:
        print("🛑 Requête annulée proprement")
        raise # Toujours relancer l'annulation !

async def main():
    try:
        # On impose une limite de 2 secondes
        async with asyncio.timeout(2):
            res = await requete_lente()
            print(res)
    except TimeoutError:
        print("⏰ Trop long ! Abandon.")

if __name__ == "__main__":
    asyncio.run(main())
```

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-44}

1.  **Quelle est la différence fondamentale entre `time.sleep(1)` et `await asyncio.sleep(1)` ?**
    `time.sleep` bloque tout le thread (et donc la boucle d'événements). `await asyncio.sleep` rend la main à la boucle, permettant à d'autres tâches de s'exécuter pendant la pause.

2.  **Peut-on appeler une fonction `async` sans `await` ?**
    Oui, mais cela ne l'exécutera pas. Cela retournera juste un objet coroutine (message "RuntimeWarning: coroutine was never awaited"). Pour l'exécuter, il faut soit l'`await`, soit en faire une `Task`, soit la passer à `asyncio.run`.

3.  **Pourquoi dit-on que l'async est "contaminant" ?**
    Car pour utiliser `await`, il faut être dans une fonction `async`. De proche en proche, toute la chaîne d'appels remonte jusqu'au point d'entrée qui doit être géré par `asyncio`.

4.  **Asyncio utilise-t-il plusieurs cœurs CPU ?**
    Non, par défaut tout tourne sur un seul thread/cœur. Pour utiliser plusieurs cœurs, il faut combiner asyncio avec `multiprocessing` ou `ProcessPoolExecutor`.

---

## Exercices : {#exercices-44}

### Exercice 1 - Le grille-pain asynchrone {#exercice-1-grille-pain}

🎯 **Objectif** : Comprendre le gain de temps du parallélisme avec `gather`.

💼 **Mise en situation** : Préparer un petit-déjeuner. Il faut griller le pain (2s) et faire le café (2s). En séquentiel, ça prend 4s. En asynchrone, ça doit prendre 2s.

📝 **Énoncé** :
1.  Créez deux coroutines : `griller_pain()` et `faire_cafe()`.
2.  Chacune doit afficher "Début...", attendre 2s (`asyncio.sleep`), puis afficher "Fin !".
3.  Dans `main()`, lancez-les d'abord séquentiellement (avec await l'un après l'autre) et mesurez le temps.
4.  Ensuite, lancez-les avec `asyncio.gather()` et mesurez le temps.

📺 **Résultat attendu** :
```text
--- Séquentiel ---
Temps: 4.01s
--- Asynchrone ---
Temps: 2.01s
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
import asyncio
import time

async def griller_pain():
    print("🍞 Pain au four...")
    await asyncio.sleep(2)
    print("🍞 Pain grillé !")

async def faire_cafe():
    print("☕ Café en cours...")
    await asyncio.sleep(2)
    print("☕ Café prêt !")

async def main():
    # Séquentiel
    print("--- Séquentiel ---")
    start = time.perf_counter()
    await griller_pain()
    await faire_cafe()
    print(f"Temps: {time.perf_counter() - start:.2f}s")

    # Asynchrone
    print("\n--- Asynchrone ---")
    start = time.perf_counter()
    # On lance les deux en même temps
    await asyncio.gather(griller_pain(), faire_cafe())
    print(f"Temps: {time.perf_counter() - start:.2f}s")

if __name__ == "__main__":
    asyncio.run(main())
```
</details>

### Exercice 2 - Timeout sur API externe {#exercice-2-timeout-api}

🎯 **Objectif** : Gérer les lenteurs externes.

💼 **Mise en situation** : Votre application contacte 3 serveurs de météo. L'un d'eux est en panne et met 10s à répondre (ou ne répond jamais). Vous ne voulez pas attendre plus de 3s au total.

📝 **Énoncé** :
1.  Créez `fetch_weather(server_name, delay)` qui attend `delay` secondes.
2.  Lancez 3 tâches : Serveur A (1s), Serveur B (5s), Serveur C (0.5s).
3.  Utilisez `asyncio.gather` pour récupérer les résultats.
4.  Enveloppez le tout dans un `asyncio.timeout(3)`.
5.  Gérez l'exception pour afficher "Annulation : trop lent" si ça dépasse.

📺 **Résultat attendu** :
```text
Serveur C fini.
Serveur A fini.
Timeout ! Le traitement global a été annulé.
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
import asyncio

async def fetch_weather(name, delay):
    print(f"🌍 Contact {name}...")
    await asyncio.sleep(delay)
    print(f"✅ {name} a répondu.")
    return f"Météo {name}"

async def main():
    try:
        # On définit un budget temps global de 3 secondes
        async with asyncio.timeout(3):
            results = await asyncio.gather(
                fetch_weather("Server A", 1),
                fetch_weather("Server B", 5), # Lui fera échouer le groupe
                fetch_weather("Server C", 0.5)
            )
            print("Résultats:", results)
            
    except TimeoutError:
        print("❌ Timeout ! Le traitement global a été annulé.")

if __name__ == "__main__":
    asyncio.run(main())
```
</details>

### Exercice 3 - Le Crawler Limité (Sémaphore) {#exercice-3-crawler-semaphore}

🎯 **Objectif** : Contrôler la concurrence (Rate Limiting).

💼 **Mise en situation** : Vous devez télécharger 100 pages. Si vous lancez 100 tâches `gather` en même temps, vous allez être banni du serveur ou planter votre réseau. Vous voulez limiter à 3 téléchargements simultanés max.

📝 **Énoncé** :
1.  Utilisez `asyncio.Semaphore(3)`.
2.  Créez une coroutine `download(sem, id)` qui acquiert le sémaphore (`async with sem:`), attend 1s, puis relâche.
3.  Lancez 10 tâches.
4.  Observez que les tâches se lancent par vagues de 3.

📺 **Résultat attendu** :
```text
Téléchargement 0, 1, 2 lancés... (attente 1s)
Fini 0, 1, 2.
Téléchargement 3, 4, 5 lancés...
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
import asyncio

async def download(sem, doc_id):
    # On attend qu'une place se libère dans le sémaphore (max 3)
    async with sem:
        print(f"📥 Start {doc_id}")
        await asyncio.sleep(1) # Simulation téléchargement
        print(f"✅ End {doc_id}")

async def main():
    # Limite de concurrence
    sem = asyncio.Semaphore(3)
    
    tasks = [download(sem, i) for i in range(10)]
    
    # On attend tout
    await asyncio.gather(*tasks)

if __name__ == "__main__":
    asyncio.run(main())
```
</details>