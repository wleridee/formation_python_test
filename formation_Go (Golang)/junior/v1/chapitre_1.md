---
sidebar_label: "Installation et environnement"
sidebar_position: 1
difficulty: "junior"
---

# Chapitre 1 : Installation et environnement {#chapitre-1-:-installation-et-environnement}

Installation, Go Modules, GOPATH, Hello World

## Installation du langage Go {#installation-du-langage-go-1}

### 1. Quoi
L'installation de Go consiste à déployer le compilateur `go` et les outils associés sur votre système d'exploitation. Depuis les versions récentes, Go utilise un système de gestion de dépendances intégré appelé **Go Modules**.

### 2. Pourquoi
Pour transformer votre code source en un binaire exécutable optimisé, vous avez besoin de la chaîne d'outils (toolchain) officielle fournie par l'équipe Go.

### 3. Comment
A. **Installation** :
- Téléchargez l'installeur officiel sur [go.dev/dl](https://go.dev/dl/).
- Suivez les instructions spécifiques à votre OS (Windows, macOS, ou Linux).
- Vérifiez l'installation dans votre terminal : `go version`.

B. **Configuration** :
- Contrairement aux anciennes versions, vous n'avez plus besoin de configurer manuellement le `GOPATH` pour vos projets.
- Utilisez `go mod init <nom-du-module>` à la racine de votre projet pour initialiser un nouveau module.

C. **Tableau comparatif** :
| Concept | Ancienne approche (Avant Go 1.11) | Approche moderne (Go Modules) |
|---------|-----------------------------------|-------------------------------|
| Stockage | Tout dans `$GOPATH/src` | N'importe où sur le disque |
| Dépendances | `go get` global | `go.mod` local au projet |

### 4. Zone de Danger
❌ **À ne pas faire** : Placer vos projets à l'intérieur du répertoire d'installation de Go ou tenter de configurer manuellement des variables d'environnement complexes sans nécessité.
✅ **Bonne Pratique** : Laissez Go gérer les dépendances via `go.mod` et travaillez dans des répertoires de projet isolés.

---

## Premier programme : Hello World {#premier-programme-:-hello-world-1}

### 1. Quoi
Le "Hello World" est le programme minimaliste permettant de valider que votre environnement est correctement configuré pour compiler et exécuter du code Go.

### 2. Pourquoi
Il permet de vérifier la chaîne de compilation, l'exécution du binaire et la structure de base d'un fichier source `.go`.

### 3. Comment
A. **Syntaxe de base** :
```go
package main // Définit le package comme exécutable

import "fmt" // Importe le package de formatage pour afficher du texte

func main() {
    // La fonction main est le point d'entrée du programme
    fmt.Println("Hello, World!") 
}
```

B. **Configuration et vérification** :
1. Créez un dossier `hello`.
2. Exécutez `go mod init hello`.
3. Créez `main.go` et collez le code ci-dessus.
4. Exécutez `go run main.go`.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Terminal affichant le résultat de la commande `go run main.go`
> **Alt Text** : "Terminal montrant l'exécution réussie du programme Hello World en Go"

### 4. Zone de Danger
❌ **À ne pas faire** : Oublier de déclarer `package main` ou de nommer la fonction `main` autrement, sinon le compilateur ne saura pas quoi exécuter.
✅ **Bonne Pratique** : Utilisez `go fmt` sur vos fichiers pour respecter automatiquement les standards de style de la communauté Go.

---

## Questions clés (validation des acquis du chapitre) {#questions-clés-(validation-des-acquis-du-chapitre)-1}

- **Quelle commande permet d'initialiser un nouveau module Go dans un répertoire ?**
  *Réponse : `go mod init <nom-du-module>`*

- **Quel est le nom obligatoire de la fonction point d'entrée d'un programme Go ?**
  *Réponse : `main`*

- **Pourquoi est-il recommandé d'utiliser `go mod` plutôt que de travailler dans le `GOPATH` ?**
  *Réponse : `go mod` permet une gestion des dépendances locale au projet, isolée et reproductible.*