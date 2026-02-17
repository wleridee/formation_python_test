---
sidebar_label: "Boucles: While et les Boucles Infinies"
sidebar_position: 8
difficulty: "junior"
---

# Boucles: While et les Boucles Infinies {#boucles-while-8}

Avec les structures conditionnelles, nous avons appris à faire des choix. Il est maintenant temps d'apprendre à faire des répétitions. Les boucles sont des structures qui permettent de répéter un bloc de code plusieurs fois, ce qui est au cœur de l'automatisation des tâches.

La boucle `while` est la plus fondamentale des boucles. Elle répète une action **tant qu'une condition reste vraie**.

## 1. La Boucle `while` : Répéter sous Condition {#boucle-while-8}

### Quoi
Une boucle `while` exécute un bloc de code de manière répétée aussi longtemps qu'une condition spécifiée est évaluée à `True`. La condition est vérifiée **avant** chaque répétition (ou "itération").

### Pourquoi
La boucle `while` est idéale lorsque vous ne savez pas à l'avance combien de fois vous devez répéter une action. Par exemple, attendre une entrée valide de l'utilisateur, traiter des éléments jusqu'à ce qu'une file soit vide, ou faire tourner la boucle principale d'un jeu.

### Comment
Une boucle `while` typique se compose de trois parties :
1.  **Initialisation** : On prépare une variable de contrôle avant la boucle.
2.  **Condition** : La condition testée par `while` qui utilise cette variable.
3.  **Mise à jour** : À l'intérieur de la boucle, on modifie la variable de contrôle pour que la condition finisse par devenir `False`.

```mermaid
graph TD
    A[1. Initialiser le compteur (ex: i = 5)] --> B{2. Condition: i > 0 ?};
    B -- True --> C[3. Exécuter le bloc de code];
    C --> D[4. Mettre à jour le compteur (i = i - 1)];
    D --> B;
    B -- False --> E[Fin de la boucle];
```

```python
# Cas d'usage : Un compte à rebours
compteur = 5 # 1. Initialisation

print("Début du compte à rebours !")
while compteur > 0: # 2. Condition
    print(compteur)
    compteur = compteur - 1 # 3. Mise à jour (ou compteur -= 1)

print("Décollage !")
```
Ce code affichera 5, 4, 3, 2, 1, puis "Décollage !". Lorsque `compteur` atteint 0, la condition `0 > 0` devient `False` et la boucle s'arrête.

### Zone de Danger : La Boucle Infinie
C'est l'erreur la plus classique avec `while`. Si vous oubliez l'étape de mise à jour (l'incrémentation ou la décrémentation), la condition restera toujours vraie et la boucle ne s'arrêtera **jamais**.

```python
# ⚠️ ATTENTION : BOUCLE INFINIE ⚠️
i = 0
while i < 5:
    print("Ceci est une boucle infinie !")
    # On a oublié de faire i = i + 1
```
Ce programme affichera "Ceci est une boucle infinie !" sans fin et bloquera votre terminal. Pour l'arrêter, vous devez utiliser le raccourci clavier `Ctrl+C`.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Un terminal rempli de la même ligne de texte qui se répète à l'infini, puis l'utilisateur qui tape `Ctrl+C` pour stopper l'exécution.
> **Alt Text** : Arrêt d'une boucle infinie en Python dans le terminal avec Ctrl+C.

---

## 2. Contrôler les Boucles : `break` et le Pattern `while True` {#controle-boucles-8}

### Quoi
Parfois, la condition pour sortir d'une boucle est complexe ou se produit au milieu du bloc de code. Le mot-clé `break` permet de **quitter immédiatement la boucle**, sans attendre la prochaine vérification de la condition.

### Pourquoi
`break` est essentiel pour gérer des événements exceptionnels ou des conditions de sortie multiples. Il est souvent utilisé dans un `if` à l'intérieur de la boucle. Cela permet de créer un pattern très courant et puissant : la boucle `while True`.

### Comment
Une boucle `while True` est une boucle infinie intentionnelle. Elle semble dangereuse, mais elle est rendue sûre par une condition de sortie interne qui utilise `break`. C'est une manière très claire de dire : "Répète indéfiniment, jusqu'à ce que X se produise".

```python
# Cas d'usage : Un menu interactif
print("Bienvenue ! Tapez 'aide' pour les commandes ou 'quitter' pour partir.")

while True: # La boucle tournera pour toujours...
    commande = input("> ") # On attend une commande

    if commande == "aide":
        print("- 'bonjour' : dit bonjour")
        print("- 'quitter' : ferme le programme")
    elif commande == "bonjour":
        print("Bonjour à vous !")
    elif commande == "quitter":
        print("Au revoir !")
        break # ...sauf si cette ligne est atteinte. On sort de la boucle.
    else:
        print("Commande inconnue.")

print("Programme terminé.") # Cette ligne n'est atteinte qu'après le 'break'
```
Ce code est plus lisible qu'une boucle `while commande != "quitter":`, car la logique de sortie est clairement contenue dans le corps de la boucle.

---

## Validation des Acquis {#validation-8}

### 3 Questions Clés

1.  Quelles sont les trois étapes cruciales pour construire une boucle `while` qui se termine correctement (sans utiliser `break`) ?
2.  Dans quel scénario est-il plus judicieux d'utiliser une boucle `while` plutôt qu'une boucle `for` (que nous verrons plus tard) ?
3.  Expliquez la différence entre une boucle infinie accidentelle et une boucle `while True` contrôlée.

### 3 Exercices Progressifs

#### Exercice 1 : La Somme des Premiers Entiers
Écrivez un script qui demande à l'utilisateur un nombre entier positif `n`. Le programme doit ensuite calculer et afficher la somme de tous les nombres de 1 à `n` en utilisant une boucle `while`.
*Exemple : si `n=5`, le calcul est `1 + 2 + 3 + 4 + 5 = 15`.*

<details>
<summary>Découvrir la solution commentée</summary>

```python
# 1. Demander un nombre à l'utilisateur
n_str = input("Entrez un nombre entier positif : ")
n = int(n_str)

# 2. Initialiser les variables pour la boucle
somme = 0 # Variable pour accumuler le résultat
compteur = 1 # Variable de contrôle qui commencera à 1

# 3. Construire la boucle while
while compteur <= n:
    somme = somme + compteur # Ajoute la valeur actuelle du compteur à la somme
    compteur = compteur + 1 # Met à jour le compteur pour l'itération suivante

# 4. Afficher le résultat final
print(f"La somme des nombres de 1 à {n} est : {somme}")
```
</details>

#### Exercice 2 : Jeu "Devinez le Nombre"
Créez un petit jeu où l'ordinateur "choisit" un nombre secret (par exemple, `7`). Le programme demande à l'utilisateur de deviner ce nombre.
*   Tant que l'utilisateur ne donne pas la bonne réponse, le programme lui indique si sa proposition est "trop grande" ou "trop petite" et lui demande de réessayer.
*   Lorsque l'utilisateur trouve le nombre, le programme le félicite et s'arrête.

<details>
<summary>Découvrir la solution commentée</summary>

```python
# 1. Définir le nombre secret et initialiser la supposition
nombre_secret = 7
supposition = 0 # On initialise à une valeur qui ne peut pas être la bonne

print("J'ai choisi un nombre entre 1 et 20. À vous de deviner !")

# 2. La boucle continue tant que la supposition n'est pas la bonne
while supposition != nombre_secret:
    supposition_str = input("Votre proposition : ")
    supposition = int(supposition_str)

    # 3. On utilise des conditions 'if/elif' à l'intérieur de la boucle
    if supposition < nombre_secret:
        print("C'est trop petit !")
    elif supposition > nombre_secret:
        print("C'est trop grand !")

# 4. Si on sort de la boucle, c'est que supposition == nombre_secret
print(f"Bravo ! Le nombre secret était bien {nombre_secret}.")
```
</details>

#### Exercice 3 : Collecteur de Données
Écrivez un programme qui demande à l'utilisateur de saisir des âges. Le programme continue de demander des âges jusqu'à ce que l'utilisateur tape "fin".
*   Le programme doit ignorer les entrées qui ne sont pas des nombres valides.
*   À la fin, il doit afficher le nombre total d'âges saisis, l'âge le plus jeune et l'âge le plus âgé.

*Indice : pour vérifier si une chaîne est un nombre, vous pouvez utiliser `entree.isdigit()`.*

<details>
<summary>Découvrir la solution commentée</summary>

```python
# Initialisation des variables de suivi
ages = [] # Une liste pour stocker tous les âges valides
print("Entrez des âges un par un. Tapez 'fin' pour terminer.")

# On utilise le pattern 'while True' pour une boucle contrôlée
while True:
    entree = input("Âge : ")

    # Condition de sortie
    if entree.lower() == 'fin':
        break # Sortie immédiate de la boucle

    # Validation de l'entrée
    if not entree.isdigit():
        print("Veuillez entrer un nombre valide.")
        continue # 'continue' saute le reste de l'itération et recommence la boucle

    # Si l'entrée est valide, on la traite
    age = int(entree)
    ages.append(age)

# Après la boucle, on analyse les résultats
if len(ages) > 0:
    nombre_ages = len(ages)
    age_min = min(ages)
    age_max = max(ages)
    
    print("\n--- Analyse des données ---")
    print(f"Nombre d'âges saisis : {nombre_ages}")
    print(f"Âge le plus jeune : {age_min}")
    print(f"Âge le plus âgé : {age_max}")
else:
    print("\nAucun âge n'a été saisi.")

```
</details>