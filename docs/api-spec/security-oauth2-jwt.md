# OAuth2 & JWT Authenticatie

Veilige API autorisatie met OAuth2 Authorization Code Flow + JWT tokens.

Referentie: RFC 6749, RFC 7519

---

## Architecture

```
┌──────────┐                    ┌──────────────────┐
│  Client  │                    │ Authorization    │
│ (Web App)│ ◄──────────────────┤ Server / IDP     │
│          │  Authorization    │ (Keycloak)       │
└──────┬───┘  Code + State      └──────────────────┘
       │
       │ (3) Exchange code → access_token
       │     (with client_secret)
       │
       ▼
┌──────────────────────┐         ┌────────────────────┐
│  Schulddienstverlening          │  Resource Server   │
│  API                 │ ◄────────┤  (API Backend)     │
│                      │  Validate JWT                  │
└──────────────────────┘         └────────────────────┘
```

---

## Flow: Authorization Code

### 1. Initiate Authorization

User clicks "Login" → redirect to:

```
GET https://auth.schulddienstverlening.nl/auth
  ?client_id=my-app-id
  &response_type=code
  &redirect_uri=https://my-app.nl/callback
  &scope=openid%20profile%20email%20schulddossier:read
  &state=random-state-string
```

**Parameters**:
- `client_id`: Your app identifier
- `response_type`: "code" (Authorization Code Flow)
- `redirect_uri`: Callback endpoint (must be whitelist ed)
- `scope`: Permissions (space-separated)
  - `openid`: OpenID Connect
  - `profile`: Name, picture
  - `email`: Email address
  - `schulddossier:read`: Read dossiers
  - `schulddossier:write`: Edit dossiers
- `state`: CSRF protection

### 2. User Logs in & Consents

User enters credentials, approves scopes → Redirected to `redirect_uri`:

```
GET https://my-app.nl/callback
  ?code=auth-code-xyz
  &state=random-state-string
```

**Server validates**:
- `state` matches (CSRF check)
- `code` is fresh (< 10 min)

### 3. Exchange Code for Tokens

Server (backend):

```
POST https://auth.schulddienstverlening.nl/token
  Content-Type: application/x-www-form-urlencoded

  grant_type=authorization_code
  &code=auth-code-xyz
  &client_id=my-app-id
  &client_secret=secret-xyz  # ← Keep secret!
  &redirect_uri=https://my-app.nl/callback
```

**Response** (200 OK):

```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 900,
  "refresh_token": "refresh-xyz...",
  "id_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

### 4. Use Access Token

Client requests with Bearer token:

```
GET https://api.schulddienstverlening.nl/v1/schulddossiers
  Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...
```

---

## JWT Token Structure

**Access Token** (short-lived, 15 min):

```json
{
  "iss": "https://auth.schulddienstverlening.nl",
  "aud": "schulddienstverlening-api",
  "sub": "user-123",
  "preferred_username": "jan@example.nl",
  "email": "jan@example.nl",
  "name": "Jan de Vries",
  "roles": ["burger", "schuldhulpverlener"],
  "burger_id": "123456789",
  "hulpverlener_id": "uuid-xyz",
  "scopes": ["schulddossier:read", "schulddossier:write"],
  "exp": 1716734400,
  "iat": 1716733500,
  "jti": "unique-id"
}
```

**ID Token** (for identification, not authorization):

```json
{
  "iss": "https://auth.schulddienstverlening.nl",
  "aud": "my-app-id",
  "sub": "user-123",
  "email": "jan@example.nl",
  "email_verified": true,
  "name": "Jan de Vries",
  "picture": "https://...",
  "exp": 1716734400,
  "iat": 1716733500
}
```

---

## Token Validation (API Server)

### 1. Signature Verification

API gets public keys from:

```
GET https://auth.schulddienstverlening.nl/.well-known/jwks.json
```

Response:

```json
{
  "keys": [
    {
      "kid": "key-1",
      "use": "sig",
      "kty": "RSA",
      "n": "0vx7agoebGcQSuuPiLJXZ...",
      "e": "AQAB"
    }
  ]
}
```

Library verifies: RS256 signature using public key

### 2. Claims Validation

- `exp > NOW`: Token not expired
- `iat <= NOW`: Issued in past
- `iss == "https://auth.schulddienstverlening.nl"`: Trusted issuer
- `aud == "schulddienstverlening-api"`: Correct audience
- Scopes include required permission

### 3. Role-Based Authorization

```python
# After JWT validation:

def authorize(required_scope):
    token = parse_jwt(request.headers['Authorization'])
    
    if required_scope not in token['scopes']:
        raise ForbiddenError("Insufficient permissions")
    
    if 'burger' in token['roles'] and required_scope == 'schulddossier:write':
        # Burger can't modify (read-only)
        raise ForbiddenError()
    
    return token
```

---

## Scopes & Permissions

### Scopes

| Scope | Target | Allows |
|-------|--------|--------|
| `openid` | Auth | ID Token |
| `profile` | Auth | Name, picture |
| `email` | Auth | Email |
| `schulddossier:read` | API | GET /schulddossiers |
| `schulddossier:write` | API | POST/PUT/DELETE /schulddossiers |
| `stats:read` | API | GET /stats/** |
| `audit:read` | API | GET /audit/** (admin only) |

**Default Scopes by Role**:

| Role | Scopes |
|------|--------|
| Burger | `openid profile email schulddossier:read` |
| Hulpverlener | `openid profile email schulddossier:read schulddossier:write` |
| Gemeente | `openid profile email stats:read` |
| SysAdmin | All |

---

## Refresh Tokens

Refresh token (7 dagen, kan geroteerd):

```
POST https://auth.schulddienstverlening.nl/token
  grant_type=refresh_token
  &refresh_token=refresh-xyz-abc
  &client_id=my-app-id
  &client_secret=secret-xyz
```

Response: New access_token + refresh_token (rotation)

---

## Revocation

Logout / token revocation:

```
POST https://auth.schulddienstverlening.nl/revoke
  token=access-token-or-refresh-token
  &client_id=my-app-id
  &client_secret=secret-xyz
```

Response: `204 No Content`

---

## Service-to-Service (Client Credentials)

For server-to-server calls (e.g., notifications service):

```
POST https://auth.schulddienstverlening.nl/token
  grant_type=client_credentials
  &client_id=notification-service
  &client_secret=secret-xyz
  &scope=schulddossier:read
```

Response: `access_token` (no refresh_token)

---

## Security Best Practices

✅ **Always**:
- Use HTTPS (TLS 1.3+)
- Store `client_secret` securely (env var, not code)
- Validate `state` parameter (CSRF)
- Verify JWT signature
- Check exp timestamp
- Use short TTL for access tokens (15 min)
- Rotate refresh tokens

❌ **Never**:
- Expose `client_secret` in frontend code
- Use "implicit" grant flow
- Trust `kid` from JWT header alone (verify in JWKS)
- Store access_token in localStorage (use secure httpOnly cookie)
- Use authorization_code for mobile app (use PKCE instead)

---

## Error Responses

### Invalid Token
```json
{
  "error": "invalid_token",
  "error_description": "Token signature verification failed"
}
```

### Expired Token
```json
{
  "error": "invalid_token",
  "error_description": "Token expired at 2026-05-26T14:00:00Z"
}
```

### Insufficient Scope
```json
{
  "error": "insufficient_scope",
  "error_description": "Required scope: schulddossier:write"
}
```

---

## Configuration

Keycloak setup (for admins):

```
Realm: schulddienstverlening
Client: schulddienstverlening-api
  - Access Type: bearer-only
  - Valid Redirect URIs: https://my-app.nl/callback
  - Web Origins: https://my-app.nl

Roles:
  - burger
  - schuldhulpverlener
  - gemeente
  - admin

Protocol Mappers:
  - Add "roles" claim to access token
  - Add "burger_id", "hulpverlener_id" claims
```

---

## Example Code

### JavaScript/NodeJS

```javascript
// Login
async function login() {
  const state = crypto.randomUUID();
  sessionStorage.setItem('auth-state', state);
  
  window.location = `https://auth.schulddienstverlening.nl/auth`
    + `?client_id=my-app-id`
    + `&response_type=code`
    + `&redirect_uri=${encodeURIComponent('https://my-app.nl/callback')}`
    + `&scope=openid profile email schulddossier:read`
    + `&state=${state}`;
}

// Callback
async function handleCallback(searchParams) {
  const code = searchParams.get('code');
  const state = searchParams.get('state');
  
  if (state !== sessionStorage.getItem('auth-state')) {
    throw new Error('CSRF check failed');
  }
  
  const response = await fetch('https://auth.schulddienstverlening.nl/token', {
    method: 'POST',
    headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
    body: new URLSearchParams({
      grant_type: 'authorization_code',
      code,
      client_id: 'my-app-id',
      client_secret: 'secret-xyz',
      redirect_uri: 'https://my-app.nl/callback'
    })
  });
  
  const data = await response.json();
  localStorage.setItem('access_token', data.access_token);
  localStorage.setItem('refresh_token', data.refresh_token);
}

// API Request
async function getdossiers() {
  const token = localStorage.getItem('access_token');
  const response = await fetch('https://api.schulddienstverlening.nl/v1/schulddossiers', {
    headers: {
      'Authorization': `Bearer ${token}`
    }
  });
  return response.json();
}
```

---

**Document version**: 1.0 | **Last update**: Mei 2026
