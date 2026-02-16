---
sidebar_label: Structures de Contrôle : Boucles While
sidebar_position: 10
---

# Chapitre 10 : Structures de Contrôle : Boucles While

Condition de boucle, Boucles infinies, Break et continue

La boucle `for` est idéale quand vous connaissez le nombre d'itérations ou la collection à parcourir. Mais que faire si vous ne savez pas combien de fois vous devez répéter une action ? Par exemple : "Tant que l'utilisateur n'a pas deviné le mot de passe" ou "Tant que la batterie est au-dessus de 5%".

C'est ici qu'intervient la boucle `while` ("Tant que"). C'est une boucle basée sur une **condition** plutôt que sur un compteur.

---

## 1. La Boucle `while` : Principes et Syntaxe {#la-boucle-while}

### 1. Quoi
Une structure qui répète un bloc de code tant qu'une condition booléenne reste `True`. L'évaluation de la condition se fait *avant* chaque tour de boucle.

### 2. Pourquoi
Pour gérer des processus dont la durée ou le nombre d'étapes est imprévisible (attente réseau, saisie utilisateur, convergence mathématique).

### 3. Comment

#### A. Syntaxe de base

```python
battery: int = 20

# Tant que la batterie est positive
while battery > 0:
    print(f"Batterie : {battery}%")
    battery -= 5 # Décrémentation essentielle !

print("Arrêt du système.")
```

#### B. Cas concret : Saisie utilisateur robuste
On force l'utilisateur à entrer une donnée valide.

```python
age: int = 0

# Tant que l'âge est invalide (exemple : mineur ou valeur absurde)
while age < 18:
    user_input: str = "20" # Simulation d'une saisie via input()
    age = int(user_input)
    
    if age < 18:
        print("Accès refusé. Vous devez être majeur.")

print(f"Bienvenue, utilisateur de {age} ans.")
```

### 4. Zone de Danger
❌ **Oublier la mise à jour de la condition** : Si la variable testée dans le `while` ne change jamais à l'intérieur de la boucle, vous créez une **boucle infinie** qui plantera votre programme (ou le rendra inarrêtable).

```python
count = 0
while count < 5:
    print(count)
    # count += 1  <-- Si cette ligne manque, boucle infinie de 0 !
```

---

## 2. Boucles Infinies Intentionnelles {#boucles-infinies-intentionnelles}

### 1. Quoi
Une boucle `while True:` qui ne s'arrête jamais d'elle-même. Elle doit contenir une instruction `break` interne pour en sortir.

### 2. Pourquoi
C'est le pattern standard pour les serveurs, les interfaces en ligne de commande (CLI), ou les jeux : "Fais tourner le programme pour toujours, jusqu'à ce que l'utilisateur tape 'Quitter'".

### 3. Comment

#### A. Le pattern "While True"

```python
is_running: bool = True

while True:
    command: str = "start" # Simulation input
    
    if command == "quit":
        print("Fermeture...")
        break # Seul moyen de sortir
        
    print(f"Exécution de : {command}")
    
    # Pour l'exemple, on force le break pour ne pas bloquer
    break 
```

#### B. Watchdog Timer (Sécurité)
Dans les systèmes critiques, on ajoute souvent un compteur de sécurité pour éviter le blocage éternel.

```python
attempts: int = 0
max_attempts: int = 3
connected: bool = False

while not connected:
    attempts += 1
    print(f"Tentative de connexion {attempts}...")
    
    # Simulation succès aléatoire
    if attempts == 2:
        connected = True
        print("Connecté !")
    
    if attempts >= max_attempts:
        print("Échec critique : Trop de tentatives.")
        break
```

---

## 3. Contrôle Avancé : `continue` et `else` {#controle-avance}

### 1. Quoi
Comme pour la boucle `for`, `while` supporte :
*   `continue` : Revenir immédiatement au début de la boucle (et réévaluer la condition).
*   `else` : S'exécuter si la boucle se termine *naturellement* (condition devenue fausse), mais PAS via un `break`.

### 2. Pourquoi
Pour sauter des étapes inutiles ou gérer des cas de succès/échec après une recherche.

### 3. Comment

#### A. Exemple avec `continue`

```python
x: int = 10

while x > 0:
    x -= 1
    
    # On ne veut pas afficher les nombres impairs
    if x % 2 != 0:
        continue 
        
    print(f"Nombre pair : {x}")
```

#### B. Exemple avec `else` (Le chercheur)

```python
n: int = 5
found: bool = False

# Cherchons si n est divisible par un nombre entre 2 et n-1 (nombre premier ?)
divisor: int = 2

while divisor < n:
    if n % divisor == 0:
        print(f"{n} n'est pas premier (divisible par {divisor})")
        break
    divisor += 1
else:
    # Ce bloc s'exécute SI le while s'est terminé car divisor == n
    # et JAMAIS via le break
    print(f"{n} est un nombre premier !")
```

### 🚨 Limitations de `while`
Le risque d'erreur logique (boucle infinie) est beaucoup plus élevé avec `while` qu'avec `for`.
👉 **Règle d'or** : Si vous pouvez utiliser `for` (itération sur une liste, range), préférez toujours `for`. Gardez `while` pour les conditions indéterminées.

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-10}

1.  **Quelle est la condition principale pour utiliser une boucle `while` plutôt qu'une boucle `for` ?**
    Quand on ne connaît pas à l'avance le nombre d'itérations nécessaires (condition d'arrêt dynamique).

2.  **Comment arrêter une boucle `while True` ?**
    En utilisant le mot-clé `break` à l'intérieur d'une condition `if`.

3.  **Que se passe-t-il si j'oublie d'incrémenter mon compteur dans une boucle `while` ?**
    Vous créez une boucle infinie (infinite loop) qui peut bloquer votre programme.

4.  **Quand le bloc `else` d'une boucle `while` est-il exécuté ?**
    Uniquement si la boucle se termine parce que la condition est devenue `False`. Il ne s'exécute pas si on sort via un `break`.

---

## Exercices : {#exercices-10}

### Exercice 1 - La Tirelire Électronique {#exercice-1---tirelire}

🎯 **Objectif** : Gérer une condition de sortie numérique.

💼 **Mise en situation** : Vous économisez pour un vélo à 500€. Vous ajoutez de l'argent chaque mois jusqu'à atteindre l'objectif.

📝 **Énoncé** :
1.  Variable `savings` = 0.
2.  Variable `goal` = 500.
3.  Tant que `savings` est inférieur à `goal` :
    - Ajoutez 50€ (simulation dépôt).
    - Affichez "Économies : X€".
4.  Une fois la boucle finie, affichez "Objectif atteint !".

📺 **Résultat attendu** :
```text
Économies : 50€
...
Économies : 500€
Objectif atteint !
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
savings: int = 0
goal: int = 500

while savings < goal:
    savings += 50
    print(f"Économies : {savings}€")

print("Objectif atteint !")
```
</details>

### Exercice 2 - Menu Interactif (Simulation) {#exercice-2---menu-interactif}

🎯 **Objectif** : Pattern `while True` avec `input` (simulé) et `break`.

💼 **Mise en situation** : Créer un menu de CLI (Command Line Interface) qui demande une action à l'utilisateur.

📝 **Énoncé** :
1.  Liste d'actions simulées : `actions = ["profil", "messages", "quitter"]`.
2.  Utilisez `actions.pop(0)` pour simuler la saisie utilisateur à chaque tour.
3.  Tant que Vrai :
    - Si action == "quitter", afficher "Au revoir" et break.
    - Si action == "profil", afficher "Votre profil...".
    - Sinon, afficher "Menu : [action]".

📺 **Résultat attendu** :
```text
Votre profil...
Menu : messages
Au revoir
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
# Simulation des entrées utilisateur
actions_queue: list[str] = ["profil", "messages", "quitter"]

while True:
    # Simulation input() : on prend le premier élément
    # Dans un vrai programme : user_choice = input("Choix ? ")
    if not actions_queue: 
        break # Sécurité si liste vide
        
    user_choice: str = actions_queue.pop(0)
    
    if user_choice == "quitter":
        print("Au revoir")
        break
    elif user_choice == "profil":
        print("Votre profil...")
    else:
        print(f"Menu : {user_choice}")
```
</details>

### Exercice 3 - Le Validateur de Mot de Passe {#exercice-3---validateur-mdp}

🎯 **Objectif** : Utiliser `continue` pour forcer une condition.

💼 **Mise en situation** : Un système demande un mot de passe. Si le mot de passe est trop court (< 4 caractères), on redemande sans vérifier le reste.

📝 **Énoncé** :
1.  Liste de tentatives : `attempts = ["123", "abc", "secret123"]`.
2.  Tant qu'il y a des tentatives :
    - Récupérez une tentative.
    - Si longueur < 4 : Affichez "Trop court" et `continue`.
    - Si longueur >= 4 : Affichez "Mot de passe accepté" et `break`.

📺 **Résultat attendu** :
```text
Tentative '123' : Trop court
Tentative 'abc' : Trop court
Mot de passe accepté (secret123)
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
attempts_queue: list[str] = ["123", "abc", "secret123"]

while attempts_queue:
    pwd: str = attempts_queue.pop(0)
    
    if len(pwd) < 4:
        print(f"Tentative '{pwd}' : Trop court")
        continue # On passe directement à l'itération suivante
    
    # Ce code n'est atteint que si len >= 4
    print(f"Mot de passe accepté ({pwd})")
    break # Sortie définitive
```
</details>