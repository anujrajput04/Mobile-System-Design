## Third-Party Logins

#### Sign-in with Apple
- Platform-first login mechanism (iOS/macOS/web)
- Key Features:
	- Uses JWT (JSON Web Tokens)
	- "Hide My Email" feature (Apple relays real email)
	- User approval required for each new device or app

**Workflow**:
1. iOS SDK presents Apple sign-in
2. User authenticates using FaceID/TouchID
3. App receives `identityToken` (JWT), sends it to backend
4. Backend verified the JWT with Apple's public key


#### Sign-in with Google / Facebook / Others
- Uses [[OAuth 2.0]] protocol under the hood
- App receives an access token from the provider
- Backend optionally verifies the token or uses it to get user profile info