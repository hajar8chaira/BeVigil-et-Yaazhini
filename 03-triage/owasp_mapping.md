# Mapping OWASP (InsecureBankv2)

## FIND-001: Communication HTTP Insecure
- **Catégorie OWASP**: MASVS-NETWORK
- **Référence spécifique**: V5.1 (*Data is encrypted on the network using TLS*)
- **Justification**: Les transferts en clair (HTTP) autorisent l'interception et l'altération des transferts de fonds et identifiants sur le réseau.

## FIND-004: Sauvegarde ADB Activée
- **Catégorie OWASP**: MASVS-STORAGE
- **Référence spécifique**: V2.8 (*No sensitive data is included in backups*)
- **Justification**: L'activation de la sauvegarde permet à une personne ayant un accès physique (via adb) de pomper l'historique bancaire de l'application.

## FIND-005: Composant Provider / Activity Exposé
- **Catégorie OWASP**: MASVS-PLATFORM
- **Référence spécifique**: V6.2 (*All inputs from external sources and the user are validated*)
- **Justification**: L'exportation non sécurisée des composants liés aux transferts et bases de données crée une entrée directe pour les applications malveillantes contournant le flux normal.

## FIND-008: Hardcoded Credentials (Secret en dur)
- **Catégorie OWASP**: MASVS-STORAGE
- **Référence spécifique**: V2.1 (*System credential storage facilities are used...*)
- **Justification**: L'inscription d'un mot de passe dans `strings.xml` annule l'authentification et rend sa découverte triviale lors de l'ingénierie inverse.

## FIND-009: Requête SQL Dynamique
- **Catégorie OWASP**: MASVS-STORAGE
- **Référence spécifique**: V2.3 (*No sensitive data is available to third-party apps... or SQL injections*)
- **Justification**: L'utilisation d'une concaténation de chaînes dans `rawQuery` pour les requêtes de base de données introduit une vulnérabilité directe d'injection SQL locale.
