# API Overview

Relyr provides a comprehensive REST API for managing proxy configurations, monitoring performance, and integrating with your applications.

## Base URL

```
https://api.relyr.io/v1
```

For local installations:

```
http://localhost:8080/api/v1
```

## Authentication

All API requests require authentication. Relyr supports multiple authentication methods:

- **API Key**: Include `X-API-Key` header
- **Bearer Token**: Include `Authorization: Bearer <token>` header
- **Basic Auth**: Include `Authorization: Basic <base64>` header

See [Authentication](authentication.md) for detailed information.

## Response Format

All API responses follow a consistent JSON format:

### Success Response

```json
{
  "success": true,
  "data": {
    // Response data here
  },
  "meta": {
    "timestamp": "2023-09-11T10:30:00Z",
    "version": "1.0.0",
    "request_id": "req_123456789"
  }
}
```

### Error Response

```json
{
  "success": false,
  "error": {
    "code": "INVALID_REQUEST",
    "message": "The request is invalid",
    "details": "Missing required field: proxy_host"
  },
  "meta": {
    "timestamp": "2023-09-11T10:30:00Z",
    "version": "1.0.0",
    "request_id": "req_123456789"
  }
}
```

## Rate Limiting

API requests are rate limited to prevent abuse:

- **Default**: 1000 requests per hour per API key
- **Burst**: 100 requests per minute
- **Headers**: Rate limit information is included in response headers

```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1631361600
```

## API Endpoints

### Core Resources

| Resource | Description |
|----------|-------------|
| [Proxies](#proxies) | Manage proxy configurations |
| [Health](#health) | Server health and status |
| [Metrics](#metrics) | Performance and usage metrics |
| [Users](#users) | User and authentication management |
| [Settings](#settings) | Server configuration |

### Proxies

Manage proxy server configurations and routing rules.

#### List Proxies

```http
GET /api/v1/proxies
```

**Response:**

```json
{
  "success": true,
  "data": {
    "proxies": [
      {
        "id": "proxy_123",
        "name": "Main Proxy",
        "host": "proxy.example.com",
        "port": 8080,
        "protocol": "http",
        "status": "active",
        "created_at": "2023-09-01T10:00:00Z"
      }
    ],
    "total": 1,
    "page": 1,
    "per_page": 50
  }
}
```

#### Create Proxy

```http
POST /api/v1/proxies
```

**Request Body:**

```json
{
  "name": "New Proxy",
  "host": "proxy.example.com",
  "port": 8080,
  "protocol": "http",
  "auth": {
    "enabled": true,
    "username": "user",
    "password": "pass"
  }
}
```

#### Get Proxy Details

```http
GET /api/v1/proxies/{proxy_id}
```

#### Update Proxy

```http
PUT /api/v1/proxies/{proxy_id}
```

#### Delete Proxy

```http
DELETE /api/v1/proxies/{proxy_id}
```

### Health

Monitor server health and status.

#### Health Check

```http
GET /api/v1/health
```

**Response:**

```json
{
  "success": true,
  "data": {
    "status": "healthy",
    "uptime": "2h 30m 45s",
    "version": "1.0.0",
    "timestamp": "2023-09-11T10:30:00Z",
    "checks": {
      "database": "healthy",
      "proxy": "healthy",
      "memory": "healthy"
    }
  }
}
```

#### System Status

```http
GET /api/v1/status
```

**Response:**

```json
{
  "success": true,
  "data": {
    "server": {
      "status": "running",
      "uptime": 9045,
      "memory_usage": "256MB",
      "cpu_usage": "5.2%"
    },
    "proxy": {
      "active_connections": 42,
      "total_requests": 15847,
      "requests_per_second": 12.5
    }
  }
}
```

### Metrics

Access performance and usage metrics.

#### Get Metrics

```http
GET /api/v1/metrics
```

**Query Parameters:**

- `start_time`: ISO 8601 timestamp (default: 1 hour ago)
- `end_time`: ISO 8601 timestamp (default: now)
- `interval`: Aggregation interval (1m, 5m, 1h, 1d)
- `metrics`: Comma-separated list of metrics

**Response:**

```json
{
  "success": true,
  "data": {
    "time_range": {
      "start": "2023-09-11T09:30:00Z",
      "end": "2023-09-11T10:30:00Z"
    },
    "metrics": {
      "requests_per_second": [
        {"timestamp": "2023-09-11T09:30:00Z", "value": 10.2},
        {"timestamp": "2023-09-11T09:35:00Z", "value": 12.5}
      ],
      "response_time_avg": [
        {"timestamp": "2023-09-11T09:30:00Z", "value": 150},
        {"timestamp": "2023-09-11T09:35:00Z", "value": 145}
      ]
    }
  }
}
```

#### Available Metrics

| Metric | Description | Unit |
|--------|-------------|------|
| `requests_per_second` | Request rate | requests/sec |
| `response_time_avg` | Average response time | milliseconds |
| `response_time_p95` | 95th percentile response time | milliseconds |
| `active_connections` | Current active connections | count |
| `error_rate` | Error percentage | percentage |
| `bandwidth_in` | Incoming bandwidth | bytes/sec |
| `bandwidth_out` | Outgoing bandwidth | bytes/sec |

### Users

Manage user accounts and authentication.

#### List Users

```http
GET /api/v1/users
```

#### Create User

```http
POST /api/v1/users
```

**Request Body:**

```json
{
  "username": "newuser",
  "password": "securepassword",
  "email": "user@example.com",
  "role": "user",
  "permissions": ["proxy.read", "proxy.write"]
}
```

#### Update User

```http
PUT /api/v1/users/{user_id}
```

#### Delete User

```http
DELETE /api/v1/users/{user_id}
```

### Settings

Manage server configuration.

#### Get Configuration

```http
GET /api/v1/settings
```

#### Update Configuration

```http
PUT /api/v1/settings
```

**Request Body:**

```json
{
  "server": {
    "port": 8080,
    "host": "0.0.0.0"
  },
  "logging": {
    "level": "info"
  },
  "proxy": {
    "timeout": 30
  }
}
```

## WebSocket API

For real-time updates, Relyr provides WebSocket endpoints:

### Connection

```javascript
const ws = new WebSocket('ws://localhost:8080/api/v1/ws');

// Authentication
ws.send(JSON.stringify({
  type: 'auth',
  token: 'your-api-token'
}));
```

### Real-time Metrics

```javascript
// Subscribe to metrics
ws.send(JSON.stringify({
  type: 'subscribe',
  channel: 'metrics',
  metrics: ['requests_per_second', 'active_connections']
}));

// Receive updates
ws.onmessage = function(event) {
  const data = JSON.parse(event.data);
  console.log('Metrics update:', data);
};
```

### Proxy Events

```javascript
// Subscribe to proxy events
ws.send(JSON.stringify({
  type: 'subscribe',
  channel: 'proxy_events'
}));

// Receive events
ws.onmessage = function(event) {
  const data = JSON.parse(event.data);
  if (data.type === 'proxy_event') {
    console.log('Proxy event:', data.event);
  }
};
```

## Error Codes

Common API error codes:

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `INVALID_REQUEST` | 400 | Request validation failed |
| `UNAUTHORIZED` | 401 | Authentication required |
| `FORBIDDEN` | 403 | Insufficient permissions |
| `NOT_FOUND` | 404 | Resource not found |
| `CONFLICT` | 409 | Resource conflict |
| `RATE_LIMITED` | 429 | Rate limit exceeded |
| `INTERNAL_ERROR` | 500 | Internal server error |
| `SERVICE_UNAVAILABLE` | 503 | Service temporarily unavailable |

## SDK and Libraries

Official SDKs are available for popular programming languages:

- **Python**: `pip install relyr-python`
- **Node.js**: `npm install relyr-node`
- **Go**: `go get github.com/relyr/relyr-go`
- **Java**: Maven/Gradle dependency available
- **PHP**: `composer require relyr/relyr-php`

### Python Example

```python
from relyr import RelyrClient

client = RelyrClient(
    api_key='your-api-key',
    base_url='http://localhost:8080'
)

# List proxies
proxies = client.proxies.list()

# Create a new proxy
proxy = client.proxies.create({
    'name': 'My Proxy',
    'host': 'proxy.example.com',
    'port': 8080
})

# Get metrics
metrics = client.metrics.get(
    start_time='2023-09-11T09:00:00Z',
    metrics=['requests_per_second', 'response_time_avg']
)
```

## OpenAPI Specification

The complete API specification is available in OpenAPI 3.0 format:

- **JSON**: `/api/v1/openapi.json`
- **YAML**: `/api/v1/openapi.yaml`
- **Interactive Docs**: `/api/v1/docs`

## Next Steps

- [Authentication](authentication.md) - Learn about API authentication
- [Endpoints](endpoints.md) - Complete endpoint reference
- [Examples](../examples/basic.md) - See practical API usage examples
