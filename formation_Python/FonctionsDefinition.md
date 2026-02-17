---
sidebar_label: "Fonctions: Définition, Paramètres et Valeurs de Retour"
sidebar_position: 14
difficulty: "junior"
---

# Fonctions: Définition, Paramètres et Valeurs de Retour {#fonctions-definition-14}

Jusqu'ici, nos scripts s'exécutaient de haut en bas. Si nous devions répéter une série d'opérations, nous devions copier-coller le code. C'est inefficace et source d'erreurs. Les **fonctions** sont la solution à ce problème.

Une fonction est un bloc de code réutilisable qui effectue une action spécifique. En définissant une fonction, vous donnez un nom à une série d'instructions, que vous pouvez ensuite "appeler" autant de fois que vous le souhaitez. C'est le principe fondamental du **DRY** : *Don't Repeat Yourself* (Ne vous répétez pas).

## 1. Définir et Appeler une Fonction {#definir-appeler-fonction-14}

### Quoi
Pour créer une fonction, on utilise le mot-clé `def`, suivi du nom de la fonction, de parenthèses `()`, et de deux-points `:`. Le code à l'intérieur de la fonction doit être indenté. Pour exécuter ce code, on "appelle" la fonction en écrivant son nom suivi des parenthèses.

### Pourquoi
Organiser son code en fonctions le rend plus lisible, plus facile à maintenir et à déboguer. Si vous devez changer une logique, vous ne le faites qu'à un seul endroit : dans la définition de la fonction.

### Comment
```python
# --- Définition de la fonction ---
def afficher_message_bienvenue():
    """Cette fonction affiche un message de bienvenue standard."""
    print("-------------------------")
    print("Bienvenue dans notre application !")
    print("-------------------------")

# --- Appels de la fonction ---
print("Début du programme.")
afficher_message_bienvenue() # Premier appel

print("\nUn nouvel utilisateur se connecte...")
afficher_message_bienvenue() # Deuxième appel
```

### Zone de Danger
L'erreur la plus courante pour un débutant est d'oublier les parenthèses `()` lors de l'appel.
*   `afficher_message_bienvenue()` **exécute** la fonction.
*   `afficher_message_bienvenue` (sans `()`) fait référence à l'objet fonction lui-même, mais ne l'exécute pas. Votre code ne fera rien et n'affichera pas d'erreur, ce qui peut être déroutant.

---

## 2. Paramètres : Rendre les Fonctions Flexibles {#parametres-fonctions-14}

### Quoi
Les **paramètres** (aussi appelés arguments) sont des variables que l'on définit entre les parenthèses lors de la création de la fonction. Ils agissent comme des "espaces réservés" pour des valeurs que l'on fournira plus tard, lors de l'appel. Cela permet à une même fonction de se comporter différemment en fonction des données qu'on lui passe.

### Pourquoi
Les paramètres rendent les fonctions génériques et réutilisables. Au lieu d'avoir une fonction `saluer_alice()` et une autre `saluer_bob()`, on peut avoir une seule fonction `saluer(nom)` qui fonctionne pour n'importe quel nom.

### Comment
```python
# 'nom' et 'ville' sont les paramètres de la fonction
def saluer_utilisateur(nom, ville):
    """Affiche un message d'accueil personnalisé."""
    print(f"Bonjour {nom} de {ville} !")

# On fournit les valeurs (arguments) lors de l'appel
saluer_utilisateur("Alice", "Paris")
saluer_utilisateur("Bob", "Lyon")
```

### Zone de Danger
*   **`TypeError` pour un nombre d'arguments incorrect** : Si vous appelez une fonction avec plus ou moins d'arguments que de paramètres définis, Python lèvera une `TypeError`.

    ```python
    saluer_utilisateur("Charlie") # ❌ TypeError: saluer_utilisateur() missing 1 required positional argument: 'ville'
    ```
    > 📸 **CAPTURE D'ÉCRAN REQUISE**
    > **Sujet** : Fenêtre de code montrant l'appel `saluer_utilisateur("Charlie")` et l'erreur `TypeError` correspondante dans le terminal.
    > **Alt Text** : Exemple d'une erreur TypeError en Python due à un argument manquant lors de l'appel d'une fonction.

---

## 3. `return` : Obtenir une Valeur en Retour {#return-valeur-14}

### Quoi
Jusqu'à présent, nos fonctions ont *fait* quelque chose (afficher du texte). Mais souvent, on veut qu'une fonction *calcule* quelque chose et nous donne le résultat. Le mot-clé `return` met fin à l'exécution de la fonction et renvoie une valeur à l'endroit où la fonction a été appelée.

```mermaid
graph TD
    A[Arguments<br>(5, 3)] -- Entrée --> B{Fonction addition(a, b)}
    B -- Logique interne<br>resultat = a + b --> C{return resultat}
    C -- Sortie --> D[Valeur de retour<br>(8)]
```

### Pourquoi
Le `return` permet de composer des fonctions. Le résultat d'une fonction peut être stocké dans une variable et utilisé comme entrée pour une autre fonction. C'est la base de la programmation modulaire. Cela sépare clairement les fonctions qui effectuent un calcul de celles qui affichent des résultats.

### Comment
```python
# Cette fonction CALCULE une valeur et la retourne
def additionner(a, b):
    """Retourne la somme de deux nombres."""
    print("Calcul en cours...") # Ce code s'exécute
    return a + b
    print("Calcul terminé !") # ❌ Ce code ne sera JAMAIS atteint

# On appelle la fonction et on stocke son résultat dans une variable
resultat_calcul = additionner(10, 5)

print(f"Le résultat du calcul est : {resultat_calcul}")
print(f"On peut réutiliser le résultat : {resultat_calcul * 2}")
```

### Zone de Danger
*   **Code inatteignable** : Tout code placé après une instruction `return` dans une fonction ne sera jamais exécuté.
*   **Retour implicite de `None`** : Une fonction qui n'a pas d'instruction `return` (ou qui a un `return` sans valeur) renvoie implicitement la valeur spéciale `None`.

    ```python
    def fonction_sans_return():
        x = 10
    
    resultat = fonction_sans_return()
    print(resultat) # Affiche: None
    ```

---

## Validation des Acquis {#validation-14}

### 3 Questions Clés

1.  Quelle est la différence entre définir une fonction et l'appeler ?
2.  Quelle est la différence entre un *paramètre* et un *argument* ?
3.  Quelle est la valeur de retour d'une fonction qui ne contient pas le mot-clé `return` ?

### 3 Exercices Progressifs

#### Exercice 1 : Calculatrice de TVA
Créez une fonction `calculer_tva` qui prend en paramètre un `prix_ht` (prix hors taxe). La fonction doit calculer le montant de la TVA (considérez un taux de 20%) et l'afficher dans une phrase claire. Appelez cette fonction avec deux prix différents.

<details>
<summary>Découvrir la solution commentée</summary>

```python
def calculer_tva(prix_ht):
    """
    Calcule et affiche le montant de la TVA (20%) pour un prix donné.
    Cette fonction ne retourne rien.
    """
    taux_tva = 0.20
    montant_tva = prix_ht * taux_tva
    print(f"Pour un prix HT de {prix_ht}€, le montant de la TVA est de {montant_tva:.2f}€.")

# Appels de la fonction
calculer_tva(100)
calculer_tva(25.50)
```
</details>

#### Exercice 2 : Vérificateur de Mot de Passe
Écrivez une fonction `est_mot_de_passe_valide` qui prend un `mot_de_passe` (chaîne de caractères) en paramètre. La fonction doit **retourner** `True` si le mot de passe a 8 caractères ou plus, et `False` sinon.
Appelez la fonction avec deux mots de passe différents et affichez le résultat retourné.

<details>
<summary>Découvrir la solution commentée</summary>

```python
def est_mot_de_passe_valide(mot_de_passe):
    """
    Vérifie si un mot de passe a une longueur d'au moins 8 caractères.
    Retourne une valeur booléenne (True ou False).
    """
    longueur = len(mot_de_passe)
    if longueur >= 8:
        return True
    else:
        return False

# Solution encore plus concise et "pythonique" :
# def est_mot_de_passe_valide(mot_de_passe):
#     return len(mot_de_passe) >= 8

# Appels et utilisation du résultat
mdp1 = "azerty"
mdp2 = "motdepassesecurise"

print(f"Le mot de passe '{mdp1}' est valide ? {est_mot_de_passe_valide(mdp1)}")
print(f"Le mot de passe '{mdp2}' est valide ? {est_mot_de_passe_valide(mdp2)}")

# On peut aussi utiliser le retour dans une condition if
if est_mot_de_passe_valide(mdp2):
    print("Le second mot de passe est conforme.")
```
</details>

#### Exercice 3 : Générateur de Salutations
Créez une fonction `generer_salutation` qui prend deux paramètres, `nom` et `langue`.
*   Si `langue` est `"fr"`, la fonction doit **retourner** `"Bonjour [nom] !"`.
*   Si `langue` est `"en"`, elle doit **retourner** `"Hello [nom] !"`.
*   Pour toute autre langue, elle doit **retourner** `"[nom], langue non supportée."`.

Stockez les résultats de plusieurs appels dans des variables et affichez-les.

<details>
<summary>Découvrir la solution commentée</summary>

```python
def generer_salutation(nom, langue):
    """
    Génère une chaîne de salutation personnalisée en fonction de la langue.
    Retourne la chaîne de caractères formatée.
    """
    if langue == "fr":
        return f"Bonjour {nom} !"
    elif langue == "en":
        return f"Hello {nom} !"
    else:
        return f"{nom}, langue non supportée."

# Appels de la fonction et stockage des résultats
salutation_fr = generer_salutation("Marie", "fr")
salutation_en = generer_salutation("John", "en")
salutation_es = generer_salutation("Carlos", "es")

# Affichage des résultats
print(salutation_fr)
print(salutation_en)
print(salutation_es)
```
</details>