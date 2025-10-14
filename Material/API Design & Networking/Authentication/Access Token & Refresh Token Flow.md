#### Access Token
- Short lived (eg. 15 mins - 1 hour)
- Used to authenticated each API request: `Authorization: Bearer <access_token>`

#### Refresh Token
- Long lived (eg. days or weeks)
- Used to get a new access token when the old one expires
- Should not be sent with each API call

#### Flow
```
1. App logs in. Gets access token + refresh token
2. Access token is used for API calls to autorize
3. On 401 Unauthorized:
	1. App uses refresh token to request a new access token
	2. Update token storage
```

### Token storage best practices

| Platform | Secure Storage Method                                                |
| -------- | -------------------------------------------------------------------- |
| iOS      | Keychain, encrypted, secure enclave                                  |
| Android  | EncryptedSharedPreferences, Keystore                                 |
| Shared   | Never store tokens in NSUserDefaults / SharedPreferences             |
| WebView  | Avoid injecting tokens into localStorage if using embedded web views |
**Rules**:
- Don't persist refresh tokens in memory alone
- Use biometric protection if supported (eg. FaceID)
- Tokens must be invalidated on logout or app uninstall