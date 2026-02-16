---
sidebar_label: Programmation Orientée Objet : Constructeurs et Destructeurs
sidebar_position: 18
---

# Chapitre 18 : Programmation Orientée Objet : Constructeurs et Destructeurs

Méthode `__init__`, Initialisation des objets, Méthode `__del__`, Gestion de la mémoire

Dans le chapitre précédent, nous avons appris à créer des plans (classes) et des maisons (objets). Mais quand on construit une maison, on ne livre pas quatre murs vides. On installe l'électricité, l'eau courante et on peint les murs **avant** que les habitants n'arrivent. De même, quand la maison est détruite, il faut couper l'eau et l'électricité pour éviter les fuites.

En Python, ces étapes cruciales de "naissance" et de "mort" d'un objet sont gérées par des méthodes spéciales : le **constructeur** et le **destructeur**.

---

## 1. L'Initialisation avec `__init__` (Le Constructeur) {#initialisation-avec-init}

### 1. Quoi
La méthode spéciale `__init__` (double underscore, ou "dunder" init) est invoquée **automatiquement** juste après qu'une nouvelle instance de la classe a été créée en mémoire. C'est ce qu'on appelle communément le **constructeur** (bien que techniquement, ce soit l'initialiseur).

### 2. Pourquoi
*   **Garantir l'état** : Assurer qu'un objet est prêt à être utilisé dès sa création (pas d'attributs manquants).
*   **Validation** : Vérifier que les données fournies à la création sont valides.
*   **Configuration** : Paramétrer l'objet selon les besoins.

### 3. Comment

#### A. Syntaxe de base
La méthode prend toujours `self` en premier argument, suivi des paramètres de configuration.

```python
class Player:
    def __init__(self, nickname: str, level: int = 1):
        print(f"Création du joueur {nickname}...")
        self.nickname = nickname
        self.level = level
        self.xp = 0 # Initialisation d'une valeur par défaut interne

# L'appel déclenche automatiquement __init__
p1 = Player("Mario")      # Création du joueur Mario...
p2 = Player("Luigi", 5)   # Création du joueur Luigi...
```

#### B. Validation des données
Le constructeur est le gardien du temple. Il doit empêcher la création d'objets "cassés".

```python
class BankAccount:
    def __init__(self, initial_balance: float):
        if initial_balance < 0:
            # On empêche la création de l'objet si la condition n'est pas remplie
            raise ValueError("Un compte ne peut pas être ouvert avec un solde négatif.")
        
        self.balance = initial_balance

try:
    acc = BankAccount(-100)
except ValueError as e:
    print(f"Erreur : {e}") # Erreur : Un compte ne peut pas être ouvert avec un solde négatif.
```

### 4. Zone de Danger
❌ **Retourner une valeur** :
`__init__` doit **toujours** retourner `None`. Retourner autre chose lèvera une `TypeError`.

```python
# ❌ INTERDIT
def __init__(self):
    return 10 

# ✅ BONNE PRATIQUE
def __init__(self):
    self.value = 10
    # return None est implicite
```

---

## 2. La Destruction avec `__del__` (Le Destructeur) {#destruction-avec-del}

### 1. Quoi
La méthode `__del__` est appelée lorsque l'objet est sur le point d'être détruit par le gestionnaire de mémoire de Python. C'est la dernière volonté de l'objet.

### 2. Pourquoi
Pour effectuer un nettoyage de ressources externes qui ne sont pas gérées automatiquement par Python :
*   Fermer une connexion réseau.
*   Fermer un fichier ouvert manuellement.
*   Sauvegarder des logs finaux.

### 3. Comment

#### A. Fonctionnement du cycle de vie

```python
class TemporaryFile:
    def __init__(self, filename: str):
        self.filename = filename
        print(f"📂 Fichier {self.filename} créé.")

    def __del__(self):
        print(f"🔥 Fichier {self.filename} détruit.")

def create_temp():
    tmp = TemporaryFile("test.txt")
    # tmp existe ici
    print("Fin de la fonction create_temp")
    # À la fin du bloc, 'tmp' n'est plus référencé -> destruction

create_temp()
print("Suite du programme...")
```

**Sortie console :**
```text
📂 Fichier test.txt créé.
Fin de la fonction create_temp
🔥 Fichier test.txt détruit.
Suite du programme...
```

### 🚨 Limitations de `__del__`
L'utilisation de `__del__` est **souvent déconseillée** en Python moderne pour plusieurs raisons :
1.  **Imprévisibilité** : Vous ne savez pas exactement *quand* le Garbage Collector (GC) va passer.
2.  **Exceptions** : Les erreurs levées dans `__del__` sont ignorées (imprimées dans stderr mais ne stoppent pas le programme).
3.  **Alternative** : Préférez toujours les **Context Managers** (blocs `with`) pour la gestion des ressources (voir Chapitre 53).

---

## 3. Gestion de la Mémoire (Reference Counting) {#gestion-memoire}

### 1. Quoi
Python utilise un système de **comptage de références**. Chaque objet possède un compteur interne qui note combien de variables pointent vers lui.
*   Compteur > 0 : L'objet reste en vie.
*   Compteur = 0 : L'objet est détruit immédiatement (et `__del__` est appelé).

### 2. Pourquoi
Pour libérer la RAM automatiquement sans que le développeur n'ait à faire des `free()` ou `delete` manuels comme en C++.

### 3. Comment

```python
import sys

class Node:
    pass

obj = Node()
# Le compteur est à 2 (1 pour 'obj' + 1 pour l'argument de getrefcount)
print(sys.getrefcount(obj)) 

alias = obj
# Le compteur augmente car 'alias' pointe aussi dessus
print(sys.getrefcount(obj)) 

del obj
# L'objet existe toujours car 'alias' pointe dessus !

del alias
# Compteur tombe à 0 -> L'objet est supprimé de la mémoire.
```

### 4. Zone de Danger
❌ **Références Circulaires** :
Si A pointe vers B et B pointe vers A, leur compteur ne tombera jamais à zéro, même si vous supprimez les variables externes.
✅ Le **Garbage Collector** cyclique de Python passe périodiquement pour détecter et nettoyer ces îlots isolés, mais cela prend du temps.

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-18}

1.  **Quelle méthode est appelée automatiquement lors de la création d'un objet ?**
    `__init__`.

2.  **Que doit impérativement retourner la méthode `__init__` ?**
    Elle doit retourner `None` (implicitement ou explicitement).

3.  **Quand la méthode `__del__` est-elle exécutée ?**
    Lorsque le compteur de références de l'objet tombe à zéro, c'est-à-dire quand plus aucune variable ne pointe vers cet objet.

4.  **Pourquoi `__del__` n'est-il pas la meilleure façon de gérer la fermeture d'un fichier ?**
    Car on ne contrôle pas le moment exact de son exécution. Si le programme plante ou si le Garbage Collector tarde, le fichier restera ouvert trop longtemps.

---

## Exercices : {#exercices-18}

### Exercice 1 - Le Portail de Sécurité (Validation) {#exercice-1---portail-securite}

🎯 **Objectif** : Validation stricte dans le constructeur.

💼 **Mise en situation** : Vous gérez les badges d'accès d'une centrale nucléaire. Un badge ne peut pas être créé si le niveau d'accréditation est insuffisant.

📝 **Énoncé** :
1.  Créez une classe `SecurityBadge`.
2.  Le constructeur prend `owner` (str) et `access_level` (int).
3.  Si `access_level` est inférieur à 1 ou supérieur à 5, levez une `ValueError` avec un message explicite.
4.  Si valide, affichez "Badge créé pour [owner]".
5.  Testez la création d'un badge valide et d'un badge invalide (avec un bloc try/except).

📺 **Résultat attendu** :
```text
Badge créé pour Alice (Niveau 3)
Erreur de création : Niveau invalide (0). Doit être entre 1 et 5.
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
class SecurityBadge:
    def __init__(self, owner: str, access_level: int):
        # Validation immédiate (Fail Fast)
        if not (1 <= access_level <= 5):
            raise ValueError(f"Niveau invalide ({access_level}). Doit être entre 1 et 5.")
        
        self.owner = owner
        self.access_level = access_level
        print(f"Badge créé pour {self.owner} (Niveau {self.access_level})")

# Test Cas Valide
try:
    b1 = SecurityBadge("Alice", 3)
except ValueError as e:
    print(e)

# Test Cas Invalide
try:
    b2 = SecurityBadge("Bob", 0)
except ValueError as e:
    print(f"Erreur de création : {e}")
```
</details>

### Exercice 2 - Le Compte à Rebours (Cycle de Vie) {#exercice-2---compte-a-rebours}

🎯 **Objectif** : Visualiser le moment exact de la destruction.

💼 **Mise en situation** : Une sonde spatiale envoie un signal lors de son lancement et un signal de détresse lors de sa destruction.

📝 **Énoncé** :
1.  Créez une classe `Probe`.
2.  `__init__` : Affiche "Sonde [id] lancée".
3.  `__del__` : Affiche "Sonde [id] détruite - Signal perdu".
4.  Créez une fonction `mission()` qui instancie une sonde et se termine.
5.  Appelez `mission()` et observez que la sonde est détruite à la fin de la fonction.
6.  Créez une sonde globale `voyager` en dehors de la fonction et supprimez-la manuellement avec `del`.

📺 **Résultat attendu** :
```text
--- Début Mission ---
Sonde Mars-1 lancée
Fin de la fonction mission
Sonde Mars-1 détruite - Signal perdu
--- Fin Mission ---
Sonde Voyager lancée
Sonde Voyager détruite - Signal perdu
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
class Probe:
    def __init__(self, mission_id: str):
        self.mission_id = mission_id
        print(f"Sonde {self.mission_id} lancée")

    def __del__(self):
        print(f"Sonde {self.mission_id} détruite - Signal perdu")

def mission():
    print("--- Début Mission ---")
    p = Probe("Mars-1")
    # p est une variable locale
    print("Fin de la fonction mission")
    # À la sortie, p est nettoyée

mission()
print("--- Fin Mission ---")

# Variable globale
voyager = Probe("Voyager")
# Suppression manuelle
del voyager
```
</details>

### Exercice 3 - Le Pool de Connexions (Mock) {#exercice-3---pool-connexions}

🎯 **Objectif** : Gestion de ressources simulée (Compteur de classe).

💼 **Mise en situation** : Vous limitez le nombre de connexions simultanées à votre base de données.

📝 **Énoncé** :
1.  Créez une classe `DatabaseConnection`.
2.  Ajoutez un attribut de classe `active_connections = 0`.
3.  Dans `__init__`, incrémentez ce compteur. Affichez le nombre total de connexions.
4.  Dans `__del__`, décrémentez ce compteur. Affichez le nombre restant.
5.  Simulez l'ouverture de 3 connexions, puis la fermeture de 2.

📺 **Résultat attendu** :
```text
Connexion ouverte. Actives : 1
Connexion ouverte. Actives : 2
Connexion ouverte. Actives : 3
Fermeture connexion. Restantes : 2
Fermeture connexion. Restantes : 1
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
class DatabaseConnection:
    # Attribut de classe partagé par toutes les instances
    active_connections: int = 0

    def __init__(self):
        DatabaseConnection.active_connections += 1
        print(f"Connexion ouverte. Actives : {DatabaseConnection.active_connections}")

    def __del__(self):
        DatabaseConnection.active_connections -= 1
        print(f"Fermeture connexion. Restantes : {DatabaseConnection.active_connections}")

# Simulation
c1 = DatabaseConnection() # 1
c2 = DatabaseConnection() # 2
c3 = DatabaseConnection() # 3

# Suppression explicite pour l'exercice
del c1 # Reste 2
del c2 # Reste 1
# c3 sera détruit à la fin du script
```
</details>