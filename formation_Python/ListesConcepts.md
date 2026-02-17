---
sidebar_label: "Listes: Création, Accès, Modification et Méthodes"
sidebar_position: 10
difficulty: "junior"
---

# Listes: Création, Accès, Modification et Méthodes {#listes-concepts-10}

Si les variables sont des boîtes pour stocker une seule information, les listes sont des étagères pour en stocker toute une collection ordonnée. Une liste de courses, une liste de notes, une liste d'utilisateurs... les listes sont partout en programmation.

Ce chapitre vous apprend à maîtriser la structure de données la plus fondamentale de Python : la liste (`list`). Vous apprendrez à créer, lire, modifier et gérer des collections de données.

## 1. Création et Accès aux Éléments {#creation-acces-listes-10}

### Quoi
Une liste est une collection **ordonnée** et **modifiable** d'éléments, séparés par des virgules et entourés de crochets `[]`. Comme les chaînes de caractères, les éléments d'une liste sont accessibles via un **index** qui commence à 0.

```mermaid
graph TD
    subgraph ma_liste = ["pomme", "banane", "cerise"]
        direction LR
        A("pomme") --- |Index 0| B("banane")
        B --- |Index 1| C("cerise")
        C --- |Index 2| D((Fin))
    end
```

### Pourquoi
Les listes permettent de regrouper des données liées sous un seul nom de variable. Cela simplifie énormément le code en permettant de manipuler l'ensemble des données (par exemple, avec une boucle `for`) au lieu de gérer des dizaines de variables séparées.

### Comment
*   **Création** : `nom_liste = [element1, element2, ...]`
*   **Accès (Indexing)** : `nom_liste[index]`
*   **Découpage (Slicing)** : `nom_liste[start:stop]`

```python
# Création d'une liste de courses
courses = ["lait", "œufs", "pain", "beurre", "fromage"]

# Accéder aux éléments
premier_element = courses[0] # "lait"
dernier_element = courses[-1] # "fromage"

print(f"Le premier article sur la liste est : {premier_element}")
print(f"Le dernier article est : {dernier_element}")

# Slicing pour obtenir une sous-liste
deux_premiers_articles = courses[0:2] # ["lait", "œufs"] (l'index 2 est exclu)
print(f"Il faut d'abord acheter : {deux_premiers_articles}")
```

### Zone de Danger
*   **`IndexError`** : C'est l'erreur la plus fréquente. Elle se produit lorsque vous essayez d'accéder à un index qui n'existe pas dans la liste. Par exemple, `courses[5]` dans notre cas, car les index vont de 0 à 4.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Fenêtre de code avec la liste `courses` et une tentative d'accès à `courses[5]`, montrant l'erreur `IndexError: list index out of range` dans le terminal.
> **Alt Text** : Exemple d'une erreur IndexError lors de l'accès à un élément de liste en Python.

---

## 2. Modifier une Liste : La Mutabilité {#modifier-listes-10}

### Quoi
Contrairement aux chaînes de caractères, les listes sont **mutables**. Cela signifie que vous pouvez modifier leur contenu après leur création : changer un élément, en ajouter ou en supprimer.

### Pourquoi
La mutabilité est ce qui rend les listes si dynamiques. Un panier d'achat en ligne, une liste de tâches à faire, le score d'un jeu... toutes ces collections évoluent avec le temps. La mutabilité permet de refléter ces changements.

### Comment

| Opération                     | Syntaxe                                     | Description                               |
| ----------------------------- | ------------------------------------------- | ----------------------------------------- |
| **Modifier** un élément       | `ma_liste[index] = nouvelle_valeur`         | Remplace l'élément à l'index donné.       |
| **Ajouter** à la fin          | `ma_liste.append(element)`                  | Ajoute un élément à la toute fin.         |
| **Insérer** à un index        | `ma_liste.insert(index, element)`           | Insère un élément à une position précise. |
| **Supprimer** par index       | `del ma_liste[index]`                       | Supprime l'élément à l'index donné.       |
| **Supprimer** par valeur      | `ma_liste.remove(valeur)`                   | Supprime la **première** occurrence de la valeur. |

```python
# Cas d'usage : Gérer une liste de tâches (To-Do List)
taches = ["Faire les courses", "Répondre aux emails", "Appeler le médecin"]
print(f"Tâches initiales : {taches}")

# Modifier une tâche
taches[2] = "Confirmer le rdv médecin"
print(f"Tâches mises à jour : {taches}")

# Ajouter une nouvelle tâche urgente
taches.insert(0, "Tâche URGENTE : Payer la facture")
print(f"Avec ajout urgent : {taches}")

# Ajouter une tâche à la fin
taches.append("Faire du sport")
print(f"Avec ajout en fin : {taches}")

# Supprimer une tâche terminée (par valeur)
taches.remove("Répondre aux emails")
print(f"Après suppression : {taches}")

# Supprimer la tâche urgente (par index)
del taches[0]
print(f"Tâches finales : {taches}")
```

### Zone de Danger
*   **`ValueError` avec `.remove()`** : Si vous essayez de `.remove()` un élément qui n'est pas dans la liste, Python lèvera une `ValueError`.
*   **`append()` vs `insert()`** : `append()` est simple et rapide, toujours à la fin. `insert()` est plus flexible mais peut être plus lent sur de très grandes listes car il doit "décaler" tous les éléments suivants.

---

## 3. Méthodes de Listes Essentielles {#methodes-listes-10}

### Quoi
Les listes viennent avec un ensemble de "fonctions intégrées" (méthodes) très pratiques pour effectuer des opérations courantes comme le tri, l'inversion ou le comptage.

| Méthode/Fonction         | Description                                                      | Exemple                                    |
| ------------------------ | ---------------------------------------------------------------- | ------------------------------------------ |
| `ma_liste.sort()`        | Trie la liste **en place** (modifie l'originale).                | `[3, 1, 2].sort()` -> la liste devient `[1, 2, 3]` |
| `ma_liste.reverse()`     | Inverse l'ordre des éléments **en place**.                       | `[1, 2, 3].reverse()` -> `[3, 2, 1]`     |
| `len(ma_liste)`          | **(Fonction)** Renvoie le nombre d'éléments dans la liste.       | `len(['a', 'b'])` -> `2`                   |
| `ma_liste.count(item)`   | Compte le nombre d'occurrences d'un `item`.                      | `[1, 2, 2].count(2)` -> `2`                |
| `sum(ma_liste)`          | **(Fonction)** Calcule la somme des éléments (doivent être des nombres). | `sum([10, 20])` -> `30`                    |

### Pourquoi
Utiliser ces méthodes est plus efficace, plus lisible et moins sujet aux erreurs que de réécrire la logique soi-même. Elles font partie de la "boîte à outils" standard de tout développeur Python.

### Comment
```python
scores = [88, 92, 77, 92, 65]
print(f"Scores initiaux : {scores}")

# Trier la liste du plus petit au plus grand
scores.sort()
print(f"Scores triés : {scores}")

# Inverser pour avoir du plus grand au plus petit
scores.reverse()
print(f"Scores inversés (top score en premier) : {scores}")

# Informations diverses
nombre_de_scores = len(scores)
nombre_de_92 = scores.count(92)
total_points = sum(scores)

print(f"\nNombre de participants : {nombre_de_scores}")
print(f"Nombre de fois que le score 92 a été obtenu : {nombre_de_92}")
print(f"Total des points : {total_points}")
```

### Zone de Danger
La confusion la plus courante concerne les méthodes "en place" (`.sort()`, `.reverse()`). Elles modifient la liste originale et **ne retournent rien (`None`)**.

```python
ma_liste = [3, 1, 2]
liste_triee = ma_liste.sort() # ❌ ERREUR COMMUNE !

print(f"ma_liste a été modifiée : {ma_liste}") # Affiche [1, 2, 3]
print(f"liste_triee est vide : {liste_triee}") # Affiche None
```
Si vous voulez une *nouvelle* liste triée sans modifier l'originale, utilisez la fonction `sorted()` : `liste_triee = sorted(ma_liste)`.

---

## Validation des Acquis {#validation-10}

### 3 Questions Clés

1.  Quelle est la différence fondamentale entre une liste et une chaîne de caractères en termes de modification ?
2.  Expliquez la différence entre `ma_liste.append('x')` et `ma_liste.insert(0, 'x')`.
3.  Que retourne l'appel `ma_liste.sort()` et quel est son effet sur `ma_liste` ?

### 3 Exercices Progressifs

#### Exercice 1 : Gestion de Playlist
1.  Créez une liste vide appelée `playlist`.
2.  Ajoutez trois de vos chansons préférées à la fin de la liste.
3.  Ajoutez une chanson que vous aimez bien au début de la liste.
4.  Supprimez la deuxième chanson de la liste (par son index).
5.  Affichez la playlist finale.

<details>
<summary>Découvrir la solution commentée</summary>

```python
# 1. Créer une liste vide
playlist = []

# 2. Ajouter trois chansons avec append()
playlist.append("Bohemian Rhapsody")
playlist.append("Stairway to Heaven")
playlist.append("Hotel California")
print(f"Après 3 ajouts : {playlist}")

# 3. Ajouter une chanson au début avec insert()
playlist.insert(0, "Imagine")
print(f"Après insertion au début : {playlist}")

# 4. Supprimer la deuxième chanson (index 1)
del playlist[1]
print(f"Après suppression de la 2ème chanson : {playlist}")

# 5. Afficher le résultat
print("\n--- Ma Playlist Finale ---")
for i, chanson in enumerate(playlist): # enumerate est un moyen avancé de boucler avec l'index
    print(f"{i + 1}. {chanson}")
```
</details>

#### Exercice 2 : Analyseur de Notes
Créez un script qui :
1.  Demande à l'utilisateur de saisir 5 notes (une par une), et les stocke dans une liste.
2.  Une fois les 5 notes saisies, affiche :
    *   La liste complète des notes.
    *   La note la plus basse (`min()`).
    *   La note la plus haute (`max()`).
    *   La moyenne des notes (`sum()` / `len()`).

<details>
<summary>Découvrir la solution commentée</summary>

```python
notes = []
nombre_de_notes = 5

print(f"Veuillez entrer {nombre_de_notes} notes.")

# 1. Boucle pour demander les notes
for i in range(nombre_de_notes):
    note_str = input(f"Entrez la note n°{i + 1} : ")
    notes.append(float(note_str)) # On convertit en float et on ajoute

# 2. Afficher les résultats de l'analyse
print("\n--- Analyse des notes ---")
print(f"Liste des notes saisies : {notes}")

note_min = min(notes)
note_max = max(notes)
moyenne = sum(notes) / len(notes)

print(f"Note la plus basse : {note_min}")
print(f"Note la plus haute : {note_max}")
# On formate la moyenne pour n'afficher que 2 décimales
print(f"Moyenne des notes : {moyenne:.2f}")
```
</details>

#### Exercice 3 : Inverseur de Mots
Écrivez un programme qui :
1.  Demande à l'utilisateur de saisir une phrase.
2.  Utilise la méthode `.split()` (vue dans le chapitre sur les strings) pour transformer la phrase en une liste de mots.
3.  Inverse l'ordre des mots dans la liste.
4.  Utilise la méthode `.join()` pour reconstituer une phrase avec les mots inversés, séparés par des espaces.
5.  Affiche la nouvelle phrase.

*Exemple : "Python est vraiment génial" -> "génial vraiment est Python"*

<details>
<summary>Découvrir la solution commentée</summary>

```python
# 1. Demander la phrase
phrase_originale = input("Entrez une phrase à inverser : ")

# 2. Transformer la phrase en liste de mots
# .split() sans argument sépare par les espaces
mots = phrase_originale.split()
print(f"La phrase en liste de mots : {mots}")

# 3. Inverser l'ordre des éléments de la liste
mots.reverse()
print(f"La liste inversée : {mots}")

# 4. Reconstituer la phrase
# ' '.join(liste) est la syntaxe pour joindre les éléments d'une liste
# en une chaîne de caractères, avec ' ' comme séparateur.
phrase_inversee = ' '.join(mots)

# 5. Afficher le résultat
print("\n--- Phrase inversée ---")
print(phrase_inversee)
```
</details>