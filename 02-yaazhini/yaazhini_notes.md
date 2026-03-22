# Notes d'analyse Yaazhini

## Éléments identifiés

### Élément 1: Hardcoded API Key / Credentials (Secret)
- **Localisation**: `resources/res/values/strings.xml`
- **Description**: La chaîne de caractères `loginscreen_password` contient potentiellement un mot de passe ou une clé d'API définis en dur dans le code source décompilé.
- **Impact potentiel**: Une exposition d'identifiants permettrait à un attaquant ou un auditeur ayant décompilé l'application d'usurper une identité système ou d'insérer de fausses données (Risque Critique).
- **Remédiation suggérée**: Retirer toute information d'identification du code source. Les secrets doivent être gérés côté serveur ou stockés de façon chiffrée via l'Android Keystore natif.

### Élément 2: Traffic réseau en clair autorisé (Configuration)
- **Localisation**: `resources/AndroidManifest.xml` (Attribut `android:usesCleartextTraffic`)
- **Description**: L'application ne possède pas de `NetworkSecurityConfig` interdisant les connexions non sécurisées, et les connexions HTTP sont autorisées par défaut pour les API minimales ciblées.
- **Impact potentiel**: Exposition de toutes les communications bancaires interceptées au travers d'une attaque *Man-In-The-Middle* (Attaque MITM). Les données naviguent sur le réseau sans chiffrement.
- **Remédiation suggérée**: Imposer `android:usesCleartextTraffic="false"` dans le Manifest et mettre en place une `Network Security Configuration` stricte exigeant le protocole HTTPS.

### Élément 3: Composant Activity exporté (Configuration Externe)
- **Localisation**: `resources/AndroidManifest.xml` (Balises `<activity>`)
- **Description**: Des activités critiques de sécurité telles que `DoTransfer` et `ChangePassword` sont configurées avec `android:exported="true"` (implicitement ou explicitement via Intent Filters vides).
- **Impact potentiel**: Des applications tierces installées sur le même périphérique peuvent forcer le lancement de ces activités bancaires en bypassant l'écran de connexion initial, menant à une faille logique fatale.
- **Remédiation suggérée**: Modifier ces activités avec l'attribut `android:exported="false"`, ou les sécuriser avec un système de permission `android:permission` personnalisé (`signature`).

### Élément 4: Autorisation de Sauvegarde activée (Configuration)
- **Localisation**: `resources/AndroidManifest.xml` (Attribut `android:allowBackup="true"`)
- **Description**: Le drapeau autorisant la sauvegarde des données de l'application via `adb backup` est demeuré actif.
- **Impact potentiel**: Un accès physique à l'appareil d'une victime, ou même au Cloud Android, permettrait d'extraire des bases de données de l'utilisateur ou le fichier local des `Shared Preferences` où des mots de passe sont conservés en clair.
- **Remédiation suggérée**: Passer ce paramètre à `android:allowBackup="false"` dans l'application si aucune solution spécifique n'est implémentée pour filtrer les données à exclure de la sauvegarde de l'OS.

### Élément 5: Requête SQL dynamique (Faille d'implémentation)
- **Localisation**: `sources/com/google/android/gms/analytics/internal/zzj.java` (et classes similaires)
- **Description**: Exécution de `rawQuery()` avec l'usage de concaténation de chaînes de caractères au lieu d'arguments liés (Prepared Statements).
- **Impact potentiel**: Exécution de commandes SQL arbitraires si l'application autorise une entrée utilisateur à remonter jusqu'à ce constructeur. Risque de corruption de base de données locale ou d'extraction de comptes.
- **Remédiation suggérée**: Adopter une approche par requêtes paramétrées utilisant `?` (binding array string) ou exploiter des frameworks ORM modernes comme *Room*.
