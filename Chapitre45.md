---
sidebar_label: Tests Unitaires avec unittest
sidebar_position: 45
---

# Chapitre 45 : Tests Unitaires avec `unittest`

Framework unittest, Cas de test, Assertions, Exécution des tests

Écrire du code est une chose ; s'assurer qu'il fonctionne (et qu'il continuera de fonctionner après modification) en est une autre. Les **tests unitaires** sont le filet de sécurité du développeur. Ils vérifient automatiquement que chaque petite partie de votre code (unité) se comporte comme prévu.

Python inclut dans sa bibliothèque standard un framework de test puissant et mature : `unittest`, inspiré par JUnit de Java. Même si des alternatives comme `pytest` existent, `unittest` reste un standard de l'industrie, présent partout, sans installation tierce.

---

## 1. La Structure d'un Test : TestCase et Méthodes {#structure-testcase}

### 1. Quoi
Pour écrire des tests avec `unittest`, vous devez créer une classe qui hérite de `unittest.TestCase`. Chaque méthode de cette classe commençant par le préfixe `test_` est considérée comme un test indépendant à exécuter.

### 2. Pourquoi
L'approche orientée objet permet de regrouper les tests liés logiquement, de partager du code de configuration (setup) et de bénéficier de nombreuses méthodes d'assertion intégrées.

### 3. Comment

#### A. Syntaxe de base

```python
import unittest

# La fonction à tester
def addition(a: int, b: int) -> int:
    return a + b

# La classe de test
class TestCalculs(unittest.TestCase):
    
    def test_addition_simple(self):
        # Vérifie que 1 + 1 = 2
        self.assertEqual(addition(1, 1), 2)
        
    def test_addition_negatifs(self):
        # Vérifie que -1 + -1 = -2
        self.assertEqual(addition(-1, -1), -2)

if __name__ == "__main__":
    unittest.main()
```

### 4. Zone de Danger
❌ **Nommage des méthodes** : Si votre méthode s'appelle `verifier_addition` (sans `test_` au début), `unittest` l'ignorera silencieusement.
✅ **Indépendance** : Chaque test doit être totalement indépendant. L'ordre d'exécution n'est pas garanti. Ne jamais compter sur l'état laissé par `test_A` pour faire fonctionner `test_B`.

---

## 2. Les Assertions : Le Cœur du Test {#assertions}

### 1. Quoi
Les assertions sont des méthodes fournies par `unittest.TestCase` pour vérifier qu'une condition est vraie. Si l'assertion échoue, le test est marqué comme "FAILED".

### 2. Pourquoi
Contrairement à un simple `assert` Python, les méthodes `self.assertX` fournissent des messages d'erreur détaillés et contextuels ("Expected X, got Y") qui facilitent le débogage.

### 3. Comment

#### D. Tableau des Assertions Courantes

| Méthode | Vérifie que... | Usage Typique |
| :--- | :--- | :--- |
| `assertEqual(a, b)` | `a == b` | Valeurs de retour, calculs |
| `assertNotEqual(a, b)` | `a != b` | Changement d'état |
| `assertTrue(x)` | `bool(x) is True` | Validation booléenne |
| `assertIn(item, list)` | `item in list` | Présence dans une collection |
| `assertIsInstance(a, b)` | `isinstance(a, b)` | Vérification de type |
| `assertRaises(Error, func)` | `func` lève `Error` | Gestion d'erreurs |

#### B. Exemple complet

```python
import unittest

class UserManager:
    def __init__(self):
        self.users = []
    
    def add_user(self, name):
        if not name:
            raise ValueError("Nom vide interdit")
        self.users.append(name)

class TestUserManager(unittest.TestCase):
    
    def test_ajout_utilisateur(self):
        manager = UserManager()
        manager.add_user("Alice")
        
        # Vérifie la taille de la liste
        self.assertEqual(len(manager.users), 1)
        # Vérifie la présence
        self.assertIn("Alice", manager.users)
        
    def test_erreur_nom_vide(self):
        manager = UserManager()
        
        # Vérifie qu'une exception est bien levée
        with self.assertRaises(ValueError):
            manager.add_user("")
```

---

## 3. Fixtures : setUp et tearDown {#fixtures-setup-teardown}

### 1. Quoi
Les méthodes `setUp()` et `tearDown()` sont des crochets (hooks) spéciaux exécutés automatiquement **avant** et **après** CHAQUE test de la classe.

### 2. Pourquoi
Pour préparer un environnement propre (connexion DB, création de fichiers temporaires, instanciation d'objets) avant chaque test, et nettoyer après, garantissant l'isolation totale des tests ("Hermetic Testing").

### 3. Comment

```python
import unittest
import tempfile
import os
import shutil

class TestFileOperations(unittest.TestCase):
    
    def setUp(self):
        """Exécuté AVANT chaque test"""
        # Création d'un dossier temporaire isolé
        self.test_dir = tempfile.mkdtemp()
        print(f"\n📁 Création environnement: {self.test_dir}")
        
    def tearDown(self):
        """Exécuté APRÈS chaque test"""
        # Nettoyage pour ne laisser aucune trace
        shutil.rmtree(self.test_dir)
        print("🧹 Nettoyage terminé")
        
    def test_creation_fichier(self):
        path = os.path.join(self.test_dir, "test.txt")
        with open(path, "w") as f:
            f.write("Hello")
        
        self.assertTrue(os.path.exists(path))
        
    def test_ecriture_fichier(self):
        # Ce test part d'un dossier VIDE, grâce au setUp/tearDown
        path = os.path.join(self.test_dir, "log.txt")
        # ... logique de test ...
```

---

## 4. Exécution et Découverte des Tests {#execution-decouverte}

### 1. Quoi
Comment lancer vos tests ? Vous pouvez lancer un fichier spécifique, ou laisser `unittest` découvrir automatiquement tous les fichiers de test d'un projet.

### 2. Pourquoi
Pour l'intégration continue (CI/CD), il est crucial de pouvoir lancer toute la suite de tests en une seule commande.

### 3. Comment

*   **Lancer un fichier spécifique :**
    ```bash
    python -m unittest tests/test_mon_module.py
    ```

*   **Auto-découverte (Discovery) :**
    Parcourt récursivement les dossiers pour trouver les fichiers `test*.py`.
    ```bash
    python -m unittest discover
    ```

*   **Mode verbeux (recommandé) :**
    Affiche le nom de chaque test exécuté.
    ```bash
    python -m unittest -v
    ```

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-45}

1.  **Quelle méthode est appelée avant chaque test individuel ?**
    `setUp()`. Elle sert à initialiser l'état nécessaire au test.

2.  **Comment vérifier qu'une fonction lève bien une exception spécifique ?**
    En utilisant le contexte manager `with self.assertRaises(ExceptionType):`.

3.  **Pourquoi les tests doivent-ils être indépendants les uns des autres ?**
    Pour éviter les effets de bord (side effects) où un test échoue non pas à cause d'un bug, mais à cause de l'état "sale" laissé par le test précédent. Cela rend le débogage impossible.

4.  **Si j'ai une méthode `def check_login(self):` dans ma classe de test, sera-t-elle exécutée ?**
    Non, car elle ne commence pas par `test_`. C'est utile pour créer des méthodes utilitaires (helpers) internes.

---

## Exercices : {#exercices-45}

### Exercice 1 - Le Testeur de Panier E-commerce {#exercice-1-panier-ecommerce}

🎯 **Objectif** : Tester une classe métier simple avec `setUp`.

💼 **Mise en situation** : Vous gérez le panier d'un site marchand. Vous devez garantir que le calcul du total et l'ajout d'articles fonctionnent sans faille avant la mise en prod.

📝 **Énoncé** :
1.  Créez une classe `Cart` avec une méthode `add(price)` et une propriété `total`.
2.  Écrivez une classe de test `TestCart`.
3.  Utilisez `setUp` pour instancier un nouveau `Cart` avant chaque test.
4.  Testez :
    - Un panier vide (total = 0).
    - L'ajout d'un article.
    - L'ajout de plusieurs articles.
    - L'ajout d'un prix négatif (doit lever `ValueError`).

📺 **Résultat attendu** :
```text
test_add_items ... OK
test_empty_cart ... OK
test_negative_price ... OK
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
import unittest

# --- Code Métier ---
class Cart:
    def __init__(self):
        self.items = []
        
    def add(self, price):
        if price < 0:
            raise ValueError("Le prix ne peut pas être négatif")
        self.items.append(price)
        
    @property
    def total(self):
        return sum(self.items)

# --- Tests ---
class TestCart(unittest.TestCase):
    
    def setUp(self):
        """Crée un panier neuf avant chaque test."""
        self.cart = Cart()
        
    def test_empty_cart(self):
        self.assertEqual(self.cart.total, 0)
        
    def test_add_items(self):
        self.cart.add(10)
        self.cart.add(20.5)
        self.assertEqual(self.cart.total, 30.5)
        
    def test_negative_price(self):
        # Vérifie que l'ajout d'un prix négatif lève une erreur
        with self.assertRaises(ValueError):
            self.cart.add(-5)

if __name__ == '__main__':
    unittest.main(verbosity=2)
```
</details>

### Exercice 2 - Validateur d'Emails (TDD) {#exercice-2-validateur-emails}

🎯 **Objectif** : Pratiquer le Test Driven Development (écrire le test *avant* le code).

💼 **Mise en situation** : On vous demande une fonction `is_valid_email(email)`. Au lieu de coder la regex tout de suite, écrivez d'abord les cas de tests.

📝 **Énoncé** :
1.  Créez `TestEmailValidator`.
2.  Définissez 3 tests :
    - `test_valid`: "user@example.com" -> True
    - `test_no_at`: "userexample.com" -> False
    - `test_empty`: "" -> False
3.  Implémentez ensuite la fonction `is_valid_email` la plus simple possible pour faire passer les tests (une simple vérification de la présence de "@" suffira pour cet exercice).

📺 **Résultat attendu** :
```text
Ran 3 tests in 0.001s
OK
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
import unittest

# --- Code Métier (écrit APRES les tests dans la logique TDD) ---
def is_valid_email(email):
    if not email:
        return False
    if "@" not in email:
        return False
    return True

# --- Tests ---
class TestEmailValidator(unittest.TestCase):
    
    def test_valid(self):
        self.assertTrue(is_valid_email("alice@example.com"))
        
    def test_no_at(self):
        self.assertFalse(is_valid_email("alice.example.com"))
        
    def test_empty(self):
        self.assertFalse(is_valid_email(""))
        self.assertFalse(is_valid_email(None))

if __name__ == '__main__':
    unittest.main()
```
</details>

### Exercice 3 - Le FizzBuzz Testé {#exercice-3-fizzbuzz}

🎯 **Objectif** : Utiliser `unittest` sur un algorithme classique.

💼 **Mise en situation** : Vous recrutez des stagiaires et voulez automatiser la correction de l'exercice FizzBuzz.

📝 **Énoncé** :
1.  Fonction `fizzbuzz(n)` : retourne "Fizz" si multiple de 3, "Buzz" si multiple de 5, "FizzBuzz" si multiple de 15, sinon le nombre en string.
2.  Écrivez les tests couvrant les 4 cas.
3.  Utilisez `subTest` (fonctionnalité avancée de unittest) pour boucler sur plusieurs valeurs d'entrée sans arrêter le test au premier échec.

📺 **Résultat attendu** :
```text
Tous les cas (3, 5, 15, 7) doivent passer.
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
import unittest

def fizzbuzz(n):
    if n % 15 == 0:
        return "FizzBuzz"
    if n % 3 == 0:
        return "Fizz"
    if n % 5 == 0:
        return "Buzz"
    return str(n)

class TestFizzBuzz(unittest.TestCase):
    
    def test_fizz(self):
        # Utilisation classique
        self.assertEqual(fizzbuzz(3), "Fizz")
        self.assertEqual(fizzbuzz(9), "Fizz")
        
    def test_buzz(self):
        self.assertEqual(fizzbuzz(5), "Buzz")
        self.assertEqual(fizzbuzz(20), "Buzz")
        
    def test_fizzbuzz(self):
        self.assertEqual(fizzbuzz(15), "FizzBuzz")
        self.assertEqual(fizzbuzz(45), "FizzBuzz")
        
    def test_numbers(self):
        # Utilisation de subtest pour des tests paramétrés propres
        cases = [(1, "1"), (2, "2"), (7, "7")]
        for n, expected in cases:
            with self.subTest(n=n):
                self.assertEqual(fizzbuzz(n), expected)

if __name__ == '__main__':
    unittest.main()
```
</details>