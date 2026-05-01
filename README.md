# Lab 7- Analyse dynamique Android avec MobSF

## Objectif

Realiser une analyse dynamique complete d'une application Android vulnerable (DIVA - Damn Insecure and Vulnerable App) en utilisant MobSF pour detecter des failles de securite en temps reel : logs runtime, stockage insecure, composants exportes et instrumentation Frida.

---

## Environnement de travail

| Element | Detail |
|---|---|
| VM d'analyse | Mobexler (Debian) |
| Emulateur | Genymotion Android 8.1 (API 27) |
| IP emulateur | 192.168.56.104:5555 |
| Outils | ADB, MobSF, Frida Server |
| Application cible | DIVA (Damn Insecure and Vulnerable App) |

Note : MobSF est deja installe dans Mobexler. Il n'est pas necessaire de le reinstaller via Docker ou de le recloner. L'emulateur Genymotion est configure avec root actif.

---

## Fiche perimetre du test

```
Application     : DIVA (Damn Insecure and Vulnerable App)
Version         : 1.0
Package         : jakhar.aseem.diva
Support         : Genymotion AVD Android 8.1 API 27
Objectif        : Detection de vulnerabilites par analyse dynamique
Donnees         : Fictives uniquement (41119770856666, testuser/supersecret123)
Reseau          : Host-Only (192.168.56.x)
Analyste        : analyste
Methode         : Analyse statique + dynamique avec MobSF
Outil           : MobSF preinstalle dans Mobexler
```

---

## Etape 1 - Verification de l'environnement

Verification de la connexion ADB entre Mobexler et l'emulateur Genymotion :

```bash
adb connect 192.168.56.104:5555
adb devices
```

Resultat attendu :

```
192.168.56.104:5555     device
```

<img width="791" height="181" alt="image" src="https://github.com/user-attachments/assets/92d67006-4a39-4dbe-b627-cef3c85561e4" />
---

 
Verification de la version Android de l'emulateur :

```bash
adb shell getprop ro.build.version.release
adb shell getprop ro.build.version.sdk
```

Resultat : Android 8.1, API 27

---
<img width="905" height="188" alt="image" src="https://github.com/user-attachments/assets/a41703d1-b80b-4369-8d7e-53bb0ec464c8" />

## Etape 2 - Lancement de MobSF

MobSF est preinstalle dans Mobexler. Lancement avec l'identifiant de l'emulateur Genymotion :

```bash
MOBSF_ANALYZER_IDENTIFIER=192.168.56.104:5555 ~/.local/bin/mobsf
```

<img width="1513" height="490" alt="image" src="https://github.com/user-attachments/assets/313b1ddc-1e8a-43d3-9c5e-0a7f7c74ad2d" />

---


Acces a l'interface depuis Firefox dans Mobexler :

```
http://127.0.0.1:8000
```


<img width="1600" height="764" alt="image" src="https://github.com/user-attachments/assets/54d34245-6bf3-4ca7-b0d9-0bc13d25cdc4" />

## Etape 3 - Compilation et installation de DIVA

L'APK DIVA n'etant plus disponible directement en telechargement, elle est compilee depuis le code source.

Compilation de l'APK :

```bash
chmod +x gradlew
./gradlew assembleDebug
```

Resultat attendu :

```
BUILD SUCCESSFUL in 2m 10s
23 actionable tasks: 23 executed
```

<img width="1407" height="542" alt="image" src="https://github.com/user-attachments/assets/b53f2394-8f23-4582-aba9-c85481e717fe" />

---


Copie et renommage de l'APK :

```bash
cp ./app/build/outputs/apk/debug/app-debug.apk ~/DivaApplication.apk
```
<img width="1437" height="310" alt="image" src="https://github.com/user-attachments/assets/94a5b1d3-61e3-45ea-a304-ad82d20ab6be" />

---


Installation sur l'emulateur :

```bash
adb connect 192.168.56.104:5555
adb -s 192.168.56.104:5555 install ~/DivaApplication.apk
```

Resultat :

```
Success
```

<img width="1012" height="151" alt="image" src="https://github.com/user-attachments/assets/6b6db784-d11d-4f36-a7fc-63a61be45e41" />

---
  
<img width="1291" height="216" alt="image" src="https://github.com/user-attachments/assets/ad0c3a8e-ae43-4516-9205-a4dc8a428cce" />

---

Verification de l'installation :

```bash
adb shell pm list packages | grep diva
```

Resultat :

```
package:jakhar.aseem.diva
```

<img width="976" height="192" alt="image" src="https://github.com/user-attachments/assets/33ae39ea-3f4f-42d1-9ea3-681401e35c88" />


---


## Etape 4 - Analyse statique avec MobSF

Dans l'interface MobSF, cliquer sur "Upload & Analyze" et selectionner le fichier DivaApplication.apk.

Attendre la fin de l'analyse (1-2 minutes).

Le rapport statique affiche les informations suivantes :

| Champ | Valeur |
|---|---|
| Nom | DIVA |
| Package | jakhar.aseem.diva |
| Min SDK | 15 |
| Target SDK | 29 |
| Security Score | 55/100 |
| Activities | 17 |
| Providers | 1 |

---

<img width="1600" height="824" alt="image" src="https://github.com/user-attachments/assets/2cce5a3b-fe78-46ea-a14d-e3a4e6d7706f" />

## Etape 5 - Configuration de Frida Server

L'emulateur Genymotion utilise une architecture x86. La version correcte de Frida Server doit etre telechargee depuis https://github.com/frida/frida/releases en choisissant la variante android-x86.

Transfert du binaire depuis le PC Windows vers l'emulateur via ADB :

```bash
adb push "C:\Users\houda\Downloads\frida-server-17.9.3-android-x86\frida-server-17.9.3-android-x86" /data/local/tmp/frida-server
adb shell chmod +x /data/local/tmp/frida-server
```

Verification de la version :

```bash
adb shell /data/local/tmp/frida-server --version
```

Resultat :

```
17.9.3
```

Lancement de Frida Server en arriere-plan :

```bash
adb shell /data/local/tmp/frida-server &
```

---
<img width="1472" height="355" alt="image" src="https://github.com/user-attachments/assets/ed12e9cc-d63d-49d3-b810-dcba620aeb42" />

## Etape 6 - Lancement de l'analyse dynamique

Dans le rapport statique de MobSF, cliquer sur "Start Dynamic Analyzer".

MobSF realise automatiquement les operations suivantes :

- Connexion a l'emulateur Genymotion
- Installation du certificat CA MobSF pour l'interception HTTPS
- Configuration du proxy HTTP/HTTPS global
- Lancement de Frida Server
- Demarrage de l'application DIVA sur l'emulateur

Le message "Testing Environment is Ready!" confirme que l'environnement est pret.

---
<img width="1600" height="794" alt="image" src="https://github.com/user-attachments/assets/e1fcbf6e-4b17-4702-9568-4b4659124453" />

---

<img width="1600" height="769" alt="image" src="https://github.com/user-attachments/assets/005cb1e6-6d3a-4d4c-94b6-acc0c640eacb" />


## Etape 7 - Challenge 1 : Insecure Logging

Dans l'emulateur, ouvrir l'application DIVA et cliquer sur "1. INSECURE LOGGING".

Saisir un numero de carte bancaire fictif dans le champ :

```
41119770856666
```

<img width="502" height="1017" alt="image" src="https://github.com/user-attachments/assets/f19b4023-8ce3-4ae8-8fec-f3f37343cdd9" />

---
Cliquer sur "CHECK OUT".

Observation des logs en temps reel depuis MobSF (onglet Logcat Stream) ou depuis le terminal Mobexler :

```bash
adb -s 192.168.56.104:5555 logcat -d | grep -i "4111\|credit\|card\|diva"
```

Resultat dans les logs :

```
E diva-log: Error while processing transaction with credit card: 41119770856666
```
<img width="1600" height="630" alt="image" src="https://github.com/user-attachments/assets/b6209195-6081-40c0-bcdb-059f0524c27c" />

---

Vulnerabilite identifiee : l'application logue des donnees sensibles (numero de carte bancaire) en clair dans les logs systeme Android. Un attaquant disposant d'un acces aux logs peut lire ces informations sans aucune autorisation particuliere.

Classification : OWASP MASVS-STORAGE-1 / Insecure Logging

---

## Etape 8 - Challenge 4 : Insecure Data Storage

Dans l'emulateur, cliquer sur "4. INSECURE DATA STORAGE - PART 2".

Saisir les donnees suivantes :

```
User name : testuser
Password  : supersecret123
```

<img width="513" height="1017" alt="image" src="https://github.com/user-attachments/assets/9e47b299-a33b-4744-9d0b-1450329378b0" />

---

Cliquer sur "SAVE".

Recherche des fichiers de stockage de l'application :

```bash
adb -s 192.168.56.104:5555 shell run-as jakhar.aseem.diva ls -la /data/data/jakhar.aseem.diva/databases/
```

Resultat :

```
-rw-rw---- 1 u0_a70 u0_a70 20480 divanotes.db
-rw-rw---- 1 u0_a70 u0_a70 16384 ids2
```

Lecture du fichier ids2 contenant les identifiants :

```bash
adb -s 192.168.56.104:5555 shell run-as jakhar.aseem.diva cat /data/data/jakhar.aseem.diva/databases/ids2
```

Resultat (extrait lisible) :

```
testuser : supersecret123
```
<img width="1600" height="209" alt="image" src="https://github.com/user-attachments/assets/6bf703bc-7b43-474b-8681-4bcd8bae28dc" />


---

Vulnerabilite identifiee : les identifiants utilisateur sont stockes en clair dans une base de donnees SQLite sans aucun chiffrement. Tout attaquant disposant d'un acces root ou d'un backup ADB peut extraire ces donnees.

Classification : OWASP MASVS-STORAGE-1 / Insecure Data Storage

-Les notes personnelles sont egalement stockees en clair dans la base SQLite.

---

## Etape 9 - Test des composants exportes

Dans MobSF Dynamic Analyzer, cliquer sur "Start Exported Activity Tester".

MobSF identifie les activities exportees de DIVA et tente de les lancer depuis l'exterieur de l'application.

Resultat :

```
jakhar.aseem.diva.APICredsActivity
jakhar.aseem.diva.APICreds2Activity
```
<img width="1600" height="616" alt="image" src="https://github.com/user-attachments/assets/0a00779b-6dc6-488a-a059-362a8faa124c" />

---

Vulnerabilite identifiee : des activities sont exportees sans controle d'acces. Elles peuvent etre demarrees par n'importe quelle application tierce installee sur l'appareil, exposant des ecrans contenant des informations sensibles.

Classification : OWASP MASVS-PLATFORM-1 / Improper Export of Android Application Components

---

## Etape 10 - Captures d'ecran

Dans MobSF Dynamic Analyzer, cliquer sur "Take a Screenshot" pour capturer l'etat de l'emulateur a chaque etape.

Les captures sont disponibles dans la section "Screenshots" du menu de gauche de l'interface Dynamic Analyzer.

---
<img width="1600" height="590" alt="image" src="https://github.com/user-attachments/assets/ad9567c1-d686-4e36-ad4e-bc682b3d4e33" />


## Etape 11 - Generation du rapport

Dans MobSF Dynamic Analyzer, cliquer sur "Generate Report".

Le rapport regroupe les elements suivants :

- Logs captures pendant l'analyse (Logcat Stream)
- Fichiers detectes sur le systeme de fichiers de l'application
- Screenshots des activities et composants exportes

---
<img width="1600" height="799" alt="image" src="https://github.com/user-attachments/assets/cc321b70-73d2-4741-b892-e0dcbf214a84" />

## Etape 12 - Nettoyage de l'environnement

Desinstallation de l'application de test :

```bash
adb uninstall jakhar.aseem.diva
```

Arret de MobSF (Ctrl+C dans le terminal Mobexler).

Deconnexion ADB :

```bash
adb disconnect
```

---

## Resume des commandes utilisees

| Commande | Objectif |
|---|---|
| `adb devices` | Verifier la connexion a l'emulateur |
| `adb root` | Activer les privileges root |
| `adb remount` | Remonter la partition systeme en lecture-ecriture |
| `adb install DivaApplication.apk` | Installer l'application DIVA |
| `adb shell pm list packages` | Lister les applications installees |
| `adb shell am start -n jakhar.aseem.diva/.MainActivity` | Lancer DIVA manuellement |
| `adb logcat \| grep -i "diva\|card"` | Capturer les logs Android en temps reel |
| `adb shell run-as jakhar.aseem.diva ls -la /data/data/jakhar.aseem.diva/databases/` | Lister les bases de donnees de l'application |
| `adb shell run-as jakhar.aseem.diva cat /data/data/jakhar.aseem.diva/databases/ids2` | Lire le fichier contenant les identifiants en clair |
| `adb shell /data/local/tmp/frida-server --version` | Verifier la version de Frida Server |
| `adb uninstall jakhar.aseem.diva` | Desinstaller l'application apres le test |

---

## Vulnerabilites identifiees

| N | Challenge | Vulnerabilite | Donnee exposee | Reference OWASP |
|---|---|---|---|---|
| 1 | Insecure Logging | Log de donnees sensibles en clair | Numero de carte bancaire : 41119770856666 | MASVS-STORAGE-1 |
| 2 | Insecure Data Storage Part 2 | Stockage d'identifiants sans chiffrement | testuser / supersecret123 dans ids2 | MASVS-STORAGE-1 |
| 3 | Exported Activities | Composants exportes sans controle d'acces | APICredsActivity, APICreds2Activity | MASVS-PLATFORM-1 |

---

## Conclusion

Ce lab a permis de :

- Configurer un environnement d'analyse dynamique avec MobSF preinstalle dans Mobexler
- Compiler l'APK DIVA depuis le code source en cas d'indisponibilite du binaire
- Detecter une faille de logging insecure exposant des numeros de carte bancaire dans les logs systeme
- Detecter une faille de stockage insecure avec des identifiants en clair dans une base SQLite
- Identifier des composants Android exportes sans protection d'acces
- Configurer Frida Server pour l'instrumentation avancee
- Generer un rapport complet d'analyse dynamique via MobSF

---

## Ressources

- DIVA source : https://github.com/payatu/diva-android
- MobSF Documentation : https://github.com/MobSF/Mobile-Security-Framework-MobSF
- Frida : https://frida.re
- OWASP MASVS : https://mas.owasp.org/MASVS/
- - OWASP MASTG : https://mas.owasp.org/MASTG/


## Auteur
**H-oubane**
