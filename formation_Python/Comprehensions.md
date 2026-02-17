---
sidebar_label: "Compréhensions de Listes, Dictionnaires et Ensembles"
sidebar_position: 20
difficulty: "confirmé"
---

# Compréhensions de Listes, Dictionnaires et Ensembles {#comprehensions-20}

Les compréhensions sont l'une des fonctionnalités les plus élégantes et "pythoniques" du langage. Elles permettent de créer de nouvelles listes, de nouveaux dictionnaires ou ensembles à partir d'itérables existants, en une seule ligne de code concise et lisible.

Elles remplacent souvent des boucles `for` verbeuses, rendant le code plus court et, une fois maîtrisé, plus facile à comprendre. De plus, elles sont généralement plus rapides car optimisées en C au cœur de l'interpréteur Python.

```mermaid
graph TD
    subgraph "Boucle For classique"
        A[Initialiser: `ma_liste = []`] --> B{Pour chaque `item` dans `iterable`};
        B --> C{Si `condition` est vraie};
        C -- Oui --> D[Transformer `item` -> `resultat`];
        D --> E[Ajouter: `ma_liste.append(resultat)`];
        E --> B;
        C -- Non --> B;
        B -- Fin --> F[Liste finale `ma_liste`];
    end

    subgraph "Compréhension (tout en une ligne)"
        G["[ `resultat` for `item` in `iterable` if `condition` ]"]
    end
    
    style G fill:#ccf,stroke:#333,stroke-width:2px
```

## 1. Compréhension de Liste (List Comprehension) {#list-comprehension-20}

### Quoi
C'est la forme la plus courante de compréhension. Elle permet de construire une nouvelle liste en appliquant une expression à chaque élément d'un itérable, avec la possibilité d'ajouter une condition de filtrage.

### Pourquoi
-   **Concise et lisible** : Une ligne pour exprimer une transformation et un filtre.
-   **Performante** : Souvent plus rapide qu'une boucle `for` avec des `append()`.
-   **Expressive** : Permet de se concentrer sur le "quoi" (le résultat) plutôt que sur le "comment" (la mécanique de la boucle).

### Comment
La syntaxe se décompose en trois parties :
`[expression for item in iterable if condition]`

-   `expression`: La valeur à inclure dans la nouvelle liste (ex: `x * 2`).
-   `for item in iterable`: La boucle sur l'itérable source.
-   `if condition` (optionnel): Le filtre pour n'inclure que certains éléments.

**Cas Réel : Créer une liste des carrés des nombres pairs de 0 à 9.**

**Version avec une boucle `for` :**
```python
carres_pairs = []
for i in range(10):
    if i % 2 == 0:
        carres_pairs.append(i * i)

print(carres_pairs)
# Résultat: [0, 4, 16, 36, 64]
```

**Version avec une compréhension de liste :**
```python
carres_pairs_comp = [i * i for i in range(10) if i % 2 == 0]

print(carres_pairs_comp)
# Résultat: [0, 4, 16, 36, 64]
```

### Zone de Danger
*   **Lisibilité avant tout** : N'abusez pas des compréhensions. Si elles deviennent trop complexes, avec plusieurs boucles `for` imbriquées et des conditions complexes, une boucle `for` classique sera beaucoup plus lisible.

    ```python
    # ❌ Difficile à lire
    result = [x+y for x in [0,1,2] for y in [10,20,30] if x > 0 and y > 10]
    ```

---

## 2. Compréhension de Dictionnaire (Dictionary Comprehension) {#dict-comprehension-20}

### Quoi
Le même principe s'applique pour créer des dictionnaires. La syntaxe est très similaire, mais on utilise des accolades `{}` et on spécifie une paire `clé: valeur`.

### Pourquoi
C'est une méthode extrêmement efficace pour créer des dictionnaires à partir d'autres données, par exemple pour inverser un dictionnaire, créer des tables de correspondance, ou transformer les données d'une liste.

### Comment
La syntaxe est : `{cle: valeur for item in iterable if condition}`

**Cas Réel : Créer un dictionnaire qui associe chaque mot d'une phrase à sa longueur.**
```python
phrase = "le code pythonique est un plaisir"
longueurs_mots = {mot: len(mot) for mot in phrase.split()}

print(longueurs_mots)
# Résultat: {'le': 2, 'code': 4, 'pythonique': 10, 'est': 3, 'un': 2, 'plaisir': 7}
```

### Zone de Danger
*   **Écrasement des clés** : Comme pour tout dictionnaire, si la clé générée par la compréhension existe déjà, sa valeur précédente sera écrasée sans avertissement. Assurez-vous que les clés que vous générez sont uniques si vous ne voulez pas perdre de données.

---

## 3. Compréhension d'Ensemble (Set Comprehension) {#set-comprehension-20}

### Quoi
Encore une fois, la syntaxe est très proche, utilisant des accolades `{}`, mais cette fois sans spécifier de paire clé-valeur. Le résultat est un ensemble (`set`), ce qui signifie que les doublons seront automatiquement éliminés.

### Pourquoi
Idéal pour créer un ensemble d'éléments uniques à partir d'un itérable qui peut contenir des doublons. C'est plus court et plus direct que de créer une liste puis de la convertir en ensemble.

### Comment
La syntaxe est : `{expression for item in iterable if condition}`

**Cas Réel : Extraire les lettres uniques (en minuscules) d'une chaîne de caractères.**
```python
texte = "Programming in Python is Fun!"
lettres_uniques = {lettre.lower() for lettre in texte if lettre.isalpha()}

print(lettres_uniques)
# Résultat: {'p', 'u', 'y', 't', 'n', 's', 'o', 'h', 'm', 'i', 'f', 'r', 'g'} (l'ordre peut varier)
```

### Zone de Danger
*   **Désordonné par nature** : Rappelez-vous que les ensembles ne sont pas ordonnés. Ne comptez pas sur l'ordre des éléments dans le `set` résultant d'une compréhension.

---

## Validation des Acquis {#validation-20}

### 3 Questions Clés
1.  Quels sont les trois composants principaux d'une compréhension de liste ?
2.  Comment écririez-vous une compréhension de dictionnaire pour créer `{1: 1, 2: 8, 3: 27}` à partir de `range(1, 4)` ?
3.  Quel est le principal avantage d'une compréhension d'ensemble par rapport à une compréhension de liste si les données sources contiennent des doublons ?

### 3 Exercices Progressifs

#### Exercice 1 : Filtrage et Transformation de Noms
Vous avez une liste de noms. Utilisez une compréhension de liste pour créer une nouvelle liste contenant uniquement les noms de plus de 5 caractères, en majuscules.

```python
noms = ["alice", "bob", "charlie", "david", "eve"]
```

<details>
<summary>Découvrir la solution commentée</summary>

```python
noms = ["alice", "bob", "charlie", "david", "eve"]

# Expression: nom.upper()
# Itérable: noms
# Condition: len(nom) > 5
noms_filtres = [nom.upper() for nom in noms if len(nom) > 5]

print(noms_filtres)
# Résultat attendu : ['CHARLIE']
```
</details>

#### Exercice 2 : Inverser un Dictionnaire
Vous avez un dictionnaire de codes de pays. Utilisez une compréhension de dictionnaire pour l'inverser (les valeurs deviennent les clés et les clés deviennent les valeurs). Supposez qu'il n'y a pas de doublons dans les valeurs.

```python
codes_pays = {"France": "FR", "Allemagne": "DE", "Espagne": "ES"}
```

<details>
<summary>Découvrir la solution commentée</summary>

```python
codes_pays = {"France": "FR", "Allemagne": "DE", "Espagne": "ES"}

# On itère sur les paires (clé, valeur) du dictionnaire avec .items()
# La nouvelle clé est l'ancienne valeur (v), et la nouvelle valeur est l'ancienne clé (k).
pays_codes = {v: k for k, v in codes_pays.items()}

print(pays_codes)
# Résultat attendu : {'FR': 'France', 'DE': 'Allemagne', 'ES': 'Espagne'}
```
</details>

#### Exercice 3 : Trouver les Fichiers Images
Vous avez une liste de noms de fichiers. Utilisez une compréhension d'ensemble pour créer un ensemble contenant uniquement les extensions de fichiers d'images ('.jpg', '.jpeg', '.png'), en minuscules.

```python
fichiers = ["document.pdf", "photo_01.JPG", "vacances.jpeg", "archive.zip", "logo.PNG", "script.py"]
```

<details>
<summary>Découvrir la solution commentée</summary>

```python
fichiers = ["document.pdf", "photo_01.JPG", "vacances.jpeg", "archive.zip", "logo.PNG", "script.py"]
extensions_images_valides = {'.jpg', '.jpeg', '.png'}

# 1. On itère sur chaque nom de fichier.
# 2. On isole l'extension, par exemple en splittant par '.' et en prenant le dernier élément.
# 3. On met l'extension en minuscule pour normaliser.
# 4. On filtre pour ne garder que celles qui sont dans notre ensemble d'extensions valides.
extensions_trouvees = {f.split('.')[-1].lower() for f in fichiers if '.' in f and f.split('.')[-1].lower() in extensions_images_valides}

print(extensions_trouvees)
# Résultat attendu : {'jpeg', 'jpg', 'png'} (l'ordre peut varier)
```
</details>