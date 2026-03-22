# Mobile Security Lab: Yaazhini & BeVigil Analysis

**Auteur :** Chaira Hajar  
**Application Cible :** `InsecureBankv2.apk`  
**Date du rapport :** 22/03/2026  
**Outils :** BeVigil / BigDvil (Web Scan), Yaazhini (SAST/DAST Tool)

## Vue d'ensemble (Overview)
Ce projet vise à évaluer la posture de sécurité d'une application Android pédagogique nommée **InsecureBankv2**. 

**Qu'est-ce que InsecureBankv2 ?**
Il s'agit d'une application bancaire factice "délibérément vulnérable" (*Vulnerable By Design*). Elle a été créée par des experts en sécurité pour permettre aux étudiants et professionnels de s'entraîner au pentest mobile. Elle regorge d'erreurs classiques de développement réel (composants exportés, trafic en clair, mots de passe codés en dur, cryptographie obsolète).

**Pourquoi BigDvil / BeVigil et Yaazhini ?**
Pour une analyse exhaustive, deux points de vue sont nécessaires :
1. **BigDvil / BeVigil (L'approche OSINT / Cloud) :** Permet une analyse statique dite "Black-Box" depuis l'extérieur. L'outil agit comme un moteur de recherche de sécurité. Sans exécuter l'application, il cartographie presque instantanément sa surface d'attaque externe globale (Endpoints d'API ouverts, sous-domaines, e-mails oubliés, vulnérabilités massives).
2. **Yaazhini (L'approche SAST en profondeur) :** Permet d'investiguer le code en lui-même (White/Grey-Box). Après décompilation de l'APK, cet outil descend dans la logique binaire. Il permet de repérer non seulement *qu'il y a une faille*, mais de donner aux développeurs le fichier de code Java exact et la ressource XML précise à corriger.

À travers une approche structurée (d'une Étape Préliminaire à l'Étape 9), ce laboratoire met en œuvre l'analyse statique et dynamique de bout en bout.

---

## Sommaire (Table of Contents)
1. [Chronologie de l'Audit](#1-chronologie-de-laudit)
   * [Étape Préliminaire — Définition Officielle du Périmètre](#étape-préliminaire--définition-officielle-du-périmètre-scope)
   * [Étape 1 — Préparation de Cadrage et Traçabilité de l'Audit](#étape-1--préparation-de-cadrage-et-traçabilité-de-laudit)
   * [Étape 2 — Validation de l'Intégrité de la Cible](#étape-2--validation-de-lintégrité-de-la-cible-hashing-sha-256)
   * [Étape 3 — Scan OSINT avec CloudSEK / BeVigil](#étape-3--scan-osint-avec-cloudsek--bevigil)
   * [Étape 4 — Reconnaissance Externe](#étape-4--reconnaissance-externe-domaines-emails-endpoints-api)
   * [Étape 5 — Mise en Cale Sèche et Décompilation binaire via Yaazhini](#étape-5--mise-en-cale-sèche-et-décompilation-binaire-via-yaazhini)
   * [Étape 6 — Investigation des Fichiers Révélés](#étape-6--investigation-des-fichiers-révélés-indices-de-sast-profond)
   * [Étape 7 — Consolidation, Nettoyage et Élimination des Faux Positifs](#étape-7--consolidation-nettoyage-et-élimination-des-faux-positifs)
   * [Étape 8 — Classification et Assignation des Risques selon Standards OWASP](#étape-8--classification-et-assignation-des-risques-selon-standards-owasp)
   * [Étape 9 — Finalisation et Extrants pour les Décideurs](#étape-9--finalisation-et-extrants-pour-les-décideurs)
2. [Analysis Results (Yaazhini)](#2-analysis-results-yaazhini)
3. [Yaazhini Notes](#3-yaazhini-notes)
4. [Observations & Recommendations](#4-observations--recommendations)

---

## 1. Chronologie de l'Audit

### Étape Préliminaire — Définition Officielle du Périmètre (Scope)
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

### Étape 1 — Préparation de Cadrage et Traçabilité de l'Audit
**Objectif :** Standardiser l'espace pour une efficacité professionnelle (Reproductibilité).
- **Étapes :** Création de l'arborescence, du fichier d'informations d'analyse et du fichier de logs séquentiels.
- **Commandes exécutées :**
  ```powershell
  mkdir 01-bevigil, 02-yaazhini, 03-triage, 04-report
  "Log init..." | Out-File -FilePath "commands.log"
  ```
- **Observations :** L'environnement est prêt et documenté. Voici le résultat de l'arborescence :
  ![Création de l'arborescence](assets/dir_folders.png)
- **Résultats :** Création de `analyse_info.txt` recensant hôte, analyste et outils.
- **Apprentissages :** Assurer un suivi granulaire des commandes via des scripts permet d'élaborer un rapport robuste a posteriori.

### Étape 2 — Validation de l'Intégrité de la Cible (Hashing SHA-256)
**Objectif :** Sceller l'empreinte d'intégrité de l'application via des algorithmes cryptographiques.
- **Étapes :** Copier l'APK fourni vers le répertoire de travail et vérifier son hash.
- **Commandes exécutées :**
  ```powershell
  $hash = Get-FileHash -Path "00-scope\InsecureBankv2.apk" -Algorithm SHA256
  ```
- **Observations :** Le calcul de l'empreinte permet d'obtenir le hash exact du binaire :
  ![Calcul du Hash SHA-256](assets/hash_sha256.png)
- **Résultats :** Hash SHA-256 consigné : `B18AF2A0E44D763...`
- **Apprentissages :** Le hachage garantit que les observations futures sont liées à cette version exacte du fichier de l'entreprise.

### Étape 3 — Scan OSINT avec CloudSEK / BeVigil
**Objectif :** S'interfacer avec l'arsenal Cloud de balayage d'applications de *CloudSEK*.
- **Étapes :** Authentification Web, configuration d'un projet, lancement du scan de l'APK (Score: 7.4) et exportation des archives CSV.
- **Commandes exécutées :**
  ```powershell
  Move-Item -Path "Downloads\*.csv" -Destination "01-bevigil\"
  ```
- **Observations :** Processus de soumission de l'APK sur la plateforme CloudSEK et génération du rapport :
  ![Upload BeVigil](assets/bevigil_upload.png)
  ![Analyse BeVigil Terminée](assets/bevigil_done.png)

#### Qu'est-ce que le scan BeVigil / BigDvil ?
BeVigil est le premier moteur de recherche de sécurité au monde dédié aux applications mobiles. Lors de la soumission de notre APK, BeVigil a effectué une **analyse statique automatisée dans le Cloud (SAST)**.
* **Fonctionnement :** L'outil désassemble le code de l'APK, dissèque le Manifest et parcourt les chaînes de caractères pour y trouver des failles connues.
* **Ce qu'il cible :** Il extrait toutes les métadonnées, recherche des **secrets oubliés** en dur (clés d'API, mots de passe), liste l'exaustivité des **Endpoints réseau et URLs**, et repère les **défauts de configuration** manifestes à l'externe.
* **Avantage :** Il permet d'obtenir la cartographie exacte de la **surface d'attaque OSINT et statique** en quelques instants, dirigeant ainsi nos efforts vers les points faibles de l'application.
- **Résultats :** Rapport de vulnérabilités, d'assets réseau et d'intégrité exporté.
- **Apprentissages :** Extraire les data vers CSV favorise un *Triage* (tri par criticité) plus facile par de puissants parsers de données.

### Étape 4 — Reconnaissance Externe (Domaines, Emails, Endpoints APi)
**Objectif :** Structurer les retours du scanner dans un rapport détaillé.
- **Étapes :** Exportation des résultats (JSON/CSV) vers `01-bevigil/` et exploration des sections (Assets, Endpoints, URLs).
- **Observations :** ![Screenshot_Task4_Collecte](assets/task4_collecte.png)
- **Résultats :** Production du fichier analytique `bevigil_notes.md` contenant 5 éléments critiques de BigDvil (Communications non sécurisées, Génération aléatoire faible, Application Debuggable, Provider exposé, Algorithme de hachage faible).

**Tableau des Vulnérabilités (BigDvil / BeVigil) :**

| Vulnérabilité | Risque | Sévérité | CVSS | Occurrences | Fichiers impactés |
|---|---|---|---|---|---|
| Insecure communication | High | High | 8.1 | 36 | ChangePassword.java, DoLogin.java, DoTransfer.java... |
| Use of insufficiently random values | Medium | Medium | 5.9 | 3 | zzl.java, Tracker.java, zzc.java |
| Android debuggable enabled | Medium | Medium | 4.9 | 1 | AndroidManifest.xml |
| Android backup vulnerability | Medium | Medium | 4.9 | 1 | AndroidManifest.xml |
| Improper export of providers | Medium | Medium | 4.9 | 1 | AndroidManifest.xml |
| Weak hash – MD5 | Medium | Medium | 4.3 | 4 | Multiples classes Java |
| Javascript enabled in WebView | Low | Low | 2.9 | 3 | ViewStatement.java, zzfd.java, zzig.java |

- **Apprentissages :** La catégorisation des alertes facilite un triage immédiat par criticité (CVSS). 

### Étape 5 — Mise en Cale Sèche et Décompilation binaire via Yaazhini
**Objectif :** Extraire le code binaire (*Disassembly*) pour remonter les failles Android spécifiques.
- **Étapes :** Déploiement de l'exécutable/script Yaazhini sur l'artefact.
- **Commandes exécutées :**
  ```powershell
  & "yaazhini.exe" -apk "00-scope\InsecureBankv2.apk" -output "02-yaazhini"
  ```
- **Observations :** L'outil a dézippé l'APK, extrait le `classes.dex`, puis décompilé au format Smali et pseudo-Java. Interface de Yaazhini :
  ![Interface Yaazhini](assets/yaazhini_ui.png)
- **Résultats :** Scan SAST approfondi des dépendances et de la structure du Manifest.
- **Apprentissages :** Différentiel majeur avec BeVigil : Yaazhini rentre dans le cœur du code pour pointer *la ligne exacte* de la faille.

### Étape 6 — Investigation des Fichiers Révélés (Indices de SAST profond)
**Objectif :** Fouiller manuellement les alertes techniques retournées par l'outil.
- **Étapes :** Revue fine du compte-rendu SAST, identification des exports de composants et données figées dans les ressources.
- **Observations :** Les *Intents* Android sont mal sécurisés.
- **Résultats :** Synthèse d'indicateurs de compromission critiques dans `yaazhini_notes.md` (Voir section "Yaazhini Notes" ci-dessous).
- **Apprentissages :** Bien que les données soient masquées dans la documentation pour des raisons d'éthique, leur impact architectural est fondamental.

### Étape 7 — Consolidation, Nettoyage et Élimination des Faux Positifs
**Objectif :** Éliminer les doublons entre les scanners OSINT et SAST afin de produire une base cohérente.
- **Étapes :** Création du fichier `03-triage/triage.csv` consolidant 10 vulnérabilités réparties par criticité.
- **Résultats :** Nous avons fusionné les découvertes (exemple : Le "Cleartext Traffic" de Yaazhini et l'"Insecure Communication" de BigDvil se sont avérés décrire la même faille macroscopique).
- **Explication pour débutants :** La *normalisation* est le fait d'uniformiser différents types de rapports d'erreurs. Le *dédoublonnage* empêche qu'un développeur corrige deux fois le même problème signalé par deux outils différents. Cette étape de nettoyage est fondamentale en entreprise pour éviter les faux positifs ("bruit").

### Étape 8 — Classification et Assignation des Risques selon Standards OWASP
**Objectif :** Contextualiser professionnellement les failles découvertes par rapport au standard de l'industrie.
- **Étapes :** Référencement de 5 failles critiques au standard `MASVS` dans le fichier `03-triage/owasp_mapping.md`.
- **Résultats :** Assignation des catégories OWASP : `MASVS-NETWORK` pour les transferts Web en clair, `MASVS-STORAGE` pour la compromission des mots de passe en dur.
- **Explication pour débutants :** **L'OWASP** est l'organisation mondiale référence pour la sécurité des applications. Le **MASVS** *(Mobile Application Security Verification Standard)* est leur guide de vérification. En liant une faille à une règle OWASP (ex: V5.1), on justifie que le problème est un risque réel reconnu par l'industrie de la cybersécurité.

### Étape 9 — Finalisation et Extrants pour les Décideurs
**Objectif :** Synthétiser un document d'action clair pour les décideurs techniques et managériaux.
- **Étapes :** Mise en œuvre de l'artefact terminal `04-report/rapport_final.md`. Cet élément retient un Top 5 formel et identifie l'attribut de Faux Positifs (comme l'application du JS dans les WebViews).
- **Résultats :** L'analyse est achevée, les livrables de la racine jusqu'au compte-rendu sont parés pour transmission.
- **Explication pour débutants :** Le *rapport exécutif* est la seule chose que liront les dirigeants (CISO, CTO). Il doit résumer la situation totale en 5 lignes, sortir un *Top 5 des urgences absolues*, et proposer des *remédiations* (solutions). Un expert doit savoir transformer sa technique en langage "Business".

---

## 2. Analysis Results (Yaazhini)

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

## 3. Yaazhini Notes

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

## 4. Observations & Recommendations

### Synthèse (Key Insights)
L'application *InsecureBankv2* contient de multiples vulnérabilités à risque élevé et modéré. Plusieurs erreurs de configuration graves de sécurité sont présentes (Attribut debuggable activé, Sauvegarde ADB autorisée, et composants exportés sans permission). La cryptographie utilisée est obsolète et dangereuse (MD5, Random values faibles), et d'importantes communications HTTP en clair sont disséminées dans le code.

### Amélioration des Pratiques (Security Practices to improve)
- Imposer le protocole **HTTPS** et des tunnels TLS sécurisés pour toutes les communications réseau.
- Désactiver formellement les options de **debugging** et de **backup** en production.
- **Restreindre les composants exportés** (Activities, Providers) à moins que ce ne soit strictement justifié.
- Mettre à jour l'implémentation cryptographique vers des algorithmes robustes comme **SHA256 ou PBKDF2**.
- Auditer l'utilisation des WebViews et désactiver l'exécution de **JavaScript** si elle n'est pas requise.

### Prochaines Étapes (Next Steps)
1. **Évaluation Dynamique DAST approfondie** : Exécuter l'application sur un périphérique rooté (comme le framework Frida/Objection) pour accrocher "en vol" (Hooking) les appels non sécurisés constatés à l'étape SAST.
2. **Remédiation Code (Développement)** : Soumettre une *Pull Request* ou révision complète du code Java et du fichier d'infrastructure *Manifest* à la lumière du top 10 OWASP Mobile. Lancer ensuite un nouveau scan de validation via Yaazhini.