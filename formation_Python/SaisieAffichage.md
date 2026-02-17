---
sidebar_label: "Saisie Utilisateur et Affichage (print, input)"
sidebar_position: 6
difficulty: "junior"
---

# Saisie Utilisateur et Affichage (print, input) {#saisie-affichage-6}

Un programme qui ne communique pas avec l'extérieur est rarement utile. L'interaction la plus fondamentale se fait avec l'utilisateur : lui afficher des informations et lui en demander. En Python, ces deux opérations sont gérées par deux fonctions piliers que vous utiliserez dans presque tous vos scripts : `print()` et `input()`.

Ce chapitre vous apprend à maîtriser ce dialogue essentiel entre votre code et l'utilisateur final.

## 1. La Fonction `print()` : Afficher des Informations {#fonction-print-6}

### Quoi
La fonction `print()` est la manière standard d'afficher du texte, des variables ou des résultats dans la console (le terminal). C'est votre principal outil pour donner un retour à l'utilisateur.

### Pourquoi
Sans `print()`, vous ne pourriez jamais voir le résultat d'un calcul, le contenu d'une variable ou un message de statut. C'est également un outil de débogage simple mais puissant pour tracer le déroulement de votre programme.

### Comment
La méthode la plus moderne et la plus lisible pour afficher des variables est d'utiliser les **f-strings**, que nous avons déjà introduites.

```python
nom_produit = "Cafetière Express"
stock = 15
prix = 89.99

# Affichage simple
print("Détails du produit :")

# Utilisation d'une f-string pour un affichage formaté et lisible
print(f"- Nom : {nom_produit}")
print(f"- Stock disponible : {stock} unités")
print(f"- Prix : {prix} €")

# On peut aussi afficher plusieurs éléments, print() les sépare par un espace
print("Le stock actuel est de", stock, "unités.")
```

La fonction `print()` ajoute automatiquement un saut de ligne à la fin de chaque affichage.

### Zone de Danger
*   **Oublier les parenthèses** : En Python 3 (la version que vous utilisez), `print` est une fonction et requiert des parenthèses. Écrire `print "Bonjour"` comme en Python 2 lèvera une `SyntaxError`.
*   **Types non-textuels** : Si vous n'utilisez pas de f-string, vous devez convertir manuellement les nombres en chaînes de caractères pour les concaténer avec `+`. C'est une source d'erreur fréquente qui est élégamment évitée par les f-strings.

---

## 2. La Fonction `input()` : Demander des Informations {#fonction-input-6}

### Quoi
La fonction `input()` met le programme en pause et attend que l'utilisateur saisisse du texte dans la console et appuie sur la touche "Entrée". La valeur saisie est alors **retournée sous forme de chaîne de caractères (`str`)**.

### Pourquoi
`input()` est la porte d'entrée pour rendre vos programmes interactifs. Au lieu de coder les données en dur dans vos variables, vous pouvez les demander à l'utilisateur, rendant votre programme dynamique et réutilisable.

### Comment
On fournit généralement un message (un "prompt") à `input()` pour indiquer à l'utilisateur quelle information est attendue.

```python
# Demander le nom de l'utilisateur
nom_utilisateur = input("Quel est votre nom ? ")

# Demander la ville
ville_utilisateur = input("Dans quelle ville habitez-vous ? ")

# Utiliser les informations recueillies
print(f"Bonjour {nom_utilisateur} de {ville_utilisateur} ! Bienvenue.")
```

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Un terminal en cours d'exécution du script ci-dessus. On voit le prompt "Quel est votre nom ? ", l'utilisateur qui a tapé "David", puis le prompt suivant.
> **Alt Text** : Interaction avec un script Python utilisant la fonction input() dans un terminal.

### Zone de Danger
Le point le plus important à retenir, et la source de nombreuses erreurs pour les débutants : **`input()` renvoie TOUJOURS une chaîne de caractères (`str`)**, même si l'utilisateur tape des chiffres.

```mermaid
graph TD
    A[Utilisateur tape "25"] -->|Touche Entrée| B(Fonction input())
    B -->|Retourne la valeur| C["'25' (type: str)"]
    
    subgraph "Console"
        A
    end
    subgraph "Programme Python"
        B --> C
    end

    style C fill:#fce4ec,stroke:#ad1457,stroke-width:2px
```

Tenter d'effectuer une opération mathématique sur le résultat direct de `input()` provoquera une `TypeError`.

```python
annee_naissance_str = input("En quelle année êtes-vous né ? ") # L'utilisateur tape 1995
# annee_naissance_str vaut "1995"

age_calcule = 2023 - annee_naissance_str # ❌ TypeError: unsupported operand type(s) for -: 'int' and 'str'
```

La solution est de **caster** (convertir) la chaîne de caractères dans le type numérique approprié (`int` ou `float`) avant de l'utiliser dans un calcul.

```python
annee_naissance_str = input("En quelle année êtes-vous né ? ") # "1995"
annee_naissance_int = int(annee_naissance_str) # Conversion en entier -> 1995

age_calcule = 2023 - annee_naissance_int # ✅ Fonctionne !
print(f"Vous avez ou aurez environ {age_calcule} ans en 2023.")
```

---

## Validation des Acquis {#validation-6}

### 3 Questions Clés

1.  Quel est le type de données de la variable `reponse` après l'exécution de `reponse = input("Entrez un chiffre : ")`, si l'utilisateur tape `123` ?
2.  Comment afficheriez-vous trois variables `jour`, `mois` et `annee` sur la même ligne, séparées par des `/`, en un seul appel à `print()` ?
3.  Pourquoi le code `total = input("Prix : ") * 1.2` ne fonctionnera-t-il pas comme prévu pour calculer une taxe ? Comment le corriger ?

### 3 Exercices Progressifs

#### Exercice 1 : Carte de Visite Personnalisée
Écrivez un script qui demande à l'utilisateur son nom, son prénom, et sa profession. Le programme doit ensuite afficher une petite "carte de visite" formatée avec ces informations.

Exemple de sortie :
```
==============================
Nom        : DOE
Prénom     : John
Profession : Développeur Python
==============================
```

<details>
<summary>Découvrir la solution commentée</summary>

```python
# 1. Demander les informations à l'utilisateur
nom = input("Veuillez entrer votre nom de famille : ")
prenom = input("Veuillez entrer votre prénom : ")
profession = input("Veuillez entrer votre profession : ")

# 2. Afficher la carte de visite formatée
print("\n==============================")
# On utilise .upper() pour mettre le nom en majuscules, comme demandé
print(f"Nom        : {nom.upper()}")
print(f"Prénom     : {prenom}")
print(f"Profession : {profession}")
print("==============================")
```
</details>

#### Exercice 2 : Calculateur de Rabais
Créez un programme qui :
1.  Demande le prix initial d'un article (`float`).
2.  Demande le pourcentage de rabais à appliquer (`int` ou `float`).
3.  Calcule le montant du rabais (`prix * (pourcentage / 100)`).
4.  Calcule le prix final (`prix - montant_rabais`).
5.  Affiche le prix initial, le pourcentage de rabais, le montant économisé et le prix final.

<details>
<summary>Découvrir la solution commentée</summary>

```python
# 1. Demander les informations et les caster immédiatement
prix_initial_str = input("Entrez le prix initial de l'article : ")
prix_initial = float(prix_initial_str)

pourcentage_rabais_str = input("Entrez le pourcentage de rabais (ex: 15 pour 15%) : ")
pourcentage_rabais = float(pourcentage_rabais_str)

# 2. Effectuer les calculs
montant_rabais = prix_initial * (pourcentage_rabais / 100)
prix_final = prix_initial - montant_rabais

# 3. Afficher un résumé clair et formaté
# L'option :.2f dans les f-strings formate le nombre pour n'afficher que 2 décimales
print("\n--- Résumé de la transaction ---")
print(f"Prix initial : {prix_initial:.2f} €")
print(f"Rabais appliqué : {pourcentage_rabais}%")
print(f"Montant économisé : {montant_rabais:.2f} €")
print(f"Prix final à payer : {prix_final:.2f} €")
```
</details>

#### Exercice 3 : Générateur de Mini-Histoire
Écrivez un programme interactif qui construit une petite histoire en demandant des mots à l'utilisateur.
1.  Demandez un nom (`str`).
2.  Demandez un lieu (`str`).
3.  Demandez un adjectif (`str`).
4.  Demandez un nombre (`int`).
5.  Insérez ces mots dans une histoire pré-écrite et affichez-la.

Exemple :
"Il était une fois, un héros nommé **[nom]** qui vivait dans la ville de **[lieu]**. Chaque jour, il combattait un **[adjectif]** dragon avec ses **[nombre]** compagnons."

<details>
<summary>Découvrir la solution commentée</summary>

```python
print("Bienvenue dans le générateur d'histoires ! Veuillez fournir les mots suivants.")

# 1. Collecter toutes les informations nécessaires
nom_heros = input("Un nom de héros : ")
nom_lieu = input("Un nom de lieu (ville, château, etc.) : ")
adjectif = input("Un adjectif (ex: courageux, maléfique) : ")
nombre_compagnons_str = input("Un nombre entier : ")

# 2. Caster le nombre en 'int'
# Même s'il n'est pas utilisé dans un calcul, c'est une bonne pratique
# si on voulait le manipuler comme un nombre plus tard.
nombre_compagnons = int(nombre_compagnons_str)

# 3. Construire et afficher l'histoire avec une f-string
print("\n--- Votre histoire ---")
histoire = f"Il était une fois, un héros nommé {nom_heros} qui vivait dans la magnifique cité de {nom_lieu}. "
histoire += f"Chaque matin, il affrontait un {adjectif} monstre. "
histoire += f"Heureusement, il était toujours aidé par ses {nombre_compagnons} fidèles compagnons."

print(histoire)
```
</details>