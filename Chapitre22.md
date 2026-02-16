---
sidebar_label: Gestion des Erreurs : Exceptions
sidebar_position: 22
---

# Chapitre 22 : Gestion des Erreurs : Exceptions

Bloc try-except, Gestion de multiples exceptions, Bloc finally, Lever des exceptions (raise)

"L'erreur est humaine, mais pour vraiment mettre la pagaille, il faut un ordinateur."
Dans le monde réel, les fichiers disparaissent, les connexions réseau lâchent, et les utilisateurs entrent du texte là où on attend des chiffres. Si votre programme plante au moindre imprévu, il est inutilisable.

La gestion des **exceptions** est le mécanisme qui permet à votre programme de réagir gracieusement aux erreurs, de les diagnostiquer, voire de les corriger automatiquement, sans crasher lamentablement devant l'utilisateur.

---

## 1. Le bloc Try-Except : Filet de sécurité {#bloc-try-except}

### 1. Quoi
Le bloc `try-except` permet de délimiter une zone de code "à risque" (`try`) et de définir quoi faire si une erreur survient (`except`).

### 2. Pourquoi
Pour empêcher l'arrêt brutal du programme. Au lieu d'afficher une traceback rouge effrayante, on capture l'erreur et on affiche un message utile ou on tente une solution de repli.

### 3. Comment

#### A. Syntaxe de base

```python
try:
    # Zone à risque
    age = int(input("Entrez votre âge : "))
    print(f"Vous avez {age} ans.")
except ValueError:
    # Zone de secours (si l'utilisateur tape "douze")
    print("Erreur : Veuillez entrer un nombre entier.")

print("Le programme continue...")
```

#### B. Capturer l'objet exception (`as e`)
Pour obtenir des détails techniques sur l'erreur (le message système).

```python
try:
    result = 10 / 0
except ZeroDivisionError as e:
    print(f"Opération impossible : {e}") 
    # Affiche : Opération impossible : division by zero
```

### 4. Zone de Danger
❌ **Except nu (`except:`)** :
```python
try:
    do_something()
except: # JAMAIS ÇA !
    print("Oups")
```
Cela capture **TOUT**, y compris `Ctrl+C` (KeyboardInterrupt) ou `SystemExit`. Votre programme devient impossible à arrêter proprement.
✅ **Bonne pratique** : Capturez toujours une exception spécifique (`ValueError`, `KeyError`...) ou au pire `Exception` (la classe mère des erreurs standard).

---

## 2. Gestion de Multiples Exceptions {#gestion-multiples-exceptions}

### 1. Quoi
Un même bloc de code peut échouer pour plusieurs raisons différentes. On peut définir des comportements distincts pour chaque type d'erreur.

### 2. Pourquoi
Traiter une "Fichier non trouvé" différemment d'une "Permission refusée". Dans le premier cas, on peut demander à l'utilisateur de vérifier le nom ; dans le second, on doit lui dire de contacter l'administrateur.

### 3. Comment

```python
def read_config(filename: str):
    try:
        # 1. Peut échouer si le fichier n'existe pas
        with open(filename, 'r') as f:
            # 2. Peut échouer si ce n'est pas un nombre
            content = int(f.read().strip())
            # 3. Peut échouer si division par zéro
            ratio = 100 / content
            print(f"Ratio calculé : {ratio}")

    except FileNotFoundError:
        print("Erreur : Le fichier de configuration est introuvable.")
    
    except ValueError:
        print("Erreur : Le fichier doit contenir un nombre valide.")
    
    except ZeroDivisionError:
        print("Erreur : Le fichier ne doit pas contenir 0.")
    
    except Exception as e:
        # Filet de sécurité pour tout autre imprévu
        print(f"Erreur inattendue : {e}")

read_config("missing.txt")
```

---

## 3. Finally et Else : Nettoyage et Succès {#finally-else}

### 1. Quoi
*   `else`: S'exécute **si et seulement si** aucune exception n'a été levée dans le `try`.
*   `finally`: S'exécute **dans tous les cas**, qu'il y ait eu erreur ou non (même s'il y a un `return` avant !).

### 2. Pourquoi
*   `else` : Pour séparer clairement le code "à risque" du code qui dépend de sa réussite.
*   `finally` : Pour garantir la libération de ressources (fermer un fichier, couper une connexion BDD) quoi qu'il arrive.

### 3. Comment

```python
def process_database():
    print("--- Début ---")
    try:
        connect_db() # Fonction imaginaire
    except ConnectionError:
        print("Échec connexion")
        return # On quitte la fonction
    else:
        print("Connexion réussie, envoi des données...")
    finally:
        # S'exécute MÊME si on est passé par le return du bloc except !
        print("Nettoyage des ressources temporaires.")
        print("--- Fin ---")

# Sortie en cas d'erreur :
# --- Début ---
# Échec connexion
# Nettoyage des ressources temporaires.
# --- Fin ---
```

---

## 4. Lever des Exceptions (`raise`) {#lever-exceptions}

### 1. Quoi
Le mot-clé `raise` permet de déclencher volontairement une erreur. Vous pouvez lever des exceptions existantes ou vos propres exceptions personnalisées.

### 2. Pourquoi
Pour signaler qu'une fonction ne peut pas faire son travail parce que les paramètres sont invalides ou que l'état du système ne le permet pas. C'est le principe du **"Fail Fast"** : mieux vaut planter tout de suite avec un message clair que de continuer avec des données corrompues.

### 3. Comment

#### A. Lever une exception standard

```python
def set_age(age: int):
    if age < 0:
        raise ValueError("L'âge ne peut pas être négatif.")
    print(f"Âge défini à {age}")

try:
    set_age(-5)
except ValueError as e:
    print(f"Validation échouée : {e}")
```

#### B. Exceptions Personnalisées
Créez une classe héritant de `Exception` pour des erreurs métier spécifiques.

```python
class InsufficientFundsError(Exception):
    """Levée quand le solde est insuffisant pour un retrait."""
    pass

def withdraw(balance: float, amount: float):
    if amount > balance:
        raise InsufficientFundsError(f"Manque {amount - balance}€")
    return balance - amount
```

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-22}

1.  **Quelle est la différence entre `except Exception:` et `except:` ?**
    `except Exception` capture les erreurs standard. `except:` (nu) capture absolument tout, y compris les interruptions système, ce qui est dangereux.

2.  **Quand le bloc `finally` est-il exécuté ?**
    Toujours. Qu'une exception soit levée, qu'elle soit attrapée ou non, ou même si la fonction retourne une valeur avant.

3.  **À quoi sert le bloc `else` dans un `try-except` ?**
    Il contient le code à exécuter uniquement si le bloc `try` s'est déroulé sans aucune erreur.

4.  **Pourquoi créer ses propres classes d'exception ?**
    Pour représenter des erreurs métier spécifiques (ex: `EmailInvalideError`) et permettre au code appelant de les capturer sélectivement sans attraper toutes les `ValueError`.

---

## Exercices : {#exercices-22}

### Exercice 1 - La Calculatrice Robuste {#exercice-1-calculatrice-robuste}

🎯 **Objectif** : Gérer `ValueError` et `ZeroDivisionError`.

💼 **Mise en situation** : Vous développez l'interface d'une calculatrice simple. L'utilisateur peut entrer n'importe quoi, votre programme ne doit pas planter.

📝 **Énoncé** :
1.  Demandez deux nombres à l'utilisateur (`a` et `b`).
2.  Tentez de diviser `a` par `b`.
3.  Gérez le cas où l'utilisateur entre du texte (ValueError) : affichez "Entrée invalide".
4.  Gérez le cas où `b` est 0 (ZeroDivisionError) : affichez "Division par zéro impossible".
5.  Utilisez un bloc `else` pour afficher le résultat si tout va bien.
6.  Utilisez un bloc `finally` pour afficher "Calcul terminé" dans tous les cas.

📺 **Résultat attendu** :
*Cas 1 :* Entrée "10", "2" -> "Résultat : 5.0" puis "Calcul terminé"
*Cas 2 :* Entrée "10", "0" -> "Division par zéro impossible" puis "Calcul terminé"
*Cas 3 :* Entrée "dix", "2" -> "Entrée invalide" puis "Calcul terminé"

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
print("--- Calculatrice ---")

try:
    # Peut lever ValueError si l'utilisateur tape du texte
    a = float(input("Nombre a : "))
    b = float(input("Nombre b : "))
    
    # Peut lever ZeroDivisionError
    result = a / b

except ValueError:
    print("Erreur : Entrée invalide. Veuillez entrer des chiffres.")

except ZeroDivisionError:
    print("Erreur : Division par zéro impossible.")

else:
    # S'exécute uniquement si aucune erreur ci-dessus
    print(f"Résultat : {result}")

finally:
    # S'exécute toujours
    print("Calcul terminé.")
```
</details>

### Exercice 2 - Validation de Formulaire (Raise) {#exercice-2-validation-formulaire}

🎯 **Objectif** : Lever manuellement des erreurs de validation (`raise`).

💼 **Mise en situation** : Inscription utilisateur. Vous devez valider le pseudo et le mot de passe avant d'enregistrer.

📝 **Énoncé** :
1.  Créez une fonction `register_user(username, password)`.
2.  Si `username` fait moins de 3 caractères, levez une `ValueError` : "Pseudo trop court".
3.  Si `password` fait moins de 8 caractères, levez une `ValueError` : "Mot de passe faible".
4.  Si tout est bon, retournez "Utilisateur créé".
5.  Appelez cette fonction dans un bloc try/except pour tester les 3 cas (pseudo court, mdp court, succès).

📺 **Résultat attendu** :
```text
Test 1 (ab/12345678) -> Erreur : Pseudo trop court
Test 2 (alice/123) -> Erreur : Mot de passe faible
Test 3 (alice/12345678) -> Succès : Utilisateur créé
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
def register_user(username: str, password: str) -> str:
    # Validation Pseudo
    if len(username) < 3:
        raise ValueError("Pseudo trop court (min 3 chars)")
    
    # Validation Mot de passe
    if len(password) < 8:
        raise ValueError("Mot de passe faible (min 8 chars)")
    
    return "Utilisateur créé"

# Fonction de test pour éviter de répéter le try/except
def test_register(u, p):
    try:
        msg = register_user(u, p)
        print(f"Succès : {msg}")
    except ValueError as e:
        print(f"Erreur : {e}")

# Scénarios
test_register("ab", "12345678")     # Pseudo court
test_register("alice", "123")       # Password court
test_register("alice", "12345678")  # OK
```
</details>

### Exercice 3 - Retry Logic (Logique de Réessai) {#exercice-3-retry-logic}

🎯 **Objectif** : Utiliser `try-except` dans une boucle `while` pour insister jusqu'à réussite.

💼 **Mise en situation** : Un serveur instable échoue parfois à répondre. Il faut réessayer 3 fois maximum avant d'abandonner.

📝 **Énoncé** :
1.  Simulez une fonction `connect_server()` qui a 2 chances sur 3 de lever une `ConnectionError` (utilisez `random`).
2.  Écrivez une boucle qui tente de se connecter.
3.  Si ça marche : `break` (sortir de la boucle) et afficher "Connecté !".
4.  Si ça échoue : afficher "Échec tentative X/3..." et continuer.
5.  Si au bout de 3 essais ça échoue toujours, affichez "Abandon.".

📺 **Résultat attendu** (aléatoire) :
```text
Tentative 1...
Échec tentative 1/3...
Tentative 2...
Connecté !
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
import random

# Exception personnalisée pour la simulation
class ConnectionError(Exception):
    pass

def connect_server():
    print("Tentative de connexion...")
    # 66% de chance d'échouer
    if random.random() < 0.66:
        raise ConnectionError("Timeout")
    return True

max_retries = 3

for i in range(1, max_retries + 1):
    try:
        print(f"Essai {i}...")
        connect_server()
        print("✅ Connecté avec succès !")
        break # Sortie immédiate de la boucle
    except ConnectionError:
        print(f"❌ Échec tentative {i}/{max_retries}...")
else:
    # Ce bloc else appartient à la boucle for !
    # Il s'exécute si la boucle se termine sans 'break'
    print("⛔ Abandon : Serveur injoignable après 3 essais.")
```
</details>