# Notes d'Analyse BeVigil

**Application analysée :** InsecureBankv2 (`com.android.insecurebankv2`)
**Score de sécurité global :** 7.4 / 10 (Average)
**Date d'analyse :** 2026-03-22
**Outil :** BeVigil Web Scan (CloudSEK)
**Hash SHA-256 de l'APK :** `B18AF2A0E44D7634BBCDF93664D9C78A2695E050393FCFBB5E8B91F902D194A4`

---

##  Ce qui est certain (preuves directes)

Ces éléments sont confirmés par des correspondances directes dans les fichiers sources décompilés de l'APK.

- **Client HTTP non sécurisé :** `new DefaultHttpClient()` est instancié sans SSL/TLS dans `DoLogin.java`, `ChangePassword.java` et `DoTransfer.java` (CWE-757).
- **Chiffrement CBC vulnérable :** `Cipher.getInstance("AES/CBC/PKCS5Padding")` est utilisé dans `CryptoClass.java`, exposant l'application à une attaque Padding Oracle.
- **Algorithme MD5 utilisé :** `getInstance("MD5")` est appelé dans plusieurs fichiers internes de Google GMS, un algorithme considéré comme cryptographiquement cassé (CWE-327).
- **Requêtes SQL brutes :** `rawQuery("SELECT * FROM " + str + ...)` dans `zzj.java` – construction dynamique d'une requête SQL avec une chaîne de caractères non paramétrée (CWE-89).
- **Désérialisation d'objets :** `new ObjectInputStream(byteArrayInputStream)` dans `zzw.java` – désérialisation potentiellement non sécurisée (CWE-502).
- **Activités Android exportées :** 4 activités sensibles exportées sans permission dans `AndroidManifest.xml` : `PostLogin`, `DoTransfer`, `ViewStatement`, `ChangePassword` (CWE-926).
- **Endpoints de l'application bancaire exposés :** Chemins `/login`, `/devlogin`, `/changepassword`, `/dotransfer`, `/getaccounts`, `/Statements_` identifiés en clair dans le code source.
- **Email interne hardcodé :** `android@android.com` apparaît dans 6 occurrences dans `zzc.java`.
- **Possible secret détecté :** Un identifiant de mot de passe (`loginscreen_password`) exposé dans `strings.xml`.

---

##  Ce qui est hypothèse (indices ou suspicions)

Ces éléments constituent des signaux d'alerte qui méritent une investigation plus approfondie mais ne sont pas formellement prouvés comme malveillants.

- **Mécanisme Anti-VM potentiel :** Le binaire `classes.dex` contient des vérifications `Build.MODEL`, `Build.MANUFACTURER` et `Build.PRODUCT`. *Hypothèse :* L'application pourrait adapter son comportement pour masquer ses fonctionnalités lors d'une analyse sur émulateur.
- **Mécanisme de build non standard (dexmerge) :** L'APKID indique l'usage du compilateur `dx` avec manipulation possible par `dexmerge`. *Suspicion :* Probablement un build automatisé ou légèrement obfusqué, à explorer en reverse engineering.
- **Tokens dans assets REST API :** Des chaînes base64-like (`ByD3U8Y1/AgbpCy...`, `GAtGQEFDwIyRr...`) identifiées dans `zzat.java`. *Hypothèse :* Il pourrait s'agir de clés API ou tokens d'authentification hardcodés.
- **URLs tierce-parties en HTTP :** La présence de `http://www.google-analytics.com` (non chiffré) dans les communications pourrait servir de vecteur MITM dans un réseau hostile.

---

##  Points d'intérêt (éléments critiques à surveiller)

-  **`/devlogin`** : Un endpoint de développeur expose une porte d'entrée alternative pouvant contourner les mécanismes d'authentification normaux.
-  **`DefaultHttpClient` dans les flux bancaires** : Toutes les transactions (`DoTransfer`, `DoLogin`, `ChangePassword`) passent par un client HTTP insécure.
-  **`AES/CBC/PKCS5Padding` dans `CryptoClass.java`** : Si des données sensibles sont chiffrées localement avec ce mode, elles sont potentiellement décryptables via une attaque Padding Oracle.
-  **Activités exportées** : `DoTransfer` et `ChangePassword` exportées signifient qu'une application tierce pourrait initier des transferts ou changements de mot de passe sans consentement utilisateur.
-  **`TrackUserContentProvider`** : Un Content Provider exposé via l'URL `content://com.android.insecurebankv2.TrackUserContentProvider/trackerusers` peut permettre la lecture de données de profil.

---

##  Domaines et sous-domaines

| Domaine | Source | Contexte |
|---|---|---|
| `www.google-analytics.com` | `zzy.java`, `zzqi.java` | Analytics (⚠️ aussi via HTTP) |
| `ssl.google-analytics.com` | `zzy.java` | Analytics (HTTPS) |
| `www.googletagmanager.com` | `zzs.java` | Tag Manager |
| `www.googleapis.com` | `Games.java`, `Drive.java`, `Scopes.java` | APIs Google |
| `accounts.google.com` | `IdentityProviders.java` | Authentification |
| `www.facebook.com` | `IdentityProviders.java` | OAuth tiers |
| `www.linkedin.com` | `IdentityProviders.java` | OAuth tiers |
| `login.live.com` | `IdentityProviders.java` | Microsoft OAuth |
| `www.paypal.com` | `IdentityProviders.java` | Paiement |
| `twitter.com` | `IdentityProviders.java` | OAuth tiers |
| `login.yahoo.com` | `IdentityProviders.java` | OAuth tiers |
| `plus.google.com` | `zzm.java` | Google+ (déprécié) |
| `pagead2.googlesyndication.com` | `zzd.java`, `zzgc.java` | Publicité Google |
| `googleads.g.doubleclick.net` | `zzbz.java` | Publicité Google |
| `csi.gstatic.com` | `zzbz.java` | Google Static |
| `schema.org` | `Action.java` | Schémas sémantiques |

---

##  Endpoints et APIs

Ces chemins ont été identifiés directement dans le code source de l'application bancaire :

| Endpoint | Fichier source | Risque |
|---|---|---|
| `/login` | `DoLogin.java` | Authentification principale |
| `/devlogin` | `DoLogin.java` |  **CRITIQUE** : Porte dérobée de développement |
| `/changepassword` | `ChangePassword.java` | Modification de mot de passe |
| `/dotransfer` | `DoTransfer.java` | Exécution de virements bancaires |
| `/getaccounts` | `DoTransfer.java` | Récupération des comptes |
| `/Statements_` | `DoTransfer.java`, `ViewStatement.java` | Relevés bancaires |
| `TrackUserContentProvider` | `TrackUserContentProvider.java` | Suivi des utilisateurs |

---

##  URLs HTTP/HTTPS

### URLs HTTP (Non chiffrées) —  Risque MITM
- `http://www.google-analytics.com` — analytics non chiffrées
- `http://plus.google.com/` — lien vers Google+ (service déprécié)
- `http://www.google.com` — requêtes non sécurisées
- `http://schema.org/...` — métadonnées sémantiques non chiffrées

### URLs HTTPS (Chiffrées) —  Acceptables
- `https://ssl.google-analytics.com` — analytics sécurisées
- `https://www.googletagmanager.com` — Tag Manager sécurisé
- `https://www.googleapis.com/auth/...` — endpoints Google API
- `https://www.facebook.com`, `https://twitter.com`, etc. — fournisseurs OAuth
- `https://googleads.g.doubleclick.net/mads/...` — SDK publicité Mobile

### Content URIs Android (internes)
- `content://com.android.insecurebankv2.TrackUserContentProvider/trackerusers` — **Exposition possible de données de suivi utilisateurs.**
- `content://com.google.android.gms.games/` — Data provider interne Google GMS.

---

##  Emails et identifiants

| Valeur | Fichier | Occurrences | Risque |
|---|---|---|---|
| `android@android.com` | `zzc.java` | 6 | Email hardcodé, peut servir de credential par défaut |
| `loginscreen_password` | `strings.xml` | 1 | Identifiant de champ lié au mot de passe |

**Recommandation :** Ne jamais hardcoder des emails dans le code source. Utiliser des ressources configurables et mettre en œuvre un contrôle d'accès approprié.

---

##  Technologies détectées

| Technologie | Valeur / Indicateur | Statut |
|---|---|---|
| Compilateur DEX | `dx` (via dexmerge) |  Manipulé |
| Librairie Crypto | `AES/CBC/PKCS5Padding` |  Vulnérable Padding Oracle |
| Librairie Crypto | `MD5` (via `getInstance("MD5")`) |  Algorithme cassé |
| SDK Google GMS | Bibliothèques internes `com.google.android.gms.*` |  Présent |
| SDK Google Analytics | `analytics.internal.zzy.java` |  Appels HTTP en clair |
| SDK Google Ads | `ads.internal.*` |  Standard |
| SDK Google Tag Manager | `tagmanager.*` |  Standard |

---

##  Vulnérabilités détectées (Code Source)

### 1. 🔴 Client HTTP Non Sécurisé — CWE-757 (Sévérité : Low)
- **Description :** L'application instancie `DefaultHttpClient()` sans forcer le chiffrement SSL/TLS. Toutes les communications bancaires (login, transfert, changement de mot de passe) passent par ce client non sécurisé.
- **Fichiers impactés :** `DoLogin.java`, `ChangePassword.java`, `DoTransfer.java`
- **Risque :** Interception des données en transit par une attaque Man-In-The-Middle (MITM) sur un réseau Wi-Fi public.
- **Remédiation :** Remplacer `DefaultHttpClient()` par `OkHttpClient` ou `HttpsURLConnection` avec validation de certificat. Forcer TLS 1.2 minimum.

---

### 2. 🔴 Attaque Padding Oracle CBC Possible (Sévérité : Medium-High)
- **Description :** `Cipher.getInstance("AES/CBC/PKCS5Padding")` est utilisé dans `CryptoClass.java`. En mode CBC sans vérification d'intégrité (HMAC), un attaquant peut progressivement décrypter le contenu chiffré en exploitant les messages d'erreur du padding.
- **Fichiers impactés :** `CryptoClass.java`, `com.google.android.gms.internal.zzar.java`
- **Risque :** Décryptage non autorisé des données bancaires sensibles chiffrées localement.
- **Remédiation :** Migrer vers `AES/GCM/NoPadding` (chiffrement authentifié) ou ajouter un HMAC avant de vérifier le padding. Utiliser des IV aléatoires générés cryptographiquement.

---

### 3. 🔴 Algorithmes Cryptographiques Faibles (MD5) — CWE-327 (Sévérité : Low)
- **Description :** `MessageDigest.getInstance("MD5")` est appelé dans les librairies Google GMS internes. MD5 est considéré comme cassé cryptographiquement depuis 2004.
- **Fichiers impactés :** `zza.java`, `zzak.java`, `zzbl.java`, `zzhl.java` (GMS interne)
- **Risque :** Vulnérable aux attaques par collision, pouvant mener à des falsifications d'empreintes ou d'identités.
- **Remédiation :** Remplacer MD5 par SHA-256 ou SHA-3. Mettre à jour les dépendances Google Play Services.

---

### 4. 🟡 Injection SQL Non-Paramétrée — CWE-89 (Sévérité : Low)
- **Description :** `rawQuery("SELECT * FROM " + str + " LIMIT 0", null)` — une requête SQL brute est construite en concaténant une variable non filtrée.
- **Fichier impacté :** `com.google.android.gms.analytics.internal.zzj.java`
- **Risque :** Injection SQL locale pouvant entraîner une fuite ou une corruption de la base de données SQLite.
- **Remédiation :** Utiliser des requêtes paramétrées (`?` comme placeholder), un ORM, ou `SQLiteQueryBuilder`. Valider et nettoyer toutes les entrées.

---

### 5. 🟡 Désérialisation d'Objets Non Sécurisée — CWE-502 (Sévérité : Low)
- **Description :** `new ObjectInputStream(byteArrayInputStream)` désérialise des données binaires sans contrôle sur le type ou le contenu des objets.
- **Fichier impacté :** `com.google.android.gms.tagmanager.zzw.java`
- **Risque :** Si les données désérialisées sont contrôlables par un attaquant, un RCE (Remote Code Execution) est envisageable.
- **Remédiation :** Éviter la désérialisation Java native. Utiliser JSON/Protobuf. Implémenter un filtre de désérialisation (`ObjectInputFilter` en Java 9+).

---

##  Secrets et Données Sensibles

### Activités Android Exportées — CWE-926 (Sévérité : Low)
- **Description :** Quatre activités sensibles sont marquées `android:exported=true` dans `AndroidManifest.xml` sans restriction de permission. N'importe quelle application Android installée sur l'appareil peut les déclencher.
- **Activités concernées :**
  - `com.android.insecurebankv2.PostLogin`
  - `com.android.insecurebankv2.DoTransfer`
  - `com.android.insecurebankv2.ViewStatement`
  - `com.android.insecurebankv2.ChangePassword`
- **Risque :** Une application malveillante peut initier un `startActivity()` vers `DoTransfer` ou `ChangePassword` sans authentification préalable.
- **Remédiation :** Définir `android:exported="false"` pour toutes les activités non accessibles depuis l'extérieur. Si une activité doit être exportée, protéger l'accès avec une permission personnalisée (`android:permission`).

---

### Secret Possible dans Strings — Sévérité : Low
- **Description :** La valeur `loginscreen_password` exposée dans `strings.xml` constitue un indicateur de credential potentiellement hardcodé accessible par décompilation.
- **Fichier :** `resources/res/values/strings.xml`
- **Risque :** Extraction triviale par tout attaquant ayant accès à l'APK.
- **Remédiation :** Ne jamais stocker de credentials, même partiels, dans `strings.xml`. Utiliser Android Keystore ou un gestionnaire de secrets.

---

##  Malware et Alertes APKID

| Indicateur | Valeur | Signification |
|---|---|---|
| **Anti-VM** | `Build.MODEL check`, `Build.MANUFACTURER check`, `Build.PRODUCT check` | L'application vérifie si elle tourne sur un émulateur, comportement typique des échantillons cherchant à éviter l'analyse. |
| **Compilateur** | `dx (possible dexmerge)` | Compilation standard mais potentiellement post-traitée. |
| **Manipulateur** | `dexmerge` | Recombination de fichiers DEX, peut indiquer une obfuscation légère ou un build automatisé. |

**Note :** InsecureBankv2 étant une application pédagogique délibérément vulnérable, les mécanismes Anti-VM sont probablement inclus à titre d'exemple. Ils seraient néanmoins un signal critique dans un contexte réel.

---

##  Synthèse Globale

### Évaluation de la posture de sécurité externe

| Catégorie | Niveau de risque | Commentaire |
|---|---|---|
| Communications réseau | 🔴 CRITIQUE | HTTP en clair pour toutes les opérations bancaires |
| Chiffrement des données | 🔴 CRITIQUE | Mode CBC vulnérable, MD5 utilisé |
| Authentification et contrôle d'accès | 🔴 CRITIQUE | Endpoint `/devlogin` + activités exportées |
| Exposition d'informations | 🟡 MOYEN | Endpoints, chemins et emails exposés |
| Intégrité du binaire | 🟡 MOYEN | Anti-VM + dexmerge détectés |
| Gestion des dépendances | 🟢 BAS | Google GMS standard, mais versions internes à vérifier |

### Recommandations finales par priorité

** Priorité 1 — Immédiat (Corrections bloquantes)**
1. Remplacer `DefaultHttpClient` par `OkHttpClient` avec pinning de certificat
2. Passer de `AES/CBC` à `AES/GCM` dans `CryptoClass.java`
3. Désactiver ou sécuriser l'endpoint `/devlogin` en production
4. Mettre `android:exported="false"` sur toutes les activités bancaires

** Priorité 2 — Court terme**
5. Remplacer MD5 par SHA-256 dans toutes les fonctions de hachage
6. Paramétrer toutes les requêtes SQL (`rawQuery` → `query()` avec `?`)
7. Retirer l'email `android@android.com` et tout credential du code source

** Priorité 3 — Moyen terme**
8. Évaluer et supprimer les checks Anti-VM si non nécessaires
9. Implémenter un filtre de désérialisation Java
10. Auditer et mettre à jour les versions des librairies GMS
11. Activer la validation de certificat SSL et implémenter certificate pinning

---

*Document généré le 2026-03-22 — Environnement : Windows, Yaazhini 2.0.2, BeVigil Web Scan*
*Ce rapport est basé sur une analyse statique. Une validation dynamique avec Yaazhini est recommandée pour confirmer l'exploitabilité.*
