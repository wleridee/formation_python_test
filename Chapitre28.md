---
sidebar_label: Module `datetime` : Manipulation des Dates et Heures
sidebar_position: 28
---

# Chapitre 28 : Module `datetime` : Manipulation des Dates et Heures

Objets datetime, date, time, Formatage (strftime, strptime), Timezones, Calculs de durée (timedelta)

Gérer le temps en programmation est notoirement difficile : années bissextiles, mois de longueurs variables, changements d'heure (DST) et fuseaux horaires... C'est une source infinie de bugs si l'on tente de tout recalculer manuellement.

Le module standard `datetime` de Python encapsule cette complexité. Il vous permet de manipuler des dates, des heures et des durées avec une précision et une simplicité remarquables. En Python 3.14, l'utilisation conjointe avec `zoneinfo` est le standard moderne pour des applications robustes.

---

## 1. Les Objets Fondamentaux : `date`, `time`, `datetime` {#les-objets-fondamentaux}

### 1. Quoi
Le module propose trois classes principales pour modéliser le temps :
*   **`date`** : Une date du calendrier (Année, Mois, Jour). Indépendant de l'heure.
*   **`time`** : Une heure d'horloge (Heure, Minute, Seconde, Microseconde). Indépendant de la date.
*   **`datetime`** : La combinaison des deux (Année, Mois, Jour, Heure, Minute...).

### 2. Pourquoi
Utiliser le bon type sémantique évite les erreurs. Un anniversaire est une `date` (on se fiche de l'heure exacte de naissance pour fêter les 30 ans). Un log serveur est un `datetime` précis.

### 3. Comment

#### A. Création et accès aux attributs

```python
from datetime import date, time, datetime

# 1. Date (AAAA, MM, JJ)
today = date(2026, 10, 25)
print(f"Année : {today.year}")

# 2. Time (HH, MM, SS)
lunch_break = time(12, 30, 0)
print(f"Pause : {lunch_break.isoformat()}")

# 3. Datetime (Combinaison)
# Obtenir le moment présent
now = datetime.now() 
print(f"Maintenant : {now}")
```

#### B. Combiner Date et Time
```python
meeting_date = date(2026, 5, 1)
meeting_time = time(14, 0)

# Fusionner les deux
full_meeting = datetime.combine(meeting_date, meeting_time)
print(full_meeting) # 2026-05-01 14:00:00
```

### 4. Zone de Danger
❌ **Confondre `time` (module) et `datetime.time` (classe)** :
Python possède aussi un module nommé `time` (fonction `time.sleep()`). Ne les confondez pas.
✅ **Importez explicitement** ce dont vous avez besoin : `from datetime import datetime`.

---

## 2. Formatage et Parsing (`strftime`, `strptime`) {#formatage-et-parsing}

### 1. Quoi
Transformer des objets Python en chaînes de caractères lisibles (**Stringify**) et inversement (**Parse**).
*   **`strftime`** (string **f**ormat time) : `datetime` → `str`
*   **`strptime`** (string **p**arse time) : `str` → `datetime`

### 2. Pourquoi
Vos utilisateurs veulent lire "25 Décembre 2026" (UI), mais votre base de données stocke "2026-12-25". Vous devez constamment convertir entre ces représentations.

### 3. Comment

#### A. Codes de formatage courants

| Code | Signification | Exemple |
| :--- | :--- | :--- |
| `%Y` | Année (4 chiffres) | 2026 |
| `%m` | Mois (01-12) | 12 |
| `%d` | Jour (01-31) | 25 |
| `%H` | Heure (00-23) | 14 |
| `%M` | Minute (00-59) | 30 |
| `%S` | Seconde (00-59) | 05 |

#### B. Exemple concret

```python
from datetime import datetime

# --- 1. Formater (Date -> Texte) ---
now = datetime.now()
readable = now.strftime("Le %d/%m/%Y à %Hh%M")
print(readable) 
# Résultat : Le 25/10/2026 à 15h30

# --- 2. Parser (Texte -> Date) ---
input_date = "2026-12-31 23:59"
# Le format doit correspondre EXACTEMENT à la chaîne d'entrée
obj = datetime.strptime(input_date, "%Y-%m-%d %H:%M")
print(obj.month) # 12
```

---

## 3. Arithmétique des Dates avec `timedelta` {#arithmetique-avec-timedelta}

### 1. Quoi
L'objet `timedelta` représente une **durée** (une différence entre deux temps). On peut l'ajouter ou la soustraire à un `datetime`.

### 2. Pourquoi
Pour calculer une date d'expiration (maintenant + 30 jours), l'âge d'un utilisateur, ou le temps écoulé entre deux événements.

### 3. Comment

```python
from datetime import datetime, timedelta

now = datetime.now()

# Dans 3 jours et 2 heures
duration = timedelta(days=3, hours=2)
future_date = now + duration

# Il y a 1 semaine
past_date = now - timedelta(weeks=1)

# Calculer une différence
diff = future_date - now
print(f"Écart total en secondes : {diff.total_seconds()}")
```

### 🚨 Limitations de `timedelta`
`timedelta` ne gère pas les "mois" ou les "années" car leur durée est variable (28, 30, 31 jours / 365, 366 jours).
Pour ajouter "1 mois" exactement, il faut utiliser des bibliothèques externes comme `python-dateutil` ou gérer la logique métier manuellement (attention aux effets de bord comme le 31 janvier + 1 mois).

---

## 4. Fuseaux Horaires (Timezones) avec `zoneinfo` {#fuseaux-horaires}

### 1. Quoi
Par défaut, les objets datetime sont **naïfs** (sans information de fuseau). Pour gérer des utilisateurs globaux, il faut les rendre **aware** (conscients du fuseau) en utilisant le module standard `zoneinfo`.

### 2. Pourquoi
Si un utilisateur à Tokyo poste un message à 10h00, un utilisateur à New York doit le voir affiché comme "posté hier à 21h00", pas "à 10h00".

### 3. Comment

#### A. La Bonne Pratique (UTC + Conversion locale)
On stocke toujours en UTC (temps universel) côté serveur, et on convertit en heure locale uniquement pour l'affichage.

```python
from datetime import datetime
from zoneinfo import ZoneInfo # Standard depuis Python 3.9+

# 1. Créer une date consciente en UTC
# Utile pour stockage BDD
now_utc = datetime.now(ZoneInfo("UTC"))
print(f"UTC : {now_utc}")

# 2. Convertir pour un utilisateur à Paris
# Gère automatiquement l'heure d'été/hiver
now_paris = now_utc.astimezone(ZoneInfo("Europe/Paris"))
print(f"Paris : {now_paris}")

# 3. Convertir pour un utilisateur à New York
now_ny = now_utc.astimezone(ZoneInfo("America/New_York"))
print(f"NY : {now_ny}")
```

### 4. Zone de Danger
❌ **Utiliser `datetime.utcnow()` (Obsolète)** :
Cette méthode renvoie un datetime *naïf*, ce qui est source d'erreurs. Utilisez `datetime.now(timezone.utc)`.
❌ **Calculer des différences entre naïf et aware** :
Python lèvera une erreur `TypeError: can't subtract offset-naive and offset-aware datetimes`.

---

## Questions clés (validation des acquis du chapitre) {#questions-cles-28}

1.  **Quelle est la différence entre `strftime` et `strptime` ?**
    `strftime` (Format) convertit une date en chaîne de caractères. `strptime` (Parse) convertit une chaîne en objet date.

2.  **Que représente un objet `timedelta` ?**
    Il représente une durée (une différence de temps), et non une date précise du calendrier.

3.  **Pourquoi ne faut-il pas utiliser des objets datetime "naïfs" pour une application internationale ?**
    Car ils ne contiennent pas d'information de fuseau horaire, rendant impossible la comparaison ou la conversion correcte entre différentes zones géographiques.

4.  **Quel module de la bibliothèque standard permet de gérer les fuseaux horaires modernes (IANA) ?**
    Le module `zoneinfo`.

---

## Exercices : {#exercices-28}

### Exercice 1 - Calculateur d'Expiration d'Abonnement {#exercice-1---abonnement}

🎯 **Objectif** : Utiliser `timedelta` et le formatage.

💼 **Mise en situation** : Vous gérez un SaaS. Lorsqu'un utilisateur s'abonne, son accès est valide 30 jours.

📝 **Énoncé** :
1.  Définissez une date de début d'abonnement (simulez `datetime.now()`).
2.  Ajoutez une durée de 30 jours via `timedelta`.
3.  Affichez : "Abonnement souscrit le [JJ/MM/AAAA], valide jusqu'au [JJ/MM/AAAA]".

📺 **Résultat attendu** :
```text
Abonnement souscrit le 25/10/2026, valide jusqu'au 24/11/2026
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
from datetime import datetime, timedelta

# 1. Date de souscription (maintenant)
start_date = datetime.now()

# 2. Calcul de la durée (30 jours)
duration = timedelta(days=30)
end_date = start_date + duration

# 3. Formatage pour l'affichage
fmt = "%d/%m/%Y"
print(f"Abonnement souscrit le {start_date.strftime(fmt)}, valide jusqu'au {end_date.strftime(fmt)}")
```
</details>

### Exercice 2 - Planificateur de Réunion Globale {#exercice-2---reunion-globale}

🎯 **Objectif** : Manipuler `zoneinfo` et `astimezone`.

💼 **Mise en situation** : Une réunion est prévue à **15h00 heure de Londres**. À quelle heure doivent se connecter vos collègues de **Tokyo** et de **Los Angeles** ?

📝 **Énoncé** :
1.  Créez un objet datetime pour "2026-10-25 15:00:00" avec le fuseau "Europe/London".
2.  Convertissez cette date pour le fuseau "Asia/Tokyo".
3.  Convertissez cette date pour le fuseau "America/Los_Angeles".
4.  Affichez les heures locales.

📺 **Résultat attendu** :
```text
Londres : 2026-10-25 15:00:00+01:00
Tokyo   : 2026-10-25 23:00:00+09:00
LA      : 2026-10-25 07:00:00-07:00
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
from datetime import datetime
from zoneinfo import ZoneInfo

# 1. Définition de l'heure de la réunion à Londres
# Attention : ne pas oublier d'attacher le timezone dès la création
meeting_london = datetime(2026, 10, 25, 15, 0, 0, tzinfo=ZoneInfo("Europe/London"))

# 2. Conversion vers Tokyo
meeting_tokyo = meeting_london.astimezone(ZoneInfo("Asia/Tokyo"))

# 3. Conversion vers Los Angeles
meeting_la = meeting_london.astimezone(ZoneInfo("America/Los_Angeles"))

print(f"Londres : {meeting_london}")
print(f"Tokyo   : {meeting_tokyo}")
print(f"LA      : {meeting_la}")
```
</details>

### Exercice 3 - Le Parser de Logs {#exercice-3---parser-logs}

🎯 **Objectif** : Utiliser `strptime` et comparer des dates.

💼 **Mise en situation** : Vous analysez un fichier de logs. Vous devez détecter si une erreur est survenue il y a moins de 5 minutes.

📝 **Énoncé** :
1.  Soit la chaîne de log : `log_time_str = "2026-10-25 14:58:00"` (Simulez une heure proche de votre heure actuelle pour tester, ou ajustez la logique).
2.  Convertissez cette chaîne en objet datetime.
3.  Prenez l'heure actuelle (`datetime.now()`). *Astuce: assurez-vous de comparer des choses comparables (naïf vs naïf)*.
4.  Calculez la différence.
5.  Si la différence est < 5 minutes (300 secondes), affichez "Alerte récente !", sinon "Vieux log".

📺 **Résultat attendu** :
```text
Analyse du log : 2026-10-25 14:58:00
Différence : 120 secondes
Alerte récente !
```

<details>
<summary>💡 Voir le code complet commenté</summary>

```python
from datetime import datetime

# Simulation : on imagine que "maintenant" est 15h00
# Dans un vrai cas, on utiliserait datetime.now()
fake_now = datetime(2026, 10, 25, 15, 0, 0)
log_time_str = "2026-10-25 14:58:00"

# 1. Parsing de la date du log
log_date = datetime.strptime(log_time_str, "%Y-%m-%d %H:%M:%S")

print(f"Analyse du log : {log_date}")

# 2. Calcul du delta
# Note : log_date est naïf (pas de timezone), fake_now aussi. C'est compatible.
delta = fake_now - log_date
seconds_diff = delta.total_seconds()

print(f"Différence : {seconds_diff} secondes")

# 3. Vérification seuil (5 minutes = 300 secondes)
if seconds_diff < 300:
    print("🚨 Alerte récente !")
else:
    print("📁 Vieux log.")
```
</details>