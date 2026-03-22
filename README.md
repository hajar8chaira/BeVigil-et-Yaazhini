# Mobile Security Lab: Yaazhini & BeVigil Analysis

**Auteur :** Chaira Hajar  
**Application Cible :** `InsecureBankv2.apk`  
**Date du rapport :** 22/03/2026  
**Outils :** BeVigil / BigDvil (Web Scan), Yaazhini (SAST/DAST Tool)

## Vue d'ensemble (Overview)
Ce projet est un laboratoire pratique d'audit de sécurité d'applications mobiles, visant à évaluer rigoureusement la posture de sécurité d'une application Android pédagogique nommée **InsecureBankv2**. Ce document sert de rapport de restitution global et démontre pas à pas une méthodologie professionnelle d'analyse statique et d'OSINT.

**Qu'est-ce que InsecureBankv2 ?**
Il s'agit d'une application bancaire factice "délibérément vulnérable" (*Vulnerable By Design*). Conçue par des experts en sécurité pour l'entraînement au pentest mobile, elle regorge des vulnérabilités de développement les plus fréquentes dans l'industrie (modèle de menaces OWASP Mobile) : composants exportés non protégés, trafic exposé, mots de passe codés en dur, et faiblesses cryptographiques majeures.

**Pourquoi BeVigil et Yaazhini ?**
L'association de ces deux outils m'a permis de réaliser un audit croisé avec deux perspectives fondamentales :
1. **BeVigil (L'approche OSINT / Analyse Cloud) :** Offre une vision type "Black-Box". Cet outil effectue un balayage massif de l'infrastructure et des métadonnées de l'application depuis l'extérieur. Il cartographie les domaines, endpoints d'API, adresses emails exposées et identifie rapidement les erreurs de configuration grossières.
2. **Yaazhini (L'approche SAST en profondeur) :** Apporte l'analyse "White-Box". Par des techniques de rétro-ingénierie (Reverse Engineering), cet outil "désassemble" l'APK pour examiner le code source interne (XML, Manifest, classes Java). Il cible la faille à la ligne près.

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

**Mise en valeur du travail :** Avant de lancer la moindre analyse technique, il est crucial d'établir les bases légales et traçables d'une mission de cybersécurité, connues sous le nom de *Rules of Engagement* (Règles d'engagement).

- **Actions effectuées :** 
  - **Délimitation du Scope :** Création du dossier `00-scope` renfermant `scope.md`. J'ai formalisé précisément la portée de l'exercice (l'APK InsecureBankv2) et ses limites (aucune tentative d'exploitation intrusive ou de déni de service n'est autorisée).
  - **Création du Workspace :** J'ai mis en place une arborescence ordonnée de niveau industriel : `01-bevigil`, `02-yaazhini`, `03-triage`, `04-report`, couplée à un journal de commandes `commands.log` pour assurer la stricte reproductibilité de mon travail.
  - **Empreinte Cryptographique :** Pour des raisons de conformité et de *Chaîne de possession* (Chain of Custody), le hachage (SHA-256) du fichier cible a été calculé et scellé (`B18AF2A0E44D7634BBCDF93664D9C78A2695E050393FCFBB5E8B9...`). Cette étape garantit sans débat possible que le fichier que j'analyse est l'exact binaire défini avec le client.

- **Preuves et livrables :**
  ![Screenshot_Task0_Scope](assets/task0_scope.png)
  ![Calcul du Hash SHA-256](assets/hash_sha256.png)

## 2. Analyse avec BeVigil

**Mise en valeur du travail :** L'objectif de cette phase est de jouer le rôle d'un attaquant externe scannant la surface exposée de l'application sans connaissance initiale approfondie du code source.

- **Actions techniques :**
  - Validation du fichier sur les moteurs de CloudSEK pour extraire les *Rulesets* d'audit automatisés.
  - Extraction de tous les endpoints API cachés ainsi que de la cartographie réseau (URLs, Domaines).
  - Export et sauvegarde des datas brutes au format JSON et CSV pour traitement ultérieur. L'application a récolté la note de vulnérabilité générale de 7.4/10.

- **Explication Pédagogique (Qu'est-ce que le scan BeVigil ?) :**
  C'est un moteur de recherche de sécurité. Il opère tel un scanner dans le cloud qui dissèque massivement l'architecture statique superficielle. En quelques secondes, sa puissance de calcul révèle les clés en dur ou les serveurs backend communiqués par les développeurs. Il me dirige directement là où la surface d'attaque est la plus "fragile".

- **Résultats mis en relief (Extrait) :**

| Top Vulnérabilité Macro | Sévérité / Risque | Score CVSS | Impact technique identifié |
|---|---|---|---|
| Mots de passe et Traffic non sécurisé | Élevée (High) | 8.1 | Révélation de transferts en HTTP sur le ChangePassword.java et DoTransfer.java |
| Faiblesse de l'aléatoire (Random Values) | Moyenne (Med) | 5.9 | Facilite la prédiction des jetons de sessions par les pirates |
| Application Debuggable | Moyenne (Med) | 4.9 | Permettrait à un individu d'attacher un outil de diagnostic interne |
| Android Backup configuré à "Vrai" | Moyenne (Med) | 4.9 | Vol complet et illicite des données sur PC via ADB |
| Algorithmes Obsoletes (MD5) | Basse (Low) | 4.3 | Cassable très rapidement par "Rainbow Tables" |

## 3. Analyse avec Yaazhini

**Mise en valeur du travail :** Tandis que BeVigil scrute la façade externe, Yaazhini (outil SAST) détoure les entrailles du code. Cette analyse approfondie implique de déconstruire le binaire et de l'inspecter comme le ferait l'équipe de développement initiale.

- **Actions techniques :**
  - **Reverse Engineering** : J'ai déployé l'exécutable local `yaazhini.exe` pointant vers l'APK cible. L'outil a dézippé l'archive, ciblé le fichier névralgique `classes.dex`, puis l'a rétro-conçu (décompilation) en code lisible (Smali / pseudo-Java).
  - Lancement des sondes de regex et d'analyse d'AST (Abstract Syntax Tree) pour identifier non seulement *quoi*, mais *où* sont les vulnérabilités (jusqu'à la ligne XML précise).

- **Découvertes majeures investiguées (Notes Techniques) :**

  * **1. Hardcoded Credentials dans les ressources**
    J'ai pu repérer et intercepter la variable de production `loginscreen_password` directement gravée en clair dans `resources/res/values/strings.xml`. C'est une négligence critique qui équivaut à laisser les clés de l'infrastructure sous le paillasson.

  * **2. Risques majeurs sur les composants exportés (Activités ouvertes)**
    Dans le fichier central `AndroidManifest.xml`, j'ai identifié des directives `android:exported=true`. Cela signifie que même sans être identifié sur l'application, l'interface de virement bancaire ou l'écran de modification de mot de passe de l'utilisateur peuvent être évoqués sans sécurité (« bypass d'authentification »).

  * **3. Failles de Flux Dynamique (SQL Injections locales)**
    Dans les couches Java (`sources/com/...`), j'ai découvert l'emploi de de jonctions dangereuses sur des curseurs SQlite (`rawQuery()`). L'historique transactionnel de la banque est directement modelable et piratable depuis l'interface client (Inputs UI compromettant).

## 4. Triage

**Mise en valeur du travail :** Les outils automatisés sont "bruyants" et génèrent énormément d'alertes, y compris des Faux Positifs (ex: fausses alertes sur des dépendances JavaScript légitimes liées aux WebViews). Mon travail d'analyste a consisté à isoler le grain de l'ivraie.

- **Actions :** Collecte consolidée. Croisement des tables issues de BeVigil (fichiers CSV) avec la sortie locale de Yaazhini. Ce traitement manuel est une preuve d'expertise car aucun logiciel ne surpasse la qualification et la décision humaine pour écarter ce qui n’est pas exploitable en conditions réelles.

## 5. Normalisation

**Mise en valeur du travail :** J'ai uniformisé et dédoublonné les rapports pour présenter une *"Single Source of Truth"* (la version unique de la vérité) pour l'entreprise.

- **Actions :** Édition du fichier `03-triage/triage.csv`. Ce nettoyage garantit qu’un développeur ne se retrouvera pas avec deux billets de failles pour un même problème. Par exemple : Bien que Yaazhini cite le `Cleartext Traffic` (Manifeste) et BeVigil cite l'`Insecure Communication` (Code Java), l'expertise en normalisation prouve qu'il ne s'agît que d'une seule faille colossale nécessitant une seule tâche corrective, optimisant ainsi le budget et le temps de l'équipe de réparation.

## 6. Corrélation OWASP

**Mise en valeur du travail :** Présenter une erreur logicielle ne suffit pas. L'auditeur de sécurité doit lier une erreur à son impact sur le monde réel. L'alignement sur un standard industriel prouve à la direction la dangerosité officielle des découvertes.

- **Actions :** Tous les constats majeurs ont été projetés (mapping) dans le référentiel normatif mondial **OWASP MASVS** (Mobile Application Security Verification Standard) dans le livrable `owasp_mapping.md`.
- **Exemples :** Le fait de lier le "Trafic en clair autorisé" à la règle imposée `MASVS-NETWORK` prouve à l'équipe conformité qu'ils sont hors-la-loi par rapport aux directives internationales sur le chiffrement en transit des données personnelles d'utilisateurs.

## 7. Rédaction Rapport

**Mise en valeur du travail :** Le pont ultime entre la technologie pure et la prise de décision stratégique des Chefs de Service et Dirigeants (CTO/CISO).

- **Actions :** Production de l'Executive Summary `04-report/rapport_final.md`. Cet artefact condense l'analyse en 5 points à impact Business fort.
- **Recommandations Cles Priorisées :**
  1. Activer stricto sensu une Configuration de Sécurité Réseau obligeant le `HTTPS` et TLS robustes.
  2. Mettre immédiatement un terme aux options de developpement actives sur l'APK (`Android Backup` et `Debuggable`).
  3. Protéger impérativement les activités dites "Exportées" du binaire pour restaurer une couche d'Authentication correcte.
  4. Recours exclusif aux algorithmes recommandés par l'ANSSI / NIST (SHA-256 vs MD5) et migration des secrets (mots de passe) vers le Keystore Android matériel protégé.

## 8. Nettoyage & Clôture

**Mise en valeur du travail :** Conformément à l'attitude d'un auditeur d'élite, j'ai garanti la confidentialité absolue du projet tout à la fin de ma mission.

- **Actions Réalisées :**
  - **Désensibilisation des données :** Audit approfondi de l'ensemble de mes propres rapports documentaires. Tous les fichiers de résultats ont été scannés pour y chercher des mots clés "secret", "password" ou "key". J'ai masqué manuellement (balise `[MASQUÉ]`) les identifiants en dur détectés afin qu'aucune donnée client confidentielle ne soit compromise lors de la livraison du rapport.
  - **Validation des Livrables & Checklist :** Validation de bout-en-bout via la création d'une charte qualité `checklist_fin.md`. J'ai contresigné numériquement (Date et Signature formelle) la garantie que la prestation (Triage CSV, Logs, Notes, Scope, Rapport final) a été délivrée telle que spécifiée.
  - **Archivage & Hygiène :** Tri du log `commands.log` et publication finale sur l'arbre Git pour horodatage officiel permanent et indexation immuable de mon travail.

**Conclusion :** Le laboratoire *InsecureBankv2* s'achève avec succès, fournissant un éventail diagnostique complet et clair, transformable instantanément par les équipes de développement en un sprint de correctifs de sécurité logiciel.
