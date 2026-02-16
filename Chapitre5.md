---
sidebar_label: Les Chaînes de Caractères (Strings)
sidebar_position: 5
---

# Chapitre 5 : Les Chaînes de Caractères (Strings)

Manipulation de base, Méthodes de strings, F-strings et formatage.

Les chaînes de caractères (type `str`) sont fondamentales en Python. Ce ne sont pas de simples tableaux de caractères, mais des séquences immuables d'Unicode, dotées d'une API riche pour la manipulation de texte.

## 1. Manipulation de base {#manipulation-de-base}

### 1. Quoi
Une chaîne de caractères est une séquence ordonnée et **immuable** de caractères Unicode. Une fois créée, elle ne peut être modifiée sur place (in-place).

### 2. Pourquoi
Le texte est omniprésent : logs, interface utilisateur, clés de base de données, communication API (JSON). Comprendre comment Python gère le texte est crucial pour la performance et l'exactitude des données.

### 3. Comment

#### A. Syntaxe de base

```python
# Différents délimiteurs
s1 = 'Simple quote'
s2 = "Double quote (strictement identique)"
s3 = """Triple quote pour
le texte multi-lignes"""

# Raw strings (ignore les échappements \n, \t...)
# Utile pour les regex ou chemins Windows
path = r"C:\Users\Name\Dossier"
```

#### B. Indexation et Slicing (Découpage)

La syntaxe `[start:stop:step]` permet d'extraire des sous-chaînes.

```python
text = "Python Architecture"

# Indexation (0-based)
first = text[0]    # 'P'
last = text[-1]    # 'e' (dernier élément)

# Slicing
lang = text[0:6]   # 'Python' (exclut l'index 6)
arch = text[7:]    # 'Architecture' (jusqu'à la fin)
copy = text[:]     # Copie complète
rev = text[::-1]   # 'erutcetihcrA nohtyP' (inverse)
```

#### C. Exemples pratiques

**Cas 1 : Extraction d'ID**
```python
order_ref = "ORD-2024-8932"
# On sait que le préfixe fait 9 caractères
order_id = order_ref[9:] # "8932"
```

**Cas 2 : Vérification de fichier**
```python
filename = "report_final.pdf"
# Récupérer l'extension (les 3 derniers caractères)
ext = filename[-3:] # "pdf"
```

#### D. L'immutabilité

```python
s = "Hello"
# ❌ ERREUR : TypeError: 'str' object does not support item assignment
# s[0] = "h"

# ✅ CORRECTION : Créer une nouvelle chaîne
s = "h" + s[1:] # "hello"
```

### 4. Zone de Danger

❌ **Concaténation en boucle** :
Évitez `s += part` dans une grande boucle. Cela crée un nouvel objet à chaque itération (coûteux en mémoire).
✅ **Utilisez** `list.append()` puis `''.join(list)` (vu au chapitre suivant) ou `io.StringIO`.

---

## 2. F-strings et Formatage {#f-strings-et-formatage}

### 1. Quoi
Les **F-strings** (Formatted String Literals), introduites en Python 3.6, sont la méthode standard pour insérer des variables dans des chaînes. Elles sont précédées d'un `f`.

### 2. Pourquoi
Elles sont plus lisibles, plus concises et **plus performantes** que les anciennes méthodes (`%` ou `.format()`).

### 3. Comment

#### A. Syntaxe de base

```python
user = "Alice"
role = "Admin"

# Injection directe
msg = f"User: {user}, Role: {role}"
```

#### B. Expressions et Formatage Avancé

Les f-strings permettent d'exécuter du code Python et de formater les nombres.

```python
price = 49.5
tax_rate = 0.2

# Calculs et formatage float (2 décimales)
total_display = f"Total TTC: {price * (1 + tax_rate):.2f}€"
# Résultat : "Total TTC: 59.40€"

# Alignement (padding)
# < (gauche), > (droite), ^ (centré)
status = "OK"
log_line = f"| {status:<10} |" # "| OK         |"
```

#### C. Debugging rapide (Python 3.8+)

Utilisez `=` pour afficher le nom de la variable et sa valeur.

```python
width = 10
height = 5
print(f"{width=} {height=}")
# Affiche : width=10 height=5
```

### 4. Zone de Danger

❌ **Injection SQL** : Ne JAMAIS utiliser les f-strings pour construire des requêtes SQL. Cela ouvre des failles de sécurité.
✅ **Utilisez** les paramètres de liaison de votre connecteur SQL (`?` ou `%s`).

---

## 3. Méthodes Essentielles de Strings {#methodes-essentielles}

### 1. Quoi
Python expose des méthodes intégrées directement sur les objets `str` pour le nettoyage, la recherche et la transformation.

### 2. Pourquoi
Nettoyer les entrées utilisateur (espaces superflus), normaliser les données (casse), ou parser des formats simples ne nécessite souvent pas d'expressions régulières complexes.

### 3. Comment

#### A. Nettoyage et Casse

```python
raw_input = "  eMaIL@ExAmPle.cOM  "

# Enchaînement de méthodes (Chainage)
clean_email = raw_input.strip().lower()
# Résultat : "email@example.com"
# strip() retire les espaces début/fin
# lower() met en minuscule
```

#### B. Analyse et Recherche

```python
text = "Error: Connection timeout"

# Vérifications booléennes
is_err = text.startswith("Error") # True
is_py = text.endswith(".py")      # False
has_digit = "User1".isalnum()     # True (Alphanumérique)

# Recherche
pos = text.find("timeout") # 18 (index de début, ou -1 si absent)
```

#### C. Découpage et Jointure (Split & Join)

Essentiel pour transformer `str` <-> `list`.

```python
# Split : String -> List
csv_line = "user,admin,2024"
data = csv_line.split(",") 
# data = ['user', 'admin', '2024']

# Join : List -> String
tags = ["python", "web", "api"]
tag_string = " | ".join(tags)
# tag_string = "python | web | api"
```

#### D. Suppression de préfixes/suffixes (Python 3.9+)

Plus sûr que le slicing si on n'est pas certain de la longueur.

```python
filename = "image_001.png"
name_only = filename.removesuffix(".png").removeprefix("image_")
# Résultat : "001"
```

### 🚨 Limitations de `.replace()`

La méthode `.replace(old, new)` remplace **toutes** les occurrences par défaut.
```python
text = "banana"
print(text.replace("a", "o")) # "bonono"
# Pour limiter :
print(text.replace("a", "o", 1)) # "bonana"
```

---

## Questions clés (validation des acquis du chapitre) {#questions-cles}

1. **Comment insérer la valeur d'une variable `score` dans la chaîne "Points: " ?**
   *Réponse : `f"Points: {score}"`*

2. **Si `s = "Python"`, que renvoie `s[1:4]` ?**
   *Réponse : "yth" (index 1 inclus à 4 exclu).*

3. **Peut-on modifier le premier caractère d'une chaîne `s` via `s[0] = 'X'` ?**
   *Réponse : Non, les chaînes sont immuables. Il faut créer une nouvelle chaîne.*

4. **Quelle méthode utiliser pour retirer les espaces au début et à la fin d'une chaîne ?**
   *Réponse : `.strip()`.*

5. **Quelle est la différence entre `find()` et `index()` ?**
   *Réponse : `find()` retourne `-1` si non trouvé, `index()` lève une erreur (exception).*

---

## Exercices {#exercices}

### Exercice 1 - Générateur de Slug URL {#exercice-1-slug-url}

**🎯 Objectif** : Manipuler la casse, le nettoyage et le remplacement.
**💼 Mise en situation** : Vous développez un CMS (Content Management System) de type WordPress. Vous devez transformer les titres d'articles en "slugs" d'URL valides (minuscules, tirets à la place des espaces).

**📝 Énoncé** :
Créez une variable `titre` contenant `"  Nouvelle version de Python 3.12 : Quoi de neuf ?  "`.
1. Nettoyez les espaces inutiles au début et à la fin.
2. Tout mettre en minuscule.
3. Remplacez les espaces internes par des tirets `-`.
4. Remplacez les deux points `:` par rien (suppression).
5. Affichez le résultat final.

**📺 Résultat attendu** :
```
nouvelle-version-de-python-3.12-quoi-de-neuf-?
```

<details>
<summary>💡 Voir la solution</summary>

```python
titre = "  Nouvelle version de Python 3.12 : Quoi de neuf ?  "

# 1. Nettoyage espaces et casse
slug = titre.strip().lower()

# 2. Remplacements
slug = slug.replace(" :", "") # On retire le " :"
slug = slug.replace(" ", "-") # On remplace les espaces par des tirets

# Note : On peut chaîner les méthodes
# slug = titre.strip().lower().replace(" :", "").replace(" ", "-")

print(slug)
```
</details>

---

### Exercice 2 - Parser de Log Simple {#exercice-2-parser-log}

**🎯 Objectif** : Utiliser `split`, le slicing et les f-strings.
**💼 Mise en situation** : Votre startup SaaS reçoit des logs serveurs bruts. Vous devez extraire les informations clés pour un dashboard de monitoring.

**📝 Énoncé** :
Voici une ligne de log brute :
`log = "[ERROR] 2024-03-15 - DB_Connection_Failed: Timeout au bout de 30s"`

1. Extrayez le niveau de log (ce qu'il y a entre crochets, sans les crochets).
2. Extrayez le message d'erreur (tout ce qui est après `" - "`).
3. Formatez une alerte lisible pour Slack : `🚨 ALERTE {Niveau} : {Message}`.

**📺 Résultat attendu** :
```
🚨 ALERTE ERROR : DB_Connection_Failed: Timeout au bout de 30s
```

<details>
<summary>💡 Voir la solution</summary>

```python
log = "[ERROR] 2024-03-15 - DB_Connection_Failed: Timeout au bout de 30s"

# Approche 1 : Slicing et Find
fin_crochet = log.find("]")
niveau = log[1:fin_crochet] # De 1 à l'index du crochet fermant

# Approche 2 : Split
# On coupe en 2 parties autour de " - "
# parties = ['[ERROR] 2024-03-15', 'DB_Connection_Failed: Timeout au bout de 30s']
message = log.split(" - ")[1]

# Affichage f-string
print(f"🚨 ALERTE {niveau} : {message}")
```
</details>

---

### Exercice 3 - Ticket de Caisse Formatté {#exercice-3-ticket-caisse}

**🎯 Objectif** : Maîtriser le formatage avancé des f-strings (alignement et décimales).
**💼 Mise en situation** : Vous codez le backend d'un terminal de paiement. Vous devez générer une ligne de ticket de caisse parfaitement alignée pour une imprimante thermique.

**📝 Énoncé** :
Vous avez trois variables :
`produit = "Café Long"`
`prix = 1.5` (float)
`quantite = 2`

Générez une ligne de 30 caractères de large au total :
- Le nom du produit aligné à gauche.
- Le total (prix * quantité) aligné à droite, avec 2 décimales et le symbole `€`.
- Remplissez l'espace vide entre les deux avec des points `.`.

**📺 Résultat attendu** :
```
Café Long.................3.00€
```

<details>
<summary>💡 Voir la solution</summary>

```python
produit = "Café Long"
prix = 1.5
quantite = 2
total = prix * quantite

# Explication du format :
# {produit:.<24} : prend 24 espaces, aligné à gauche (<), comblé par des points (.)
# {total:.2f}€   : total formaté float 2 décimales
# Note : Ajuster la largeur (24) selon la longueur totale souhaitée

ligne = f"{produit:.<24}{total:.2f}€"

print(ligne)
print(f"Longueur totale : {len(ligne)}") # Vérification
```
</details>