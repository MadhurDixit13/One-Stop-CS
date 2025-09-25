# CORS - Cross-Origin Resource Sharing

CORS (Cross-Origin Resource Sharing) is a security feature implemented by web browsers that controls how web pages from one domain can access resources from another domain. It's a crucial concept for modern web development, especially when building APIs and SPAs.

---

## What is CORS?

CORS is a mechanism that allows web servers to specify which origins (domains, protocols, or ports) are permitted to access their resources. It's enforced by browsers to prevent malicious websites from making unauthorized requests to other domains.

### Same-Origin Policy
Browsers enforce the **Same-Origin Policy** by default:
- **Same Origin**: Same protocol, domain, and port
- **Cross-Origin**: Different protocol, domain, or port

```javascript
// Same origin examples
https://example.com/page1 → https://example.com/page2 ✅
http://example.com → https://example.com ❌ (different protocol)
https://example.com → https://api.example.com ❌ (different subdomain)
https://example.com:3000 → https://example.com:8080 ❌ (different port)
```

---

## How CORS Works

### Simple Requests
For simple requests (GET, HEAD, POST with certain content types), browsers send the request directly and check the response headers.

```http
GET /api/users HTTP/1.1
Host: api.example.com
Origin: https://myapp.com
```

```http
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://myapp.com
Content-Type: application/json

{"users": [...]}
```

### Preflight Requests
For complex requests, browsers send a preflight OPTIONS request first:

```http
OPTIONS /api/users HTTP/1.1
Host: api.example.com
Origin: https://myapp.com
Access-Control-Request-Method: POST
Access-Control-Request-Headers: Content-Type, Authorization
```

```http
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://myapp.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Max-Age: 86400
```

---

## CORS Headers

### Server Response Headers
```http
# Allow specific origin
Access-Control-Allow-Origin: https://myapp.com

# Allow any origin (use with caution)
Access-Control-Allow-Origin: *

# Allow multiple methods
Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS

# Allow specific headers
Access-Control-Allow-Headers: Content-Type, Authorization, X-Requested-With

# Allow credentials (cookies, authorization headers)
Access-Control-Allow-Credentials: true

# Cache preflight response
Access-Control-Max-Age: 86400

# Expose headers to client
Access-Control-Expose-Headers: X-Total-Count, X-Page-Count
```

### Client Request Headers
```http
# Origin of the request
Origin: https://myapp.com

# Requested method (preflight)
Access-Control-Request-Method: POST

# Requested headers (preflight)
Access-Control-Request-Headers: Content-Type, Authorization
```

---

## Server Implementation

### Express.js (Node.js)
```javascript
import express from 'express';
import cors from 'cors';

const app = express();

// Basic CORS - allow all origins
app.use(cors());

// Specific origin
app.use(cors({
  origin: 'https://myapp.com'
}));

// Multiple origins
app.use(cors({
  origin: ['https://myapp.com', 'https://admin.myapp.com']
}));

// Dynamic origin function
app.use(cors({
  origin: function (origin, callback) {
    // Allow requests with no origin (mobile apps, Postman, etc.)
    if (!origin) return callback(null, true);
    
    const allowedOrigins = [
      'https://myapp.com',
      'https://admin.myapp.com',
      'http://localhost:3000' // Development
    ];
    
    if (allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  },
  credentials: true, // Allow cookies
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS'],
  allowedHeaders: ['Content-Type', 'Authorization', 'X-Requested-With'],
  exposedHeaders: ['X-Total-Count'],
  maxAge: 86400 // Cache preflight for 24 hours
}));

// Manual CORS implementation
app.use((req, res, next) => {
  const origin = req.headers.origin;
  const allowedOrigins = ['https://myapp.com', 'http://localhost:3000'];
  
  if (allowedOrigins.includes(origin)) {
    res.header('Access-Control-Allow-Origin', origin);
  }
  
  res.header('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, OPTIONS');
  res.header('Access-Control-Allow-Headers', 'Content-Type, Authorization, X-Requested-With');
  res.header('Access-Control-Allow-Credentials', 'true');
  res.header('Access-Control-Max-Age', '86400');
  
  // Handle preflight requests
  if (req.method === 'OPTIONS') {
    res.sendStatus(200);
  } else {
    next();
  }
});

app.listen(3000);
```

### Python Flask
```python
from flask import Flask, jsonify
from flask_cors import CORS

app = Flask(__name__)

# Basic CORS
CORS(app)

# Specific configuration
CORS(app, 
     origins=['https://myapp.com', 'http://localhost:3000'],
     methods=['GET', 'POST', 'PUT', 'DELETE'],
     allow_headers=['Content-Type', 'Authorization'],
     supports_credentials=True)

# Manual CORS implementation
@app.after_request
def after_request(response):
    origin = request.headers.get('Origin')
    allowed_origins = ['https://myapp.com', 'http://localhost:3000']
    
    if origin in allowed_origins:
        response.headers['Access-Control-Allow-Origin'] = origin
    
    response.headers['Access-Control-Allow-Methods'] = 'GET, POST, PUT, DELETE, OPTIONS'
    response.headers['Access-Control-Allow-Headers'] = 'Content-Type, Authorization'
    response.headers['Access-Control-Allow-Credentials'] = 'true'
    response.headers['Access-Control-Max-Age'] = '86400'
    
    return response

@app.route('/api/users', methods=['GET', 'OPTIONS'])
def get_users():
    if request.method == 'OPTIONS':
        return '', 200
    
    return jsonify({'users': []})

if __name__ == '__main__':
    app.run(debug=True)
```

### Python FastAPI
```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

# Add CORS middleware
app.add_middleware(
    CORSMiddleware,
    allow_origins=[
        "https://myapp.com",
        "http://localhost:3000",
        "http://localhost:8080",
    ],
    allow_credentials=True,
    allow_methods=["GET", "POST", "PUT", "DELETE"],
    allow_headers=["*"],
    max_age=86400,
)

# Or with dynamic origins
def is_origin_allowed(origin: str) -> bool:
    allowed_origins = [
        "https://myapp.com",
        "https://admin.myapp.com",
        "http://localhost:3000"
    ]
    return origin in allowed_origins

app.add_middleware(
    CORSMiddleware,
    allow_origin_regex=r"https://.*\.myapp\.com",
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

@app.get("/api/users")
async def get_users():
    return {"users": []}
```

### Django
```python
# settings.py
CORS_ALLOWED_ORIGINS = [
    "https://myapp.com",
    "http://localhost:3000",
]

CORS_ALLOW_CREDENTIALS = True

CORS_ALLOW_ALL_ORIGINS = False  # Only for development

CORS_ALLOWED_HEADERS = [
    'accept',
    'accept-encoding',
    'authorization',
    'content-type',
    'dnt',
    'origin',
    'user-agent',
    'x-csrftoken',
    'x-requested-with',
]

# Custom CORS middleware
class CustomCorsMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):
        response = self.get_response(request)
        
        origin = request.META.get('HTTP_ORIGIN')
        allowed_origins = ['https://myapp.com', 'http://localhost:3000']
        
        if origin in allowed_origins:
            response['Access-Control-Allow-Origin'] = origin
        
        response['Access-Control-Allow-Methods'] = 'GET, POST, PUT, DELETE, OPTIONS'
        response['Access-Control-Allow-Headers'] = 'Content-Type, Authorization'
        response['Access-Control-Allow-Credentials'] = 'true'
        
        return response
```

---

## Client Implementation

### JavaScript Fetch
```javascript
// Basic fetch with CORS
fetch('https://api.example.com/users', {
  method: 'GET',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': 'Bearer token'
  },
  credentials: 'include' // Include cookies
})
.then(response => response.json())
.then(data => console.log(data))
.catch(error => console.error('CORS Error:', error));

// Handle CORS errors
async function fetchWithCORS(url, options = {}) {
  try {
    const response = await fetch(url, {
      ...options,
      credentials: 'include',
      headers: {
        'Content-Type': 'application/json',
        ...options.headers
      }
    });
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    return await response.json();
  } catch (error) {
    if (error.name === 'TypeError' && error.message.includes('CORS')) {
      console.error('CORS Error: Request blocked by browser');
      throw new Error('CORS policy prevents this request');
    }
    throw error;
  }
}
```

### Axios
```javascript
import axios from 'axios';

// Configure axios with CORS
const api = axios.create({
  baseURL: 'https://api.example.com',
  withCredentials: true, // Include cookies
  headers: {
    'Content-Type': 'application/json'
  }
});

// Request interceptor
api.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem('token');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => Promise.reject(error)
);

// Response interceptor for CORS errors
api.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.code === 'ERR_NETWORK' && error.message.includes('CORS')) {
      console.error('CORS Error: Request blocked');
      throw new Error('CORS policy prevents this request');
    }
    return Promise.reject(error);
  }
);

// Usage
api.get('/users')
  .then(response => console.log(response.data))
  .catch(error => console.error(error));
```

### React Hook
```javascript
import { useState, useEffect } from 'react';

function useApi(url, options = {}) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchData = async () => {
      try {
        setLoading(true);
        setError(null);
        
        const response = await fetch(url, {
          ...options,
          credentials: 'include',
          headers: {
            'Content-Type': 'application/json',
            ...options.headers
          }
        });

        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }

        const result = await response.json();
        setData(result);
      } catch (err) {
        if (err.name === 'TypeError' && err.message.includes('CORS')) {
          setError('CORS policy prevents this request');
        } else {
          setError(err.message);
        }
      } finally {
        setLoading(false);
      }
    };

    fetchData();
  }, [url, JSON.stringify(options)]);

  return { data, loading, error };
}

// Usage
function UsersList() {
  const { data, loading, error } = useApi('https://api.example.com/users');

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <div>
      {data?.users?.map(user => (
        <div key={user.id}>{user.name}</div>
      ))}
    </div>
  );
}
```

---

## Common CORS Issues & Solutions

### Issue 1: "Access to fetch at 'X' from origin 'Y' has been blocked by CORS policy"
**Solution**: Configure server to allow the requesting origin
```javascript
// Server-side fix
app.use(cors({
  origin: 'https://myapp.com' // Add the blocked origin
}));
```

### Issue 2: "Response to preflight request doesn't pass access control check"
**Solution**: Handle OPTIONS requests and set proper headers
```javascript
// Express.js
app.options('*', (req, res) => {
  res.header('Access-Control-Allow-Origin', req.headers.origin);
  res.header('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, OPTIONS');
  res.header('Access-Control-Allow-Headers', 'Content-Type, Authorization');
  res.sendStatus(200);
});
```

### Issue 3: "Credentials flag is 'true', but the 'Access-Control-Allow-Credentials' header is not set"
**Solution**: Set credentials header on server
```javascript
// Server
app.use(cors({
  origin: 'https://myapp.com',
  credentials: true // This sets Access-Control-Allow-Credentials
}));

// Client
fetch(url, {
  credentials: 'include' // This sends credentials
});
```

### Issue 4: "The value of the 'Access-Control-Allow-Origin' header must not be the wildcard '*' when the request's credentials mode is 'include'"
**Solution**: Use specific origins instead of wildcard
```javascript
// Wrong
app.use(cors({
  origin: '*',
  credentials: true // This combination is not allowed
}));

// Correct
app.use(cors({
  origin: ['https://myapp.com', 'https://admin.myapp.com'],
  credentials: true
}));
```

---

## Security Considerations

### Production CORS Configuration
```javascript
// Production-ready CORS configuration
const corsOptions = {
  origin: function (origin, callback) {
    // Allow requests with no origin (mobile apps, Postman, etc.)
    if (!origin) return callback(null, true);
    
    const allowedOrigins = [
      'https://myapp.com',
      'https://www.myapp.com',
      'https://admin.myapp.com'
    ];
    
    // Check if origin is in allowed list
    if (allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      // Log suspicious requests
      console.warn(`CORS blocked request from: ${origin}`);
      callback(new Error('Not allowed by CORS'));
    }
  },
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS'],
  allowedHeaders: [
    'Content-Type',
    'Authorization',
    'X-Requested-With',
    'X-CSRFToken'
  ],
  exposedHeaders: ['X-Total-Count', 'X-Page-Count'],
  maxAge: 86400, // Cache preflight for 24 hours
  optionsSuccessStatus: 200 // Some legacy browsers choke on 204
};
```

### Environment-Based Configuration
```javascript
const isDevelopment = process.env.NODE_ENV === 'development';
const isProduction = process.env.NODE_ENV === 'production';

const corsOptions = {
  origin: isDevelopment 
    ? ['http://localhost:3000', 'http://localhost:8080'] // Development
    : ['https://myapp.com', 'https://www.myapp.com'], // Production
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS'],
  allowedHeaders: ['Content-Type', 'Authorization', 'X-Requested-With'],
  maxAge: isProduction ? 86400 : 0 // Cache in production only
};

app.use(cors(corsOptions));
```

---

## Testing CORS

### Manual Testing
```bash
# Test preflight request
curl -X OPTIONS \
  -H "Origin: https://myapp.com" \
  -H "Access-Control-Request-Method: POST" \
  -H "Access-Control-Request-Headers: Content-Type, Authorization" \
  https://api.example.com/users

# Test actual request
curl -X GET \
  -H "Origin: https://myapp.com" \
  -H "Content-Type: application/json" \
  https://api.example.com/users
```

### Automated Testing
```javascript
import request from 'supertest';
import app from '../app.js';

describe('CORS', () => {
  test('should allow requests from allowed origin', async () => {
    const response = await request(app)
      .get('/api/users')
      .set('Origin', 'https://myapp.com')
      .expect(200);
    
    expect(response.headers['access-control-allow-origin']).toBe('https://myapp.com');
  });
  
  test('should handle preflight requests', async () => {
    const response = await request(app)
      .options('/api/users')
      .set('Origin', 'https://myapp.com')
      .set('Access-Control-Request-Method', 'POST')
      .set('Access-Control-Request-Headers', 'Content-Type, Authorization')
      .expect(200);
    
    expect(response.headers['access-control-allow-methods']).toContain('POST');
    expect(response.headers['access-control-allow-headers']).toContain('Content-Type');
  });
  
  test('should block requests from disallowed origin', async () => {
    const response = await request(app)
      .get('/api/users')
      .set('Origin', 'https://malicious.com')
      .expect(403);
    
    expect(response.headers['access-control-allow-origin']).toBeUndefined();
  });
});
```

---

## Best Practices

### 1. Security
- **Never use wildcard (*) with credentials**: Always specify exact origins
- **Validate origins**: Use functions to validate origins dynamically
- **Log blocked requests**: Monitor for suspicious CORS violations
- **Use HTTPS**: Always use HTTPS in production

### 2. Performance
- **Cache preflight responses**: Set appropriate `Access-Control-Max-Age`
- **Minimize preflight requests**: Use simple requests when possible
- **Optimize headers**: Only expose necessary headers

### 3. Development
- **Environment-specific configs**: Different settings for dev/prod
- **Local development**: Allow localhost origins in development
- **Testing**: Include CORS tests in your test suite

### 4. Error Handling
- **Graceful degradation**: Handle CORS errors in client code
- **User feedback**: Provide clear error messages for CORS issues
- **Fallback strategies**: Consider proxy solutions for development

---

## Common Patterns

### Proxy for Development
```javascript
// webpack.config.js or vite.config.js
module.exports = {
  devServer: {
    proxy: {
      '/api': {
        target: 'http://localhost:3000',
        changeOrigin: true,
        secure: false
      }
    }
  }
};
```

### Nginx Proxy
```nginx
server {
    listen 80;
    server_name api.example.com;
    
    location / {
        add_header 'Access-Control-Allow-Origin' 'https://myapp.com';
        add_header 'Access-Control-Allow-Methods' 'GET, POST, PUT, DELETE, OPTIONS';
        add_header 'Access-Control-Allow-Headers' 'Content-Type, Authorization';
        add_header 'Access-Control-Allow-Credentials' 'true';
        
        if ($request_method = 'OPTIONS') {
            add_header 'Access-Control-Max-Age' 86400;
            add_header 'Content-Type' 'text/plain; charset=utf-8';
            add_header 'Content-Length' 0;
            return 204;
        }
        
        proxy_pass http://backend;
    }
}
```

CORS is essential for modern web applications. Understanding how to properly configure CORS on both server and client sides will help you build secure, cross-origin compatible web applications.
