# BeVigil / BigDvil Analysis Notes

## Element 1: Insecure Communication
- **Location:** ChangePassword.java, DoLogin.java, DoTransfer.java
- **Description:** HTTP used instead of HTTPS.
- **Impact Potential:** Intercepted credentials, sensitive data leakage.
- **Remediation Suggested:** Switch to HTTPS and enforce TLS.

## Element 2: Weak Random Number Generation
- **Location:** zzl.java, Tracker.java
- **Description:** java.util.Random used instead of SecureRandom.
- **Impact Potential:** Predictable values, security compromise.
- **Remediation Suggested:** Use java.security.SecureRandom.

## Element 3: Debuggable App
- **Location:** AndroidManifest.xml
- **Description:** android:debuggable="true"
- **Impact Potential:** Allows debugging and reverse engineering.
- **Remediation Suggested:** Set android:debuggable="false".

## Element 4: Exposed Provider
- **Location:** TrackUserContentProvider in AndroidManifest.xml
- **Description:** android:exported="true" on sensitive provider
- **Impact Potential:** Unauthorized access to internal data.
- **Remediation Suggested:** Set android:exported="false" if not needed externally.

## Element 5: Weak Hash Algorithm
- **Location:** Multiple Java classes (MD5/SHA1)
- **Description:** Insecure hashing algorithms used
- **Impact Potential:** Weak password hashing, integrity issues.
- **Remediation Suggested:** Use SHA-256 or PBKDF2.
