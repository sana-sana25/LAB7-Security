# LAB7-Security - Rapport de TP — Analyse de sécurité Android avec DIVA et MobSF

## Réalisé par :

ASSEKNOUR Sana
ENSA Marrakech — Génie Cyberdéfense et Systèmes de Télécommunications 

---

# 1. Introduction

Dans le cadre de ce TP, nous avons réalisé une analyse de sécurité d’une application Android vulnérable appelée DIVA (Damn Insecure and Vulnerable App).

L’objectif principal était de :

* comprendre les vulnérabilités Android les plus courantes,
* effectuer une analyse statique et dynamique,
* exploiter certaines failles de sécurité,
* utiliser des outils professionnels d’analyse mobile.

L’application DIVA a été conçue spécialement pour l’apprentissage du pentest mobile et contient volontairement plusieurs vulnérabilités.

---

# 2. Objectifs du TP

Les objectifs du TP étaient les suivants :

* Installer et configurer un environnement Android sécurisé
* Utiliser Android Studio et un émulateur Android
* Installer et exécuter l’application DIVA
* Réaliser une analyse statique avec MobSF
* Réaliser une analyse dynamique avec ADB
* Exploiter plusieurs vulnérabilités Android
* Comprendre les mauvaises pratiques de développement mobile

---

# 3. Environnement de travail

## Matériel et logiciels utilisés

| Outil                      | Description                        |
| -------------------------- | ---------------------------------- |
| Android Studio             | Développement et émulation Android |
| Émulateur Pixel 5 API 30   | Exécution de DIVA                  |
| ADB (Android Debug Bridge) | Interaction avec l’émulateur       |
| MobSF                      | Analyse statique de l’APK          |
| Windows PowerShell         | Exécution des commandes            |
| SQLite3                    | Analyse des bases de données       |

---

# 4. Installation de l’application DIVA

Au début du TP, plusieurs problèmes de compatibilité sont apparus :

* incompatibilité Gradle/JDK,
* erreurs de dépendances,
* incompatibilité avec Android récent.

Après plusieurs modifications :

* ajout des repositories `google()` et `mavenCentral()`,
* utilisation d’un émulateur Android API 30,
* installation manuelle de l’APK,

l’application DIVA a finalement été exécutée avec succès sur l’émulateur Android.



---

# 5. Analyse statique avec MobSF

Une analyse statique complète a été réalisée à l’aide de MobSF.




<img width="1919" height="971" alt="Screenshot 2026-05-22 015022" src="https://github.com/user-attachments/assets/f784f79c-5d9c-4f87-a08a-ab9d612b557b" /><img width="1917" height="904" alt="Screenshot 2026-05-22 015115" src="https://github.com/user-attachments/assets/69d481c1-f336-4453-b77a-aba081c683cd" /><img width="1919" height="870" alt="Screenshot 2026-05-22 015132" src="https://github.com/user-attachments/assets/8d1c852d-72a5-4449-a41d-35e4ba237837" /><img width="1392" height="618" alt="Screenshot 2026-05-22 015226" src="https://github.com/user-attachments/assets/58d1d7bb-e597-43dc-bfb3-70d891c1efcf" />





## Informations générales de l’application

| Élément                  | Valeur            |
| ------------------------ | ----------------- |
| Application              | DIVA              |
| Package                  | jakhar.aseem.diva |
| Version Android minimale | API 15            |
| Target SDK               | API 23            |

---

# 5.1 Vulnérabilités détectées

## 1. Application signée avec un certificat Debug

MobSF a détecté que l’application utilise un certificat de debug.

### Risque :

Une application de production ne doit jamais être signée avec un certificat debug car cela facilite le reverse engineering.

---

## 2. Vulnérabilité Janus

L’application utilise uniquement la signature v1.

### Risque :

Cette configuration rend l’application vulnérable à l’attaque Janus sur certaines versions Android.

---

## 3. Debuggable = true

L’application autorise le debugging.

### Risque :

Un attaquant peut :

* attacher un debugger,
* manipuler l’application,
* observer son comportement interne.

---

## 4. allowBackup = true

L’application autorise la sauvegarde ADB.

### Risque :

Les données internes peuvent être extraites via ADB.

---

## 5. Activities exportées

MobSF a détecté plusieurs Activities exportées.

### Risque :

D’autres applications peuvent lancer ces activités sans authentification.

---

## 6. Permissions dangereuses

Permissions détectées :

* READ_EXTERNAL_STORAGE
* WRITE_EXTERNAL_STORAGE

### Risque :

Lecture et modification du stockage externe.

---

## 7. Librairies natives non sécurisées

Les bibliothèques `.so` ne possèdent pas :

* NX bit,
* Stack Canary,
* Fortify.

### Risque :

Possibilité d’exploitation mémoire.

---

# 6. Analyse dynamique

L’analyse dynamique a été réalisée principalement avec ADB et l’émulateur Android.

---

# 6.1 Vérification de l’émulateur

Commande utilisée :

```bash
.\adb devices
```

Résultat :

```text
List of devices attached
emulator-5554 device
```

L’émulateur Android était correctement détecté.

---

# 6.2 Vulnérabilité : Insecure Logging

## Description

L’application écrit des données sensibles dans les logs Android.


---

## Procédure

Ouverture des logs :

```bash
.\adb logcat | findstr diva
```

Dans l’application DIVA :

* ouverture de “Insecure Logging”
* saisie d’un faux numéro de carte bancaire
* clic sur “CHECK OUT”

---

## Résultat obtenu

Les informations sensibles apparaissent dans Logcat :

```text
Error while processing transaction with credit card: 1234567890
```
<img width="957" height="243" alt="image" src="https://github.com/user-attachments/assets/d545b585-ecb8-4616-9f4c-483a123b58c9" />


---

## Impact

Un attaquant ayant accès aux logs peut récupérer :

* numéros de cartes,
* mots de passe,
* informations sensibles.

---

# 6.3 Vulnérabilité : Insecure Data Storage — SQLite

## Description

L’application stocke des données sensibles dans une base SQLite non protégée.

---

## Accès au dossier interne

```bash
.\adb shell
cd /data/data/jakhar.aseem.diva/databases
ls
```
<img width="958" height="225" alt="Screenshot 2026-05-22 012406" src="https://github.com/user-attachments/assets/61ef9e8d-a3c4-4235-93a3-0bfe3d87064c" />

Résultat :

```text
divanotes.db
```

---

## Ouverture de la base SQLite

```bash
sqlite3 divanotes.db
```


<img width="960" height="98" alt="Screenshot 2026-05-22 012514" src="https://github.com/user-attachments/assets/42644395-3328-4797-8701-0dd030e89c1a" />

---

## Affichage des tables

```sql
.tables
```

Résultat :

```text
android_metadata
notes
```

---

## Extraction des données

```sql
SELECT * FROM notes;
```

<img width="962" height="280" alt="Screenshot 2026-05-22 012621" src="https://github.com/user-attachments/assets/f52e51a6-cb17-4468-9e09-087102c32173" />

Résultat :

```text
1|office|10 Meetings. 5 Calls. Lunch with CEO
2|home|Buy toys for baby, Order dinner
3|holiday|Either Goa or Amsterdam
4|Expense|Spent too much on home theater
5|Exercise|Alternate days running
```

---

## Impact

Les données sensibles sont stockées en clair dans SQLite.

---

# 6.4 Vulnérabilité : Shared Preferences non sécurisées

## Description

Les identifiants utilisateurs sont stockés en clair dans SharedPreferences.

---

## Procédure

Dans DIVA :

* ouverture de “Insecure Data Storage Part 1”
* saisie :

  * username : sana
  * password : sana@2026

<img width="598" height="793" alt="Screenshot 2026-05-22 012743" src="https://github.com/user-attachments/assets/13c62a85-fa22-4142-8023-ccb5ad4f500c" />

---

## Accès au fichier XML

```bash
cd /data/data/jakhar.aseem.diva/shared_prefs
ls
```

Résultat :

```text
jakhar.aseem.diva_preferences.xml
```

<img width="963" height="222" alt="Screenshot 2026-05-22 013628" src="https://github.com/user-attachments/assets/61aa1018-ec93-416e-a44d-375e84a5595c" />

---

## Lecture du fichier

```bash
cat jakhar.aseem.diva_preferences.xml
```

<img width="961" height="323" alt="Screenshot 2026-05-22 013644" src="https://github.com/user-attachments/assets/7902055c-d3b4-4cc1-8393-cd6d45da6891" />

---

## Résultat obtenu

```xml
<map>
    <string name="password">sana@2026</string>
    <string name="user">sana</string>
</map>
```

---

## Impact

Les credentials sont stockés sans chiffrement.

Un attaquant peut facilement :

* récupérer les mots de passe,
* voler des comptes utilisateurs.

---

# 6.5 Vulnérabilité : Access Control Issues

## Description

Certaines Activities sont exportées sans protection.

<img width="487" height="693" alt="Screenshot 2026-05-22 014154" src="https://github.com/user-attachments/assets/638dee31-c2d4-4aac-bd69-91fd7e108c75" />

---

## Exploitation

Commande utilisée :

```bash
am start -n jakhar.aseem.diva/.APICredsActivity
```

<img width="959" height="104" alt="Screenshot 2026-05-22 014304" src="https://github.com/user-attachments/assets/e4a86ffa-63b0-4a12-a581-9f1518d47f50" />

---

## Résultat

L’activité s’ouvre directement sans authentification.

Les informations suivantes sont affichées :

<img width="603" height="391" alt="Screenshot 2026-05-22 014209" src="https://github.com/user-attachments/assets/e0cc52c1-f3eb-4f0f-86b3-f5ff86fbba1e" />

```text
API Key: 123secretapikey123
API Username: diva
API Password: p@ssword
```

---

## Impact

Un attaquant peut accéder à des informations critiques sans permission.

---

# 7. Difficultés rencontrées

Pendant le TP, plusieurs problèmes techniques ont été rencontrés :

* incompatibilités Gradle,
* erreurs de dépendances,
* incompatibilité Android récente,
* problèmes MobSF Dynamic Analysis,
* problèmes d’installation APK.

Ces problèmes ont été résolus progressivement grâce :

* à la modification des configurations Gradle,
* à l’utilisation d’un émulateur API 30,
* à l’utilisation directe d’ADB.

---

# 8. Conclusion

Ce TP a permis de comprendre plusieurs vulnérabilités Android importantes.

Les principales failles observées sont :

* stockage non sécurisé,
* logging sensible,
* mauvais contrôle d’accès,
* permissions dangereuses,
* mauvaise configuration de sécurité.

L’utilisation de MobSF et ADB a permis de :

* réaliser une analyse statique,
* effectuer une analyse dynamique,
* exploiter concrètement plusieurs vulnérabilités.

Ce TP montre l’importance :

* du chiffrement des données,
* de la sécurisation des logs,
* du contrôle d’accès,
* des bonnes pratiques de développement Android.

---

# 9. Outils utilisés

* Android Studio
* Android Emulator
* ADB
* MobSF
* SQLite3
* PowerShell


