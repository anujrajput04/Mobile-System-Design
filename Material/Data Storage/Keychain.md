## iOS: Keychain
The Keychain is Apple's secure storage system for small, sensitive data. Optimized for credentials, tokens, certificates, keys but not for bulk data. Data is encrypted at rest and in transit, tied to device's security hardware (Secure Enclave when available). Apps don't manage encryption directly, Keychain handles it.

### Use Cases
- Storing user credentials (username/password, OAuth Token)
- Storing cryptographic keys (RSA, EC)
- Securely persisting session tokens between app launches
- Sharing login credentials between apps from the same developer team
- Persisting credentials across device backups and restores

### How it works
1. APIs
    - `Keychain Services` framework (`SecItemAdd`, `SecItemUpdate`, `SecItemCopyMatching`, `SecItemDelete`)
    - Swift wrappers exist (eg. KeychainAccess, Locksmith)
2. Security Levels
    - Accessibility attributes define when the item is available:
        - `kSecAttrAccessibleWhenUnlocked`
        - `kSecAttrAccessibleAfterFirstUnlock`
        - `kSecAttrAccessibleAlways`
        - `kSecAttrAccessibleWhenPasscodeSetThisDeviceOnly` (highest security, requires passcode, doesn’t sync).
3. Hardware Security
    - On modern devices, secrets can be stored in the **Secure Enclave** (isolated processor, resistant to extraction).
    - Biometrics (Face ID/Touch ID) can gate access to Keychain items.
4. **Sharing**
    - Apps from the same team ID can share Keychain data via Access Groups.
    - Used for Single Sign-On (SSO) within a suite of apps.

---

## iOS vs Android vs Backend

- **iOS Keychain** → OS-managed, hardware-backed, item-based secure storage.
- **Android equivalent** → **Android Keystore + EncryptedSharedPreferences**.
    - Keystore stores keys; EncryptedSharedPreferences stores small secrets.
- **Backend equivalent** → **Secret Managers** (AWS Secrets Manager, HashiCorp Vault, Google Secret Manager).
    - Used to securely store API keys, DB credentials, certificates.

---

## Pros & Limitations

**Pros:**
- Strong hardware-backed security.
- Automatically encrypted and decrypted by the OS.
- Integrates with Face ID / Touch ID.
- Works with iCloud Keychain to sync credentials across devices (if allowed).

Limitations:
- Not for large data (binary blobs, files).
- API can be verbose and low-level.
- Migration between accessibility levels and syncing needs careful handling.
- Access can be slower than in-memory or local DB.

---

## 🔍 Interview Talking Points

When asked about Keychain, emphasize:
- **Appropriate usage** → only for sensitive small data (tokens, credentials).
- **Security properties** → encrypted, hardware-backed, supports biometrics.
- **Alternatives** → SQLite/UserDefaults for non-sensitive data, Keychain for secrets.
- **Cross-platform awareness** → compare with Android Keystore and server-side Secret Managers.
- **Design trade-offs** → e.g., don’t store entire user profiles in Keychain, just the token.