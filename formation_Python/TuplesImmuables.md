---
sidebar_label: "Tuples: Séquences Immuables et leurs Usages"
sidebar_position: 11
difficulty: "junior"
---

# Tuples: Séquences Immuables et leurs Usages {#tuples-immuables-11}

Nous avons exploré les listes, des collections dynamiques et modifiables. Python nous offre une autre structure de séquence très similaire mais avec une différence capitale : le **tuple**. Un tuple est une liste en "lecture seule".

Ce chapitre vous présente les tuples, vous explique pourquoi leur caractère non modifiable (immuable) est une fonctionnalité puissante, et vous montre quand les préférer aux listes.

## 1. Le Tuple et l'Immuabilité {#tuple-immuabilite-11}

### Quoi
Un tuple est une collection **ordonnée** et **non modifiable** (immuable) d'éléments. On le crée en utilisant des parenthèses `()` au lieu des crochets des listes. Une fois un tuple créé, on ne peut ni ajouter, ni supprimer, ni modifier ses éléments.

```mermaid
graph TD
    subgraph List (Mutable)
        direction LR
        L1[A] --> L2[B]
        L2 --> L3[C]
        style L2 fill:#f8bbd0,stroke:#c2185b
    end
    subgraph Tuple (Immutable)
        direction LR
        T1((A)) --> T2((B))
        T2 --> T3((C))
        style T2 fill:#b2ebf2,stroke:#0097a7
    end
    
    L2 -- Peut être modifié --> M[X]
    T2 -- Ne peut PAS être modifié --> XM(( ))
    style XM fill:#ffcdd2,stroke:#d32f2f,stroke-width:2px,stroke-dasharray: 5 5
```

### Pourquoi
L'immuabilité n'est pas une contrainte, c'est une garantie. Elle assure que les données contenues dans le tuple ne seront pas altérées accidentellement ailleurs dans le programme. C'est une forme de sécurité. Les tuples sont aussi légèrement plus rapides et consomment moins de mémoire que les listes, car Python n'a pas besoin de leur réserver de l'espace supplémentaire pour de futurs ajouts.

### Comment
La création et l'accès à un tuple sont très similaires à ceux d'une liste.

```python
# Cas d'usage : stocker les coordonnées d'un point qui ne doivent pas changer
point_fixe = (10.5, 25.2)

# On peut accéder aux éléments par leur index
latitude = point_fixe[0]
longitude = point_fixe[1]

print(f"Le point est situé à la latitude {latitude} et longitude {longitude}.")

# On peut boucler dessus comme avec une liste
print("Coordonnées :", end=" ")
for coord in point_fixe:
    print(coord, end=" ")
```

### Zone de Danger
*   **La `TypeError`** : La principale erreur est d'oublier qu'un tuple est immuable et d'essayer de le modifier.

    ```python
    mon_tuple = (1, 2, 3)
    mon_tuple[0] = 99 # ❌ TypeError: 'tuple' object does not support item assignment
    ```

    > 📸 **CAPTURE D'ÉCRAN REQUISE**
    > **Sujet** : Fenêtre de code montrant la tentative de modification d'un tuple et l'erreur `TypeError` qui en résulte dans le terminal.
    > **Alt Text** : Exemple d'une TypeError en Python en essayant de modifier un tuple.

*   **Le tuple à un seul élément** : C'est un piège classique. Écrire `(42)` ne crée pas un tuple, Python le voit comme le nombre `42` entre parenthèses. Pour créer un tuple d'un seul élément, il faut **obligatoirement** ajouter une virgule finale : `(42,)`.

    ```python
    pas_un_tuple = (42)
    vrai_tuple = (42,)
    print(type(pas_un_tuple)) # <class 'int'>
    print(type(vrai_tuple))   # <class 'tuple'>
    ```

---

## 2. Packing et Unpacking de Tuples {#packing-unpacking-11}

### Quoi
Le "packing" (empaquetage) consiste à créer un tuple en regroupant plusieurs valeurs. L'"unpacking" (dépaquetage) est l'opération inverse : extraire les valeurs d'un tuple pour les assigner à plusieurs variables en une seule ligne.

### Pourquoi
C'est une des fonctionnalités les plus élégantes et pratiques de Python. L'unpacking rend le code plus concis et plus lisible, notamment pour manipuler des données structurées comme des coordonnées, des paires clé-valeur, ou pour récupérer plusieurs résultats d'une fonction.

### Comment
Les parenthèses pour le packing sont souvent optionnelles, mais il est recommandé de les utiliser pour la clarté.

```python
# Packing : on groupe des valeurs dans un tuple
contact = ("John Doe", "john.doe@email.com", "0123456789")

# Unpacking : on assigne les valeurs du tuple à des variables
# Le nombre de variables doit correspondre au nombre d'éléments dans le tuple
nom, email, telephone = contact

print(f"Nom : {nom}")
print(f"Email : {email}")
print(f"Téléphone : {telephone}")
```

Un cas d'usage très courant est l'échange de la valeur de deux variables :
```python
a = 10
b = 20

# La manière classique (avec une variable temporaire)
# temp = a
# a = b
# b = temp

# La manière Pythonique avec l'unpacking de tuple
a, b = b, a

print(f"Après échange, a = {a} et b = {b}") # a = 20 et b = 10
```

### Zone de Danger
*   **`ValueError` à l'unpacking** : Si le nombre de variables à gauche du `=` ne correspond pas exactement au nombre d'éléments dans le tuple à droite, Python lèvera une `ValueError`.

    ```python
    data = (200, "OK")
    status_code, message, details = data # ❌ ValueError: not enough values to unpack (expected 3, got 2)
    ```

---

## Validation des Acquis {#validation-11}

### 3 Questions Clés

1.  Quelle est la différence fondamentale entre une liste et un tuple ? Citez un avantage de cette différence.
2.  Comment crée-t-on un tuple ne contenant qu'un seul élément, par exemple le nombre `100` ?
3.  Que se passe-t-il si vous essayez d'exécuter la ligne `nom, age = ("Alice", 25, "Paris")` ? Quelle est l'erreur et pourquoi ?

### 3 Exercices Progressifs

#### Exercice 1 : Configuration de Connexion
Les informations de connexion à une base de données (hôte, port, utilisateur) sont des données qui ne devraient pas changer pendant l'exécution d'un programme.
1.  Créez un tuple `db_config` contenant l'hôte (`"localhost"`), le port (`5432`), et le nom d'utilisateur (`"admin"`).
2.  Utilisez l'unpacking pour extraire ces trois valeurs dans des variables `host`, `port`, et `user`.
3.  Affichez une phrase de connexion formatée en utilisant ces variables.

<details>
<summary>Découvrir la solution commentée</summary>

```python
# 1. Créer le tuple de configuration
# C'est un cas parfait pour un tuple : ces données sont fixes.
db_config = ("localhost", 5432, "admin")

# 2. Unpacking des valeurs
host, port, user = db_config

# 3. Afficher le message de connexion
print(f"Connexion à la base de données sur {host}:{port} avec l'utilisateur '{user}'...")
```
</details>

#### Exercice 2 : Retour de Fonction Multiple
Créez une fonction `analyser_texte` qui prend une chaîne de caractères en entrée. Cette fonction doit retourner un tuple contenant deux valeurs :
1.  Le nombre de caractères dans le texte (`len()`).
2.  Le nombre de mots dans le texte (`split()` puis `len()`).

Ensuite, appelez cette fonction avec une phrase de votre choix et utilisez l'unpacking pour stocker les résultats dans deux variables, puis affichez-les.

<details>
<summary>Découvrir la solution commentée</summary>

```python
def analyser_texte(texte):
  """Analyse un texte et retourne sa longueur et son nombre de mots."""
  longueur = len(texte)
  mots = texte.split()
  nombre_mots = len(mots)
  
  # On retourne un tuple contenant les deux résultats.
  # Les parenthèses sont optionnelles mais recommandées.
  return (longueur, nombre_mots)

# Phrase à analyser
ma_phrase = "Les tuples sont vraiment très utiles"

# Appel de la fonction et unpacking direct des résultats
nb_caracteres, nb_mots = analyser_texte(ma_phrase)

# Affichage des résultats
print(f"--- Analyse de la phrase : '{ma_phrase}' ---")
print(f"Nombre de caractères : {nb_caracteres}")
print(f"Nombre de mots : {nb_mots}")
```
</details>

#### Exercice 3 : Traitement de Données Structurées
Vous recevez une liste de tuples. Chaque tuple représente un produit et contient `(nom_produit, prix, quantite_stock)`.
Écrivez un script qui parcourt cette liste et qui, pour chaque produit :
1.  Utilise l'unpacking pour accéder facilement au nom, prix et quantité.
2.  Calcule la valeur totale du stock pour ce produit (`prix * quantite_stock`).
3.  Affiche une ligne résumant ces informations.
4.  À la fin, calcule et affiche la valeur totale de l'inventaire (la somme des valeurs de stock de tous les produits).

```python
# Liste de tuples fournie
inventaire = [
    ("Pommes", 0.5, 100),
    ("Bananes", 0.3, 150),
    ("Cerises", 2.0, 50)
]
```

<details>
<summary>Découvrir la solution commentée</summary>

```python
inventaire = [
    ("Pommes", 0.5, 100),
    ("Bananes", 0.3, 150),
    ("Cerises", 2.0, 50)
]

valeur_inventaire_total = 0.0

print("--- État de l'Inventaire ---")

# On boucle sur la liste. 'produit' sera un tuple à chaque itération.
for produit in inventaire:
    # 1. Unpacking du tuple pour plus de lisibilité
    nom, prix, quantite = produit
    
    # 2. Calcul de la valeur du stock pour ce produit
    valeur_stock_produit = prix * quantite
    
    # 3. Affichage du résumé pour le produit
    print(f"- {nom}: {quantite} unités x {prix}€ = {valeur_stock_produit:.2f}€")
    
    # On ajoute cette valeur au total
    valeur_inventaire_total += valeur_stock_produit

# 4. Affichage du total final après la boucle
print("\n---------------------------------")
print(f"Valeur totale de l'inventaire : {valeur_inventaire_total:.2f}€")
```
</details>