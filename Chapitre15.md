---
sidebar_label: Fonctions : Arguments Variadiques et Mots-clés
sidebar_position: 15
---

# Chapitre 15 : Fonctions : Arguments Variadiques et Mots-clés

Arguments positionnels (*args), Arguments mots-clés (**kwargs), Passage d'arguments

Imaginez que vous deviez écrire une fonction `calculer_total` pour un panier d'achat. Parfois l'utilisateur a 2 articles, parfois 50. Allez-vous écrire une fonction avec 50 paramètres optionnels ? Certainement pas.

Python offre une flexibilité incroyable grâce aux arguments **variadiques**. Ces outils permettent de créer des fonctions capables d'accepter un nombre indéfini d'arguments, rendant votre code plus générique et robuste. C'est la magie derrière des fonctions natives comme `print()` qui peut accepter 1 ou 100 éléments à afficher.

---

## 1. Arguments Positionnels Variables (`*args`) {#arguments-positionnels-args}

### 1. Quoi
L'opérateur `*` (splat operator) placé devant un nom de paramètre (conventionnellement `*args`) permet à une fonction de recevoir **un nombre arbitraire d'arguments positionnels**. Ces arguments sont regroupés dans un **tuple**.

### 2. Pourquoi
Pour traiter des listes d'éléments de longueur inconnue sans forcer l'utilisateur à créer une liste au préalable.

### 3. Comment

#### A. Syntaxe de base

```python
def sum_all(*numbers: int) -> int:
    # numbers est un tuple : (1, 2, 3...)
    total = 0
    for n in numbers:
        total += n
    return total

print(sum_all(10, 20))      # 30
print(sum_all(1, 1, 1, 1))  # 4
```

#### B. Cas concret : Logger flexible
Une fonction de log qui accepte un message principal et des détails optionnels.

```python
def log_event(message: str, *details: str):
    print(f"[EVENT] {message}")
    for detail in details:
        print(f"  - {detail}")

log_event("Connexion utilisateur", "IP: 192.168.1.1", "User: Admin", "Time: 12:00")
```

#### C. Dépaquetage (Unpacking) à l'appel
Si vous avez déjà une liste et que vous voulez la passer à une fonction attendant des `*args`, utilisez `*` lors de l'appel.

```python
values = [10, 20, 30]
# sum_all(values)      <- Erreur (passe une liste au lieu d'entiers)
print(sum_all(*values)) # Équivalent à sum_all(10, 20, 30)
```

### 4. Zone de Danger
❌ **Ordre des paramètres** :
`*args` doit toujours être placé **après** les arguments positionnels obligatoires.
```python
# ❌ SyntaxError
# def wrong(*args, first): ...

# ✅ Correct
def right(first, *args): ...
```

---

## 2. Arguments Nommés Variables (`**kwargs`) {#arguments-nommes-kwargs}

### 1. Quoi
L'opérateur `**` (double splat) permet de capturer un nombre arbitraire d'arguments passés par **mot-clé** (keyword arguments). Ces arguments sont regroupés dans un **dictionnaire**.

### 2. Pourquoi
Idéal pour les fonctions de configuration, les constructeurs d'objets complexes, ou pour passer des options optionnelles sans alourdir la signature de la fonction.

### 3. Comment

#### A. Syntaxe de base

```python
def print_profile(**attributes):
    # attributes est un dict : {'key': value, ...}
    for key, value in attributes.items():
        print(f"{key}: {value}")

print_profile(name="Alice", job="Dev", city="Paris")
```

#### B. Cas concret : Constructeur HTML
Générer une balise HTML avec des attributs dynamiques.

```python
def create_element(tag: str, text: str, **attributes: str) -> str:
    # Construction de la chaîne d'attributs (ex: class="btn" id="submit")
    attrs_str = " ".join([f'{k}="{v}"' for k, v in attributes.items()])
    
    if attrs_str:
        return f"<{tag} {attrs_str}>{text}</{tag}>"
    return f"<{tag}>{text}</{tag}>"

# Utilisation
btn = create_element("button", "Cliquez ici", cls="btn-primary", id="main-btn")
print(btn) 
# <button cls="btn-primary" id="main-btn">Cliquez ici</button>
```

#### C. Dépaquetage de dictionnaire
Passer un dictionnaire comme arguments nommés.

```python
user_data = {"name": "Bob", "job": "Builder"}
print_profile(**user_data) # Équivalent à print_profile(name="Bob", job="Builder")
```

### 🚨 Limitations
Les clés passées dans `**kwargs` doivent être des chaînes de caractères valides comme identifiants Python si vous les passez directement (`func(1=2)` est impossible).

---

## 3. Combinaison et Ordre des Paramètres {#combinaison-et-ordre}

### 1. Quoi
Une fonction peut combiner tous les types de paramètres, mais l'ordre est **strict**.

### 2. Comment

**Ordre obligatoire :**
1.  Paramètres positionnels standards (`arg`)
2.  `*args`
3.  Paramètres nommés par défaut ou spécifiques (`kwonly`)
4.  `**kwargs`

```python
def master_func(a, b, *args, default_val=10, **kwargs):
    print(f"a={a}, b={b}")
    print(f"args={args}")
    print(f"default={default_val}")
    print(f"kwargs={kwargs}")

master_func(1, 2, 3, 4, default_val=99, user="Admin")
# Sortie:
# a=1, b=2
# args=(3, 4)
# default=99
# kwargs={'user': 'Admin'}
```

#### D. Forcer les arguments nommés (Keyword-Only Arguments)
Pour forcer l'utilisateur à nommer certains arguments (pour la clarté), placez-les après `*args` ou après un `*` solitaire.

```python
# L'argument 'wait' DOIT être nommé
def send_packet(data, *, wait: bool = True):
    pass

send_packet("Hello", wait=False) # ✅ OK
# send_packet("Hello", False)    # ❌ TypeError
```

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-15}

1.  **Quel est le type de la variable `args` dans `def f(*args)` ?**
    C'est un **tuple**.

2.  **Quel est le type de la variable `kwargs` dans `def f(**kwargs)` ?**
    C'est un **dictionnaire**.

3.  **Peut-on définir `def f(**kwargs, *args)` ?**
    **Non**. `*args` doit impérativement être déclaré avant `**kwargs`. L'ordre est : positionnels, `*args`, nommés, `**kwargs`.

4.  **À quoi sert l'étoile seule `*` dans une signature de fonction (ex: `def f(a, *, b)`) ?**
    Elle force tous les arguments qui la suivent (ici `b`) à être passés obligatoirement par nom (keyword-only arguments).

---

## Exercices : {#exercices-15}

### Exercice 1 - La Super Calculatrice {#exercice-1---super-calculatrice}

🎯 **Objectif** : Utiliser `*args` pour une opération mathématique flexible.

💼 **Mise en situation** : Vous créez un outil statistique qui doit pouvoir multiplier une série de nombres, peu importe la longueur de la série.

📝 **Énoncé** :
1.  Créez une fonction `multiply_all(*numbers: int | float) -> int | float`.
2.  Si aucun argument n'est fourni, retournez 0.
3.  Sinon, retournez le produit de tous les nombres.
4.  Testez avec `(2, 3, 4)` et `()`.

📺 **Résultat attendu** :
```text
Produit (2, 3, 4) : 24
Produit () : 0
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
def multiply_all(*numbers: int | float) -> int | float:
    # Gestion du cas vide
    if not numbers:
        return 0
    
    result = 1
    for n in numbers:
        result *= n
        
    return result

print(f"Produit (2, 3, 4) : {multiply_all(2, 3, 4)}")
print(f"Produit () : {multiply_all()}")
```
</details>

### Exercice 2 - Le Générateur de CSS {#exercice-2---generateur-css}

🎯 **Objectif** : Utiliser `**kwargs` pour formater des données.

💼 **Mise en situation** : Vous développez un petit framework web. Vous avez besoin d'une fonction qui transforme des paramètres Python en style CSS.

📝 **Énoncé** :
1.  Créez une fonction `make_css(selector: str, **styles) -> str`.
2.  Le sélecteur est obligatoire.
3.  Les styles sont passés en arguments nommés (ex: `color="red"`).
4.  Attention : en Python `background_color` doit devenir `background-color` en CSS (remplacez les `_` par des `-`).
5.  Retournez la règle CSS formatée.

📺 **Résultat attendu** :
```text
h1 {
    color: red;
    font-size: 16px;
    text-align: center;
}
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
def make_css(selector: str, **styles: str) -> str:
    css_lines = []
    for prop, value in styles.items():
        # Transformation python_case -> css-case
        css_prop = prop.replace("_", "-")
        css_lines.append(f"    {css_prop}: {value};")
    
    block_content = "\n".join(css_lines)
    return f"{selector} {{\n{block_content}\n}}"

print(make_css("h1", color="red", font_size="16px", text_align="center"))
```
</details>

### Exercice 3 - Le Wrapper de Fonction (Logger) {#exercice-3---wrapper-logger}

🎯 **Objectif** : Utiliser `*args` et `**kwargs` ensemble pour transférer des arguments.

💼 **Mise en situation** : C'est un concept avancé essentiel pour les décorateurs. Vous voulez "espionner" l'exécution d'une autre fonction sans connaître ses paramètres à l'avance.

📝 **Énoncé** :
1.  Soit une fonction existante `save_user(id, name, active=True)`.
2.  Créez une fonction `log_and_call(func, *args, **kwargs)` qui :
    - Affiche "Appel de [NomFonction] avec args=[...] kwargs=[...]".
    - Exécute la fonction passée en premier paramètre avec tous les autres arguments reçus.
    - Retourne le résultat de l'exécution.
3.  Testez en appelant `save_user` via `log_and_call`.

📺 **Résultat attendu** :
```text
--- LOG: Appel de save_user ---
Args: (42, 'Alice')
Kwargs: {'active': False}
Utilisateur Alice (42) sauvegardé. Actif: False
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
# Fonction cible
def save_user(user_id: int, name: str, active: bool = True):
    print(f"Utilisateur {name} ({user_id}) sauvegardé. Actif: {active}")

# Fonction wrapper générique
def log_and_call(func, *args, **kwargs):
    print(f"--- LOG: Appel de {func.__name__} ---")
    print(f"Args: {args}")
    print(f"Kwargs: {kwargs}")
    
    # On "forward" (transfère) les arguments tels quels
    return func(*args, **kwargs)

# Test
log_and_call(save_user, 42, "Alice", active=False)
```
</details>