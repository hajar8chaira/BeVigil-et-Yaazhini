# Rapport d'analyse de sécurité mobile

## A. Informations générales
- **Date**: $(Get-Date -Format "yyyy-MM-dd")
- **Analyste**: [Prénom Nom]
- **Cible**: InsecureBankv2
- **Version/Hash**: B18AF2A0E44D7634BBCDF93664D9C78A2695E050393FCFBB5E8B91F902D194A4
- **Outils utilisés**: BeVigil / BigDvil Web Scan, Yaazhini 2.0.2

## B. Résumé exécutif
L'application bancaire InsecureBankv2 a fait l'objet d'un audit de sécurité. Au total, 10 vulnérabilités distinctes ont été consolidées, dont 4 de niveau critique (High). Le profil de risque est alarmant puisque l'application omet l'intégralité des défenses de base (communications en clair, cryptographie obsolète, mots de passe stockés en clair, composants critiques the plateforme non protégés). Il s'agit d'une preuve de concept démontrant ce qu'il ne faut pas faire en développement mobile.

## C. Top 5 constats

### 1. Hardcoded Credentials (Secret en dur) - FIND-008
- **Sévérité**: High
- **Preuve**: Fichier `resources/res/values/strings.xml` (clé de connexion maître).
- **Impact**: Accès direct et total à l'application avec privilèges maximum suite à une simple décompilation et lecture des chaînes de caractères.
- **Remédiation**: Bannir le codage en dur. Migrer vers le système natif protecteur `Android Keystore`.
- **Référence OWASP**: MASVS-STORAGE-14 / V2.1

### 2. Activités Applicatives Exportées - FIND-010
- **Sévérité**: High
- **Preuve**: Fichier `AndroidManifest.xml` (Composants ChangePassword et DoTransfer).
- **Impact**: Les flux métiers cruciaux (dont les virements monétaires) peuvent être invoqués de force par d'autres applications résidant sur le même téléphone, bypassant l'écran de connexion.
- **Remédiation**: Définir `android:exported="false"` ou contraindre l'accès par des permissions via signature.
- **Référence OWASP**: MASVS-PLATFORM-1

### 3. Communication HTTP Insecure - FIND-001
- **Sévérité**: High
- **Preuve**: Classes Java de connexion réseau (ex: `DoTransfer.java`).
- **Impact**: La surveillance (Attaque MITM) du réseau local permet d'intercepter les identifiants et les montants en clair car aucune couche de validation d'intégrité ou chiffrement n'existe au sein du tunnel de la requête.
- **Remédiation**: Imposer HTTPS formellement via une ressource d'infrastructure "Network Security Config".
- **Référence OWASP**: MASVS-NETWORK-1 / V5.1

### 4. Sauvegarde ADB Activée - FIND-004
- **Sévérité**: Medium
- **Preuve**: Fichier manifeste principal (attribut `android:allowBackup="true"`).
- **Impact**: Toute personne non autorisée ayant un accès momentané au terminal (ex: débogage USB déverrouillé) extraira localement la base de données SQLite de l'application en quelques secondes.
- **Remédiation**: Inscrire l'attribut à `false` universellement pour le cycle de build en production.
- **Référence OWASP**: MASVS-STORAGE-4 / V2.8

### 5. Weak Hash MD5 & Faible Randomisation - FIND-006 & FIND-002
- **Sévérité**: Medium
- **Preuve**: Observé dans les traces `zzl.java` ou requêtes identifiées par BeVigil.
- **Impact**: Les données générées de session et le hash de persistance recourent à des implémentations dépréciées (MD5/java.utils.Random). Résultat : Cassable expéditivement via dictionnaire et prédictibilité des token applicatifs.
- **Remédiation**: Hausser le niveau cryptographique vers une base minimale de `SHA-256`, ou mieux le `PBKDF2`, via le `SecureRandom`.
- **Référence OWASP**: MASVS-CRYPTO-4 / MASVS-CRYPTO-6

## D. Faux positifs notables
- **FIND-007 (JavaScript activé dans WebView)** : Un scan statique remonte souvent l'exécution JS comme une compromission moyenne/faible. Bien qu'une WebView ne doive exécuter JS que si les sources relèvent de la confiance, si tout le contenu affiché est inaltérable/local/API-sécurisé, ce signal est acceptable. Une analyse `DAST` manuelle viendrait confirmer ce statut.

## E. Recommandations prioritaires
1. **Verrouiller le trafic réseau** : Appliquer par défaut le blocage du trafic HTTP pur, en reconfigurant les endpoints en architecture `https://` avec TLS et Validation des certificats racine.
2. **Corriger le Manifest** : Bloquer les activités dynamiques (`android:exported="false"`) et retirer expressément le risque de portabilité avec le flag d'interdiction au backup.
3. **Nettoyer la base de code** : Purger le design et les XML des secrets applicatifs résidents ; modifier la branche de code qui emploie MD5 vers une méthode native Android moderne.

## F. Annexes
- Base OSINT Web : [Exports BigDvil/BeVigil](../01-bevigil/)
- Fichiers de SAST Décompilés: [Scan local Yaazhini](../02-yaazhini/)
- Base technique de triage : [Triage (CSV)](../03-triage/triage.csv)
- Tableau de conformité cyber : [Mapping OWASP](../03-triage/owasp_mapping.md)
