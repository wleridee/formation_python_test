---
sidebar_label: Module `logging` : Gestion des Logs
sidebar_position: 32
---

# Chapitre 32 : Module `logging` : Gestion des Logs

Niveaux de log, Configuration de base, Handlers, Formatters

Abandonnez les `print()` pour le débogage. Dans une application réelle (serveur web, script d'automatisation, analyse de données), vous avez besoin de traçabilité, de niveaux de gravité et de persistance.

Le module `logging` est la solution standard de Python pour émettre des messages d'état. Il permet de filtrer ce qui est affiché (ne pas inonder la console de détails inutiles), de rediriger les erreurs vers des fichiers, et de formater les messages pour qu'ils soient lisibles par des machines ou des humains.

---

## 1. Les Niveaux de Log (Log Levels) {#les-niveaux-de-log}

### 1. Quoi
Chaque message de log possède une **gravité** (un niveau). `logging` définit 5 niveaux standards, qui sont des entiers croissants :

| Niveau | Valeur | Usage |
| :--- | :--- | :--- |
| **DEBUG** | 10 | Détails techniques pointus pour le développeur (valeurs de variables, étapes). |
| **INFO** | 20 | Confirmation que tout marche comme prévu (démarrage, arrêt, étapes clés). |
| **WARNING** | 30 | Indication que quelque chose est inattendu, mais le programme continue (disque presque plein). **Niveau par défaut**. |
| **ERROR** | 40 | Problème sérieux, une fonction n'a pas pu s'exécuter (connexion échouée). |
| **CRITICAL** | 50 | Erreur grave, le programme risque de s'arrêter (plus de mémoire, base de données HS). |

### 2. Pourquoi
Cela permet de filtrer l'information.
*   En **développement**, on veut tout voir (`DEBUG`).
*   En **production**, on ne veut voir que les problèmes (`WARNING` ou `ERROR`) pour ne pas remplir les disques dur avec du bruit.

### 3. Comment

#### A. Utilisation basique

```python
import logging

# Par défaut, le niveau est WARNING. 
# Donc DEBUG et INFO ne s'afficheront pas.
logging.debug("Ceci est un détail technique (invisible par défaut)")
logging.info("Le programme a démarré (invisible par défaut)")
logging.warning("Attention : Espace disque faible")
logging.error("Erreur : Impossible d'ouvrir le fichier")
logging.critical("Arrêt d'urgence du système !")
```

#### B. Configuration du niveau minimum
Pour voir les messages `DEBUG` et `INFO`, il faut configurer le logger racine.

```python
import logging

# On configure le niveau MINIMUM à DEBUG
logging.basicConfig(level=logging.DEBUG)

logging.debug("Maintenant, ce message est visible !")
```

### 4. Zone de Danger
❌ **Utiliser `print` pour les erreurs** : `print` écrit sur `stdout` (sortie standard). Les erreurs doivent aller sur `stderr` (sortie d'erreur) et inclure un timestamp. `logging` gère cela automatiquement.

---

## 2. Configuration de Base (`basicConfig`) {#configuration-de-base}

### 1. Quoi
La fonction `logging.basicConfig()` est le moyen le plus rapide de configurer le système de logs global. Elle permet de définir :
*   Le niveau minimum (`level`).
*   Le format du message (`format`).
*   La destination (`filename` pour écrire dans un fichier, sinon console).

### 2. Pourquoi
Pour passer de "scripts amateurs" à "applications professionnelles" en une ligne de code. Avoir des timestamps (date/heure) est crucial pour corréler des événements.

### 3. Comment

#### A. Log vers la console avec formatage

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s',
    datefmt='%Y-%m-%d %H:%M:%S'
)

logging.info("Serveur démarré.")
# Résultat : 2026-10-25 14:30:00 - INFO - Serveur démarré.
```

#### B. Log vers un fichier

```python
import logging

# filemode='a' (append) est le défaut. 'w' écrase le fichier à chaque lancement.
logging.basicConfig(
    filename='app.log', 
    filemode='a', 
    level=logging.ERROR,
    format='%(asctime)s : %(message)s'
)

logging.error("Connexion perdue") 
# Rien dans la console, tout est écrit dans app.log
```

### 🚨 Limitations de `basicConfig`
Cette fonction ne peut être appelée qu'**une seule fois**. Les appels suivants sont ignorés silencieusement. Si une autre librairie a déjà appelé `basicConfig`, votre configuration ne s'appliquera pas.
Pour des configurations complexes, il faut utiliser des objets Logger, Handler et Formatter explicites (voir section suivante).

---

## 3. Architecture Avancée : Loggers, Handlers et Formatters {#architecture-avancee}

### 1. Quoi
Le module `logging` est modulaire :
*   **Logger** : L'objet que vous appelez dans votre code (`logger.info(...)`).
*   **Handler** : Décide *où* envoyer le message (Console, Fichier, Email, HTTP...). Un logger peut avoir plusieurs handlers.
*   **Formatter** : Décide *comment* présenter le message (texte brut, JSON...).

### 2. Pourquoi
Vous voulez souvent :
1.  Afficher **tout** dans la console pour le développeur.
2.  N'écrire que les **erreurs** critiques dans un fichier pour l'admin système.
3.  Envoyer un **email** en cas de CRITICAL.
Tout cela simultanément.

### 3. Comment

```python
import logging
import sys

# 1. Création du Logger personnalisé (bonne pratique : nom du module)
logger = logging.getLogger("mon_application_ecommerce")
logger.setLevel(logging.DEBUG) # Le logger capture tout

# 2. Création des Handlers
# Console : Affiche tout (DEBUG et plus)
console_handler = logging.StreamHandler(sys.stdout)
console_handler.setLevel(logging.DEBUG)

# Fichier : N'enregistre que les ERREURS (ERROR et CRITICAL)
file_handler = logging.FileHandler("errors.log")
file_handler.setLevel(logging.ERROR)

# 3. Création des Formatters
simple_format = logging.Formatter('%(levelname)s: %(message)s')
detailed_format = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')

# 4. Association
console_handler.setFormatter(simple_format)
file_handler.setFormatter(detailed_format)

logger.addHandler(console_handler)
logger.addHandler(file_handler)

# 5. Utilisation
logger.debug("Variable x = 10")      # Console uniquement
logger.error("Base de données HS")   # Console + Fichier errors.log
```

### D. Tableau Comparatif des Handlers Courants

| Handler | Destination | Usage typique |
| :--- | :--- | :--- |
| `StreamHandler` | Console (stdout/stderr) | Développement, Docker logs |
| `FileHandler` | Fichier texte | Archivage local |
| `RotatingFileHandler` | Fichiers multiples | Évite les fichiers logs de 100 Go (rotation par taille) |
| `TimedRotatingFileHandler`| Fichiers datés | Rotation par jour/semaine |
| `SMTPHandler` | Email | Alertes critiques immédiates |

---

## 4. Capturer les Exceptions (`exc_info`) {#capturer-les-exceptions}

### 1. Quoi
Lorsqu'une erreur survient dans un bloc `try/except`, on veut souvent logger la trace complète (Stack Trace) pour pouvoir déboguer, pas juste le message d'erreur.

### 2. Pourquoi
Un log `ERROR: Division par zéro` est inutile si on ne sait pas dans quelle ligne et quel fichier cela s'est produit.

### 3. Comment
Utilisez `logger.exception()` dans un bloc `except`. C'est équivalent à `logger.error(..., exc_info=True)`.

```python
import logging

logging.basicConfig(level=logging.ERROR)

def risky_math():
    return 1 / 0

try:
    risky_math()
except ZeroDivisionError:
    # Capture automatiquement la Stack Trace complète
    logging.exception("Oups, un calcul a échoué")
```

**Sortie :**
```text
ERROR:root:Oups, un calcul a échoué
Traceback (most recent call last):
  File "script.py", line 8, in <module>
    risky_math()
  File "script.py", line 5, in risky_math
    return 1 / 0
ZeroDivisionError: division by zero
```

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-32}

1.  **Quel est le niveau de log par défaut si on ne configure rien ?**
    Le niveau `WARNING` (30). Les messages `INFO` et `DEBUG` sont ignorés.

2.  **Quelle est la différence entre `logging.error()` et `logging.exception()` ?**
    `logging.exception()` émet un message de niveau ERROR *mais ajoute automatiquement la trace de l'exception (stack trace)*. Elle ne doit être utilisée que dans un bloc `except`.

3.  **Pourquoi est-il recommandé d'utiliser `logger = logging.getLogger(__name__)` plutôt que le logger racine ?**
    Cela permet de savoir exactement de quel module vient le log (ex: `myproject.database` vs `myproject.ui`) et de configurer des niveaux différents par module.

4.  **Comment éviter qu'un fichier de log ne remplisse tout le disque dur ?**
    En utilisant un `RotatingFileHandler` qui limite la taille des fichiers et conserve un nombre fixe d'archives.

---

## Exercices : {#exercices-32}

### Exercice 1 - Le Logger de Simulation Bancaire {#exercice-1-banque}

🎯 **Objectif** : Configurer `basicConfig` avec un format personnalisé.

💼 **Mise en situation** : Vous développez le backend d'une néo-banque. Chaque transaction doit être loggée avec l'heure précise.

📝 **Énoncé** :
1.  Configurez le logging pour écrire dans la console.
2.  Format requis : `[HEURE] NIVEAU : Message` (ex: `[14:05:01] INFO : Virement effectué`).
3.  Niveau minimum : `INFO`.
4.  Simulez 3 actions :
    - Un debug ("Vérification solde" -> ne doit pas s'afficher).
    - Une info ("Virement de 50€ envoyé").
    - Un warning ("Tentative de connexion échouée").

📺 **Résultat attendu** :
```text
[HH:MM:SS] INFO : Virement de 50€ envoyé
[HH:MM:SS] WARNING : Tentative de connexion échouée
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
import logging

# Configuration unique au début du script
logging.basicConfig(
    level=logging.INFO, # On ignore DEBUG
    format='[%(asctime)s] %(levelname)s : %(message)s',
    datefmt='%H:%M:%S'  # Heure seulement
)

# Simulation
logging.debug("Vérification solde (variable hidden=True)") # Ignoré
logging.info("Virement de 50€ envoyé")
logging.warning("Tentative de connexion échouée (IP 192.168.1.55)")
```
</details>

### Exercice 2 - Le Rotation Log Handler {#exercice-2-rotation}

🎯 **Objectif** : Utiliser `RotatingFileHandler` pour gérer le volume.

💼 **Mise en situation** : Votre script tourne en boucle infinie. Vous voulez garder des traces dans un fichier `loop.log`, mais ce fichier ne doit jamais dépasser 1 Ko (pour l'exercice). On garde 2 archives max.

📝 **Énoncé** :
1.  Créez un logger nommé "LoopLogger".
2.  Attachez-lui un `RotatingFileHandler` (fichier `loop.log`, taille max 100 octets, 2 backups).
3.  Faites une boucle qui écrit 10 lignes de log assez longues.
4.  Observez (via votre explorateur de fichiers ou `ls`) la création de `loop.log`, `loop.log.1`, `loop.log.2`.

📺 **Résultat attendu** :
Vous devriez voir plusieurs fichiers créés car 100 octets sont vite atteints.

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
import logging
from logging.handlers import RotatingFileHandler

logger = logging.getLogger("LoopLogger")
logger.setLevel(logging.INFO)

# Handler qui découpe les fichiers
# maxBytes=100 (très petit pour tester), backupCount=2 (garde .log et .log.1 et .log.2)
handler = RotatingFileHandler('loop.log', maxBytes=100, backupCount=2)
formatter = logging.Formatter('%(asctime)s - %(message)s')
handler.setFormatter(formatter)

logger.addHandler(handler)

# Génération de logs
for i in range(10):
    logger.info(f"Ceci est la ligne de log numéro {i} qui est assez longue pour remplir le fichier.")
    
print("Logs générés. Vérifiez votre dossier pour loop.log, loop.log.1, etc.")
```
</details>

### Exercice 3 - Filtrage Console vs Fichier {#exercice-3-double-handler}

🎯 **Objectif** : Différencier les destinations selon la gravité.

💼 **Mise en situation** : Un script de traitement de données (ETL).
- L'opérateur qui regarde l'écran veut voir l'avancement (`INFO`).
- Le fichier `errors.log` ne doit contenir QUE les problèmes (`ERROR`) pour être envoyé par mail plus tard.

📝 **Énoncé** :
1.  Créez un logger.
2.  Ajoutez un `StreamHandler` (Console) au niveau `INFO`.
3.  Ajoutez un `FileHandler` (Fichier `etl_errors.log`) au niveau `ERROR`.
4.  Loggez :
    - "Démarrage du traitement" (INFO) -> Console seulement.
    - "Traitement ligne 1..." (INFO) -> Console seulement.
    - "Erreur de format ligne 2" (ERROR) -> Console ET Fichier.
5.  Vérifiez le contenu du fichier : il ne doit y avoir qu'une seule ligne.

📺 **Résultat attendu** :
*Console :*
```text
Démarrage du traitement
Traitement ligne 1...
Erreur de format ligne 2
```
*Fichier etl_errors.log :*
```text
Erreur de format ligne 2
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
import logging
import sys

logger = logging.getLogger("ETL_System")
logger.setLevel(logging.DEBUG) # Le logger global accepte tout, les handlers filtreront

# 1. Handler Console (Info+)
c_handler = logging.StreamHandler(sys.stdout)
c_handler.setLevel(logging.INFO)
c_format = logging.Formatter('%(levelname)s: %(message)s')
c_handler.setFormatter(c_format)

# 2. Handler Fichier (Error+)
f_handler = logging.FileHandler('etl_errors.log', mode='w') # mode='w' pour reset à chaque test
f_handler.setLevel(logging.ERROR)
f_format = logging.Formatter('%(asctime)s - %(levelname)s - %(message)s')
f_handler.setFormatter(f_format)

# Ajout des handlers
logger.addHandler(c_handler)
logger.addHandler(f_handler)

# Test
logger.info("Démarrage du traitement")      # Console uniquement
logger.info("Traitement ligne 1...")        # Console uniquement
logger.error("Erreur de format ligne 2")    # Console + Fichier
```
</details>