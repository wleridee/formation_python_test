---
sidebar_label: Introduction à pytest
sidebar_position: 46
---

# Chapitre 46 : Introduction à `pytest`

Installation de pytest, Tests simplifiés, Fixtures, Paramétrisation des tests

Si `unittest` est l'outil standard, **`pytest`** est le favori incontesté de la communauté Python. Plus simple, plus puissant, plus lisible, il élimine le code superflu (boilerplate) pour vous laisser vous concentrer sur ce qui compte : vérifier que votre code fonctionne.

Une fois que vous aurez goûté à `assert resultat == attendu` au lieu de `self.assertEqual(resultat, attendu)`, vous ne voudrez plus revenir en arrière.

---

## 1. Pytest : La Révolution de la Simplicité {#pytest-simplicite}

### 1. Quoi
`pytest` est un framework de test tiers (doit être installé via pip) qui permet d'écrire des tests unitaires et fonctionnels de manière extrêmement concise. Il est compatible avec les tests écrits pour `unittest`, mais offre une syntaxe beaucoup plus légère.

### 2. Pourquoi
*   **Pas de classes obligatoires** : De simples fonctions suffisent.
*   **Assertions natives** : Utilise le mot-clé standard `assert` de Python.
*   **Rapports d'erreurs détaillés** : Pytest dissèque vos échecs et montre exactement *pourquoi* `assert` a échoué (valeurs comparées, différences de listes, etc.).

### 3. Comment

#### A. Installation et Premier Test

```bash
# Installation
pip install pytest
```

```python
# test_demo.py

def addition(x, y):
    return x + y

# Pas besoin de classe, ni de self !
def test_addition():
    assert addition(1, 2) == 3
    assert addition(-1, 1) == 0
```

Pour lancer les tests, exécutez simplement `pytest` dans votre terminal. Il découvrira automatiquement tous les fichiers `test_*.py` ou `*_test.py`.

#### B. Rapport d'erreur intelligent

Si un test échoue :
```python
def test_echec():
    assert addition(1, 2) == 5
```
Pytest affichera :
```text
>       assert addition(1, 2) == 5
E       assert 3 == 5
E        +  where 3 = addition(1, 2)
```
Il vous montre la valeur calculée (`3`) face à la valeur attendue (`5`).

### 4. Zone de Danger
❌ **Nommage des fichiers** : Pytest ne trouvera PAS vos tests si le fichier ne commence pas par `test_` ou ne finit pas par `_test.py`.
✅ **Configuration** : Créez un fichier `pytest.ini` ou `pyproject.toml` à la racine pour configurer les options par défaut (ex: `addopts = -v`).

---

## 2. Les Fixtures : L'Injection de Dépendances {#fixtures}

### 1. Quoi
Les **Fixtures** remplacent les méthodes `setUp` et `tearDown` de `unittest`. Ce sont des fonctions décorées par `@pytest.fixture` qui retournent des données ou des objets. Vous les "injectez" dans vos tests simplement en les ajoutant comme arguments de la fonction de test.

### 2. Pourquoi
*   **Modularité** : Une fixture peut utiliser d'autres fixtures.
*   **Portée (Scope)** : On peut définir une fixture qui ne s'exécute qu'une fois par session, par module, ou par fonction.
*   **Clarté** : On voit immédiatement de quoi un test a besoin (ex: `def test_login(browser, database):`).

### 3. Comment

#### A. Syntaxe de base

```python
import pytest

# Définition de la fixture
@pytest.fixture
def utilisateur_test():
    """Retourne un dictionnaire utilisateur simulé."""
    return {"username": "alice", "role": "admin", "active": True}

# Injection dans le test
def test_droit_admin(utilisateur_test):
    # 'utilisateur_test' contient le retour de la fonction fixture
    assert utilisateur_test["role"] == "admin"

def test_est_actif(utilisateur_test):
    assert utilisateur_test["active"] is True
```

#### B. Setup et Teardown avec `yield`

Si vous devez nettoyer des ressources (fermer un fichier, supprimer une DB), utilisez `yield` au lieu de `return`. Tout ce qui est après `yield` s'exécute après le test.

```python
import pytest
import os

@pytest.fixture
def fichier_temporaire():
    # Setup
    nom_fichier = "temp_test.txt"
    with open(nom_fichier, "w") as f:
        f.write("Données initiales")
    
    # Passe le contrôle au test
    yield nom_fichier
    
    # Teardown (Nettoyage)
    if os.path.exists(nom_fichier):
        os.remove(nom_fichier)
        print("🧹 Fichier nettoyé")

def test_lecture_fichier(fichier_temporaire):
    with open(fichier_temporaire, "r") as f:
        contenu = f.read()
    assert contenu == "Données initiales"
```

---

## 3. Paramétrisation : Éviter la Répétition {#parametrisation}

### 1. Quoi
La paramétrisation permet d'exécuter **le même test** plusieurs fois avec des **données d'entrée différentes**.

### 2. Pourquoi
Pour éviter de copier-coller 10 fois la même fonction de test juste pour changer les arguments. C'est l'équivalent moderne et lisible de `subTest` ou des boucles dans les tests.

### 3. Comment

#### A. Décorateur `@pytest.mark.parametrize`

```python
import pytest

def est_pair(n):
    return n % 2 == 0

# On définit les noms des arguments, puis une liste de tuples de valeurs
@pytest.mark.parametrize("nombre, attendu", [
    (2, True),
    (4, True),
    (3, False),
    (0, True),
    (-2, True),
    (11, False)
])
def test_est_pair(nombre, attendu):
    # Ce test sera exécuté 6 fois
    assert est_pair(nombre) == attendu
```

L'exécution (`pytest -v`) affichera chaque cas comme un test distinct :
```text
test_est_pair[2-True] PASSED
test_est_pair[4-True] PASSED
test_est_pair[3-False] PASSED...
```

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-46}

1.  **Quelle est la commande pour lancer tous les tests du projet ?**
    `pytest` (à la racine du projet).

2.  **Comment injecter une fixture dans un test ?**
    Il suffit d'ajouter le nom de la fonction fixture comme argument de la fonction de test. Pytest fait le lien automatiquement.

3.  **Quelle est la différence entre `return` et `yield` dans une fixture ?**
    `return` termine la fixture (setup uniquement). `yield` permet de suspendre la fixture, exécuter le test, puis reprendre l'exécution pour faire le nettoyage (setup + teardown).

4.  **Comment vérifier qu'une exception est levée avec pytest ?**
    Avec le contexte manager `with pytest.raises(ExceptionType):`.

---

## Exercices : {#exercices-46}

### Exercice 1 - Le Convertisseur de Devises {#exercice-1-convertisseur-devises}

🎯 **Objectif** : Utiliser `parametrize` pour couvrir de nombreux cas.

💼 **Mise en situation** : Vous développez une fonction `convertir(montant, taux)` pour un site de voyage. Elle doit gérer les arrondis à 2 décimales.

📝 **Énoncé** :
1.  Fonction `convertir(montant, taux)` qui retourne `round(montant * taux, 2)`.
2.  Créez un test paramétré couvrant :
    - Conversion simple (10 * 1.2 -> 12.0)
    - Arrondi supérieur (10 * 1.255 -> 12.55 ou 12.56 selon la stratégie de round)
    - Montant zéro
    - Taux négatif (la fonction doit l'accepter mathématiquement pour cet exercice)
3.  Lancez `pytest -v`.

📺 **Résultat attendu** :
```text
test_convertir[10-1.2-12.0] PASSED
test_convertir[0-1.5-0.0] PASSED
...
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
import pytest

def convertir(montant, taux):
    return round(montant * taux, 2)

@pytest.mark.parametrize("montant, taux, resultat_attendu", [
    (100, 1.2, 120.0),       # Cas standard
    (10, 1.255, 12.55),      # Cas d'arrondi (Note: round(x.5) va vers le pair le plus proche en Python 3)
    (0, 50.0, 0.0),          # Zéro
    (100, -1.0, -100.0)      # Négatif
])
def test_convertir(montant, taux, resultat_attendu):
    assert convertir(montant, taux) == resultat_attendu
```
</details>

### Exercice 2 - Fixture de Base de Données (Mock) {#exercice-2-fixture-db}

🎯 **Objectif** : Comprendre le cycle de vie d'une fixture (Setup/Teardown).

💼 **Mise en situation** : Vos tests ont besoin d'une connexion à une "base de données" (ici un simple dictionnaire) qui doit être propre (vide) avant chaque test et fermée après.

📝 **Énoncé** :
1.  Créez une classe `MockDB` avec `connect()`, `disconnect()`, `insert(k, v)` et `get(k)`.
2.  Créez une fixture `db` qui instancie `MockDB`, connecte, `yield` l'instance, puis déconnecte.
3.  Écrivez deux tests utilisant cette fixture :
    - Test A : Insère une donnée et vérifie sa présence.
    - Test B : Vérifie que la base est bien vide (prouvant que c'est une NOUVELLE instance ou qu'elle a été nettoyée).

📺 **Résultat attendu** :
```text
[DB] Connexion...
test_insert PASSED
[DB] Déconnexion...
[DB] Connexion...
test_empty PASSED
[DB] Déconnexion...
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
import pytest

class MockDB:
    def __init__(self):
        self.data = {}
        self.connected = False
        
    def connect(self):
        self.connected = True
        print("\n[DB] Connexion...")
        
    def disconnect(self):
        self.connected = False
        self.data = {}
        print("[DB] Déconnexion...")
        
    def insert(self, key, value):
        if not self.connected: raise ConnectionError
        self.data[key] = value
        
    def get(self, key):
        if not self.connected: raise ConnectionError
        return self.data.get(key)

@pytest.fixture
def db():
    # Setup
    database = MockDB()
    database.connect()
    
    # Injection
    yield database
    
    # Teardown
    database.disconnect()

def test_insert(db):
    db.insert("user:1", "Alice")
    assert db.get("user:1") == "Alice"

def test_empty(db):
    # Ce test doit réussir car db est une nouvelle instance
    assert db.data == {}
```
</details>

### Exercice 3 - Exceptions et Marqueurs {#exercice-3-exceptions-markers}

🎯 **Objectif** : Gérer les erreurs attendues et catégoriser les tests.

💼 **Mise en situation** : Une fonction `diviser(a, b)` doit lever `ValueError` si `b` est 0. De plus, ce test est critique (smoke test).

📝 **Énoncé** :
1.  Fonction `diviser(a, b)`: si `b == 0` raise `ValueError("Div par zero")`, sinon `a / b`.
2.  Test `test_div_zero` : utilisez `with pytest.raises(ValueError) as excinfo` pour vérifier le type d'exception ET le message ("Div par zero").
3.  Ajoutez un marqueur personnalisé `@pytest.mark.smoke` sur ce test (nécessite une petite config ou juste l'usage dans le code).

📺 **Résultat attendu** :
```text
pytest -v -m smoke
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
import pytest

def diviser(a, b):
    if b == 0:
        raise ValueError("Division par zéro interdite")
    return a / b

@pytest.mark.smoke
def test_div_zero():
    # On capture l'exception
    with pytest.raises(ValueError) as excinfo:
        diviser(10, 0)
    
    # On vérifie le message d'erreur
    assert "Division par zéro interdite" in str(excinfo.value)

def test_div_classique():
    assert diviser(10, 2) == 5.0
```
</details>