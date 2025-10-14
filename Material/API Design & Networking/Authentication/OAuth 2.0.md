## OAuth 2.0 & OpenID Connect

#### OAuth 2.0
- Purpose: Authorization
- Actors:
	- Resource Owner (user)
	- Client (app)
	- Authorization Server
	- Resource Server (API backend)
- Grant Types:
	- Authorization Code Flow: recommended for mobile, secure, with server involvement
	- Client Credentials: non-user flows
	- Device Code Flow: for Smart TVs etc

#### OpenID Connect
- Extension of OAuth 2.0 for authentication
- Adds ID Token (JWT) on top of OAuth