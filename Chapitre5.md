---
sidebar_label: Chaînes de Caractères (Strings)
sidebar_position: 5
---

# Chapitre 5 : Chaînes de Caractères (Strings)

Déclaration, Opérations de base, Méthodes de chaîne, F-strings (formatage)

La manipulation de texte est l'une des tâches les plus courantes en programmation : logs, traitement de données, interfaces utilisateur, communication API. Python excelle dans ce domaine grâce à son type `str` (String) puissant et flexible.

Contrairement à d'autres langages (comme C ou Java ancien), manipuler du texte en Python est intuitif et direct, grâce notamment à une innovation majeure : les **f-strings**.

---

## 1. Déclaration et Syntaxe de Base {#declaration-et-syntaxe-de-base}

### 1. Quoi
Une **chaîne de caractères** (string) est une séquence immuable de caractères Unicode. Elle peut être définie par des guillemets simples `'`, doubles `"` ou triples `"""`.

### 2. Pourquoi
La flexibilité des délimiteurs permet d'inclure facilement des citations ou des apostrophes sans avoir à "échapper" les caractères constamment.

### 3. Comment

#### A. Syntaxe de base

```python
simple: str = 'Bonjour'
double: str = "Le monde"
multiline: str = """Ceci est un texte
sur plusieurs lignes
très pratique pour la doc."""
```

#### B. Gestion des guillemets
Plus besoin de `\` (backslash) partout si on alterne intelligemment.

```python
# Pas besoin d'échapper l'apostrophe ici
sentence: str = "L'arbre est grand" 

# Pas besoin d'échapper les guillemets ici
quote: str = 'Il a dit : "Python est génial"'
```

#### C. Caractères spéciaux
Le backslash `\` reste utile pour les caractères invisibles.
*   `\n` : Nouvelle ligne
*   `\t` : Tabulation

```python
header: str = "Nom\tPrénom\nDupont\tJean"
print(header)
# Affiche :
# Nom     Prénom
# Dupont  Jean
```

### 4. Zone de Danger
❌ **Oublier l'immutabilité** : Une string ne peut pas être modifiée après création.
```python
text = "Hallo"
# text[1] = "e"  <- ERREUR (TypeError)
text = text.replace("a", "e") # ✅ OK : On crée une NOUVELLE string
```

---

## 2. Opérations de Base sur les Strings {#operations-de-base-sur-les-strings}

### 1. Quoi
On peut "additionner" et "multiplier" des chaînes, ainsi qu'accéder à des morceaux spécifiques (slicing).

### 2. Comment

#### A. Concaténation et Répétition

```python
prefix: str = "Super"
suffix: str = "Man"
hero: str = prefix + suffix # "SuperMan"

laugh: str = "Ha" * 3 # "HaHaHa"
```

#### B. Indexing et Slicing (Découpage)
Python utilise des index commençant à 0. Les index négatifs comptent depuis la fin.

```python
alphabet: str = "ABCDEFGHIJKLMNOPQRSTUVWXYZ"

first: str = alphabet[0]    # "A"
last: str = alphabet[-1]    # "Z" (Dernier caractère)

# Slicing [début : fin_exclue : pas]
segment: str = alphabet[0:3]  # "ABC"
reverse: str = alphabet[::-1] # "ZYX...CBA" (Inverse la chaîne)
```

### 4. Zone de Danger
❌ **IndexError** : Accéder à `text[100]` si le texte ne fait que 10 caractères plantera le programme. Le slicing `text[0:100]` est plus tolérant (il s'arrête à la fin sans erreur).

---

## 3. Méthodes de Chaîne Essentielles {#methodes-de-chaine-essentielles}

### 1. Quoi
Le type `str` possède des dizaines de méthodes intégrées pour transformer, nettoyer ou analyser le texte.

### 2. Pourquoi
Pour éviter de réécrire des boucles complexes pour des tâches simples comme "mettre en majuscule" ou "trouver un mot".

### 3. Comment

#### A. Nettoyage et Case

```python
raw_input: str = "  pYtHoN  "

clean: str = raw_input.strip() # "pYtHoN" (Enlève espaces début/fin)
lower: str = clean.lower()     # "python"
title: str = clean.title()     # "Python"
```

#### B. Recherche et Remplacement

```python
text: str = "Python version 3.14"

has_python: bool = "Python" in text # True
position: int = text.find("3.14")   # 15 (Index du début)
new_text: str = text.replace("3.14", "4.0") # "Python version 4.0"
```

#### C. Découpage et Jointure
Indispensable pour traiter des fichiers CSV ou des logs.

```python
csv_line: str = "Jean,Dupont,Paris"

# String vers Liste
data: list[str] = csv_line.split(",") # ['Jean', 'Dupont', 'Paris']

# Liste vers String
path_parts: list[str] = ["home", "user", "docs"]
path: str = "/".join(path_parts) # "home/user/docs"
```

### 🚨 Limitations
Ces méthodes renvoient toutes une **nouvelle** chaîne. Elles ne modifient jamais la chaîne d'origine "sur place" (in-place).

---

## 4. F-strings : Le Formatage Moderne {#f-strings-le-formatage-moderne}

### 1. Quoi
Introduites en Python 3.6, les **Formatted String Literals** (f-strings) permettent d'insérer des expressions Python directement dans une chaîne en la préfixant par la lettre `f`.

### 2. Pourquoi
C'est la méthode la plus **lisible**, la plus **concise** et la plus **rapide** (en performance) pour formater du texte. Elle remplace avantageusement `%` et `.format()`.

### 3. Comment

#### A. Syntaxe de base

```python
name: str = "Alice"
score: int = 100

# Les accolades {} évaluent le code à l'intérieur
message: str = f"Joueur {name} a fait {score} points."
# "Joueur Alice a fait 100 points."
```

#### B. Formatage avancé (Nombres et Dates)
On peut spécifier le format après deux points `:`.

```python
price: float = 1234.5678
ratio: float = 0.75

# .2f = 2 décimales flottantes
# .0% = Pourcentage sans décimale
display: str = f"Prix: {price:.2f}€ | Taux: {ratio:.0%}"
# "Prix: 1234.57€ | Taux: 75%"
```

#### C. Debugging rapide (Python 3.8+)
Ajouter `=` après une variable affiche son nom et sa valeur.

```python
user_id: int = 42
print(f"{user_id=}") 
# Affiche : "user_id=42" (Très pratique pour le debug !)
```

### 4. Zone de Danger
❌ **Injections de code** : Ne jamais utiliser de f-strings pour construire des requêtes SQL brutes (`f"SELECT * FROM users WHERE name={input}"`). C'est une faille de sécurité majeure (SQL Injection). Utilisez les paramètres de votre bibliothèque SQL.

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-5}

1.  **Les chaînes de caractères sont-elles mutables en Python ?**
    Non, elles sont immuables. Toute opération de modification (comme `replace` ou `upper`) retourne une *nouvelle* chaîne sans modifier l'originale.

2.  **Quelle est la différence entre `text.find("x")` et `text.index("x")` ?**
    Si "x" n'est pas trouvé, `find` renvoie `-1` (plus sûr), tandis que `index` lève une exception `ValueError` (plus strict).

3.  **Pourquoi préférer les f-strings (`f"{var}"`) à la concaténation (`"a" + var`) ?**
    Pour la lisibilité, la gestion automatique des types (pas besoin de `str(var)`), et les performances légèrement supérieures.

4.  **Comment inverser une chaîne `s` simplement ?**
    Avec le slicing : `s[::-1]`.

---

## Exercices : {#exercices-5}

### Exercice 1 - Le Nettoyeur d'Email {#exercice-1---le-nettoyeur-d-email}

🎯 **Objectif** : Utiliser `strip()` et `lower()` pour normaliser une entrée utilisateur.

💼 **Mise en situation** : Les utilisateurs saisissent souvent leur email avec des majuscules ou des espaces accidentels. Votre base de données a besoin d'emails propres.

📝 **Énoncé** :
1.  Déclarez une variable `raw_email` contenant `"  Jean.Dupont@GMAIL.com "`.
2.  Nettoyez les espaces autour.
3.  Passez tout en minuscules.
4.  Affichez le résultat et sa longueur.

📺 **Résultat attendu** :
```text
Email normalisé : jean.dupont@gmail.com
Longueur : 21
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
raw_email: str = "  Jean.Dupont@GMAIL.com "

# On peut enchaîner les méthodes (Method Chaining)
# 1. strip() enlève les espaces début/fin
# 2. lower() met tout en minuscules
clean_email: str = raw_email.strip().lower()

print(f"Email normalisé : {clean_email}")
print(f"Longueur : {len(clean_email)}")
```
</details>

### Exercice 2 - Générateur de Slug URL {#exercice-2---generateur-de-slug-url}

🎯 **Objectif** : Manipuler `replace()` et `lower()`.

💼 **Mise en situation** : Pour votre blog, vous devez transformer un titre d'article en "slug" URL (tout en minuscule, espaces remplacés par des tirets).

📝 **Énoncé** :
1.  Variable `title` = `"Python 3.14 est Sorti !"`.
2.  Transformez le titre pour qu'il devienne `python-3.14-est-sorti-!`.
3.  Bonus : Essayez de retirer le `!` à la fin (avec slicing ou replace).

📺 **Résultat attendu** :
```text
Slug : python-3.14-est-sorti
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
title: str = "Python 3.14 est Sorti !"

# 1. Minuscules
slug: str = title.lower()

# 2. Remplacer espaces par tirets
slug = slug.replace(" ", "-")

# 3. Retirer le '!' (Option simple via replace)
slug = slug.replace("!", "")

# Astuce pro: On aurait aussi pu utiliser strip("!") si le ! était juste à la fin
# slug = slug.strip("!-")

print(f"Slug : {slug}")
```
</details>

### Exercice 3 - Analyseur de Log {#exercice-3---analyseur-de-log}

🎯 **Objectif** : Utiliser `split()` et les f-strings avancées.

💼 **Mise en situation** : Vous recevez une ligne de log serveur au format `IP:STATUS:USER`. Vous devez extraire les infos.

📝 **Énoncé** :
1.  Variable `log_line` = `"192.168.1.1:200:admin"`.
2.  Utilisez `split()` pour séparer les éléments.
3.  Affichez un rapport formaté : "L'utilisateur [USER] s'est connecté depuis [IP] (Statut: [STATUS])".

📺 **Résultat attendu** :
```text
L'utilisateur admin s'est connecté depuis 192.168.1.1 (Statut: 200)
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
log_line: str = "192.168.1.1:200:admin"

# Découpage selon le séparateur ":"
parts: list[str] = log_line.split(":")

# Extraction (Assignation multiple pour être élégant)
ip, status, user = parts
# Ou manuellement :
# ip = parts[0]
# status = parts[1]
# user = parts[2]

# Affichage avec f-string
print(f"L'utilisateur {user} s'est connecté depuis {ip} (Statut: {status})")
```
</details>