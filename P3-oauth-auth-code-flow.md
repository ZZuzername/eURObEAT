# Exercise P3: OAuth 2.0 Authorization Code Flow

## Diagram (all four roles)

```
 Resource Owner        Client (app)        Authorization Server     Resource Server (API)
   (user)                                    (sign-in server)
     |                      |                         |                        |
     | 1. Click "Connect"   |                         |                        |
     |--------------------->|                         |                        |
     |                      | 2. Redirect to /authorize                        |
     |                      |  (client_id, redirect_uri, scope, state)         |
     |                      |------------------------>|                        |
     | 3. Sign-in + consent screen                    |                        |
     |<-----------------------------------------------|                        |
     | 4. Sign in, approve scopes                     |                        |
     |----------------------------------------------->|                        |
     |                      | 5. Redirect back with   |                        |
     |                      |    one-time code + state|                        |
     |                      |<------------------------|                        |
     |                      | 6. POST /token          |                        |
     |                      |  (code, client_id, secret)                       |
     |                      |------------------------>|                        |
     |                      | 7. access_token,        |                        |
     |                      |    refresh_token, expires_in                     |
     |                      |<------------------------|                        |
     |                      | 8. GET /workers                                  |
     |                      |    Authorization: Bearer <access_token>          |
     |                      |------------------------------------------------->|
     |                      | 9. 200 OK + data (valid token, right scope)      |
     |                      |<-------------------------------------------------|
```

## Expiry handling

When the access token expires partway through a task, the API starts answering 401 Unauthorized, often with a `WWW-Authenticate` header saying the token expired. Calls that already succeeded still count, but anything sent with the old token fails. My code should read `expires_in` and refresh a little before expiry so most tasks never hit a 401. If a 401 still comes back, it sends the refresh token to `/token` once, saves the new access token (and the new refresh token if one comes back), and retries only the request that failed. If the refresh fails too, for example with `invalid_grant`, it stops and asks the user to sign in again instead of looping. It doesn't blindly resend a POST that might have gone through, unless it has an idempotency key, and it never logs or commits either token.
