---
sidebar_label: "Boucles: For, Itération et la fonction range()"
sidebar_position: 9
difficulty: "junior"
---

# Boucles: For, Itération et la fonction range() {#boucles-for-range-9}

Après avoir vu la boucle `while` qui répète une action *tant qu'une condition est vraie*, nous allons découvrir la boucle `for`, conçue pour un autre type de répétition : parcourir les éléments d'une collection.

La boucle `for` est l'outil de choix en Python pour **itérer** sur des séquences comme des chaînes de caractères, des listes ou des séquences de nombres générées par `range()`. C'est l'une des structures les plus utilisées et les plus "pythoniques" qui soient.

## 1. La Boucle `for` : Itérer sur une Séquence {#boucle-for-9}

### Quoi
Une boucle `for` parcourt chaque élément d'un objet "itérable" (comme une `string` ou une liste), assigne cet élément à une variable temporaire, et exécute un bloc de code pour cet élément. Une fois tous les éléments parcourus, la boucle se termine.

### Pourquoi
C'est la manière la plus directe et la plus lisible de traiter chaque élément d'une collection. Contrairement à `while`, vous n'avez pas besoin de gérer manuellement un compteur ou une condition de sortie. La boucle sait quand s'arrêter : à la fin de la séquence. C'est plus simple et moins sujet aux erreurs de boucles infinies.

### Comment
La syntaxe est très expressive : `for variable_temporaire in sequence:`.

```python
# Cas d'usage 1 : Parcourir une liste de noms
invites = ["Alice", "Bob", "Charlie"]

print("Liste des invités :")
for nom in invites:
    # À chaque tour, 'nom' prend la valeur d'un élément de la liste
    print(f"- {nom}")

# Cas d'usage 2 : Parcourir les caractères d'une chaîne
mot = "Python"
for lettre in mot:
    # 'lettre' prendra successivement la valeur 'P', 'y', 't', 'h', 'o', 'n'
    print(lettre.upper(), end=" ") # 'end=" "' empêche le saut de ligne
# Sortie : P Y T H O N
```

### Zone de Danger
*   **Ne pas modifier une liste pendant qu'on la parcourt** : Ajouter ou supprimer des éléments de la liste sur laquelle vous itérez peut entraîner des comportements très étranges et des erreurs difficiles à déboguer. Python peut "se perdre" dans les indices. C'est une pratique à éviter.
*   **La variable temporaire est écrasée** : La variable que vous définissez dans la boucle (`nom` ou `lettre` dans nos exemples) est réassignée à chaque itération. Après la boucle, elle conservera la valeur du *dernier* élément de la séquence.

---

## 2. La Fonction `range()` : Générer des Séquences de Nombres {#fonction-range-9}

### Quoi
Souvent, on ne veut pas itérer sur une liste existante, mais simplement répéter une action un nombre de fois précis. La fonction `range()` est l'outil parfait pour cela. Elle génère une séquence de nombres sur laquelle la boucle `for` peut itérer.

`range()` peut être utilisée de trois manières :
*   `range(stop)` : de 0 jusqu'à `stop - 1`.
*   `range(start, stop)` : de `start` jusqu'à `stop - 1`.
*   `range(start, stop, step)` : de `start` jusqu'à `stop - 1`, en avançant par pas de `step`.

### Pourquoi
`range()` est extrêmement efficace en mémoire. Au lieu de créer une liste géante de nombres, il les génère à la volée, un par un, au fur et à mesure que la boucle en a besoin. C'est la méthode standard pour créer des boucles qui doivent s'exécuter *N* fois.

### Comment

```mermaid
graph TD
    subgraph Boucle for i in range(1, 5)
        A(i = 1) --> B(Exécute le code)
        B --> C(i = 2) --> D(Exécute le code)
        D --> E(i = 3) --> F(Exécute le code)
        F --> G(i = 4) --> H(Exécute le code)
    end
    H --> I((Fin de la boucle))
```

```python
# Cas d'usage 1 : Répéter une action 5 fois
# range(5) génère 0, 1, 2, 3, 4
print("Exemple avec range(5):")
for i in range(5):
    print(f"Répétition numéro {i}")

# Cas d'usage 2 : Compter de 1 à 10
# range(1, 11) génère 1, 2, 3, 4, 5, 6, 7, 8, 9, 10
print("\nCompter de 1 à 10:")
for nombre in range(1, 11):
    print(nombre, end=", ")

# Cas d'usage 3 : Compter de 10 à 0 à rebours par pas de 2
# range(10, -1, -2) génère 10, 8, 6, 4, 2, 0
print("\n\nCompte à rebours par pas de 2:")
for i in range(10, -1, -2):
    print(i)
```

### Zone de Danger
L'erreur la plus commune est d'oublier que la valeur `stop` de `range()` est **toujours exclue**. Pour boucler 10 fois de 1 à 10, il faut écrire `range(1, 11)`, pas `range(1, 10)`. C'est ce qu'on appelle une "off-by-one error" (erreur d'un décalage).

---

## 3. Combiner `for` et `if` {#for-if-9}

### Quoi
La véritable puissance des boucles se révèle lorsqu'on les combine avec des structures conditionnelles. En plaçant un bloc `if/elif/else` à l'intérieur d'une boucle `for`, on peut appliquer une logique différente à chaque élément de la séquence.

### Pourquoi
Cela permet de filtrer, catégoriser ou transformer des données. Par exemple : extraire tous les nombres pairs d'une liste, trouver tous les fichiers `.txt` dans un répertoire, ou valider chaque champ d'un formulaire.

### Comment
L'indentation est la clé. Le bloc `if` est indenté à l'intérieur du `for`, et le code du `if` est indenté une fois de plus.

```python
# Cas d'usage : Filtrer les nombres pairs et impairs
nombres = [12, 7, 8, 15, 6, 11, 14]
pairs = []
impairs = []

for nombre in nombres:
    # On teste chaque nombre de la liste
    if nombre % 2 == 0:
        # Si le reste de la division par 2 est 0, c'est un nombre pair
        print(f"{nombre} est pair.")
        pairs.append(nombre)
    else:
        # Sinon, il est impair
        print(f"{nombre} est impair.")
        impairs.append(nombre)

print("\n--- Résultats ---")
print(f"Nombres pairs trouvés : {pairs}")
print(f"Nombres impairs trouvés : {impairs}")
```

---

## Validation des Acquis {#validation-9}

### 3 Questions Clés

1.  Quelle est la principale différence d'utilisation entre une boucle `for` et une boucle `while` ?
2.  Quelle séquence de nombres la fonction `range(10, 0, -3)` va-t-elle générer ?
3.  Si vous itérez sur la chaîne `"Python!"` avec une boucle `for`, quelle sera la valeur de la variable d'itération lors du dernier tour de boucle ?

### 3 Exercices Progressifs

#### Exercice 1 : Table de Multiplication
Écrivez un script qui demande à l'utilisateur un nombre entier. Le programme doit ensuite afficher la table de multiplication de ce nombre, de 1 à 10, en utilisant une boucle `for` et `range()`.

*Exemple pour le nombre 7 :*
```
7 x 1 = 7
7 x 2 = 14
...
7 x 10 = 70
```

<details>
<summary>Découvrir la solution commentée</summary>

```python
# 1. Demander le nombre à l'utilisateur
nombre_str = input("Entrez un nombre pour voir sa table de multiplication : ")
nombre = int(nombre_str)

print(f"\n--- Table de multiplication de {nombre} ---")

# 2. Utiliser une boucle for avec range pour aller de 1 à 10
# On utilise range(1, 11) car 11 est exclu.
for i in range(1, 11):
    resultat = nombre * i
    print(f"{nombre} x {i} = {resultat}")
```
</details>

#### Exercice 2 : Compteur de Voyelles
Créez un programme qui demande à l'utilisateur de saisir une phrase. Le script doit ensuite compter et afficher le nombre total de voyelles (`a, e, i, o, u`, insensible à la casse) dans cette phrase.

<details>
<summary>Découvrir la solution commentée</summary>

```python
# 1. Demander une phrase
phrase = input("Veuillez entrer une phrase : ")

# 2. Initialiser un compteur à zéro
nombre_voyelles = 0
voyelles = "aeiou"

# 3. Parcourir chaque lettre de la phrase
# On convertit la phrase en minuscules pour ne pas avoir à tester 'a' et 'A', etc.
for lettre in phrase.lower():
    # 4. Vérifier si la lettre est une voyelle
    if lettre in voyelles:
        nombre_voyelles += 1 # Raccourci pour nombre_voyelles = nombre_voyelles + 1

# 5. Afficher le résultat final
print(f"La phrase contient {nombre_voyelles} voyelle(s).")
```
</details>

#### Exercice 3 : Dessiner un Triangle
Écrivez un script qui demande à l'utilisateur la hauteur d'un triangle. Le programme doit ensuite dessiner un triangle d'étoiles `*` de cette hauteur.

*Exemple pour une hauteur de 5 :*
```
*
**
***
****
*****
```
*Indice : vous pouvez multiplier une chaîne par un nombre pour la répéter (ex: `'*' * 3` donne `'***'`).*

<details>
<summary>Découvrir la solution commentée</summary>

```python
# 1. Demander la hauteur souhaitée
hauteur_str = input("Entrez la hauteur du triangle : ")
hauteur = int(hauteur_str)

print("\nVoici votre triangle :")

# 2. Boucler de 1 jusqu'à la hauteur incluse
# La variable de boucle 'i' représentera le nombre d'étoiles sur la ligne actuelle
for i in range(1, hauteur + 1):
    # 3. Créer la chaîne d'étoiles en la multipliant
    ligne_etoiles = '*' * i
    
    # 4. Afficher la ligne
    print(ligne_etoiles)
```
</details>