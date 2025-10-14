## Multi-Factor Authentication
Multi-Factor Authentication (MFA) adds an additional layer of security during user authentication by requiring two or more independent credentials (factors)

**The 3 common factors**:
- Something you know: Password, PIN
- Something you have: Phone, hardware token, TOTP app
- Something you are: Fingerprint, facial recognition

**Common MFA Methods**:

| Method                              | Factor Type        | Description                                                                                 |
| ----------------------------------- | ------------------ | ------------------------------------------------------------------------------------------- |
| TOTP (Time-based One Time Password) | Something you have | Code changes every 30s, RFC 6238 standard. eg. Apple Passwords, Google Authenticator, Authy |
| SMS-based OTP                       | Something you have | Code sent via SMS. Vulnerable to SIM swap or SMS interception                               |
| Push Notification                   | Something you have | eg. Okta, Approve/deny on trusted device                                                    |
| Email OTP                           | Something you have | OTP sent to email. Weaker, used when others not available                                   |
| Hardware Key (FIDO2 / U2F)          | Something you have | eg. Yubikey, Strong cryptographic security                                                  |
| Biometric Authentication            | Something you are  | FaceID, TouchID, Secure if paired with hardware backed key storage                          |

**Implementation Flow**:
1. User logs in with username + password
2. Server verifies credentials
3. If correct, prompt for 2FA code
4. User enters 6 digit TOTP from app
5. Server verifies code using shared secret
6. If TOTP is valid, session or token is granted

**Backend Handling**:
- Store TOTP securely eg. encrypted DB field
- Use libraries like `PyOTP`, `otplib`, or `Google Authenticator-compatible` algorithms
- Rate-limit MFA attempts
- Enable backup codes or fallback methods

**Security Considerations**:
- NEVER use only SMS for MFA in high-risk applications
- Protect recovery options eg. don't let email OTP bypass strong MFA
- Regularly expire trusted devices and re-prompt MFA
- Use step-up authentication for sensitive actions (eg. changing password)

**When to use MFA**:
- On every login, optional "remember device"
- On sensitive actions eg. money transfer, email change
- During abnormal behavior eg. login from a new country/device