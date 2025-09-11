# API Authentication

Relyr provides multiple authentication methods to secure your API access. Choose the method that best fits your use case and security requirements.

## Authentication Methods

### API Key Authentication

The simplest method using API keys in request headers.

#### Generate API Key

```bash
# Generate a new API key
relyr api-key generate --name "My Application"

# List existing keys
relyr api-key list

# Revoke an API key
relyr api-key revoke --id key_123456
```

#### Usage

Include the API key in the `X-API-Key` header:

```bash
curl -H "X-API-Key: your-api-key-here" \
     http://localhost:8080/api/v1/proxies
```

**Python Example:**

```python
import requests

headers = {
    'X-API-Key': 'your-api-key-here',
    'Content-Type': 'application/json'
}

response = requests.get(
    'http://localhost:8080/api/v1/proxies',
    headers=headers
)
```

### Bearer Token Authentication

Use JWT or OAuth2 bearer tokens for more advanced authentication.

#### Obtain Token

```bash
# Login to get bearer token
curl -X POST http://localhost:8080/api/v1/auth/login \
     -H "Content-Type: application/json" \
     -d '{"username": "admin", "password": "password"}'
```

**Response:**

```json
{
  "success": true,
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expires_in": 3600,
    "token_type": "Bearer"
  }
}
```

#### Usage

Include the token in the `Authorization` header:

```bash
curl -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
     http://localhost:8080/api/v1/proxies
```

**Python Example:**

```python
import requests

headers = {
    'Authorization': 'Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...',
    'Content-Type': 'application/json'
}

response = requests.get(
    'http://localhost:8080/api/v1/proxies',
    headers=headers
)
```

### Basic Authentication

Traditional username/password authentication using HTTP Basic Auth.

#### Usage

```bash
curl -u username:password \
     http://localhost:8080/api/v1/proxies
```

**Python Example:**

```python
import requests
from requests.auth import HTTPBasicAuth

response = requests.get(
    'http://localhost:8080/api/v1/proxies',
    auth=HTTPBasicAuth('username', 'password')
)
```

## Token Management

### Refresh Tokens

When using bearer tokens, refresh expired tokens:

```bash
curl -X POST http://localhost:8080/api/v1/auth/refresh \
     -H "Content-Type: application/json" \
     -d '{"refresh_token": "your-refresh-token"}'
```

**Response:**

```json
{
  "success": true,
  "data": {
    "access_token": "new-access-token",
    "expires_in": 3600
  }
}
```

### Token Validation

Validate a token's authenticity and expiration:

```bash
curl -X POST http://localhost:8080/api/v1/auth/validate \
     -H "Authorization: Bearer your-token"
```

### Logout

Invalidate a token:

```bash
curl -X POST http://localhost:8080/api/v1/auth/logout \
     -H "Authorization: Bearer your-token"
```

## OAuth2 Integration

For enterprise environments, Relyr supports OAuth2 providers.

### Configuration

```yaml
# relyr.yml
auth:
  oauth2:
    enabled: true
    provider: "google"  # google, github, microsoft, custom
    client_id: "your-client-id"
    client_secret: "your-client-secret"
    redirect_uri: "http://localhost:8080/auth/callback"
    scopes: ["openid", "email", "profile"]
```

### Authorization Flow

1. **Authorization URL**: Redirect users to OAuth2 provider

```bash
curl http://localhost:8080/api/v1/auth/oauth2/authorize?provider=google
```

2. **Callback Handling**: Handle the callback from OAuth2 provider

```bash
curl -X POST http://localhost:8080/api/v1/auth/oauth2/callback \
     -H "Content-Type: application/json" \
     -d '{"code": "authorization-code", "state": "csrf-state"}'
```

3. **Token Exchange**: Exchange authorization code for access token

## API Key Management

### Create API Key

```bash
curl -X POST http://localhost:8080/api/v1/api-keys \
     -H "Authorization: Bearer your-token" \
     -H "Content-Type: application/json" \
     -d '{
       "name": "My Application",
       "permissions": ["proxy.read", "proxy.write"],
       "expires_at": "2024-12-31T23:59:59Z"
     }'
```

### List API Keys

```bash
curl -H "Authorization: Bearer your-token" \
     http://localhost:8080/api/v1/api-keys
```

**Response:**

```json
{
  "success": true,
  "data": {
    "api_keys": [
      {
        "id": "key_123456",
        "name": "My Application",
        "prefix": "rlyr_1234...",
        "permissions": ["proxy.read", "proxy.write"],
        "created_at": "2023-09-01T10:00:00Z",
        "last_used": "2023-09-11T10:30:00Z",
        "expires_at": "2024-12-31T23:59:59Z"
      }
    ]
  }
}
```

### Revoke API Key

```bash
curl -X DELETE http://localhost:8080/api/v1/api-keys/key_123456 \
     -H "Authorization: Bearer your-token"
```

## Permissions and Scopes

### Available Permissions

| Permission | Description |
|------------|-------------|
| `proxy.read` | Read proxy configurations |
| `proxy.write` | Create/update proxy configurations |
| `proxy.delete` | Delete proxy configurations |
| `metrics.read` | Access metrics and statistics |
| `users.read` | List and view users |
| `users.write` | Create/update users |
| `users.delete` | Delete users |
| `settings.read` | View system settings |
| `settings.write` | Modify system settings |
| `admin` | Full administrative access |

### Scope-based Access

```json
{
  "name": "Limited Access Key",
  "permissions": [
    "proxy.read",
    "metrics.read"
  ],
  "scope": {
    "proxies": ["proxy_123", "proxy_456"],
    "ip_whitelist": ["192.168.1.0/24"]
  }
}
```

## Security Best Practices

### API Key Security

1. **Never expose API keys in client-side code**
2. **Use environment variables for API keys**
3. **Rotate API keys regularly**
4. **Use least privilege principle**
5. **Monitor API key usage**

```bash
# Good: Use environment variables
export RELYR_API_KEY=your-api-key
relyr --api-key $RELYR_API_KEY proxy list

# Bad: Hard-coded in scripts
relyr --api-key rlyr_1234567890abcdef proxy list
```

### Token Security

1. **Store tokens securely**
2. **Use HTTPS in production**
3. **Implement proper token refresh logic**
4. **Set appropriate token expiration times**

### Network Security

1. **Use HTTPS for all API calls**
2. **Implement IP whitelisting**
3. **Use VPN for sensitive environments**
4. **Enable request logging**

```yaml
# Security configuration
security:
  https_only: true
  ip_whitelist:
    - "192.168.1.0/24"
    - "10.0.0.0/8"
  request_logging: true
  rate_limiting:
    enabled: true
    requests_per_minute: 100
```

## Error Handling

### Authentication Errors

Common authentication error responses:

#### Invalid API Key

```json
{
  "success": false,
  "error": {
    "code": "INVALID_API_KEY",
    "message": "The provided API key is invalid or expired",
    "details": "API key 'rlyr_1234...' not found or revoked"
  }
}
```

#### Expired Token

```json
{
  "success": false,
  "error": {
    "code": "TOKEN_EXPIRED",
    "message": "The access token has expired",
    "details": "Token expired at 2023-09-11T10:00:00Z"
  }
}
```

#### Insufficient Permissions

```json
{
  "success": false,
  "error": {
    "code": "INSUFFICIENT_PERMISSIONS",
    "message": "Insufficient permissions to access this resource",
    "details": "Required permission: proxy.write"
  }
}
```

## SDK Examples

### Python SDK

```python
from relyr import RelyrClient
from relyr.auth import APIKeyAuth, BearerTokenAuth

# API Key authentication
client = RelyrClient(
    base_url='http://localhost:8080',
    auth=APIKeyAuth('your-api-key')
)

# Bearer token authentication
client = RelyrClient(
    base_url='http://localhost:8080',
    auth=BearerTokenAuth('your-bearer-token')
)

# Basic authentication
from requests.auth import HTTPBasicAuth
client = RelyrClient(
    base_url='http://localhost:8080',
    auth=HTTPBasicAuth('username', 'password')
)
```

### Node.js SDK

```javascript
const { RelyrClient } = require('relyr-node');

// API Key authentication
const client = new RelyrClient({
  baseURL: 'http://localhost:8080',
  apiKey: 'your-api-key'
});

// Bearer token authentication
const client = new RelyrClient({
  baseURL: 'http://localhost:8080',
  bearerToken: 'your-bearer-token'
});

// Basic authentication
const client = new RelyrClient({
  baseURL: 'http://localhost:8080',
  username: 'admin',
  password: 'password'
});
```

## Testing Authentication

### Test API Key

```bash
# Test API key validity
curl -H "X-API-Key: your-api-key" \
     http://localhost:8080/api/v1/auth/test

# Expected response for valid key:
{
  "success": true,
  "data": {
    "authenticated": true,
    "key_id": "key_123456",
    "permissions": ["proxy.read", "proxy.write"]
  }
}
```

### Test Bearer Token

```bash
# Test bearer token validity
curl -H "Authorization: Bearer your-token" \
     http://localhost:8080/api/v1/auth/test
```

## Next Steps

- [API Overview](overview.md) - Learn about the API structure
- [Endpoints](endpoints.md) - Explore available endpoints
- [Examples](../examples/basic.md) - See authentication in action
