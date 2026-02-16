---
sidebar_label: Module `functools` : Outils d'Ordre Supérieur
sidebar_position: 31
---

# Chapitre 31 : Module `functools` : Outils d'Ordre Supérieur

partial, reduce, lru_cache, wraps (pour les décorateurs)

Le module `functools` est la boîte à outils des programmeurs Python qui aiment manipuler les fonctions comme des objets. Il propose des **fonctions d'ordre supérieur** : des fonctions qui agissent sur ou renvoient d'autres fonctions.

Ces outils permettent de simplifier le code, d'améliorer les performances sans effort et de créer des décorateurs robustes. C'est un passage obligé pour maîtriser le style fonctionnel en Python.

---

## 1. Application Partielle : `partial` {#application-partielle-partial}

### 1. Quoi
La fonction `partial` permet de créer une nouvelle version d'une fonction existante en **pré-remplissant** certains de ses arguments. On dit qu'on "gèle" une partie des paramètres.

### 2. Pourquoi
Pour réutiliser une fonction générique dans un contexte spécifique sans avoir à réécrire une fonction wrapper (`def ...: return ...`). C'est très utile pour les callbacks (GUI, asynchrone) ou pour simplifier des signatures de fonctions complexes.

### 3. Comment

#### A. Syntaxe de base

```python
from functools import partial

def power(base, exponent):
    return base ** exponent

# On crée une nouvelle fonction 'square' où exponent est figé à 2
square = partial(power, exponent=2)

# On crée une fonction 'cube'
cube = partial(power, exponent=3)

print(square(10)) # 100
print(cube(10))   # 1000
```

#### B. Cas concret : Configuration de logs
Imaginez une fonction de log qui prend un niveau et un message. Vous voulez des raccourcis pour `info` et `error`.

```python
import sys
from functools import partial

def log_message(level, message, stream=sys.stdout):
    stream.write(f"[{level.upper()}] {message}\n")

# Création de fonctions spécialisées
log_info = partial(log_message, level="info")
log_error = partial(log_message, level="error", stream=sys.stderr)

# Utilisation simplifiée
log_info(message="Serveur démarré")
log_error(message="Connexion échouée")
```

### 4. Zone de Danger
❌ **Arguments positionnels vs nommés** :
Si vous "gèlez" le premier argument positionnel avec `partial(func, 10)`, vous ne pourrez plus passer cet argument par mot-clé lors de l'appel final sans risquer un conflit ou une erreur.
✅ **Bonne Pratique** : Privilégiez les arguments nommés (`keyword arguments`) lors de l'utilisation de `partial` pour éviter toute ambiguïté.

---

## 2. Réduction de Données : `reduce` {#reduction-de-donnees-reduce}

### 1. Quoi
`reduce` prend une fonction (à deux arguments) et une liste. Elle applique la fonction cumulativement aux éléments, de gauche à droite, pour réduire la liste à une **valeur unique**.

### 2. Pourquoi
Pour transformer une collection en un seul résultat : somme, produit, concaténation, fusion de dictionnaires, etc. Bien que `sum()` existe pour l'addition, `reduce` est l'outil générique.

### 3. Comment

#### A. Syntaxe de base

```python
from functools import reduce

numbers = [1, 2, 3, 4]

# Calcule (((1 * 2) * 3) * 4)
product = reduce(lambda x, y: x * y, numbers)

print(product) # 24
```

#### B. Cas concret : Chemin imbriqué dans un dictionnaire
Récupérer une valeur profonde `config["database"]["host"]["ip"]` sans planter si une clé manque.

```python
from functools import reduce

data = {
    "database": {
        "host": {
            "ip": "192.168.1.10",
            "port": 5432
        }
    }
}

path = ["database", "host", "ip"]

# On "plonge" dans le dictionnaire niveau par niveau
# get() renvoie {} par défaut si la clé manque, évitant le crash
ip_address = reduce(lambda d, key: d.get(key, {}), path, data)

print(f"IP: {ip_address}") 
# IP: 192.168.1.10
```

### 🚨 Limitations de `reduce`
`reduce` est souvent moins lisible qu'une boucle `for` explicite. Guido van Rossum (créateur de Python) a même retiré `reduce` des built-ins en Python 3 pour le mettre dans `functools`.
Utilisez-le si la logique est mathématique ou conceptuellement une "réduction". Sinon, une boucle est souvent préférable.

---

## 3. Mise en Cache : `lru_cache` {#mise-en-cache-lru-cache}

### 1. Quoi
`@lru_cache` (Least Recently Used Cache) est un décorateur qui **mémorise** les résultats d'une fonction en fonction de ses arguments. Si on rappelle la fonction avec les mêmes arguments, elle renvoie le résultat stocké sans recalculer.

### 2. Pourquoi
*   **Performance** : Accélère drastiquement les fonctions coûteuses (calculs lourds, appels API, récursivité).
*   **Simplicité** : Ajoute du caching en une seule ligne (`@`).

### 3. Comment

#### A. Syntaxe de base

```python
from functools import lru_cache
import time

# maxsize=None : cache illimité
# maxsize=128 : garde les 128 derniers appels (par défaut)
@lru_cache(maxsize=32)
def heavy_computation(x):
    print(f"Calcul pour {x}...")
    time.sleep(1) # Simulation charge
    return x * x

print(heavy_computation(4)) # Prend 1 seconde + affiche "Calcul..."
print(heavy_computation(4)) # Instantané ! (Pas d'affichage)
```

#### B. Cas concret : Suite de Fibonacci (Récursif)
Sans cache, la complexité est exponentielle. Avec cache, elle devient linéaire.

```python
@lru_cache(maxsize=None)
def fib(n):
    if n < 2:
        return n
    return fib(n-1) + fib(n-2)

# Calcule instantanément même pour n=100
print(fib(50)) 
```

### 4. Zone de Danger
❌ **Arguments mutables** :
Les arguments de la fonction décorée DOIVENT être **hachables** (immuables). Vous ne pouvez pas passer une liste `[1, 2]` ou un dictionnaire à une fonction mise en cache. Utilisez des tuples.

---

## 4. Métadonnées de Décorateur : `wraps` {#metadonnees-de-decorateur-wraps}

### 1. Quoi
`@wraps` est un décorateur... pour créer des décorateurs ! Il copie les métadonnées de la fonction originale (nom, docstring, annotations) vers la fonction wrapper.

### 2. Pourquoi
Sans `wraps`, quand vous décorez une fonction `ma_fonction`, elle perd son identité et devient `wrapper`. Cela casse l'introspection, l'aide (`help()`) et certains outils de test.

### 3. Comment

#### A. Sans wraps (Le problème)

```python
def my_decorator(func):
    def wrapper(*args, **kwargs):
        """Docstring du wrapper"""
        return func(*args, **kwargs)
    return wrapper

@my_decorator
def add(a, b):
    """Ajoute deux nombres."""
    return a + b

print(add.__name__) # Affiche 'wrapper' 😱
print(add.__doc__)  # Affiche 'Docstring du wrapper' 😱
```

#### B. Avec wraps (La solution)

```python
from functools import wraps

def my_decorator_pro(func):
    @wraps(func) # ✅ Magie ici
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

@my_decorator_pro
def sub(a, b):
    """Soustrait deux nombres."""
    return a - b

print(sub.__name__) # Affiche 'sub'
print(sub.__doc__)  # Affiche 'Soustrait deux nombres.'
```

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-31}

1.  **Quel est l'avantage principal de `@lru_cache` ?**
    Il permet de mettre en cache automatiquement les résultats d'une fonction pour éviter de refaire des calculs coûteux si les arguments sont identiques.

2.  **Pourquoi `partial` est-il utile dans des frameworks comme GUI ou Réseau ?**
    Il permet de passer des fonctions "pré-configurées" (callbacks) qui ne prennent aucun argument lors de l'appel, alors qu'elles en nécessitent en réalité (arguments qui ont été "gelés" à la création).

3.  **Pourquoi doit-on toujours utiliser `@wraps` quand on écrit un décorateur ?**
    Pour préserver le nom (`__name__`) et la documentation (`__doc__`) de la fonction décorée, ce qui est crucial pour le débogage et la documentation automatique.

4.  **Quelle contrainte impose `@lru_cache` sur les arguments de la fonction ?**
    Les arguments doivent être **hachables** (immuables). Pas de listes ni de dictionnaires en entrée.

---

## Exercices : {#exercices-31}

### Exercice 1 - Le Convertisseur de Devises Intelligent {#exercice-1-convertisseur}

🎯 **Objectif** : Utiliser `partial` pour créer des convertisseurs spécifiques.

💼 **Mise en situation** : Vous travaillez sur un site e-commerce international. Vous avez une fonction générique de conversion, mais vous voulez fournir à vos collègues des fonctions simples `to_usd`, `to_eur`, `to_jpy`.

📝 **Énoncé** :
1.  Créez une fonction `convert(amount, rate)` qui renvoie `amount * rate`.
2.  Utilisez `partial` pour créer trois fonctions :
    - `to_usd` (taux 1.1)
    - `to_gbp` (taux 0.85)
    - `to_jpy` (taux 130)
3.  Testez-les avec un montant de 100€.

📺 **Résultat attendu** :
```text
100€ -> 110.0 USD
100€ -> 85.0 GBP
100€ -> 13000 JPY
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
from functools import partial

# Fonction générique
def convert(amount, rate):
    return amount * rate

# Création des fonctions spécialisées
# On fige l'argument 'rate'
to_usd = partial(convert, rate=1.1)
to_gbp = partial(convert, rate=0.85)
to_jpy = partial(convert, rate=130)

# Tests
amount = 100
print(f"{amount}€ -> {to_usd(amount)} USD")
print(f"{amount}€ -> {to_gbp(amount)} GBP")
print(f"{amount}€ -> {to_jpy(amount)} JPY")
```
</details>

### Exercice 2 - Optimisation d'API Simulateur {#exercice-2-cache-api}

🎯 **Objectif** : Utiliser `@lru_cache` pour optimiser.

💼 **Mise en situation** : Vous interrogez une API de météo payante et lente. Pour économiser des crédits et du temps, vous ne voulez pas refaire la même requête deux fois.

📝 **Énoncé** :
1.  Créez une fonction `get_weather(city)` qui :
    - Affiche "📡 Requête API pour [city]..."
    - Simule une attente de 1 seconde (`time.sleep`).
    - Renvoie une température fictive (basée sur la longueur du nom de la ville par exemple).
2.  Décorez cette fonction avec `@lru_cache`.
3.  Appelez la fonction 3 fois : "Paris", "London", "Paris".
4.  Vérifiez que "Paris" n'a déclenché le message "Requête API" qu'une seule fois.

📺 **Résultat attendu** :
```text
📡 Requête API pour Paris...
Température Paris : 5°C
📡 Requête API pour London...
Température London : 6°C
Température Paris : 5°C  <-- Instantané, pas de log "Requête"
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
import time
from functools import lru_cache

# Le cache gardera jusqu'à 10 villes en mémoire
@lru_cache(maxsize=10)
def get_weather(city):
    print(f"📡 Requête API pour {city}...")
    time.sleep(1) # Simulation latence
    return len(city) # Température arbitraire

# Premier appel : lent
print(f"Température Paris : {get_weather('Paris')}°C")

# Deuxième appel (autre argument) : lent
print(f"Température London : {get_weather('London')}°C")

# Troisième appel (argument déjà vu) : instantané + pas de print
start = time.time()
print(f"Température Paris : {get_weather('Paris')}°C")
print(f"Temps écoulé : {time.time() - start:.4f}s")
```
</details>

### Exercice 3 - Le Validateur Universel {#exercice-3-reduce-validator}

🎯 **Objectif** : Utiliser `reduce` pour combiner des booléens.

💼 **Mise en situation** : Un formulaire d'inscription doit passer une série de validateurs (Email valide ? Mot de passe fort ? Âge légal ?). Si TOUS les validateurs sont True, l'inscription est validée.

📝 **Énoncé** :
1.  Liste de règles (fonctions lambdas ou normales) qui prennent un dictionnaire `user` et renvoient `True`/`False`.
    - Règle 1 : age >= 18
    - Règle 2 : len(password) >= 8
    - Règle 3 : "@" in email
2.  Un utilisateur test : `{'age': 20, 'password': 'secret123', 'email': 'bob@mail.com'}`.
3.  Utilisez `reduce` pour appliquer toutes les règles. (Astuce: `True and True` donne `True`).
4.  Affichez si l'utilisateur est valide.

📺 **Résultat attendu** :
```text
Utilisateur valide : True
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
from functools import reduce

# Données
user_valid = {'age': 20, 'password': 'secret123', 'email': 'bob@mail.com'}
user_invalid = {'age': 15, 'password': '123', 'email': 'bob'}

# Liste de fonctions de validation
rules = [
    lambda u: u['age'] >= 18,
    lambda u: len(u['password']) >= 8,
    lambda u: "@" in u['email']
]

def check_user(user):
    # reduce applique 'and' entre le résultat accumulé et le résultat de la règle courante
    # Initialiseur = True (neutre pour AND)
    return reduce(lambda acc, rule: acc and rule(user), rules, True)

print(f"Utilisateur valide : {check_user(user_valid)}")   # True
print(f"Utilisateur valide : {check_user(user_invalid)}") # False
```
</details>