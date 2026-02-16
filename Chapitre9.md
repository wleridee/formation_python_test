---
sidebar_label: Structures de Contrôle : Boucles For
sidebar_position: 9
---

# Chapitre 9 : Structures de Contrôle : Boucles For

Itération sur les listes, Fonction range(), Break et continue

L'automatisation est au cœur de la programmation. Si vous devez envoyer un email à 1000 utilisateurs ou traiter 500 fichiers CSV, vous n'allez pas écrire 500 fois les mêmes lignes de code.

La boucle `for` est l'outil principal en Python pour parcourir des séquences (listes, chaînes, plages de nombres). Contrairement au `for` du langage C qui est basé sur un compteur, le `for` de Python est un **itérateur** : il prend chaque élément d'une collection, un par un.

---

## 1. Itération sur les Listes et Séquences {#iteration-sur-les-listes}

### 1. Quoi
Le mécanisme qui permet de répéter un bloc de code pour chaque élément contenu dans une collection (liste, tuple, chaîne, etc.).

### 2. Pourquoi
Pour appliquer une logique métier (calcul, affichage, transformation) à un ensemble de données sans se soucier de la taille de cet ensemble.

### 3. Comment

#### A. Syntaxe de base

```python
fruits: list[str] = ["Pomme", "Poire", "Banane"]

# "fruit" est une variable temporaire qui prendra 
# successivement la valeur de chaque élément
for fruit in fruits:
    print(f"J'aime la {fruit}")
```

#### B. Cas concret : Calcul de total (Panier e-commerce)

```python
prices: list[float] = [19.99, 4.50, 9.99, 15.00]
total: float = 0.0

for price in prices:
    total += price # Ajoute le prix courant au total

print(f"Montant total : {total}€")
```

#### C. Itération sur une String
Une chaîne de caractères est aussi une séquence.

```python
dna_sequence: str = "ATGCGAT"
count_a: int = 0

for base in dna_sequence:
    if base == "A":
        count_a += 1

print(f"Nombre de A : {count_a}")
```

### 4. Zone de Danger
❌ **Modification de la liste pendant l'itération** :
Ne faites jamais `liste.remove(item)` ou `liste.append(x)` DANS une boucle `for item in liste`. Cela perturbe les index internes de l'itérateur.
✅ Si vous devez filtrer, créez une nouvelle liste ou utilisez une compréhension de liste (vu plus tard).

---

## 2. La Fonction `range()` {#la-fonction-range}

### 1. Quoi
Une fonction native qui génère une séquence immuable de nombres. Indispensable quand on veut boucler un nombre précis de fois, ou générer des index.

### 2. Pourquoi
Python ne possède pas de syntaxe `for (i=0; i<10; i++)`. `range()` est l'équivalent Pythonique, plus puissant et plus lisible.

### 3. Comment

#### A. Syntaxe de base

*   `range(stop)` : de 0 à stop (exclu).
*   `range(start, stop)` : de start à stop (exclu).
*   `range(start, stop, step)` : de start à stop (exclu) par pas de step.

```python
# De 0 à 4
for i in range(5):
    print(i) # 0, 1, 2, 3, 4

# De 1 à 10 par pas de 2
for i in range(1, 11, 2):
    print(i) # 1, 3, 5, 7, 9
```

#### B. Cas concret : Accéder à l'index et à la valeur
Si vous avez besoin de la position de l'élément, la méthode moderne est `enumerate()`, mais comprendre `range(len())` reste utile pour le code legacy.

```python
users: list[str] = ["Alice", "Bob", "Charlie"]

# Méthode Moderne et Recommandée ✅
for index, name in enumerate(users):
    print(f"Utilisateur n°{index + 1} : {name}")

# Méthode "Old School" (Moins lisible) ❌
for i in range(len(users)):
    print(f"Utilisateur n°{i + 1} : {users[i]}")
```

### 4. Zone de Danger
❌ **Range est exclusif sur la fin** : `range(1, 5)` s'arrête à 4. C'est une source d'erreurs fréquentes ("Off-by-one error").

---

## 3. Contrôle de boucle : `break` et `continue` {#break-et-continue}

### 1. Quoi
Des mots-clés pour altérer le flux naturel de la boucle.
*   **`break`** : Arrête immédiatement la boucle (sortie définitive).
*   **`continue`** : Arrête l'itération *actuelle* et passe directement à la suivante.

### 2. Pourquoi
Pour optimiser les performances (s'arrêter dès qu'on a trouvé ce qu'on cherche) ou sauter des cas non pertinents.

### 3. Comment

#### A. Syntaxe `break` (Recherche)

```python
emails: list[str] = ["a@a.com", "b@b.com", "admin@site.com", "c@c.com"]
found_admin: bool = False

for email in emails:
    if "admin" in email:
        found_admin = True
        print("Admin trouvé !")
        break # Inutile de vérifier les autres, on sort.

if not found_admin:
    print("Aucun admin.")
```

#### B. Syntaxe `continue` (Filtrage)

```python
scores: list[int] = [10, -1, 15, -5, 20] # -1 indique une erreur de capteur

for score in scores:
    if score < 0:
        continue # On ignore les erreurs et on passe au suivant
    
    # Traitement complexe simulé
    print(f"Traitement du score valide : {score}")
```

#### C. Le `else` de boucle (Rare mais utile)
Un bloc `else` après un `for` s'exécute **si et seulement si** la boucle s'est terminée normalement (sans `break`).

```python
target: int = 42
numbers: list[int] = [1, 2, 3]

for n in numbers:
    if n == target:
        print("Trouvé !")
        break
else:
    # S'exécute uniquement si le break n'a JAMAIS été atteint
    print("Non trouvé dans la liste.")
```

### 4. Zone de Danger
❌ **Abus de `break/continue`** : Trop les utiliser peut rendre le flux logique difficile à suivre ("Spaghetti code"). Si votre boucle a 5 `break` et 3 `continue`, il faut probablement extraire une fonction.

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-9}

1.  **À quoi sert la fonction `range(5)` ?**
    Elle génère une séquence de nombres de 0 à 4 inclus (5 est exclus).

2.  **Quelle est la différence entre `break` et `continue` ?**
    `break` quitte totalement la boucle, tandis que `continue` saute juste le reste du code de l'itération en cours et passe à l'élément suivant.

3.  **Comment obtenir à la fois l'index et la valeur dans une boucle `for` ?**
    En utilisant la fonction `enumerate(liste)` : `for i, valeur in enumerate(liste):`.

4.  **Est-il sûr de modifier la longueur d'une liste (ajout/suppression) pendant qu'on boucle dessus ?**
    Non, c'est fortement déconseillé car cela décale les index. Il vaut mieux boucler sur une copie ou créer une nouvelle liste.

---

## Exercices : {#exercices-9}

### Exercice 1 - Le Filtrage de Logs {#exercice-1---filtrage-logs}

🎯 **Objectif** : Utiliser `continue` pour ignorer des données.

💼 **Mise en situation** : Vous analysez des logs serveur. Vous voulez afficher uniquement les messages d'erreur ("ERROR"), en ignorant les infos ("INFO") et les warnings ("WARN").

📝 **Énoncé** :
1.  Liste `logs = ["INFO: Start", "ERROR: DB Crash", "WARN: Low Memory", "ERROR: Timeout", "INFO: End"]`.
2.  Bouclez sur les logs.
3.  Si le log ne commence pas par "ERROR", passez au suivant (`continue`).
4.  Sinon, affichez "Alerte reçue : [Message]".

📺 **Résultat attendu** :
```text
Alerte reçue : ERROR: DB Crash
Alerte reçue : ERROR: Timeout
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
logs: list[str] = [
    "INFO: Start", 
    "ERROR: DB Crash", 
    "WARN: Low Memory", 
    "ERROR: Timeout", 
    "INFO: End"
]

for log in logs:
    # Si ce n'est pas une erreur, on saute
    if not log.startswith("ERROR"):
        continue
    
    print(f"Alerte reçue : {log}")
```
</details>

### Exercice 2 - La Table de Multiplication {#exercice-2---table-multiplication}

🎯 **Objectif** : Boucles imbriquées (Nested Loops).

💼 **Mise en situation** : Créer un outil éducatif pour afficher les tables de multiplication de 1 à 3.

📝 **Énoncé** :
1.  Utilisez une boucle pour `i` allant de 1 à 3.
2.  À l'intérieur, une boucle pour `j` allant de 1 à 3.
3.  Calculez le produit.
4.  Affichez le format "i x j = résultat".
5.  Ajoutez un séparateur "---" à la fin de chaque table de `i`.

📺 **Résultat attendu** :
```text
1 x 1 = 1
1 x 2 = 2
1 x 3 = 3
---
2 x 1 = 2
...
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
# Boucle externe pour le premier nombre (table de 1, 2, 3)
for i in range(1, 4):
    
    # Boucle interne pour le multiplicateur
    for j in range(1, 4):
        product: int = i * j
        print(f"{i} x {j} = {product}")
    
    # Séparateur à la fin de chaque table 'i'
    print("---")
```
</details>

### Exercice 3 - Recherche de Seuil {#exercice-3---recherche-seuil}

🎯 **Objectif** : Utiliser `break` pour optimiser.

💼 **Mise en situation** : Un capteur de température envoie des données toutes les minutes. Dès que la température dépasse 40°C, on doit déclencher l'arrêt d'urgence et arrêter de lire les données suivantes.

📝 **Énoncé** :
1.  Liste `temps = [25, 30, 35, 42, 38, 20]`.
2.  Parcourez la liste.
3.  Affichez "Lecture : [t]°C".
4.  Si `t > 40`, affichez "SURCHAUFFE ! Arrêt immédiat." et stoppez la boucle.

📺 **Résultat attendu** :
```text
Lecture : 25°C
Lecture : 30°C
Lecture : 35°C
Lecture : 42°C
SURCHAUFFE ! Arrêt immédiat.
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
temps: list[int] = [25, 30, 35, 42, 38, 20]

for t in temps:
    print(f"Lecture : {t}°C")
    
    if t > 40:
        print("SURCHAUFFE ! Arrêt immédiat.")
        # On sort immédiatement de la boucle, 
        # les valeurs 38 et 20 ne seront jamais lues.
        break
```
</details>