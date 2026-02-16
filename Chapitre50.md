---
sidebar_label: Débogage et Profilage de Code
sidebar_position: 50
---

# Chapitre 50 : Débogage et Profilage de Code

pdb (débogueur intégré), Breakpoints, Outils de profilage (cProfile), Optimisation des performances

Écrire du code est une chose, comprendre pourquoi il ne fonctionne pas (ou pourquoi il est lent) en est une autre. Avant ce chapitre, votre meilleur ami était peut-être `print("ICI")`. Il est temps de passer à des outils professionnels.

Python 3.14 offre des outils natifs puissants pour inspecter l'exécution pas à pas (**débogage**) et mesurer précisément où le temps est dépensé (**profilage**).

---

## 1. Le Débogueur Interactif (`pdb` / `breakpoint`) {#debogueur-interactif}

### 1. Quoi
Un débogueur permet de mettre en pause l'exécution d'un programme à une ligne précise, d'inspecter la valeur des variables à cet instant T, et d'avancer ligne par ligne. Python intègre le module `pdb` (Python Debugger).

### 2. Pourquoi
L'approche "Print Debugging" (ajouter des `print` partout) est :
1.  **Lente** : Il faut modifier le code, relancer, nettoyer après.
2.  **Limitée** : On ne voit que ce qu'on a prévu d'afficher.
3.  **Risquée** : On risque de laisser des `print` en production.

Le débogueur permet une exploration interactive immédiate.

### 3. Comment

#### A. Syntaxe de base (Moderne)
Depuis Python 3.7, une fonction native (built-in) existe pour invoquer le débogueur :

```python
def calculer_ratio(a: int, b: int) -> float:
    resultat = 0
    # Le programme se mettra en PAUSE ici
    breakpoint() 
    resultat = a / b
    return resultat

calculer_ratio(10, 0)
```

#### B. Commandes essentielles de `pdb`
Une fois le programme en pause, le terminal affiche `(Pdb)`. Voici les commandes de survie :

| Commande | Raccourci | Action |
| :--- | :--- | :--- |
| `next` | **n** | Exécuter la ligne actuelle et passer à la suivante. |
| `step` | **s** | "Rentrer" dans la fonction appelée sur la ligne actuelle. |
| `continue` | **c** | Reprendre l'exécution normale jusqu'au prochain breakpoint ou la fin. |
| `list` | **l** | Afficher le code source autour de la ligne actuelle. |
| `print` | **p** | Afficher la valeur d'une variable (ex: `p ma_var`). |
| `quit` | **q** | Quitter brutalement le débogueur et le programme. |

#### C. Exemple Pratique de Session
Imaginez que vous lancez le script ci-dessus.

```text
> script.py(5)calculer_ratio()
-> resultat = a / b
(Pdb) p a
10
(Pdb) p b
0
(Pdb) p resultat
0
(Pdb) n
ZeroDivisionError: division by zero
> script.py(5)calculer_ratio()
-> resultat = a / b
```
*Analyse : On voit immédiatement que `b` vaut 0 avant que l'erreur ne soit levée.*

### 4. Zone de Danger
❌ **Laisser `breakpoint()` en production** : Votre application web ou votre script s'arrêtera net et attendra une entrée clavier qui n'arrivera jamais. Le serveur "hangera" indéfiniment.
✅ **Variable d'environnement** : En Python 3.14, définir `PYTHONBREAKPOINT=0` dans l'environnement désactive tous les appels à `breakpoint()`. C'est une sécurité utile pour la prod.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Session de débogage dans un terminal VS Code.
> **Annotation** : Montrez le prompt `(Pdb)` et une inspection de variable.
> **Alt Text suggéré** : Terminal montrant une session interactive pdb avec inspection de variables.

---

## 2. Profilage avec `cProfile` {#profilage-cprofile}

### 1. Quoi
Le **profilage** (Profiling) consiste à mesurer le temps d'exécution de chaque fonction de votre programme. `cProfile` est le profiler déterministe standard de Python (écrit en C pour un impact minimal).

### 2. Pourquoi
**"L'optimisation prématurée est la racine de tous les maux"** (Donald Knuth).
Ne devinez jamais pourquoi votre code est lent ("C'est sûrement la base de données !"). Mesurez-le. Souvent, la lenteur vient d'une boucle Python inefficace ou d'une opération répétée inutilement.

### 3. Comment

#### A. En ligne de commande (Le plus simple)
Pas besoin de modifier votre code. Lancez votre script via le module `cProfile`.

```bash
# -s time : Trie les résultats par temps d'exécution (cumulé)
python -m cProfile -s time mon_script_lent.py
```

#### B. Interprétation du rapport
Le résultat ressemble à ceci :

```text
   200005 function calls in 1.450 seconds

   Ordered by: internal time

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    1.200    1.200    1.450    1.450 mon_script.py:10(gros_calcul)
   100000    0.200    0.000    0.200    0.000 {built-in method builtins.sum}
        1    0.050    0.050    1.450    1.450 mon_script.py:1(<module>)
```

*   **ncalls** : Nombre d'appels.
*   **tottime** : Temps passé DANS la fonction (sans compter les sous-fonctions appelées). **C'est la colonne la plus importante pour trouver les goulots d'étranglement.**
*   **cumtime** : Temps cumulé (fonction + sous-fonctions).

#### C. Dans le code (Pour cibler une fonction)
```python
import cProfile
import pstats

def main():
    # ... code complexe ...
    pass

if __name__ == "__main__":
    with cProfile.Profile() as pr:
        main()
    
    # Formatage propre des stats
    stats = pstats.Stats(pr)
    stats.sort_stats(pstats.SortKey.TIME)
    stats.print_stats(10) # Affiche le top 10
```

---

## 3. Optimisation des Performances {#optimisation-performances}

### 1. Quoi
Une fois le goulot d'étranglement identifié par `cProfile`, on applique des techniques pour réduire la complexité temporelle ou spatiale.

### 2. Pourquoi
Pour réduire les coûts d'infrastructure (CPU/RAM) et améliorer l'expérience utilisateur (latence).

### 3. Comment

#### A. Complexité Algorithmique (Big O)
C'est le levier le plus puissant. Passer de O(N) à O(1) change tout.

*   **Recherche dans une Liste** : O(N) (doit parcourir toute la liste).
*   **Recherche dans un Set/Dict** : O(1) (accès instantané par hash).

**Exemple :**
```python
# ❌ LENT (Si 'interdits' contient 1M d'éléments)
interdits = [i for i in range(1_000_000)] # Liste
def verifier(user_id):
    if user_id in interdits: # O(N)
        pass

# ✅ RAPIDE
interdits_set = set(interdits) # Conversion unique
def verifier(user_id):
    if user_id in interdits_set: # O(1)
        pass
```

#### B. Préférer les Built-ins
Les fonctions natives (`sum`, `map`, `max`, list comprehensions) sont implémentées en C. Elles sont presque toujours plus rapides que des boucles `for` Python manuelles.

```python
# ❌ Lent
total = 0
for x in ma_liste:
    total += x

# ✅ Rapide
total = sum(ma_liste)
```

#### C. Variables Locales vs Globales
L'accès aux variables locales est plus rapide que l'accès aux globales ou aux attributs d'objets (moins de recherche dans les dictionnaires internes).

### 4. Zone de Danger
❌ **Micro-optimisations illisibles** : Remplacer une fonction claire par une lambda complexe pour gagner 0.001s est une mauvaise idée si personne ne peut relire le code. La lisibilité prime, sauf preuve (profiling) du contraire.

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-50}

1.  **Quelle fonction native permet de lancer le débogueur interactif dans le code ?**
    `breakpoint()`.

2.  **Dans `pdb`, quelle est la différence entre `next` (n) et `step` (s) ?**
    `next` exécute la ligne courante entièrement. `step` tente d'entrer à l'intérieur de la fonction appelée sur la ligne courante.

3.  **Dans un rapport `cProfile`, quelle colonne indique le temps passé *uniquement* dans le code de la fonction (hors appels enfants) ?**
    `tottime`.

4.  **Pourquoi la recherche `item in my_set` est-elle plus rapide que `item in my_list` ?**
    Les Sets utilisent une table de hachage (complexité O(1)) alors que les Listes nécessitent un parcours séquentiel (complexité O(N)).

---

## Exercices : {#exercices-50}

### Exercice 1 - Enquête au Breakpoint {#exercice-1-enquete-breakpoint}

🎯 **Objectif** : Utiliser `pdb` pour trouver une erreur logique sans modifier le code avec des `print`.

💼 **Mise en situation** : Un script de traitement de commandes e-commerce plante aléatoirement avec une erreur de type, mais vous ne savez pas quelle commande cause le problème.

📝 **Énoncé** :
1.  Copiez le code ci-dessous.
2.  Insérez un `breakpoint()` dans le bloc `except` pour intercepter l'erreur au vol.
3.  Lancez le script.
4.  Utilisez `p commande` pour identifier l'ID de la commande fautive.

```python
data = [
    {"id": 1, "total": 50.0},
    {"id": 2, "total": "100.0"}, # 👈 Erreur ici (string au lieu de float)
    {"id": 3, "total": 25.5}
]

def traiter_commandes(commandes):
    total_global = 0
    for commande in commandes:
        try:
            # Simulation d'un calcul complexe
            total_global += commande["total"] * 1.2
        except TypeError:
            # TODO: Inspecter ici
            pass 
    return total_global

traiter_commandes(data)
```

📺 **Résultat attendu** :
Le script se met en pause. En tapant `p commande` dans le terminal, vous voyez le dictionnaire avec `id: 2`.

<details>
<summary>💡 Voir la solution</summary>

```python
# ...
        except TypeError:
            breakpoint() # 👈 Ajout du point d'arrêt
            print(f"Erreur sur la commande {commande['id']}")
# ...
```
Ensuite dans le terminal `(Pdb)` :
```text
(Pdb) p commande
{'id': 2, 'total': '100.0'}
(Pdb) c
```
</details>

### Exercice 2 - Profilage d'un Algorithme Lent {#exercice-2-profilage-algo-lent}

🎯 **Objectif** : Identifier un goulot d'étranglement avec `cProfile`.

💼 **Mise en situation** : Votre startup analyse des séquences ADN (chaînes de caractères). Le script actuel est trop lent.

📝 **Énoncé** :
1.  Créez un script `dna_analysis.py` avec le code ci-dessous (intentionnellement inefficace).
2.  Lancez le profilage : `python -m cProfile -s tottime dna_analysis.py`.
3.  Identifiez quelle fonction consomme le plus de `tottime`.

```python
import random

def generer_adn(taille):
    return "".join(random.choice("ATCG") for _ in range(taille))

def trouver_doublons(sequence):
    # Algorithme naïf O(N^2)
    doublons = []
    for i in range(len(sequence)):
        # Cette vérification est très lente sur une longue chaîne
        if sequence[i] in sequence[i+1:]: 
            doublons.append(sequence[i])
    return len(doublons)

def main():
    seq = generer_adn(10000)
    print("Analyse en cours...")
    trouver_doublons(seq)

if __name__ == "__main__":
    main()
```

📺 **Résultat attendu** :
Le rapport doit montrer que `trouver_doublons` (ou une méthode de recherche de string) prend 99% du temps.

<details>
<summary>💡 Voir l'analyse attendue</summary>

```text
ncalls  tottime  percall  cumtime  percall filename:lineno(function)
     1    1.234    1.234    1.234    1.234 dna_analysis.py:6(trouver_doublons)
```
La fonction `trouver_doublons` fait une recherche linéaire (`in`) à l'intérieur d'une boucle, créant une complexité quadratique.
</details>

### Exercice 3 - Optimisation Set vs List {#exercice-3-optimisation-set}

🎯 **Objectif** : Optimiser le code de l'exercice précédent.

💼 **Mise en situation** : Vous devez accélérer la fonction `trouver_doublons`.

📝 **Énoncé** :
1.  Réécrivez `trouver_doublons` pour utiliser un `set` afin de mémoriser les caractères déjà vus.
2.  Relancez le profilage. Le temps d'exécution doit devenir négligeable (< 0.01s).

📺 **Résultat attendu** :
Une exécution quasi-instantanée.

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
def trouver_doublons_optimise(sequence):
    # Utilisation d'un Set pour lookup en O(1)
    vus = set()
    doublons = []
    
    for char in sequence:
        if char in vus:
            doublons.append(char)
        else:
            vus.add(char)
            
    return len(doublons)
```
Passage d'une complexité **O(N²)** à **O(N)**.
</details>