---
sidebar_label: Fonctions : Portée des Variables et Closures
sidebar_position: 16
---

# Chapitre 16 : Fonctions : Portée des Variables et Closures

Portée locale et globale (LEGB), Mot-clé nonlocal, Fonctions imbriquées, Closures

Imaginez une grande entreprise où tout le monde, du stagiaire au PDG, partageait le même bureau ouvert et le même tableau blanc. Si le stagiaire efface par erreur le plan stratégique du PDG pour y écrire sa liste de courses, c'est le chaos.

En programmation, c'est pareil. Pour éviter le chaos des variables, Python utilise des **portées** (scopes). Une variable créée dans une fonction est "privée" à cette fonction. Mais Python permet aussi des mécanismes puissants comme les **closures**, où une fonction "se souvient" de l'environnement dans lequel elle a été créée, même après la fin de son exécution.

---

## 1. La Règle LEGB (Portée des Variables) {#regle-legb}

### 1. Quoi
La règle **LEGB** définit l'ordre dans lequel Python cherche une variable lorsqu'il rencontre un nom.
*   **L**ocal : Dans la fonction actuelle.
*   **E**nclosing : Dans la fonction englobante (si fonctions imbriquées).
*   **G**lobal : Au niveau du module (fichier).
*   **B**uilt-in : Dans les fonctions natives de Python (`print`, `len`, etc.).

### 2. Pourquoi
Pour éviter les conflits de noms ("shadowing") et protéger l'intégrité des données. Cela permet d'avoir une variable `count` dans une fonction `A` et une autre variable `count` dans une fonction `B` sans qu'elles interfèrent.

### 3. Comment

#### A. Exemple de conflit (Shadowing)

```python
x = "Global"

def outer():
    x = "Enclosing"
    
    def inner():
        x = "Local"
        print(f"Inner voit : {x}") # Cherche Local -> Trouvé
    
    inner()
    print(f"Outer voit : {x}") # Cherche Local -> Trouvé "Enclosing"

outer()
print(f"Module voit : {x}") # Cherche Global -> Trouvé "Global"
```

#### B. Accès Global (`global`)
Pour modifier une variable globale depuis une fonction locale (déconseillé mais possible).

```python
score = 0

def update_score():
    global score # On dit à Python d'utiliser la variable du module
    score += 10

update_score()
print(score) # 10
```

### 4. Zone de Danger
❌ **Abus de `global`** : L'utilisation excessive de variables globales rend le code imprévisible et difficile à tester. Préférez passer des arguments et retourner des valeurs.

---

## 2. Fonctions Imbriquées (Nested Functions) {#fonctions-imbriquees}

### 1. Quoi
Définir une fonction **à l'intérieur** d'une autre fonction.

### 2. Pourquoi
*   **Encapsulation** : Cacher une fonction utilitaire qui n'a pas d'utilité en dehors de sa fonction parente.
*   **Closures** : Créer des "usines de fonctions" (voir section suivante).

### 3. Comment

#### A. Syntaxe de base

```python
def process_data(data: list[int]):
    # Fonction helper invisible depuis l'extérieur
    def is_valid(n: int) -> bool:
        return n > 0 and n % 2 == 0
    
    # Utilisation interne
    valid_data = [x for x in data if is_valid(x)]
    return valid_data

# is_valid(10) # ❌ NameError: name 'is_valid' is not defined
```

#### B. Mot-clé `nonlocal`
Permet à une fonction interne de modifier une variable de la fonction englobante (Enclosing scope), sans toucher à la globale.

```python
def compteur_externe():
    count = 0
    
    def increment():
        nonlocal count # Cible le 'count' de compteur_externe
        count += 1
        return count
    
    print(increment()) # 1
    print(increment()) # 2

compteur_externe()
```

---

## 3. Closures (Fermetures) {#closures}

### 1. Quoi
Une **closure** est une fonction interne renvoyée par sa fonction parente, qui a gardé en mémoire les variables de son environnement de création (le scope Enclosing), même après que la fonction parente ait terminé son exécution.

### 2. Pourquoi
*   **Persistance de l'état** : Alternative légère à la Programmation Orientée Objet (une classe avec une seule méthode).
*   **Configuration** : Créer des fonctions "sur mesure" à la volée.
*   **Décorateurs** : C'est le mécanisme de base des décorateurs (voir chapitre dédié).

### 3. Comment

#### A. La Fabrique de Fonctions (Exemple classique)

```python
def multiplier_factory(factor: int):
    # Cette fonction 'inner' capture la variable 'factor'
    def inner(number: int) -> int:
        return number * factor
    
    return inner # On retourne la fonction (l'objet), sans l'appeler !

# Création de fonctions spécialisées
doubler = multiplier_factory(2)
tripler = multiplier_factory(3)

# Utilisation
print(doubler(10)) # 20 (factor=2 est mémorisé)
print(tripler(10)) # 30 (factor=3 est mémorisé)
```

#### B. Cas concret : Configuration d'API

```python
def api_request_builder(base_url: str):
    def fetch(endpoint: str):
        full_url = f"{base_url}/{endpoint}"
        print(f"Fetching: {full_url}")
        # Simulation d'appel réseau...
    
    return fetch

# On configure deux clients différents
github_api = api_request_builder("https://api.github.com")
google_api = api_request_builder("https://api.google.com")

# Les clients se "souviennent" de leur base URL
github_api("users") # Fetching: https://api.github.com/users
google_api("maps")  # Fetching: https://api.google.com/maps
```

### 🚨 Limitations
Les variables capturées par la closure sont des **références**. Si l'objet capturé est mutable (comme une liste) et qu'on le modifie ailleurs, la closure verra le changement.

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-16}

1.  **Que signifie l'acronyme LEGB ?**
    Local, Enclosing, Global, Built-in. C'est l'ordre de recherche des variables par Python.

2.  **Quelle est la différence entre `global` et `nonlocal` ?**
    `global` permet de modifier une variable définie au niveau du module. `nonlocal` permet de modifier une variable définie dans la fonction parente immédiate (scope Enclosing).

3.  **Qu'est-ce qu'une closure ?**
    C'est une fonction qui "capture" et se souvient des variables de son environnement de création, même après la fin de l'exécution de la fonction créatrice.

4.  **Peut-on appeler une fonction imbriquée depuis l'extérieur de sa fonction parente ?**
    Non, pas directement. Elle n'est visible que dans la portée locale de la fonction parente, sauf si cette dernière la retourne explicitement (closure).

---

## Exercices : {#exercices-16}

### Exercice 1 - Le Compteur Intelligent {#exercice-1---compteur-intelligent}

🎯 **Objectif** : Créer une closure avec état mutable (`nonlocal`).

💼 **Mise en situation** : Vous avez besoin d'un compteur pour générer des IDs uniques, mais vous ne voulez pas utiliser de variable globale polluante.

📝 **Énoncé** :
1.  Créez une fonction `make_counter(start: int = 0)`.
2.  Elle doit retourner une fonction interne `next_id()`.
3.  À chaque appel de `next_id()`, la valeur doit augmenter de 1.
4.  Créez deux compteurs indépendants : `c1` commençant à 0, `c2` commençant à 100.

📺 **Résultat attendu** :
```text
C1: 1
C1: 2
C2: 101
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
def make_counter(start: int = 0):
    count = start
    
    def next_id() -> int:
        nonlocal count # Indispensable pour modifier 'count'
        count += 1
        return count
    
    return next_id

# Création des instances
c1 = make_counter(0)
c2 = make_counter(100)

print(f"C1: {c1()}")
print(f"C1: {c1()}")
print(f"C2: {c2()}")
```
</details>

### Exercice 2 - Le Formateur HTML Configurable {#exercice-2---formateur-html}

🎯 **Objectif** : Utiliser une closure pour capturer une configuration immuable.

💼 **Mise en situation** : Vous générez beaucoup de balises HTML. Au lieu de répéter `format("div", "texte")`, vous voulez des fonctions dédiées `make_div("texte")`, `make_p("texte")`.

📝 **Énoncé** :
1.  Créez une fonction `tag_factory(tag_name: str)`.
2.  Elle retourne une fonction interne `wrap_text(content: str)`.
3.  La fonction interne retourne `<tag>content</tag>`.
4.  Générez les fonctions `h1_maker` et `p_maker`.
5.  Utilisez-les.

📺 **Résultat attendu** :
```text
<h1>Titre Principal</h1>
<p>Ceci est un paragraphe.</p>
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
def tag_factory(tag_name: str):
    # Capture 'tag_name' dans la closure
    def wrap_text(content: str) -> str:
        return f"<{tag_name}>{content}</{tag_name}>"
    
    return wrap_text

# Usine à fonctions
h1_maker = tag_factory("h1")
p_maker = tag_factory("p")

print(h1_maker("Titre Principal"))
print(p_maker("Ceci est un paragraphe."))
```
</details>

### Exercice 3 - Le Système de Moyenne Mobile {#exercice-3---moyenne-mobile}

🎯 **Objectif** : Closure maintenant une liste d'état.

💼 **Mise en situation** : En finance ou monitoring, on veut souvent la moyenne des N dernières valeurs reçues.

📝 **Énoncé** :
1.  Créez `make_averager()`.
2.  La closure doit garder en mémoire une liste de nombres (vide au départ).
3.  La fonction retournée `add_value(new_val)` doit :
    - Ajouter la valeur à la liste.
    - Retourner la moyenne de tous les nombres stockés.
4.  Testez avec 10, puis 20 (moyenne 15), puis 30 (moyenne 20).

📺 **Résultat attendu** :
```text
Ajout 10 -> Moyenne: 10.0
Ajout 20 -> Moyenne: 15.0
Ajout 30 -> Moyenne: 20.0
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
def make_averager():
    series: list[float] = [] # La liste est mutable, pas besoin de nonlocal
    
    def add_value(new_val: float) -> float:
        series.append(new_val)
        total = sum(series)
        return total / len(series)
    
    return add_value

avg = make_averager()

print(f"Ajout 10 -> Moyenne: {avg(10)}")
print(f"Ajout 20 -> Moyenne: {avg(20)}")
print(f"Ajout 30 -> Moyenne: {avg(30)}")
```
</details>