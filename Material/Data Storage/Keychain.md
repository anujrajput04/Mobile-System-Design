## iOS: Keychain
The Keychain is Apple's secure storage system for small, sensitive data. Optimized for credentials, tokens, certificates, keys but not for bulk data. Data is encrypted at rest and in transit, tied to device's security hardware (Secure Enclave when available). Apps don't manage encryption directly, Keychain handles it.

### Use Cases
- Storing user credentials (username/password, OAuth Token)
- Storing cryptographic keys (RSA, EC)
- Securely persisting session tokens between app launches
- Sharing login credentials between apps from the same developer team
- Persisting credentials across device backups and restores

### How it works