---
sidebar_label: Décorateurs
sidebar_position: 41
---

# Chapitre 41 : Décorateurs

Fonctions comme objets, Décorateurs de fonctions, Décorateurs avec arguments, Mot-clé @

Les décorateurs sont l'une des fonctionnalités les plus puissantes et élégantes de Python. Ils permettent de **modifier le comportement d'une fonction ou d'une classe** de manière dynamique, sans toucher à son code source. C'est le principe du design pattern *Decorator*.

Si vous avez déjà utilisé `@property`, `@staticmethod` ou `@login_required` dans un framework Web, vous avez utilisé des décorateurs. Ce chapitre va vous apprendre à créer les vôtres pour écrire du code plus propre, réutilisable et modulaire.

---

## 1. Les Fonctions comme Objets (First-Class Citizens) {#fonctions-comme-objets}

### 1. Quoi
En Python, tout est objet, y compris les fonctions. Cela signifie qu'une fonction peut être :
*   Assignée à une variable.
*   Passée comme argument à une autre fonction.
*   Retournée par une autre fonction.

### 2. Pourquoi
C'est le mécanisme fondamental qui rend les décorateurs possibles. Un décorateur est simplement une fonction qui prend une fonction en entrée et retourne une nouvelle fonction en sortie.

### 3. Comment

```python
def crier(texte: str) -> str:
    return texte.upper() + " !!!"

def chuchoter(texte: str) -> str:
    return texte.lower() + "..."

# 1. Fonction passée en argument (High-Order Function)
def saluer(func, nom: str):
    message = func(f"Bonjour {nom}")
    print(message)

saluer(crier, "Alice")    # BONJOUR ALICE !!!
saluer(chuchoter, "Bob")  # bonjour bob...

# 2. Fonction retournée par une fonction (Closure)
def generateur_de_puissance(exposant: int):
    def puissance(x: int) -> int:
        return x ** exposant
    return puissance

carre = generateur_de_puissance(2)
cube = generateur_de_puissance(3)

print(carre(5)) # 25
print(cube(5))  # 125
```

---

## 2. Décorateurs de Fonctions (Syntaxe @) {#decorateurs-de-fonctions}

### 1. Quoi
Un décorateur est une fonction qui "enveloppe" une autre fonction pour exécuter du code **avant** et **après** son appel, sans modifier la fonction originale.
Le mot-clé `@mon_decorateur` est un sucre syntaxique (raccourci) pour `ma_fonction = mon_decorateur(ma_fonction)`.

### 2. Pourquoi
Pour factoriser du code transversal (Cross-Cutting Concerns) :
*   Logging (tracer les appels).
*   Timing (mesurer les performances).
*   Contrôle d'accès (authentification).
*   Mise en cache.

### 3. Comment

#### A. Syntaxe de base (Le Wrapper)

```python
import functools

def mon_decorateur(fonction_originale):
    
    # functools.wraps est crucial : il préserve le nom et la docstring
    # de la fonction originale, sinon ils seraient remplacés par ceux de 'wrapper'
    @functools.wraps(fonction_originale)
    def wrapper(*args, **kwargs):
        print("🔔 Avant l'exécution")
        
        resultat = fonction_originale(*args, **kwargs)
        
        print("🔔 Après l'exécution")
        return resultat
    
    return wrapper

# Utilisation
@mon_decorateur
def dire_bonjour():
    print("Bonjour le monde !")

dire_bonjour()
# Sortie :
# 🔔 Avant l'exécution
# Bonjour le monde !
# 🔔 Après l'exécution
```

#### B. Cas concret : Chronomètre (Benchmark)

```python
import time
import functools

def timer(func):
    """Décorateur pour mesurer le temps d'exécution d'une fonction."""
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start_time = time.perf_counter() # Démarrage
        
        value = func(*args, **kwargs)    # Exécution réelle
        
        end_time = time.perf_counter()   # Fin
        run_time = end_time - start_time
        print(f"⏱️ {func.__name__} exécutée en {run_time:.4f} secs")
        return value
    return wrapper

@timer
def heavy_computation(n):
    total = sum(i**2 for i in range(n))
    return total

res = heavy_computation(1_000_000)
# ⏱️ heavy_computation exécutée en 0.2543 secs
```

### 4. Zone de Danger
❌ **Oublier `*args` et `**kwargs`** : Si votre wrapper ne les accepte pas, vous ne pourrez décorer que des fonctions sans arguments.
✅ **Toujours utiliser `@functools.wraps`** : Sinon, vos fonctions décorées perdront leur identité (nom, docstring), ce qui rend le débogage cauchemardesque.

---

## 3. Décorateurs avec Arguments {#decorateurs-avec-arguments}

### 1. Quoi
Parfois, on veut configurer le décorateur lui-même. Par exemple `@retry(times=3)` ou `@route("/home")`. Pour cela, il faut ajouter un niveau d'imbrication supplémentaire : une fonction qui retourne le décorateur.

### 2. Pourquoi
Pour rendre vos décorateurs génériques et paramétrables.

### 3. Comment

#### A. Syntaxe (La poupée russe : 3 niveaux)

```python
import functools
import time

def repeat(num_times):
    """Niveau 1 : Reçoit les arguments du décorateur."""
    
    def decorator_repeat(func):
        """Niveau 2 : Reçoit la fonction à décorer."""
        
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            """Niveau 3 : Le wrapper qui s'exécute à l'appel."""
            for _ in range(num_times):
                value = func(*args, **kwargs)
            return value
            
        return wrapper
        
    return decorator_repeat

@repeat(num_times=3)
def greet(name):
    print(f"Hello {name}")

greet("Python")
# Hello Python
# Hello Python
# Hello Python
```

### 🚨 Limitations
La complexité cognitive augmente vite avec 3 niveaux de fonctions imbriquées. Si votre décorateur devient trop complexe, envisagez d'utiliser une **classe décorateur** (concept avancé) pour plus de clarté.

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-41}

1.  **Pourquoi utilise-t-on `*args` et `**kwargs` dans la fonction wrapper ?**
    Pour que le décorateur puisse accepter n'importe quelle fonction, quel que soit le nombre ou le type de ses arguments, et les transmettre intacts à la fonction originale.

2.  **À quoi sert `@functools.wraps(func)` ?**
    Il copie les métadonnées de la fonction originale (nom, docstring) vers la fonction wrapper. Sans cela, `help(ma_fonction_decoree)` afficherait l'aide du wrapper, ce qui est déroutant.

3.  **Quelle est la différence entre `@mon_deco` et `@mon_deco(arg)` ?**
    `@mon_deco` appelle `mon_deco` avec la fonction cible comme argument unique.
    `@mon_deco(arg)` appelle d'abord `mon_deco(arg)` qui *doit retourner* un véritable décorateur, qui sera ensuite appliqué à la fonction cible.

4.  **L'ordre des décorateurs importe-t-il ?**
    Oui ! Les décorateurs sont appliqués de bas en haut (le plus proche de la fonction `def` en premier).
    ```python
    @A
    @B
    def f(): ...
    ```
    Équivaut à `f = A(B(f))`.

---

## Exercices : {#exercices-41}

### Exercice 1 - Le Contrôleur d'Accès (Mock Auth) {#exercice-1-controleur-acces}

🎯 **Objectif** : Créer un décorateur de sécurité simple.

💼 **Mise en situation** : Vous développez une API interne. Certaines fonctions administratives ne doivent être exécutées que si l'utilisateur courant a le rôle "admin".

📝 **Énoncé** :
1.  Simulez une variable globale `CURRENT_USER = {"username": "guest", "role": "user"}`.
2.  Créez un décorateur `@admin_required`.
3.  Si le rôle de `CURRENT_USER` n'est pas "admin", le décorateur doit lever une `PermissionError` avec le message "Accès refusé".
4.  Sinon, la fonction s'exécute normalement.
5.  Testez avec une fonction `delete_database()` et changez le rôle de l'utilisateur pour vérifier les deux cas.

📺 **Résultat attendu** :
```text
Cas User : Exception levée : Accès refusé
Cas Admin : Base de données supprimée !
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
import functools

# Simulation du contexte utilisateur
CURRENT_USER = {"username": "guest", "role": "user"}

def admin_required(func):
    """Décorateur vérifiant si l'utilisateur est admin."""
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        # Vérification avant exécution
        if CURRENT_USER.get("role") != "admin":
            raise PermissionError("⛔ Accès refusé : Admin requis.")
        
        # Exécution si tout va bien
        return func(*args, **kwargs)
    return wrapper

@admin_required
def delete_database():
    print("🔥 Base de données supprimée !")

# Test 1 : Échec attendu
try:
    delete_database()
except PermissionError as e:
    print(f"Test 1 : {e}")

# Test 2 : Succès
CURRENT_USER["role"] = "admin"
print("Test 2 : ", end="")
delete_database()
```
</details>

### Exercice 2 - Le Décorateur Cache (Mémoïsation) {#exercice-2-decorateur-cache}

🎯 **Objectif** : Optimiser une fonction récursive lente.

💼 **Mise en situation** : Vous calculez des suites mathématiques lourdes. Pour éviter de recalculer sans cesse les mêmes valeurs, vous voulez mettre en cache les résultats.

📝 **Énoncé** :
1.  Créez un décorateur `@memoize`.
2.  Il doit utiliser un dictionnaire interne `cache = {}` pour stocker les résultats. La clé sera les arguments de la fonction `args`.
3.  Appliquez-le à une fonction `fib(n)` (Suite de Fibonacci) qui est notoirement lente sans cache.
4.  Comparez (visuellement ou mentalement) la vitesse pour `fib(35)`.

📺 **Résultat attendu** :
```text
Calcul de fib(35) instantané avec le cache.
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
import functools

def memoize(func):
    """Cache simple pour stocker les résultats des appels précédents."""
    cache = {}
    
    @functools.wraps(func)
    def wrapper(*args):
        # Si le résultat est déjà connu, on le retourne direct
        if args in cache:
            return cache[args]
        
        # Sinon, on calcule
        result = func(*args)
        
        # On stocke pour la prochaine fois
        cache[args] = result
        return result
        
    return wrapper

@memoize
def fib(n):
    if n < 2:
        return n
    return fib(n-1) + fib(n-2)

# Sans cache, fib(35) prendrait plusieurs secondes
print(f"Fib(35) = {fib(35)}") 
# Calcul instantané car tous les sous-calculs (fib(34), fib(33)...) sont cachés
```
</details>

### Exercice 3 - Validation de Types (Type Check Runtime) {#exercice-3-validation-types}

🎯 **Objectif** : Utiliser un décorateur pour forcer le type des arguments.

💼 **Mise en situation** : Python est dynamiquement typé, mais pour une fonction critique de calcul financier, vous voulez interdire impérativement tout argument qui n'est pas un `int` ou un `float`.

📝 **Énoncé** :
1.  Créez un décorateur `@validate_numeric`.
2.  Il doit parcourir tous les `args` fournis à la fonction.
3.  Si un argument n'est ni `int` ni `float`, levez `TypeError`.
4.  Appliquez-le à `add(a, b)`.

📺 **Résultat attendu** :
```text
add(10, 20.5) -> 30.5
add(10, "20") -> TypeError: Argument '20' invalide.
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
import functools

def validate_numeric(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        # Vérification des arguments positionnels
        for arg in args:
            if not isinstance(arg, (int, float)):
                raise TypeError(f"Argument invalide : {arg} ({type(arg).__name__}). Attendu : int ou float.")
        
        # Note : On pourrait aussi vérifier kwargs ici
        
        return func(*args, **kwargs)
    return wrapper

@validate_numeric
def add(a, b):
    return a + b

print(f"Succès : {add(10, 20.5)}")

try:
    add(10, "vingt")
except TypeError as e:
    print(f"Erreur attrapée : {e}")
```
</details>