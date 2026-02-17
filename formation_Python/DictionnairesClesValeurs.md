---
sidebar_label: "Dictionnaires: Stockage de Données par Clé-Valeur"
sidebar_position: 12
difficulty: "junior"
---

# Dictionnaires: Stockage de Données par Clé-Valeur {#dictionnaires-cles-valeurs-12}

Jusqu'à présent, nous avons vu des séquences où l'on accède aux données par leur position (index 0, 1, 2...). Mais que faire si l'on veut stocker des informations et y accéder par un nom significatif, comme dans un vrai dictionnaire ou un annuaire ? C'est là qu'intervient le **dictionnaire** (`dict`), la structure de données la plus polyvalente de Python.

Un dictionnaire ne stocke pas des éléments seuls, mais des paires **clé-valeur**. Chaque valeur est associée à une clé unique qui sert à la retrouver.

## 1. Création et Accès aux Données {#creation-acces-dictionnaires-12}

### Quoi
Un dictionnaire est une collection **non ordonnée** (avant Python 3.7) et **modifiable** de paires clé-valeur. On le crée avec des accolades `{}`. Chaque clé doit être unique et de type immuable (comme une chaîne de caractères ou un nombre).

```mermaid
graph TD
    D(Dictionnaire 'utilisateur')
    D -- Clé 'nom' --> V1[Valeur: "Alice"]
    D -- Clé 'age' --> V2[Valeur: 30]
    D -- Clé 'ville' --> V3[Valeur: "Paris"]
    D -- Clé 'est_admin' --> V4[Valeur: true]
```

### Pourquoi
Les dictionnaires sont parfaits pour représenter des objets du monde réel ou des données structurées. Un profil utilisateur, la configuration d'une application, les caractéristiques d'un produit... `utilisateur['email']` est bien plus explicite que `utilisateur[1]`. L'accès par clé est aussi extrêmement rapide.

### Comment
*   **Création** : `mon_dict = {"cle1": "valeur1", "cle2": "valeur2"}`
*   **Accès** : `mon_dict["cle1"]`

```python
# Cas d'usage : Représenter un profil utilisateur
utilisateur = {
    "nom": "Alice",
    "age": 30,
    "ville": "Paris",
    "est_admin": True
}

# Accéder aux valeurs en utilisant les clés
nom_utilisateur = utilisateur["nom"]
age_utilisateur = utilisateur["age"]

print(f"L'utilisatrice s'appelle {nom_utilisateur} et a {age_utilisateur} ans.")

# La méthode .get() est une alternative plus sûre
# Elle retourne None (ou une valeur par défaut) si la clé n'existe pas
email = utilisateur.get("email") 
print(f"Email de l'utilisateur : {email}") # Affiche None

email_avec_defaut = utilisateur.get("email", "non renseigné")
print(f"Email de l'utilisateur : {email_avec_defaut}") # Affiche "non renseigné"
```

### Zone de Danger
*   **`KeyError`** : C'est l'équivalent de l'`IndexError` pour les listes. Elle se produit si vous essayez d'accéder à une clé qui n'existe pas dans le dictionnaire avec la syntaxe des crochets `[]`. C'est pourquoi la méthode `.get()` est souvent préférée pour un accès sécurisé.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Fenêtre de code montrant `print(utilisateur["email"])` et l'erreur `KeyError: 'email'` qui en résulte dans le terminal.
> **Alt Text** : Exemple d'une erreur KeyError en Python en accédant à une clé de dictionnaire inexistante.

---

## 2. Ajout, Modification et Suppression {#modification-dictionnaires-12}

### Quoi
Les dictionnaires sont **mutables**. On peut dynamiquement ajouter de nouvelles paires clé-valeur, modifier la valeur associée à une clé existante, ou supprimer des paires.

### Pourquoi
Les données sont rarement statiques. Un utilisateur peut ajouter un numéro de téléphone à son profil, changer de ville, ou supprimer son compte. Les dictionnaires permettent de gérer facilement ces mises à jour.

### Comment

| Opération        | Syntaxe                               | Description                                         |
| ---------------- | ------------------------------------- | --------------------------------------------------- |
| **Ajouter/Modifier** | `mon_dict["cle"] = "valeur"`        | Si la clé existe, la valeur est mise à jour. Sinon, la nouvelle paire clé-valeur est créée. |
| **Supprimer**      | `del mon_dict["cle"]`                 | Supprime la paire clé-valeur. Lève une `KeyError` si la clé n'existe pas. |

```python
# Cas d'usage : Mise à jour d'un inventaire de produit
produit = {
    "nom": "Ordinateur Portable",
    "prix": 1200,
    "stock": 42
}
print(f"Produit initial : {produit}")

# Modifier le prix (promotion)
produit["prix"] = 1099.99
print(f"Après promotion : {produit}")

# Ajouter une nouvelle information
produit["marque"] = "TechCorp"
print(f"Avec ajout de la marque : {produit}")

# Mettre à jour le stock après une vente
produit["stock"] = produit["stock"] - 1
print(f"Stock mis à jour : {produit}")

# Supprimer une information (ex: si on ne stocke plus la marque)
del produit["marque"]
print(f"Après suppression de la marque : {produit}")
```

### Zone de Danger
*   **L'écrasement silencieux** : Lorsque vous assignez une valeur à une clé qui existe déjà, l'ancienne valeur est remplacée sans aucun avertissement. Soyez prudent lorsque vous ajoutez des données pour ne pas écraser des informations importantes par inadvertance.

---

## 3. Itérer sur un Dictionnaire {#iteration-dictionnaires-12}

### Quoi
Parcourir le contenu d'un dictionnaire est une opération très courante. Il existe plusieurs façons de le faire, selon que vous ayez besoin des clés, des valeurs, ou des deux.

| Méthode                 | Description                                    |
| ----------------------- | ---------------------------------------------- |
| `for cle in mon_dict`     | Itère sur les **clés** (comportement par défaut). |
| `mon_dict.keys()`         | Renvoie une vue de toutes les clés.            |
| `mon_dict.values()`       | Renvoie une vue de toutes les valeurs.         |
| `mon_dict.items()`        | Renvoie une vue de toutes les paires (clé, valeur) sous forme de tuples. |

### Pourquoi
L'itération permet d'afficher, de traiter ou de transformer toutes les données contenues dans un dictionnaire. La méthode `.items()` est particulièrement puissante car elle donne accès à la clé et à la valeur en même temps de manière très lisible.

### Comment
```python
notes_etudiants = {
    "Alice": 18,
    "Bob": 14,
    "Charlie": 16
}

# Itérer sur les clés (le plus simple)
print("\n--- Noms des étudiants ---")
for etudiant in notes_etudiants:
    print(etudiant)

# Itérer sur les valeurs
print("\n--- Notes obtenues ---")
for note in notes_etudiants.values():
    print(note)

# Itérer sur les paires clé-valeur (le plus courant et le plus utile !)
print("\n--- Bulletin de notes ---")
for etudiant, note in notes_etudiants.items():
    print(f"{etudiant} a obtenu la note de {note}/20.")
```

---

## Validation des Acquis {#validation-12}

### 3 Questions Clés

1.  Quelle est la principale différence entre accéder à une donnée dans une liste et dans un dictionnaire ?
2.  Que se passe-t-il si vous essayez d'assigner une valeur à une clé qui n'existe pas encore dans un dictionnaire ? Et si elle existe déjà ?
3.  Quelle est la méthode la plus "pythonique" pour boucler sur un dictionnaire si vous avez besoin à la fois de la clé et de la valeur à chaque tour de boucle ?

### 3 Exercices Progressifs

#### Exercice 1 : Mon Dictionnaire Personnel
1.  Créez un dictionnaire pour vous décrire, contenant les clés `"prenom"`, `"nom"`, et `"ville"`.
2.  Ajoutez une nouvelle clé `"profession"` avec votre profession.
3.  Modifiez la valeur de la clé `"ville"` pour une autre ville.
4.  Affichez chaque information sur une ligne séparée en parcourant le dictionnaire.

<details>
<summary>Découvrir la solution commentée</summary>

```python
# 1. Création du dictionnaire
profil = {
    "prenom": "Jean",
    "nom": "Dupont",
    "ville": "Paris"
}
print(f"Profil initial: {profil}")

# 2. Ajout d'une clé
profil["profession"] = "Développeur Python"
print(f"Profil avec profession: {profil}")

# 3. Modification d'une valeur
profil["ville"] = "Lyon"
print(f"Profil avec nouvelle ville: {profil}")

# 4. Affichage final en bouclant
print("\n--- Fiche de Profil ---")
for cle, valeur in profil.items():
    # .capitalize() met la première lettre en majuscule
    print(f"{cle.capitalize()}: {valeur}")
```
</details>

#### Exercice 2 : Compteur de Mots
Écrivez un script qui prend une phrase en entrée et compte la fréquence d'apparition de chaque mot. Le résultat doit être un dictionnaire où les clés sont les mots et les valeurs sont leur nombre d'occurrences.

*Exemple : "le chat est sur le tapis" -> `{'le': 2, 'chat': 1, 'est': 1, 'sur': 1, 'tapis': 1}`*

<details>
<summary>Découvrir la solution commentée</summary>

```python
phrase = "le chat est sur le tapis et le chat dort"
# On met tout en minuscules et on enlève la ponctuation simple pour mieux compter
phrase_nettoyee = phrase.lower() 
mots = phrase_nettoyee.split()

frequences = {} # Dictionnaire vide pour stocker les comptes

# On parcourt la liste des mots
for mot in mots:
    # Si le mot est déjà une clé dans notre dictionnaire
    if mot in frequences:
        # On incrémente son compteur
        frequences[mot] += 1
    else:
        # Sinon, c'est la première fois qu'on le voit, on crée la clé avec la valeur 1
        frequences[mot] = 1

print(f"Phrase originale : '{phrase}'")
print("Fréquence des mots :")
print(frequences)
```
</details>

#### Exercice 3 : Gestion d'un Panier d'Achat
Simulez un panier d'achat en utilisant un dictionnaire. Les clés sont les noms des articles et les valeurs sont leurs prix.
1.  Créez un dictionnaire `panier` avec quelques articles et leurs prix.
2.  Écrivez un code qui parcourt le dictionnaire pour calculer et afficher le prix total du panier.
3.  Ajoutez un article au panier. Mettez à jour le prix total et affichez-le de nouveau.

<details>
<summary>Découvrir la solution commentée</summary>

```python
# 1. Création du panier initial
panier = {
    "Pommes": 2.50,
    "Lait": 1.20,
    "Pain": 0.90
}

# --- Calcul du total initial ---
total = 0.0
# On a seulement besoin des valeurs (les prix) pour le total
for prix in panier.values():
    total += prix

print("--- Panier Initial ---")
for article, prix in panier.items():
    print(f"- {article}: {prix:.2f}€")
print(f"Total initial : {total:.2f}€")


# 3. Ajout d'un nouvel article
print("\n...Ajout de 'Fromage' au panier...")
panier["Fromage"] = 4.50

# --- Recalcul du nouveau total ---
nouveau_total = sum(panier.values()) # sum() est un raccourci très pratique !

print("\n--- Panier Mis à Jour ---")
for article, prix in panier.items():
    print(f"- {article}: {prix:.2f}€")
print(f"Nouveau total : {nouveau_total:.2f}€")
```
</details>