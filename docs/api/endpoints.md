# API Endpoints Reference

Complete reference for all Relyr API endpoints with request/response examples.

## Base URL

```
http://localhost:8080/api/v1
```

## Authentication

All endpoints require authentication. See [Authentication](authentication.md) for details.

## Proxies

Manage proxy server configurations.

### List Proxies

List all configured proxy servers.

```http
GET /api/v1/proxies
```

**Query Parameters:**

| Parameter | Type | Description | Default |
|-----------|------|-------------|---------|
| `page` | integer | Page number | 1 |
| `per_page` | integer | Items per page (1-100) | 50 |
| `status` | string | Filter by status (`active`, `inactive`) | all |
| `protocol` | string | Filter by protocol (`http`, `https`, `socks4`, `socks5`) | all |

**Example Request:**

```bash
curl -H "X-API-Key: your-api-key" \
     "http://localhost:8080/api/v1/proxies?page=1&per_page=20&status=active"
```

**Example Response:**

```json
{
  "success": true,
  "data": {
    "proxies": [
      {
        "id": "proxy_123",
        "name": "Main HTTP Proxy",
        "host": "proxy.example.com",
        "port": 8080,
        "protocol": "http",
        "status": "active",
        "auth_enabled": true,
        "created_at": "2023-09-01T10:00:00Z",
        "updated_at": "2023-09-10T15:30:00Z",
        "stats": {
          "total_requests": 15847,
          "active_connections": 42,
          "uptime": "9d 12h 30m"
        }
      }
    ],
    "pagination": {
      "page": 1,
      "per_page": 20,
      "total": 1,
      "pages": 1
    }
  }
}
```

### Create Proxy

Create a new proxy configuration.

```http
POST /api/v1/proxies
```

**Request Body:**

```json
{
  "name": "New Proxy Server",
  "host": "proxy.example.com",
  "port": 8080,
  "protocol": "http",
  "auth": {
    "enabled": true,
    "username": "proxyuser",
    "password": "securepass"
  },
  "settings": {
    "timeout": 30,
    "keep_alive": true,
    "max_connections": 100
  },
  "tags": ["production", "us-east"]
}
```

**Example Response:**

```json
{
  "success": true,
  "data": {
    "proxy": {
      "id": "proxy_456",
      "name": "New Proxy Server",
      "host": "proxy.example.com",
      "port": 8080,
      "protocol": "http",
      "status": "active",
      "created_at": "2023-09-11T10:30:00Z"
    }
  }
}
```

### Get Proxy

Retrieve details of a specific proxy.

```http
GET /api/v1/proxies/{proxy_id}
```

**Example Response:**

```json
{
  "success": true,
  "data": {
    "proxy": {
      "id": "proxy_123",
      "name": "Main HTTP Proxy",
      "host": "proxy.example.com",
      "port": 8080,
      "protocol": "http",
      "status": "active",
      "auth": {
        "enabled": true,
        "username": "proxyuser"
      },
      "settings": {
        "timeout": 30,
        "keep_alive": true,
        "max_connections": 100
      },
      "stats": {
        "total_requests": 15847,
        "active_connections": 42,
        "bytes_transferred": "2.5GB",
        "uptime": "9d 12h 30m",
        "last_request": "2023-09-11T10:29:45Z"
      },
      "created_at": "2023-09-01T10:00:00Z",
      "updated_at": "2023-09-10T15:30:00Z"
    }
  }
}
```

### Update Proxy

Update an existing proxy configuration.

```http
PUT /api/v1/proxies/{proxy_id}
```

**Request Body:**

```json
{
  "name": "Updated Proxy Name",
  "settings": {
    "timeout": 45,
    "max_connections": 150
  },
  "tags": ["production", "us-east", "high-performance"]
}
```

### Delete Proxy

Delete a proxy configuration.

```http
DELETE /api/v1/proxies/{proxy_id}
```

**Example Response:**

```json
{
  "success": true,
  "data": {
    "message": "Proxy proxy_123 deleted successfully"
  }
}
```

### Proxy Actions

#### Start Proxy

```http
POST /api/v1/proxies/{proxy_id}/start
```

#### Stop Proxy

```http
POST /api/v1/proxies/{proxy_id}/stop
```

#### Restart Proxy

```http
POST /api/v1/proxies/{proxy_id}/restart
```

#### Test Proxy

```http
POST /api/v1/proxies/{proxy_id}/test
```

**Request Body:**

```json
{
  "target_url": "https://httpbin.org/ip",
  "timeout": 10
}
```

**Example Response:**

```json
{
  "success": true,
  "data": {
    "test_result": {
      "status": "success",
      "response_time": 245,
      "status_code": 200,
      "external_ip": "203.0.113.45",
      "timestamp": "2023-09-11T10:30:00Z"
    }
  }
}
```

## Health & Status

Monitor system health and status.

### Health Check

```http
GET /api/v1/health
```

**Example Response:**

```json
{
  "success": true,
  "data": {
    "status": "healthy",
    "timestamp": "2023-09-11T10:30:00Z",
    "uptime": "9d 12h 30m 45s",
    "version": "1.0.0",
    "checks": {
      "database": {
        "status": "healthy",
        "response_time": "2ms"
      },
      "proxy_service": {
        "status": "healthy",
        "active_proxies": 5
      },
      "memory": {
        "status": "healthy",
        "usage": "45%",
        "total": "8GB",
        "used": "3.6GB"
      },
      "disk": {
        "status": "healthy",
        "usage": "25%",
        "free": "75GB"
      }
    }
  }
}
```

### System Status

```http
GET /api/v1/status
```

**Example Response:**

```json
{
  "success": true,
  "data": {
    "server": {
      "status": "running",
      "uptime": 788745,
      "pid": 12345,
      "memory": {
        "rss": "256MB",
        "heap_used": "128MB",
        "heap_total": "200MB"
      },
      "cpu": {
        "usage": "5.2%",
        "load_average": [0.5, 0.8, 1.2]
      }
    },
    "proxy": {
      "active_proxies": 5,
      "total_connections": 142,
      "requests_per_second": 12.5,
      "total_requests": 158470,
      "error_rate": "0.02%"
    },
    "network": {
      "bandwidth_in": "125KB/s",
      "bandwidth_out": "89KB/s",
      "total_bytes_in": "2.5GB",
      "total_bytes_out": "1.8GB"
    }
  }
}
```

## Metrics

Access detailed performance metrics.

### Get Metrics

```http
GET /api/v1/metrics
```

**Query Parameters:**

| Parameter | Type | Description | Default |
|-----------|------|-------------|---------|
| `start_time` | string | ISO 8601 timestamp | 1 hour ago |
| `end_time` | string | ISO 8601 timestamp | now |
| `interval` | string | Aggregation interval (1m, 5m, 1h, 1d) | 5m |
| `metrics` | string | Comma-separated metric names | all |
| `proxy_id` | string | Filter by specific proxy | all |

**Example Request:**

```bash
curl -H "X-API-Key: your-api-key" \
     "http://localhost:8080/api/v1/metrics?start_time=2023-09-11T09:00:00Z&end_time=2023-09-11T10:00:00Z&interval=5m&metrics=requests_per_second,response_time_avg"
```

**Example Response:**

```json
{
  "success": true,
  "data": {
    "time_range": {
      "start": "2023-09-11T09:00:00Z",
      "end": "2023-09-11T10:00:00Z",
      "interval": "5m"
    },
    "metrics": {
      "requests_per_second": [
        {"timestamp": "2023-09-11T09:00:00Z", "value": 10.2},
        {"timestamp": "2023-09-11T09:05:00Z", "value": 12.5},
        {"timestamp": "2023-09-11T09:10:00Z", "value": 11.8}
      ],
      "response_time_avg": [
        {"timestamp": "2023-09-11T09:00:00Z", "value": 150},
        {"timestamp": "2023-09-11T09:05:00Z", "value": 145},
        {"timestamp": "2023-09-11T09:10:00Z", "value": 152}
      ]
    }
  }
}
```

### Real-time Metrics

```http
GET /api/v1/metrics/realtime
```

**Example Response:**

```json
{
  "success": true,
  "data": {
    "timestamp": "2023-09-11T10:30:00Z",
    "metrics": {
      "requests_per_second": 12.5,
      "active_connections": 42,
      "response_time_avg": 145,
      "error_rate": 0.02,
      "cpu_usage": 5.2,
      "memory_usage": 45.8,
      "bandwidth_in": 128000,
      "bandwidth_out": 91000
    }
  }
}
```

## Users

Manage user accounts and permissions.

### List Users

```http
GET /api/v1/users
```

**Example Response:**

```json
{
  "success": true,
  "data": {
    "users": [
      {
        "id": "user_123",
        "username": "admin",
        "email": "admin@example.com",
        "role": "administrator",
        "status": "active",
        "permissions": ["*"],
        "last_login": "2023-09-11T10:15:00Z",
        "created_at": "2023-09-01T10:00:00Z"
      }
    ],
    "total": 1
  }
}
```

### Create User

```http
POST /api/v1/users
```

**Request Body:**

```json
{
  "username": "newuser",
  "email": "user@example.com",
  "password": "securepassword",
  "role": "user",
  "permissions": ["proxy.read", "proxy.write", "metrics.read"]
}
```

### Update User

```http
PUT /api/v1/users/{user_id}
```

### Delete User

```http
DELETE /api/v1/users/{user_id}
```

## Settings

Manage system configuration.

### Get Settings

```http
GET /api/v1/settings
```

**Example Response:**

```json
{
  "success": true,
  "data": {
    "server": {
      "host": "0.0.0.0",
      "port": 8080,
      "ssl_enabled": false,
      "max_connections": 1000
    },
    "logging": {
      "level": "info",
      "format": "json",
      "output": "stdout"
    },
    "auth": {
      "enabled": true,
      "session_timeout": 3600,
      "max_login_attempts": 5
    },
    "proxy": {
      "default_timeout": 30,
      "keep_alive_timeout": 60,
      "max_idle_connections": 100
    }
  }
}
```

### Update Settings

```http
PUT /api/v1/settings
```

**Request Body:**

```json
{
  "logging": {
    "level": "debug"
  },
  "proxy": {
    "default_timeout": 45
  }
}
```

## Logs

Access and search system logs.

### Get Logs

```http
GET /api/v1/logs
```

**Query Parameters:**

| Parameter | Type | Description | Default |
|-----------|------|-------------|---------|
| `level` | string | Log level filter | all |
| `start_time` | string | ISO 8601 timestamp | 1 hour ago |
| `end_time` | string | ISO 8601 timestamp | now |
| `limit` | integer | Maximum number of entries | 100 |
| `search` | string | Search term | - |

**Example Response:**

```json
{
  "success": true,
  "data": {
    "logs": [
      {
        "timestamp": "2023-09-11T10:30:00Z",
        "level": "info",
        "message": "Proxy request completed",
        "proxy_id": "proxy_123",
        "client_ip": "192.168.1.100",
        "target_url": "https://api.example.com",
        "response_time": 145,
        "status_code": 200
      }
    ],
    "total": 1,
    "has_more": false
  }
}
```

## WebHooks

Manage webhook configurations for real-time notifications.

### List Webhooks

```http
GET /api/v1/webhooks
```

### Create Webhook

```http
POST /api/v1/webhooks
```

**Request Body:**

```json
{
  "name": "Slack Notifications",
  "url": "https://hooks.slack.com/services/...",
  "events": ["proxy.down", "proxy.error", "high_error_rate"],
  "headers": {
    "Content-Type": "application/json"
  },
  "enabled": true
}
```

### Test Webhook

```http
POST /api/v1/webhooks/{webhook_id}/test
```

## Error Responses

All endpoints may return these common error responses:

### 400 Bad Request

```json
{
  "success": false,
  "error": {
    "code": "INVALID_REQUEST",
    "message": "Request validation failed",
    "details": {
      "field": "port",
      "error": "must be between 1 and 65535"
    }
  }
}
```

### 401 Unauthorized

```json
{
  "success": false,
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Authentication required",
    "details": "Missing or invalid API key"
  }
}
```

### 403 Forbidden

```json
{
  "success": false,
  "error": {
    "code": "FORBIDDEN",
    "message": "Insufficient permissions",
    "details": "Required permission: proxy.write"
  }
}
```

### 404 Not Found

```json
{
  "success": false,
  "error": {
    "code": "NOT_FOUND",
    "message": "Resource not found",
    "details": "Proxy with ID 'proxy_999' does not exist"
  }
}
```

### 429 Too Many Requests

```json
{
  "success": false,
  "error": {
    "code": "RATE_LIMITED",
    "message": "Rate limit exceeded",
    "details": "Maximum 1000 requests per hour exceeded"
  }
}
```

### 500 Internal Server Error

```json
{
  "success": false,
  "error": {
    "code": "INTERNAL_ERROR",
    "message": "An internal server error occurred",
    "details": "Please contact support with request ID: req_123456789"
  }
}
```

## Rate Limiting

All endpoints are subject to rate limiting:

- **Default**: 1000 requests per hour
- **Burst**: 100 requests per minute
- **Headers**: Rate limit info in response headers

```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1631361600
Retry-After: 3600
```

## Pagination

List endpoints support pagination:

```json
{
  "pagination": {
    "page": 1,
    "per_page": 50,
    "total": 150,
    "pages": 3,
    "has_next": true,
    "has_prev": false
  }
}
```

## Next Steps

- [Authentication](authentication.md) - Learn about API authentication
- [API Overview](overview.md) - Understand the API structure
- [Examples](../examples/basic.md) - See practical usage examples
