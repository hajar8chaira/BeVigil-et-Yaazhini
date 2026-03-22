# Mobile Security Lab: Yaazhini & BeVigil Analysis

**Auteur :** [Votre Prénom/Nom]  
**Application Cible :** `InsecureBankv2.apk`  
**Date du rapport :** $(Get-Date -Format "yyyy-MM-dd")  
**Outils :** BeVigil (Cloud/Web Scan), Yaazhini (SAST/DAST Tool)

## 📌 Vue d'ensemble (Overview)
Ce projet vise à évaluer la posture de sécurité d'une application Android pédagogique nommée **InsecureBankv2**. À travers une approche structurée (Task 0 à Task 6), ce laboratoire met en œuvre l'analyse statique et dynamique. Nous avons utilisé **BeVigil** pour cartographier l'intelligence des menaces externes (Surface d'attaque OSINT) et **Yaazhini** pour une fouille en profondeur (analyse de code binaire, vulnérabilités logiques, configurations Manifest).

---

## 📑 Sommaire (Table of Contents)
1. [Tasks Summary (Résumé des Actions)](#-1-tasks-summary-résumé-des-actions)
2. [Analysis Results (Yaazhini)](#-2-analysis-results-yaazhini)
3. [Yaazhini Notes](#-3-yaazhini-notes)
4. [Observations & Recommendations](#-4-observations--recommendations)

---

## 🚀 1. Tasks Summary (Résumé des Actions)

### Task 0 — Règles, périmètre et éthique
**Objectif :** Définir le cadre légal pour se prémunir d'incidents juridiques.
- **Étapes :** Création du dossier `00-scope` et définition de l'artefact cible (InsecureBankv2) et des limites (aucun test d'exploitation intrusif réel).
- **Commandes exécutées :**
  ```powershell
  mkdir 00-scope
  # Création du fichier périmètre scope.md via Out-File
  ```
- **Observations :** ![Screenshot_Task0_Scope](assets/task0_scope.png)
- **Résultats :** Périmètre défini avec précision dans `scope.md`.
- **Apprentissages :** La délimitation d'un "scope d'audit" est la clef de voûte de toute prestation de pentest éthique.

### Task 1 — Préparation du workspace et traçabilité
**Objectif :** Standardiser l'espace pour une efficacité professionnelle (Reproductibilité).
- **Étapes :** Création de l'arborescence, du fichier d'informations d'analyse et du fichier de logs séquentiels.
- **Commandes exécutées :**
  ```powershell
  mkdir 01-bevigil, 02-yaazhini, 03-triage, 04-report
  "Log init..." | Out-File -FilePath "commands.log"
  ```
- **Observations :** L'environnement est prêt et documenté.
- **Résultats :** Création de `analyse_info.txt` recensant hôte, analyste et outils.
- **Apprentissages :** Assurer un suivi granulaire des commandes via des scripts permet d'élaborer un rapport robuste a posteriori.

### Task 2 — Préparer l'artefact autorisé
**Objectif :** Sceller l'empreinte d'intégrité de l'application via des algorithmes cryptographiques.
- **Étapes :** Copier l'APK fourni vers le répertoire de travail et vérifier son hash.
- **Commandes exécutées :**
  ```powershell
  $hash = Get-FileHash -Path "00-scope\InsecureBankv2.apk" -Algorithm SHA256
  ```
- **Observations :** ![Screenshot_Task2_Hash](assets/task2_hash.png)
- **Résultats :** Hash SHA-256 consigné : `B18AF2A0E44D763...`
- **Apprentissages :** Le hachage garantit que les observations futures sont liées à cette version exacte du fichier de l'entreprise.

### Task 3 — Démarrage et prise en main BeVigil
**Objectif :** S'interfacer avec l'arsenal Cloud de balayage d'applications de *CloudSEK*.
- **Étapes :** Authentification Web, configuration d'un projet, lancement du scan de l'APK (Score: 7.4) et exportation des archives CSV.
- **Commandes exécutées :**
  ```powershell
  Move-Item -Path "Downloads\*.csv" -Destination "01-bevigil\"
  ```
- **Observations :** ![Screenshot_Task3_Bevigil](assets/task3_bevigil.png)
- **Résultats :** Rapport de vulnérabilités, d'assets réseau et d'intégrité exporté.
- **Apprentissages :** Extraire les data vers CSV favorise un *Triage* (tri par criticité) plus facile par de puissants parsers de données.

### Task 4 — Collecte BeVigil : Endpoints, Domaines, Emails
**Objectif :** Structurer les retours BeVigil (Séparation entre "Évidences" et "Hypothèses").
- **Étapes :** Analyse des fichiers `.csv`. Localisation spécifique de `/devlogin`, présence d'emails de tests, d'adresses Google externes.
- **Observations :** ![Screenshot_Task4_Collecte](assets/task4_collecte.png)
- **Résultats :** Production du fichier analytique `bevigil_notes.md`.
- **Apprentissages :** Les scanners rapportent des faux positifs, d'où l'importance de catégoriser (Certain vs Hypothèse). 

### Task 5 — Démarrage et prise en main Yaazhini
**Objectif :** Extraire le code binaire (*Disassembly*) pour remonter les failles Android spécifiques.
- **Étapes :** Déploiement de l'exécutable/script Yaazhini sur l'artefact.
- **Commandes exécutées :**
  ```powershell
  & "yaazhini.exe" -apk "00-scope\InsecureBankv2.apk" -output "02-yaazhini"
  ```
- **Observations :** L'outil a dézippé l'APK, extrait le `classes.dex`, puis décompilé au format Smali et pseudo-Java. ![Screenshot_Task5_Yaazhini](assets/task5_yaazhini.png)
- **Résultats :** Scan SAST approfondi des dépendances et de la structure du Manifest.
- **Apprentissages :** Différentiel majeur avec BeVigil : Yaazhini rentre dans le cœur du code pour pointer *la ligne exacte* de la faille.

### Task 6 — Collecte Yaazhini : Indices d'exposition
**Objectif :** Fouiller manuellement les alertes techniques retournées par l'outil.
- **Étapes :** Revue fine du compte-rendu SAST, identification des exports de composants et données figées dans les ressources.
- **Observations :** Les *Intents* Android sont mal sécurisés.
- **Résultats :** Synthèse d'indicateurs de compromission critiques dans `yaazhini_notes.md` (Voir section "Yaazhini Notes" ci-dessous).
- **Apprentissages :** Bien que les données soient masquées dans la documentation pour des raisons d'éthique, leur impact architectural est fondamental.

---

## 🛡️ 2. Analysis Results (Yaazhini)

Le tableau ci-dessous consolide les vulnérabilités primaires détectées via le module d'analyse *SAST* de Yaazhini sur `InsecureBankv2.apk`.

| Vulnérabilité / Identification | Niveau de Risque | Sévérité | CVSS (Estimé) | Fréquence | Chemin du Fichier (*File Path*) |
|---|---|---|---|---|---|
| **Composant Activity Exporté non protégé** | Critique | Haute | 8.8 (High) | 4+ | `resources/AndroidManifest.xml` |
| **Utilisation Crypto Obsolète (AES/CBC)** | Élevé | Moyenne | 5.9 (Med) | 3 | `sources/com/android/insecurebankv2/CryptoClass.java` |
| **Cleartext Traffic Enabled (HTTP Autorisé)** | Critique | Haute | 7.5 (High) | 1 (Global) | `resources/AndroidManifest.xml` |
| **Backup ADB Activé (`allowBackup=true`)** | Modéré | Basse | 4.3 (Low) | 1 | `resources/AndroidManifest.xml` |
| **Insecure Data Storage (SharedPrefs)** | Critique | Haute | 7.5 (High) | 2 | `sources/.../DoLogin.java` |

### Descriptions Techniques :
1. **Composant Exporté :** Des Activités (ex: `DoTransfer`) peuvent être invoquées depuis l'extérieur, permettant de court-circuiter l'authentification.
2. **Crypto Obsolète :** Chiffrement attaqué par Oracle Paddings.
3. **Cleartext Traffic :** L'application ne possède aucune restriction l'empêchant de faire des requêtes en clair (`HTTP://`). Les transferts bancaires sont à la merci du réseau local.
4. **Backup ADB :** Un utilisateur ou technicien pourrait extraire les données chiffrées/en dur directement du téléphone (si débogage USB activé).
5. **Stockage Insécure :** Les mots de passe et noms d'utilisateur ne sont ni "hashés" correctement ni protégés dans l'écosystème Keystore. 

---

## 📝 3. Yaazhini Notes

Cette section agrège notre rapport `yaazhini_notes.md` interne, reprenant 5 découvertes fondamentales :

* **Élément 1 : Hardcoded API Key / Credentials (Secret)**
  * *Localisation* : `resources/res/values/strings.xml`
  * *Description* : Le champ `loginscreen_password` contient des données brutes en clair [MASQUÉ]. 
  * *Impact* : Le binaire est une passoire ; les assaillants récupèrent immédiatement des pass par défaut ou mots de passe admin.
  * *Remédiation* : Bannir le codage de mot de passe en dur. Utiliser Android Keystore system.

* **Élément 2 : Traffic réseau en clair autorisé (Configuration)**
  * *Localisation* : `resources/AndroidManifest.xml` (`usesCleartextTraffic`)
  * *Description* : Aucun flag n'oblige le recours à HTTPS strict.
  * *Impact* : Écoute passive de mot de passe et altération des requêtes API REST au vol.
  * *Remédiation* : Activer une Network Security Config pour forcer le HTTP(S).

* **Élément 3 : Activités non authentifiées (Composants)**
  * *Localisation* : `AndroidManifest.xml` (`android:exported=true`)
  * *Description* : Les façades d'applications comme la fonction de virement sont appelables en direct (`adb shell am start -n com...DoTransfer`).
  * *Impact* : Bypasse total de sécurité logique de l'utilisateur. 
  * *Remédiation* : Définir `exported="false"` ou limiter via des vérifications de permissions fortes.

* **Élément 4 : Autorisation de sauvegarde système**
  * *Localisation* : `AndroidManifest.xml` (`allowBackup="true"`)
  * *Description* : Le droit de pomper les données d'user/applicative via outil ADB est ouvert.
  * *Impact* : Exfiltration complète de l'historique bancaire par action physique sur l'appareil.
  * *Remédiation* : Définir globalement à `false`.

* **Élément 5 : Requêtes dynamiques vulnérables (SQLi)**
  * *Localisation* : Fichiers Java sous `sources/com/google/android/gms/`
  * *Description* : Concaténations `rawQuery()` non assainies.
  * *Impact* : L'attaquant manipule l'historique et la structure depuis des "Inputs" en UI.
  * *Remédiation* : Passer au Prepared Statements (Paramétrisation via des Room/SQLite query builders protecteurs).

---

## 💡 4. Observations & Recommendations

### Synthèse (Key Insights)
L'application *InsecureBankv2* (comme prévu) est un modèle d'anti-patterns sécurité mobile. Tout ce qu'un développeur ne doit pas faire y figure : codage des secrets en base, transmissions claires HTTP, sauvegarde de données locales en `.xml` brutes non chiffrées, fonctions d'obfuscations vaines (Anti-VM) mais pas de protection architecturale réelle sur l'application elle-même. Les endpoints `/devlogin` laissent d'énormes trous dans la raquette applicative.

### Amélioration des Pratiques (Security Practices to improve)
- **Principe du Moindre Privilège :** Restreindre drastiquement la portée des composants locaux (Intent, Services, Content Provider) avec des permissions signatures.
- **Chiffrement "By Default" :** Mettre un terme à la communication HTTP en remplaçant la bibliothèque `DefaultHttpClient()` par `OkHttpClient` incluant du *Certificate Pinning*.
- **Gestion de Secret :** Déléguer la charge virale de l'authentification soit à un serveur d'API renforcé par des tokens OAUTH/JWT, soit aux TEE des appareils Android modernes. Ne jamais conserver des données dans les `Shared Preferences` ou `strings.xml`.

### Prochaines Étapes (Next Steps)
1. **Évaluation Dynamique DAST approfondie** : Exécuter l'application sur un périphérique rooté (comme le framework Frida/Objection) pour accrocher "en vol" (Hooking) les appels non sécurisés constatés à l'étape SAST.
2. **Remédiation Code (Développement)** : Soumettre une *Pull Request* ou révision complète du code Java et du fichier d'infrastructure *Manifest* à la lumière du top 10 OWASP Mobile. Lancer ensuite un nouveau scan de validation via Yaazhini.