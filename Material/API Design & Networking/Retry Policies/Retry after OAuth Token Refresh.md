## Retry after OAuth Token Refresh
**Problem**:
The API request returns `401 Unauthorized` due to expired access token

**Solution**:
1. Detect `401` or `403` response
2. Pause queued requests
3. Send refresh token request
4. If successful, retry the original requests with the new access token
5. If failed, redirect to login or handle logout

iOS: Alamofire `RequestInterceptor`
Android: OkHttp's `Authenticator` interface