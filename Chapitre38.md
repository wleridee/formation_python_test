---
sidebar_label: Module `copy` : Copie Superficielle et Profonde
sidebar_position: 38
---

# Chapitre 38 : Module `copy` : Copie Superficielle et Profonde

Opération de copie, Shallow copy, Deep copy, Impact sur les objets mutables

En Python, l'affectation standard (`a = b`) ne copie pas l'objet. Elle crée simplement une nouvelle **référence** (une étiquette) pointant vers le même objet en mémoire. Cela surprend souvent les débutants lorsqu'ils modifient une liste et réalisent qu'ils ont aussi modifié l'original !

Le module `copy` est essentiel pour gérer correctement la duplication des objets, en particulier lorsqu'ils contiennent d'autres objets (comme des listes de listes ou des objets de classe).

---

## 1. L'Affectation vs La Copie {#affectation-vs-copie}

### 1. Quoi
L'affectation avec `=` ne crée jamais de nouvel objet. Elle lie un nouveau nom à l'objet existant.

### 2. Pourquoi
Pour des raisons de performance. Python évite de dupliquer des données lourdes en mémoire tant que ce n'est pas explicitement demandé.

### 3. Comment

```python
# 1. Affectation simple (Aliasing)
original_list = [1, 2, 3]
new_ref = original_list

print(f"ID original: {id(original_list)}")
print(f"ID nouvelle ref: {id(new_ref)}") # Identique !

new_ref.append(4)
print(original_list) # [1, 2, 3, 4] -> L'original est modifié !
```

---

## 2. Copie Superficielle (Shallow Copy) {#copie-superficielle}

### 1. Quoi
Une **shallow copy** crée un *nouvel* objet conteneur, mais remplit ce conteneur avec des **références** vers les objets contenus dans l'original.
En d'autres termes : l'enveloppe est nouvelle, mais le contenu est partagé.

### 2. Pourquoi
C'est plus rapide et moins gourmand en mémoire qu'une copie complète. Suffisant si les objets contenus sont immuables (entiers, chaînes) ou si vous ne comptez pas modifier le contenu imbriqué.

### 3. Comment

#### A. Syntaxe de base

```python
import copy

original = [1, [2, 3], 4]
shallow = copy.copy(original) # Ou original[:] ou list(original)

# L'enveloppe est différente
print(original is shallow) # False

# Mais le contenu imbriqué est partagé !
print(original[1] is shallow[1]) # True

# Modification du niveau 1 (l'enveloppe) -> OK, pas d'impact
shallow.append(5)
print(original) # [1, [2, 3], 4]

# Modification du niveau 2 (l'objet imbriqué) -> DANGER
shallow[1][0] = 99
print(original) # [1, [99, 3], 4] -> L'original est touché !
```

### 4. Zone de Danger
❌ **Ne pas utiliser pour des structures imbriquées mutables** : Si vous copiez une liste de dictionnaires ou une liste de listes avec `copy.copy()`, modifier un dictionnaire interne modifiera aussi celui de la liste originale.

---

## 3. Copie Profonde (Deep Copy) {#copie-profonde}

### 1. Quoi
Une **deep copy** crée un nouvel objet conteneur, puis récursivement, crée des **copies** des objets trouvés dans l'original. Tout est dupliqué, jusqu'au dernier niveau.

### 2. Pourquoi
Pour obtenir une indépendance totale entre l'objet original et sa copie. Indispensable pour les configurations par défaut, les états de jeux, ou les snapshots de données complexes.

### 3. Comment

#### A. Syntaxe de base

```python
import copy

original = [1, [2, 3], 4]
deep = copy.deepcopy(original)

# Tout est différent
print(original is deep)      # False
print(original[1] is deep[1]) # False

# Modification du niveau 2 -> SÉCURISÉ
deep[1][0] = 99
print(original) # [1, [2, 3], 4] -> Intact
print(deep)     # [1, [99, 3], 4]
```

#### B. Cas concret : Configuration par défaut

```python
import copy

# Configuration par défaut de l'app (complexe)
DEFAULT_CONFIG = {
    "version": 1,
    "plugins": ["core"],
    "db": {
        "host": "localhost",
        "port": 5432
    }
}

def create_user_config(custom_db_host):
    # ❌ Si on fait juste : user_conf = DEFAULT_CONFIG
    # On risque de modifier la config globale pour tout le monde !
    
    # ✅ Deep copy pour isoler complètement la nouvelle config
    user_conf = copy.deepcopy(DEFAULT_CONFIG)
    user_conf["db"]["host"] = custom_db_host
    return user_conf

conf1 = create_user_config("192.168.1.10")
print(DEFAULT_CONFIG["db"]["host"]) # Toujours "localhost"
```

### 🚨 Limitations de `deepcopy`
1.  **Performance** : `deepcopy` peut être lent et coûteux en mémoire si l'objet est gigantesque.
2.  **Boucles infinies** : `deepcopy` gère les références circulaires (un objet qui se contient lui-même) correctement, mais c'est un mécanisme complexe.
3.  **Objets non copiables** : Certains objets comme les connexions réseau, les fichiers ouverts ou les threads ne peuvent pas être copiés (lève une `TypeError`).

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-38}

1.  **Quelle est la différence entre `list_a = list_b` et `list_a = copy.copy(list_b)` ?**
    La première instruction ne copie rien (crée un alias). La seconde crée une nouvelle liste (enveloppe), mais partage les éléments internes.

2.  **Si j'ai une liste d'entiers `[1, 2, 3]`, est-ce que `copy.copy` suffit pour être indépendant ?**
    Oui, car les entiers sont immuables. On ne peut pas modifier `1` pour qu'il devienne `2`. On remplace l'entier dans la liste, ce qui touche l'enveloppe (indépendante).

3.  **Pourquoi `deepcopy` est-il nécessaire pour une liste de listes ?**
    Car une liste est mutable. Si on modifie la sous-liste partagée via une copie superficielle, l'original est affecté. `deepcopy` duplique aussi les sous-listes.

4.  **Comment copier un dictionnaire simplement (shallow copy) sans importer `copy` ?**
    Avec la méthode `.copy()` du dictionnaire ou le constructeur `dict(original)`.

---

## Exercices : {#exercices-38}

### Exercice 1 - Le Piège de l'Inventaire {#exercice-1-piege-inventaire}

🎯 **Objectif** : Observer la différence entre affectation et copie.

💼 **Mise en situation** : Vous gérez l'inventaire de deux magasins. Le magasin B démarre avec le même stock que le magasin A.

📝 **Énoncé** :
1.  Créez une liste `stock_A` contenant ["Pommes", "Poires"].
2.  Créez `stock_B` en faisant `stock_B = stock_A`.
3.  Ajoutez "Bananes" à `stock_B`.
4.  Affichez `stock_A`. Que constatez-vous ?
5.  Corrigez le code pour que `stock_A` ne soit pas modifié, en utilisant une technique de copie superficielle (slice `[:]` ou `copy()`).

📺 **Résultat attendu** :
```text
Avant correction : stock_A contient Bananes (Aïe !)
Après correction : stock_A ne contient PAS Bananes.
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
import copy

print("--- AVANT CORRECTION ---")
stock_A = ["Pommes", "Poires"]
# Affectation : même objet en mémoire
stock_B = stock_A 

stock_B.append("Bananes")
print(f"Stock A : {stock_A}") # Contient Bananes !

print("\n--- APRÈS CORRECTION ---")
stock_A = ["Pommes", "Poires"]
# Copie superficielle (les strings sont immuables, donc c'est safe ici)
stock_B = copy.copy(stock_A) # Ou stock_A[:]

stock_B.append("Bananes")
print(f"Stock A : {stock_A}") # Intact
print(f"Stock B : {stock_B}")
```
</details>

### Exercice 2 - Clonage de Prototypes (Deep Copy) {#exercice-2-clonage-prototypes}

🎯 **Objectif** : Manipuler des structures imbriquées mutables.

💼 **Mise en situation** : Vous construisez un jeu de stratégie. Vous avez un modèle d'unité "Soldat" avec un équipement standard. Chaque nouveau soldat doit avoir son propre sac à dos, pas partager celui du modèle !

📝 **Énoncé** :
1.  Définissez un dictionnaire `prototype_soldier` :
    ```python
    {
        "type": "Infantry",
        "stats": {"hp": 100, "atk": 10},
        "inventory": ["Ration", "Bandage"]
    }
    ```
2.  Créez `soldier_1` avec une `copy.copy()` du prototype.
3.  Créez `soldier_2` avec une `copy.deepcopy()` du prototype.
4.  Ajoutez "Grenade" à l'inventaire de `soldier_1`.
5.  Vérifiez si l'inventaire du prototype a changé.
6.  Ajoutez "Medkit" à l'inventaire de `soldier_2`. Vérifiez le prototype.

📺 **Résultat attendu** :
```text
Après modif soldier_1 (shallow) : Prototype a maintenant une Grenade (Problème !)
Après modif soldier_2 (deep) : Prototype n'a pas de Medkit (Correct !)
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
import copy

prototype_soldier = {
    "type": "Infantry",
    "stats": {"hp": 100, "atk": 10},
    "inventory": ["Ration", "Bandage"]
}

print("--- TEST SHALLOW COPY ---")
soldier_1 = copy.copy(prototype_soldier)
# On modifie la liste mutable imbriquée
soldier_1["inventory"].append("Grenade")

print(f"Inventaire Soldat 1 : {soldier_1['inventory']}")
print(f"Inventaire Prototype : {prototype_soldier['inventory']}")
print("Le prototype a été affecté ! 😱")

# Reset pour le test 2
prototype_soldier["inventory"].remove("Grenade")

print("\n--- TEST DEEP COPY ---")
soldier_2 = copy.deepcopy(prototype_soldier)
soldier_2["inventory"].append("Medkit")

print(f"Inventaire Soldat 2 : {soldier_2['inventory']}")
print(f"Inventaire Prototype : {prototype_soldier['inventory']}")
print("Le prototype est sauf. ✅")
```
</details>

### Exercice 3 - Copie d'Objets Personnalisés {#exercice-3-objets-custom}

🎯 **Objectif** : Voir le comportement de copy sur des instances de classe.

💼 **Mise en situation** : Vous gérez des projets avec une liste de tâches.

📝 **Énoncé** :
1.  Créez une classe `Project` avec un attribut `name` (str) et `tasks` (list).
2.  Instanciez `p1` ("Site Web", ["Design", "Dev"]).
3.  Créez `p2` comme une `deepcopy` de `p1`.
4.  Ajoutez "Test" aux tâches de `p2` et changez le nom de `p2` en "App Mobile".
5.  Affichez les deux projets pour prouver leur indépendance totale.

📺 **Résultat attendu** :
```text
P1: Site Web - ['Design', 'Dev']
P2: App Mobile - ['Design', 'Dev', 'Test']
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
import copy

class Project:
    def __init__(self, name, tasks):
        self.name = name
        self.tasks = tasks

    def __repr__(self):
        return f"{self.name} - {self.tasks}"

# 1. Création
p1 = Project("Site Web", ["Design", "Dev"])

# 2. Copie profonde
p2 = copy.deepcopy(p1)

# 3. Modification de la copie
p2.name = "App Mobile"     # Modifie l'attribut str (indépendant par nature car immuable/réassigné)
p2.tasks.append("Test")    # Modifie la liste mutable (indépendante grâce au deepcopy)

# 4. Vérification
print(f"P1: {p1}")
print(f"P2: {p2}")
```
</details>