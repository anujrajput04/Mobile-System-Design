## Basic Login/Password over HTTPS
- The concept is to use traditional username + password credentials sent via `POST /login` over HTTPS.
- **Security Assumptions**: HTTPS is mandatory - TLS encryption is what protects credentials in transit.
- **Best Practices**:
	- Rate-limit login attempts, prevent brute-force
	- Don't store passwords directly, hash with bcrypt/scrypt + salt on the backend
	- Never log credentials