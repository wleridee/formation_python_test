---
sidebar_label: "Variables et types de base"
sidebar_position: 3
difficulty: "junior"
---

# Chapitre 3 : Variables et types de base {#chapitre-3-:-variables-et-types-de-base}

Déclaration, inférence, types scalaires, constantes

## Déclaration de variables {#déclaration-de-variables-3}

### 1. Quoi
En Go, une **variable** est un espace de stockage nommé dont la valeur peut être modifiée. Go est un langage à **typage statique**, ce qui signifie que le type d'une variable est connu à la compilation.

### 2. Pourquoi
La déclaration explicite ou inférée permet au compilateur de garantir la sécurité mémoire et d'optimiser les performances, tout en offrant une syntaxe concise pour le développeur.

### 3. Comment
A. **Syntaxe de base** :
```go
var nom string = "Go" // Déclaration explicite
age := 25             // Inférence de type (syntaxe courte)
```

B. **Cas concret et robuste** :
```go
package main

import "fmt"

func main() {
    // Utilisation de var pour des variables globales ou non initialisées
    var statut bool 
    
    // Utilisation de := pour des variables locales
    compteur := 10 
    
    fmt.Printf("Statut: %v, Compteur: %d\n", statut, compteur)
}
```

C. **Exemples pratiques** :
- `var` : Idéal pour les variables de package ou quand l'initialisation est différée.
- `:=` : Idéal pour les variables temporaires à l'intérieur des fonctions.

### 4. Zone de Danger
❌ **À ne pas faire** : Déclarer des variables inutilisées (Go provoquera une erreur de compilation).
✅ **Bonne Pratique** : Utilisez des noms de variables courts mais explicites (`i` pour un index, `user` pour un utilisateur).

---

## Types de base et constantes {#types-de-base-et-constantes-3}

### 1. Quoi
Les **types de base** incluent les entiers (`int`), flottants (`float64`), booléens (`bool`) et chaînes de caractères (`string`). Les **constantes** sont des valeurs immuables définies avec `const`.

### 2. Pourquoi
L'utilisation de types précis permet de prévenir les erreurs de logique (ex: mélanger des entiers et des flottants) et d'optimiser l'occupation mémoire.

### 3. Comment
A. **Syntaxe de base** :
```go
const Pi = 3.14159
var score int = 100
```

B. **Cas concret** :
```go
const (
    MaxConnexions = 100
    Timeout       = 30
)

func main() {
    // Les constantes ne peuvent pas être modifiées
    // MaxConnexions = 200 // Erreur de compilation
}
```

C. **Tableau comparatif** :
| Type | Description |
|------|-------------|
| `int` | Entier signé (taille dépend de l'architecture) |
| `float64` | Flottant double précision |
| `string` | Chaîne de caractères (UTF-8) |
| `bool` | Vrai ou Faux |

### 4. Zone de Danger
❌ **À ne pas faire** : Tenter de modifier une constante après sa déclaration.
✅ **Bonne Pratique** : Utilisez des constantes pour les valeurs de configuration qui ne doivent jamais changer durant l'exécution.

---

## Questions clés (validation des acquis du chapitre) {#questions-clés-(validation-des-acquis-du-chapitre)-3}

- **Quelle est la différence entre `var` et `:=` ?**
  *Réponse : `var` permet une déclaration explicite (avec ou sans type), tandis que `:=` est une syntaxe courte qui infère automatiquement le type.*

- **Que se passe-t-il si vous déclarez une variable sans l'utiliser ?**
  *Réponse : Le compilateur Go génère une erreur et refuse de compiler le programme.*

- **Peut-on changer la valeur d'une constante après sa déclaration ?**
  *Réponse : Non, une constante est immuable par définition.*

---

## Exercices : {#exercices-:-3}

### Exercice 1 - Profil utilisateur {#exercice-1---profil-utilisateur}

🎯 **Objectif** : Manipuler les variables de base.

💼 **Mise en situation** : Vous développez un système de gestion d'utilisateurs.

📝 **Énoncé** : Déclarez trois variables : `nom` (string), `age` (int) et `estActif` (bool). Initialisez-les avec des valeurs de votre choix et affichez-les avec `fmt.Printf`.

📺 **Résultat attendu** : Affichage dans la console : `Utilisateur: Alice, Age: 30, Actif: true`.

💡 **Solution** :
<details><summary>Voir le code complet commenté</summary>

```go
package main

import "fmt"

func main() {
    // Utilisation de l'inférence de type pour la concision
    nom := "Alice"
    age := 30
    estActif := true

    fmt.Printf("Utilisateur: %s, Age: %d, Actif: %t\n", nom, age, estActif)
}
```
</details>

### Exercice 2 - Calcul de périmètre {#exercice-2---calcul-de-périmètre}

🎯 **Objectif** : Utiliser des constantes.

💼 **Mise en situation** : Vous devez calculer le périmètre d'un cercle.

📝 **Énoncé** : Déclarez une constante `Pi` égale à 3.14. Déclarez une variable `rayon` égale à 5. Calculez et affichez le périmètre (2 * Pi * rayon).

📺 **Résultat attendu** : `Périmètre: 31.4`.

💡 **Solution** :
<details><summary>Voir le code complet commenté</summary>

```go
package main

import "fmt"

func main() {
    const Pi = 3.14
    rayon := 5.0 // Utilisation de float64 pour la précision

    perimetre := 2 * Pi * rayon
    fmt.Printf("Périmètre: %.1f\n", perimetre)
}
```
</details>

### Exercice 3 - Conversion de type {#exercice-3---conversion-de-type}

🎯 **Objectif** : Comprendre la rigueur du typage.

💼 **Mise en situation** : Vous devez additionner un entier et un flottant.

📝 **Énoncé** : Déclarez `a := 10` (int) et `b := 5.5` (float64). Essayez de les additionner. *Note : Go ne permet pas l'addition directe de types différents.*

📺 **Résultat attendu** : `Résultat: 15.5`.

💡 **Solution** :
<details><summary>Voir le code complet commenté</summary>

```go
package main

import "fmt"

func main() {
    a := 10
    b := 5.5

    // Conversion explicite de 'a' en float64 pour permettre l'addition
    resultat := float64(a) + b
    
    fmt.Printf("Résultat: %.1f\n", resultat)
}
```
</details>