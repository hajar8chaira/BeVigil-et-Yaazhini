# Mobile Security Lab: Yaazhini & BeVigil Analysis

**Auteur :** Chaira Hajar  
**Application Cible :** `InsecureBankv2.apk`  
**Date du rapport :** 22/03/2026  
**Outils :** BeVigil / BigDvil (Web Scan), Yaazhini (SAST/DAST Tool)

## Vue d'ensemble (Overview)
Ce projet vise à évaluer la posture de sécurité d'une application Android pédagogique nommée **InsecureBankv2**. 

**Qu'est-ce que InsecureBankv2 ?**
Il s'agit d'une application bancaire factice "délibérément vulnérable" (*Vulnerable By Design*). Elle a été créée par des experts en sécurité pour permettre aux étudiants et professionnels de s'entraîner au pentest mobile. Elle regorge d'erreurs classiques de développement réel (composants exportés, trafic en clair, mots de passe codés en dur, cryptographie obsolète).

**Pourquoi BeVigil et Yaazhini ?**
Pour une analyse exhaustive, deux points de vue sont nécessaires :
1. **BeVigil (L'approche OSINT / Cloud) :** Permet une analyse statique dite "Black-Box" depuis l'extérieur. Sans exécuter l'application, il cartographie presque instantanément sa surface d'attaque externe globale.
2. **Yaazhini (L'approche SAST en profondeur) :** Permet d'investiguer le code en lui-même (White/Grey-Box). Après décompilation de l'APK, cet outil descend dans la logique binaire. 

À travers une approche structurée selon un workflow professionnel, ce laboratoire met en œuvre l'analyse statique et dynamique de bout en bout.

---

## Sommaire (Table of Contents)
1. [Préparation & Périmètre](#1-préparation--périmètre)
2. [Analyse avec BeVigil](#2-analyse-avec-bevigil)
3. [Analyse avec Yaazhini](#3-analyse-avec-yaazhini)
4. [Triage](#4-triage)
5. [Normalisation](#5-normalisation)
6. [Corrélation OWASP](#6-corrélation-owasp)
7. [Rédaction Rapport](#7-rédaction-rapport)
8. [Nettoyage & Clôture](#8-nettoyage--clôture)

---

## 1. Préparation & Périmètre

**Objectif :** Définir le cadre légal et standardiser l'espace de travail.

- **Étapes :** 
  - Création du dossier `00-scope` et définition de l'artefact cible (InsecureBankv2) et des limites (aucun test d'exploitation intrusif réel).
  - Création de l'arborescence (`01-bevigil`, `02-yaazhini`, `03-triage`, `04-report`).
  - Lancement des logs de commandes `commands.log`.
  - Calcul du hachage de l'APK (SHA-256) consigné : `B18AF2A0E44D763...`

- **Observations :** L'environnement est prêt et documenté. La délimitation d'un "scope d'audit" est la clef de voûte de l'éthique du projet. Le hachage garantit que toutes les observations sont liées à une version exacte.

  ![Screenshot_Task0_Scope](assets/task0_scope.png)
  ![Création de l'arborescence](assets/dir_folders.png)
  ![Calcul du Hash SHA-256](assets/hash_sha256.png)

## 2. Analyse avec BeVigil

**Objectif :** S'interfacer avec l'arsenal Cloud de balayage d'applications (*CloudSEK*) pour structurer les retours du scanner.

- **Étapes :** Soumission de l'APK, lancement du scan (Score global de 7.4/10) et exportation des archives JSON/CSV. Exploration des sections (Assets, Endpoints, URLs).

- **Observations :** Processus de soumission de l'APK sur la plateforme CloudSEK et génération du rapport.
  ![Upload BeVigil](assets/bevigil_upload.png)
  ![Analyse BeVigil Terminée](assets/bevigil_done.png)
  ![Screenshot_Task4_Collecte](assets/task4_collecte.png)

#### Qu'est-ce que le scan BeVigil / BigDvil ?
BeVigil effectue une **analyse statique automatisée (SAST)**. Il extrait toutes les métadonnées, recherche des **secrets oubliés** (clés d'API, mots de passe), et liste l'exaustivité des **Endpoints réseau**. La production du fichier `bevigil_notes.md` contient les éléments critiques de l'outil.

**Tableau des Vulnérabilités (BigDvil / BeVigil) :**

| Vulnérabilité | Risque | Sévérité | CVSS | Occurrences |
|---|---|---|---|---|
| Insecure communication | High | High | 8.1 | 36 |
| Use of insufficiently random values | Medium | Medium | 5.9 | 3 |
| Android debuggable enabled | Medium | Medium | 4.9 | 1 |
| Android backup vulnerability | Medium | Medium | 4.9 | 1 |
| Improper export of providers | Medium | Medium | 4.9 | 1 |
| Weak hash – MD5 | Medium | Medium | 4.3 | 4 |

## 3. Analyse avec Yaazhini

**Objectif :** Extraire le code binaire (*Disassembly*) pour pointer la ligne exacte des failles Android spécifiques.

- **Étapes :** Déploiement de l'exécutable Yaazhini sur l'artefact. L'outil extrait le `classes.dex`, puis décompile au format Smali et Java. Revue fine du compte-rendu SAST, identification des exports de composants et données figées dans les ressources.

- **Observations :** 
  ![Interface Yaazhini](assets/yaazhini_ui.png)

#### Résultats de l'Analyse Yaazhini

La fouille manuelle et automatisée révèle des résultats primaires qui ont été catégorisés :

| Vulnérabilité / Identification | Risque | Sévérité | CVSS (Estimé) | Chemin du Fichier |
|---|---|---|---|---|
| **Composant Activity Exporté non protégé** | Critique | Haute | 8.8 (High) | `AndroidManifest.xml` |
| **Utilisation Crypto Obsolète (AES/CBC)** | Élevé | Moyenne | 5.9 (Med) | `CryptoClass.java` |
| **Cleartext Traffic Enabled (HTTP Autorisé)** | Critique | Haute | 7.5 (High) | `AndroidManifest.xml` |
| **Backup ADB Activé (`allowBackup=true`)** | Modéré | Basse | 4.3 (Low) | `AndroidManifest.xml` |
| **Insecure Data Storage (SharedPrefs)** | Critique | Haute | 7.5 (High) | `DoLogin.java` |

#### Yaazhini Notes et Explications techniques internes :

* **Élément 1 : Hardcoded API Key / Credentials (Secret)**
  * *Localisation* : `resources/res/values/strings.xml`
  * *Description* : Le champ `loginscreen_password` contient des données brutes en clair.

* **Élément 2 : Traffic réseau en clair autorisé (Configuration)**
  * *Localisation* : `AndroidManifest.xml` (`usesCleartextTraffic`)
  * *Description* : Aucun flag n'oblige le recours à HTTPS strict, autorisant des transferts HTTP en clair.

* **Élément 3 : Activités non authentifiées (Composants)**
  * *Localisation* : `AndroidManifest.xml` (`android:exported=true`)
  * *Description* : Des composants d'interface (comme le Transfert d'argent) sont accessibles depuis l'extérieur.

* **Élément 4 : Autorisation de sauvegarde système**
  * *Localisation* : `AndroidManifest.xml` (`allowBackup="true"`)
  * *Description* : Le droit d'extraire les données applicatives par un terminal ADB est ouvert.

* **Élément 5 : Requêtes dynamiques vulnérables (SQLi)**
  * *Localisation* : Concaténations `rawQuery()` non assainies dans les sources.
  * *Description* : Manipulations faciles par injections SQL sur les requêtes locales.

## 4. Triage

**Objectif :** Regrouper les constatations issues de multiples scanners locaux ou distants.
- **Actions :** Collecte en bloc de l'ensemble des résultats de BeVigil et des alertes brutes de Yaazhini vers les fichiers préparatoires. Le triage sert à identifier ce qui tient de la faille potentielle versus le fonctionnement prévu.

## 5. Normalisation

**Objectif :** Éliminer les doublons pour produire une base cohérente ("Single Source of Truth").
- **Actions :** Création du fichier `03-triage/triage.csv` consolidant les vulnérabilités réparties par criticité.
- **Explications :** Nous fusionnons les découvertes similaires. Par exemple : Le "Cleartext Traffic" mis en lumière par Yaazhini et "Insecure Communication" cité par BeVigil documentent la même faille macroscopique réseau. Le dédoublonnage est fondamental pour éviter de repasser deux fois sur le même faux positif.

## 6. Corrélation OWASP

**Objectif :** Contextualiser professionnellement les failles par rapport au standard Mobile de l'industrie.
- **Actions :** Référencement des failles critiques au standard `MASVS` dans le fichier `owasp_mapping.md`.
- **Explications :** L'assignation formelle ajoute du poids à une vulnérabilité. Par exemple : Assignation de `MASVS-NETWORK` pour les transferts Web en clair et de `MASVS-STORAGE` pour la compromission des clés en dur. On prouve ainsi la dangerosité reconnue des découvertes au regard de ce framework mondial (OWASP).

## 7. Rédaction Rapport

**Objectif :** Synthétiser un document d'action clair pour les décideurs.
- **Actions :** Mise en œuvre de l'artefact exécutif `04-report/rapport_final.md`. Cet élément retient un Top 5 formel et traduit la complexité technique en risques business clairs.

#### Observations & Recommandations Cles :
* Imposer le protocole **HTTPS** stricts.
* Désactiver formellement les options de **debugging/Backup** en production.
* Restreindre et encadrer **les permissions des composants exportés**.
* Actualiser les algorithmes obsolètes et interdire **les mots de passe hardcodés**.
* Remédiation par des tests DAST avec Frida / Objection pour la validation ultérieure de comportement.

## 8. Nettoyage & Clôture

**Objectif :** Protéger le matériel confidentiel et clôturer proprement la séquence d'engagement.
- **Actions :**
  - Nettoyage des fichiers temporaires mis en cache par les scanners (suppression des fichiers Dex extraits).
  - Validation finale du rapport, sauvegarde sécurisée des exports JSON de l'arborescence, et indexation finale.
  - Push final sur le répertoire Git et préparation du dépôt (Clôture de la séquence).
