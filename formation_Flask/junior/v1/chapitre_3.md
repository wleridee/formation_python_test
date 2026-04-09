---
sidebar_label: "Mode Debug"
sidebar_position: 3
difficulty: "junior"
---

# Chapitre 3 : Mode Debug

Mode Debug, Rechargement automatique, Débogueur interactif

## Le mode Debug {#le-mode-debug-3}

### 1. Quoi
Le **mode Debug** est une configuration de développement qui active deux fonctionnalités majeures : le **rechargement automatique** du serveur à chaque modification du code et l'affichage d'un **débogueur interactif** dans le navigateur en cas d'erreur.

### 2. Pourquoi
Il accélère considérablement le cycle de développement en évitant de redémarrer manuellement le serveur après chaque changement et permet d'inspecter l'état de l'application directement au moment où une exception survient.

### 3. Comment
A. **Syntaxe de base** :
```python
from flask import Flask

app = Flask(__name__)

# Activation via le paramètre debug
if __name__ == "__main__":
    app.run(debug=True)
```

B. **Cas concret** :
Lorsqu'une erreur survient, Flask affiche une page HTML interactive. Vous pouvez cliquer sur les lignes de code pour voir les variables locales et exécuter des commandes Python dans le contexte de l'erreur.

C. **Exemples pratiques** :
- **Modification de code** : Vous changez une chaîne de caractères dans une route, le terminal affiche `Detected change in 'app.py', reloading...` et le serveur redémarre instantanément.
- **Gestion d'erreur** : Une division par zéro provoque l'affichage du traceback complet dans le navigateur, facilitant le diagnostic immédiat.
- **Inspection** : Le débogueur permet d'explorer la pile d'appels (*stack trace*) pour comprendre quel composant a provoqué l'échec.

> 📸 **CAPTURE D'ÉCRAN REQUISE**
> **Sujet** : Page d'erreur interactive de Flask dans le navigateur.
> **Alt Text** : Interface de débogage Flask montrant une erreur Python avec le code source et la console interactive.

### 4. Zone de Danger
❌ **À ne pas faire** : Activer `debug=True` sur un serveur accessible publiquement.
✅ **Bonne Pratique** : Utilisez des variables d'environnement (ex: `FLASK_ENV=development`) pour activer le mode debug uniquement sur votre machine locale.

### 🚨 Limitations de l'approche `debug=True` {#limitations-de-l-approche-debug-true-3}

- **Problèmes concrets** : Le débogueur interactif permet l'exécution de code arbitraire. Si un attaquant accède à cette page, il peut prendre le contrôle total de votre serveur.
- **Solutions modernes** : Utilisez des outils de monitoring comme **Sentry** pour capturer les erreurs en production de manière sécurisée, et des outils de logging structurés.
- **Pourquoi l'enseigner** : C'est l'outil fondamental pour comprendre le flux d'exécution et diagnostiquer les problèmes lors de la phase de création.

## Questions clés (validation des acquis du chapitre) {#questions-clés-(validation-des-acquis-du-chapitre)-3}

1. **Quelles sont les deux fonctionnalités principales apportées par le mode Debug ?**
   Réponse : Le rechargement automatique du serveur et l'affichage d'un débogueur interactif en cas d'erreur.

2. **Pourquoi le mode Debug est-il dangereux en production ?**
   Réponse : Il permet l'exécution de code arbitraire par n'importe quel utilisateur accédant à la page d'erreur.

3. **Comment peut-on activer le mode Debug sans modifier le code source ?**
   Réponse : En utilisant des variables d'environnement (ex: `FLASK_DEBUG=1` ou `FLASK_ENV=development`).

## Exercices : {#exercices-:-3}

### Exercice 1 - Le serveur réactif {#exercice-1---le-serveur-réactif-3}

- 🎯 **Objectif** : Observer le rechargement automatique.
- 💼 **Mise en situation** : Vous développez une page d'accueil et devez modifier le texte fréquemment.
- 📝 **Énoncé** : Lancez votre application avec `debug=True`. Modifiez le message de retour de votre route `/` pendant que le serveur tourne. Vérifiez que le terminal détecte le changement et que le navigateur affiche le nouveau texte sans redémarrage manuel.
- 📺 **Résultat attendu** : Le terminal affiche "Detected change" et le navigateur montre le texte mis à jour.

<details>
<summary>Découvrir la solution commentée</summary>

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def index():
    # On modifie ce texte pour tester le rechargement automatique
    return "Le serveur a bien rechargé !"

if __name__ == "__main__":
    # debug=True active le rechargement automatique
    app.run(debug=True)
```
</details>

### Exercice 2 - Déclenchement d'erreur {#exercice-2---déclenchement-d-erreur-3}

- 🎯 **Objectif** : Utiliser le débogueur interactif.
- 💼 **Mise en situation** : Vous voulez comprendre comment Flask réagit face à une exception.
- 📝 **Énoncé** : Créez une route `/erreur` qui provoque volontairement une erreur (ex: `1 / 0`). Accédez à cette route dans votre navigateur et explorez l'interface de débogage.
- 📺 **Résultat attendu** : Une page d'erreur interactive s'affiche avec le détail de l'exception `ZeroDivisionError`.

<details>
<summary>Découvrir la solution commentée</summary>

```python
@app.route("/erreur")
def provoquer_erreur():
    # Division par zéro pour déclencher une exception
    return str(1 / 0)
```
</details>

### Exercice 3 - Sécurisation locale {#exercice-3---sécurisation-locale-3}

- 🎯 **Objectif** : Configurer le mode debug via variable d'environnement.
- 💼 **Mise en situation** : Vous voulez éviter d'oublier `debug=True` dans votre code source.
- 📝 **Énoncé** : Supprimez `debug=True` de votre code `app.run()`. Lancez votre application en utilisant la variable d'environnement `FLASK_DEBUG=1` dans votre terminal.
- 📺 **Résultat attendu** : Le serveur démarre en mode debug sans que le code source ne contienne explicitement `debug=True`.

<details>
<summary>Découvrir la solution commentée</summary>

```bash
# Dans le terminal (Linux/macOS)
export FLASK_APP=app.py
export FLASK_DEBUG=1
flask run

# Dans le terminal (Windows PowerShell)
$env:FLASK_APP="app.py"
$env:FLASK_DEBUG="1"
flask run
```
</details>