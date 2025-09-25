# REST APIs - Representational State Transfer

REST (Representational State Transfer) is an architectural style for designing networked applications. It provides a set of constraints and principles for creating web services that are scalable, maintainable, and easy to understand.

---

## What is REST?

REST is an architectural style that defines:
- **Stateless Communication**: Each request contains all necessary information
- **Resource-Based**: Everything is a resource identified by URLs
- **HTTP Methods**: Use standard HTTP verbs (GET, POST, PUT, DELETE)
- **JSON/XML**: Data exchange in standard formats
- **Uniform Interface**: Consistent API design patterns

### Key Principles
- **Client-Server Architecture**: Separation of concerns
- **Stateless**: No client context stored on server
- **Cacheable**: Responses can be cached
- **Uniform Interface**: Consistent interaction patterns
- **Layered System**: Hierarchical layers of servers

---

## HTTP Methods & Status Codes

### HTTP Methods
```http
GET    /api/users          # Retrieve all users
GET    /api/users/123      # Retrieve specific user
POST   /api/users          # Create new user
PUT    /api/users/123      # Update entire user
PATCH  /api/users/123      # Partial update user
DELETE /api/users/123      # Delete user
```

### HTTP Status Codes
```http
# Success (2xx)
200 OK                    # Request successful
201 Created              # Resource created
204 No Content           # Successful, no content returned

# Client Error (4xx)
400 Bad Request          # Invalid request
401 Unauthorized         # Authentication required
403 Forbidden           # Access denied
404 Not Found           # Resource not found
422 Unprocessable Entity # Validation failed

# Server Error (5xx)
500 Internal Server Error # Server error
502 Bad Gateway          # Invalid response from upstream
503 Service Unavailable  # Service temporarily unavailable
```

---

## API Design Best Practices

### URL Structure
```http
# Good - Resource-based URLs
GET    /api/v1/users
GET    /api/v1/users/123
GET    /api/v1/users/123/posts
POST   /api/v1/users/123/posts

# Bad - Action-based URLs
GET    /api/v1/getUsers
POST   /api/v1/createUser
POST   /api/v1/deleteUser
```

### Naming Conventions
```http
# Use nouns, not verbs
GET    /api/users          # Good
GET    /api/getUsers       # Bad

# Use plural nouns
GET    /api/users          # Good
GET    /api/user           # Bad

# Use lowercase with hyphens
GET    /api/user-profiles  # Good
GET    /api/userProfiles   # Bad
```

### Versioning
```http
# URL versioning (recommended)
GET    /api/v1/users
GET    /api/v2/users

# Header versioning
GET    /api/users
Accept: application/vnd.api+json;version=1

# Query parameter versioning
GET    /api/users?version=1
```

---

## Request/Response Formats

### JSON Request/Response
```json
// POST /api/users
{
  "name": "John Doe",
  "email": "john@example.com",
  "age": 30
}

// Response
{
  "id": 123,
  "name": "John Doe",
  "email": "john@example.com",
  "age": 30,
  "created_at": "2024-01-15T10:30:00Z",
  "updated_at": "2024-01-15T10:30:00Z"
}
```

### Error Response Format
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": [
      {
        "field": "email",
        "message": "Email format is invalid"
      }
    ]
  }
}
```

### Pagination
```json
{
  "data": [
    {
      "id": 1,
      "name": "User 1"
    },
    {
      "id": 2,
      "name": "User 2"
    }
  ],
  "pagination": {
    "page": 1,
    "per_page": 10,
    "total": 100,
    "total_pages": 10,
    "has_next": true,
    "has_prev": false
  }
}
```

---

## Server Implementation

### Express.js (Node.js)
```javascript
import express from 'express';
import cors from 'cors';
import helmet from 'helmet';
import rateLimit from 'express-rate-limit';

const app = express();

// Middleware
app.use(helmet());
app.use(cors());
app.use(express.json({ limit: '10mb' }));
app.use(express.urlencoded({ extended: true }));

// Rate limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // limit each IP to 100 requests per windowMs
  message: 'Too many requests from this IP'
});
app.use('/api/', limiter);

// Routes
app.get('/api/v1/users', getUsers);
app.get('/api/v1/users/:id', getUser);
app.post('/api/v1/users', createUser);
app.put('/api/v1/users/:id', updateUser);
app.delete('/api/v1/users/:id', deleteUser);

// Error handling middleware
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({
    error: {
      code: 'INTERNAL_SERVER_ERROR',
      message: 'Something went wrong!'
    }
  });
});

// 404 handler
app.use('*', (req, res) => {
  res.status(404).json({
    error: {
      code: 'NOT_FOUND',
      message: 'Route not found'
    }
  });
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

### Route Handlers
```javascript
import { User } from './models/user.js';

// GET /api/v1/users
export const getUsers = async (req, res) => {
  try {
    const { page = 1, limit = 10, search } = req.query;
    const offset = (page - 1) * limit;
    
    const users = await User.findAndCountAll({
      where: search ? { name: { [Op.like]: `%${search}%` } } : {},
      limit: parseInt(limit),
      offset: parseInt(offset),
      order: [['created_at', 'DESC']]
    });
    
    res.json({
      data: users.rows,
      pagination: {
        page: parseInt(page),
        per_page: parseInt(limit),
        total: users.count,
        total_pages: Math.ceil(users.count / limit),
        has_next: page < Math.ceil(users.count / limit),
        has_prev: page > 1
      }
    });
  } catch (error) {
    res.status(500).json({
      error: {
        code: 'INTERNAL_SERVER_ERROR',
        message: error.message
      }
    });
  }
};

// GET /api/v1/users/:id
export const getUser = async (req, res) => {
  try {
    const { id } = req.params;
    const user = await User.findByPk(id);
    
    if (!user) {
      return res.status(404).json({
        error: {
          code: 'NOT_FOUND',
          message: 'User not found'
        }
      });
    }
    
    res.json({ data: user });
  } catch (error) {
    res.status(500).json({
      error: {
        code: 'INTERNAL_SERVER_ERROR',
        message: error.message
      }
    });
  }
};

// POST /api/v1/users
export const createUser = async (req, res) => {
  try {
    const { name, email, age } = req.body;
    
    // Validation
    if (!name || !email) {
      return res.status(400).json({
        error: {
          code: 'VALIDATION_ERROR',
          message: 'Name and email are required',
          details: [
            { field: 'name', message: !name ? 'Name is required' : null },
            { field: 'email', message: !email ? 'Email is required' : null }
          ].filter(detail => detail.message)
        }
      });
    }
    
    const user = await User.create({ name, email, age });
    res.status(201).json({ data: user });
  } catch (error) {
    if (error.name === 'SequelizeValidationError') {
      return res.status(422).json({
        error: {
          code: 'VALIDATION_ERROR',
          message: 'Validation failed',
          details: error.errors.map(err => ({
            field: err.path,
            message: err.message
          }))
        }
      });
    }
    
    res.status(500).json({
      error: {
        code: 'INTERNAL_SERVER_ERROR',
        message: error.message
      }
    });
  }
};

// PUT /api/v1/users/:id
export const updateUser = async (req, res) => {
  try {
    const { id } = req.params;
    const { name, email, age } = req.body;
    
    const user = await User.findByPk(id);
    if (!user) {
      return res.status(404).json({
        error: {
          code: 'NOT_FOUND',
          message: 'User not found'
        }
      });
    }
    
    await user.update({ name, email, age });
    res.json({ data: user });
  } catch (error) {
    res.status(500).json({
      error: {
        code: 'INTERNAL_SERVER_ERROR',
        message: error.message
      }
    });
  }
};

// DELETE /api/v1/users/:id
export const deleteUser = async (req, res) => {
  try {
    const { id } = req.params;
    const user = await User.findByPk(id);
    
    if (!user) {
      return res.status(404).json({
        error: {
          code: 'NOT_FOUND',
          message: 'User not found'
        }
      });
    }
    
    await user.destroy();
    res.status(204).send();
  } catch (error) {
    res.status(500).json({
      error: {
        code: 'INTERNAL_SERVER_ERROR',
        message: error.message
      }
    });
  }
};
```

### Python (FastAPI)
```python
from fastapi import FastAPI, HTTPException, Depends, Query
from pydantic import BaseModel, EmailStr
from typing import List, Optional
import uvicorn

app = FastAPI(title="User API", version="1.0.0")

class UserCreate(BaseModel):
    name: str
    email: EmailStr
    age: Optional[int] = None

class UserUpdate(BaseModel):
    name: Optional[str] = None
    email: Optional[EmailStr] = None
    age: Optional[int] = None

class User(BaseModel):
    id: int
    name: str
    email: str
    age: Optional[int] = None
    created_at: str
    updated_at: str

# GET /api/v1/users
@app.get("/api/v1/users", response_model=List[User])
async def get_users(
    page: int = Query(1, ge=1),
    limit: int = Query(10, ge=1, le=100),
    search: Optional[str] = None
):
    # Implementation here
    pass

# GET /api/v1/users/{id}
@app.get("/api/v1/users/{user_id}", response_model=User)
async def get_user(user_id: int):
    # Implementation here
    pass

# POST /api/v1/users
@app.post("/api/v1/users", response_model=User, status_code=201)
async def create_user(user: UserCreate):
    # Implementation here
    pass

# PUT /api/v1/users/{id}
@app.put("/api/v1/users/{user_id}", response_model=User)
async def update_user(user_id: int, user: UserUpdate):
    # Implementation here
    pass

# DELETE /api/v1/users/{id}
@app.delete("/api/v1/users/{user_id}", status_code=204)
async def delete_user(user_id: int):
    # Implementation here
    pass

if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

---

## Client Implementation

### JavaScript (Fetch API)
```javascript
class UserAPI {
  constructor(baseURL = 'http://localhost:3000/api/v1') {
    this.baseURL = baseURL;
    this.token = localStorage.getItem('token');
  }

  async request(endpoint, options = {}) {
    const url = `${this.baseURL}${endpoint}`;
    const config = {
      headers: {
        'Content-Type': 'application/json',
        ...(this.token && { Authorization: `Bearer ${this.token}` }),
        ...options.headers,
      },
      ...options,
    };

    if (config.body && typeof config.body === 'object') {
      config.body = JSON.stringify(config.body);
    }

    try {
      const response = await fetch(url, config);
      
      if (!response.ok) {
        const error = await response.json();
        throw new Error(error.error?.message || 'Request failed');
      }

      // Handle 204 No Content
      if (response.status === 204) {
        return null;
      }

      return await response.json();
    } catch (error) {
      console.error('API request failed:', error);
      throw error;
    }
  }

  // GET /users
  async getUsers(params = {}) {
    const queryString = new URLSearchParams(params).toString();
    const endpoint = queryString ? `/users?${queryString}` : '/users';
    return this.request(endpoint);
  }

  // GET /users/:id
  async getUser(id) {
    return this.request(`/users/${id}`);
  }

  // POST /users
  async createUser(userData) {
    return this.request('/users', {
      method: 'POST',
      body: userData,
    });
  }

  // PUT /users/:id
  async updateUser(id, userData) {
    return this.request(`/users/${id}`, {
      method: 'PUT',
      body: userData,
    });
  }

  // PATCH /users/:id
  async patchUser(id, userData) {
    return this.request(`/users/${id}`, {
      method: 'PATCH',
      body: userData,
    });
  }

  // DELETE /users/:id
  async deleteUser(id) {
    return this.request(`/users/${id}`, {
      method: 'DELETE',
    });
  }
}

// Usage
const api = new UserAPI();

// Get all users
const users = await api.getUsers({ page: 1, limit: 10 });

// Get specific user
const user = await api.getUser(123);

// Create user
const newUser = await api.createUser({
  name: 'John Doe',
  email: 'john@example.com',
  age: 30
});

// Update user
const updatedUser = await api.updateUser(123, {
  name: 'Jane Doe',
  email: 'jane@example.com'
});

// Delete user
await api.deleteUser(123);
```

### Axios Implementation
```javascript
import axios from 'axios';

class UserAPI {
  constructor(baseURL = 'http://localhost:3000/api/v1') {
    this.client = axios.create({
      baseURL,
      timeout: 10000,
      headers: {
        'Content-Type': 'application/json',
      },
    });

    // Request interceptor
    this.client.interceptors.request.use(
      (config) => {
        const token = localStorage.getItem('token');
        if (token) {
          config.headers.Authorization = `Bearer ${token}`;
        }
        return config;
      },
      (error) => Promise.reject(error)
    );

    // Response interceptor
    this.client.interceptors.response.use(
      (response) => response.data,
      (error) => {
        if (error.response?.data?.error) {
          throw new Error(error.response.data.error.message);
        }
        throw error;
      }
    );
  }

  async getUsers(params = {}) {
    return this.client.get('/users', { params });
  }

  async getUser(id) {
    return this.client.get(`/users/${id}`);
  }

  async createUser(userData) {
    return this.client.post('/users', userData);
  }

  async updateUser(id, userData) {
    return this.client.put(`/users/${id}`, userData);
  }

  async patchUser(id, userData) {
    return this.client.patch(`/users/${id}`, userData);
  }

  async deleteUser(id) {
    return this.client.delete(`/users/${id}`);
  }
}
```

### React Hook Implementation
```javascript
import { useState, useEffect, useCallback } from 'react';

function useUsers() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  const fetchUsers = useCallback(async (params = {}) => {
    setLoading(true);
    setError(null);
    
    try {
      const api = new UserAPI();
      const response = await api.getUsers(params);
      setUsers(response.data);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  }, []);

  const createUser = useCallback(async (userData) => {
    try {
      const api = new UserAPI();
      const newUser = await api.createUser(userData);
      setUsers(prev => [...prev, newUser.data]);
      return newUser;
    } catch (err) {
      setError(err.message);
      throw err;
    }
  }, []);

  const updateUser = useCallback(async (id, userData) => {
    try {
      const api = new UserAPI();
      const updatedUser = await api.updateUser(id, userData);
      setUsers(prev => prev.map(user => 
        user.id === id ? updatedUser.data : user
      ));
      return updatedUser;
    } catch (err) {
      setError(err.message);
      throw err;
    }
  }, []);

  const deleteUser = useCallback(async (id) => {
    try {
      const api = new UserAPI();
      await api.deleteUser(id);
      setUsers(prev => prev.filter(user => user.id !== id));
    } catch (err) {
      setError(err.message);
      throw err;
    }
  }, []);

  useEffect(() => {
    fetchUsers();
  }, [fetchUsers]);

  return {
    users,
    loading,
    error,
    fetchUsers,
    createUser,
    updateUser,
    deleteUser
  };
}

// Usage in component
function UsersList() {
  const { users, loading, error, createUser, updateUser, deleteUser } = useUsers();

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <div>
      <h2>Users</h2>
      {users.map(user => (
        <div key={user.id}>
          <h3>{user.name}</h3>
          <p>{user.email}</p>
          <button onClick={() => deleteUser(user.id)}>Delete</button>
        </div>
      ))}
    </div>
  );
}
```

---

## Authentication & Authorization

### JWT Authentication
```javascript
import jwt from 'jsonwebtoken';

// Login endpoint
app.post('/api/v1/auth/login', async (req, res) => {
  try {
    const { email, password } = req.body;
    
    // Validate credentials
    const user = await User.findOne({ where: { email } });
    if (!user || !await bcrypt.compare(password, user.password)) {
      return res.status(401).json({
        error: {
          code: 'INVALID_CREDENTIALS',
          message: 'Invalid email or password'
        }
      });
    }
    
    // Generate JWT
    const token = jwt.sign(
      { userId: user.id, email: user.email },
      process.env.JWT_SECRET,
      { expiresIn: '24h' }
    );
    
    res.json({
      data: {
        token,
        user: {
          id: user.id,
          name: user.name,
          email: user.email
        }
      }
    });
  } catch (error) {
    res.status(500).json({
      error: {
        code: 'INTERNAL_SERVER_ERROR',
        message: error.message
      }
    });
  }
});

// Middleware for protected routes
const authenticateToken = (req, res, next) => {
  const authHeader = req.headers['authorization'];
  const token = authHeader && authHeader.split(' ')[1];
  
  if (!token) {
    return res.status(401).json({
      error: {
        code: 'UNAUTHORIZED',
        message: 'Access token required'
      }
    });
  }
  
  jwt.verify(token, process.env.JWT_SECRET, (err, user) => {
    if (err) {
      return res.status(403).json({
        error: {
          code: 'FORBIDDEN',
          message: 'Invalid or expired token'
        }
      });
    }
    req.user = user;
    next();
  });
};

// Protected route
app.get('/api/v1/profile', authenticateToken, (req, res) => {
  res.json({ data: req.user });
});
```

### API Key Authentication
```javascript
// Middleware for API key authentication
const authenticateApiKey = (req, res, next) => {
  const apiKey = req.headers['x-api-key'];
  
  if (!apiKey) {
    return res.status(401).json({
      error: {
        code: 'UNAUTHORIZED',
        message: 'API key required'
      }
    });
  }
  
  // Validate API key
  if (apiKey !== process.env.API_KEY) {
    return res.status(403).json({
      error: {
        code: 'FORBIDDEN',
        message: 'Invalid API key'
      }
    });
  }
  
  next();
};

// Apply to routes
app.use('/api/v1/', authenticateApiKey);
```

---

## Error Handling

### Global Error Handler
```javascript
// Custom error class
class APIError extends Error {
  constructor(message, statusCode, code) {
    super(message);
    this.statusCode = statusCode;
    this.code = code;
    this.name = 'APIError';
  }
}

// Error handling middleware
const errorHandler = (err, req, res, next) => {
  console.error(err.stack);
  
  if (err instanceof APIError) {
    return res.status(err.statusCode).json({
      error: {
        code: err.code,
        message: err.message
      }
    });
  }
  
  if (err.name === 'ValidationError') {
    return res.status(422).json({
      error: {
        code: 'VALIDATION_ERROR',
        message: 'Validation failed',
        details: err.details
      }
    });
  }
  
  if (err.name === 'SequelizeValidationError') {
    return res.status(422).json({
      error: {
        code: 'VALIDATION_ERROR',
        message: 'Validation failed',
        details: err.errors.map(e => ({
          field: e.path,
          message: e.message
        }))
      }
    });
  }
  
  res.status(500).json({
    error: {
      code: 'INTERNAL_SERVER_ERROR',
      message: 'Something went wrong!'
    }
  });
};

app.use(errorHandler);
```

### Client-Side Error Handling
```javascript
class APIError extends Error {
  constructor(message, statusCode, code) {
    super(message);
    this.statusCode = statusCode;
    this.code = code;
  }
}

async function handleApiResponse(response) {
  if (!response.ok) {
    const error = await response.json();
    throw new APIError(
      error.error?.message || 'Request failed',
      response.status,
      error.error?.code
    );
  }
  
  if (response.status === 204) {
    return null;
  }
  
  return response.json();
}

// Usage
try {
  const data = await fetch('/api/users')
    .then(handleApiResponse);
  console.log(data);
} catch (error) {
  if (error instanceof APIError) {
    console.error(`API Error ${error.code}: ${error.message}`);
  } else {
    console.error('Network error:', error.message);
  }
}
```

---

## Testing

### Server Testing (Jest + Supertest)
```javascript
import request from 'supertest';
import app from '../app.js';

describe('User API', () => {
  let authToken;
  
  beforeAll(async () => {
    // Login to get auth token
    const response = await request(app)
      .post('/api/v1/auth/login')
      .send({
        email: 'test@example.com',
        password: 'password123'
      });
    
    authToken = response.body.data.token;
  });
  
  describe('GET /api/v1/users', () => {
    test('should return users list', async () => {
      const response = await request(app)
        .get('/api/v1/users')
        .set('Authorization', `Bearer ${authToken}`)
        .expect(200);
      
      expect(response.body.data).toBeInstanceOf(Array);
      expect(response.body.pagination).toBeDefined();
    });
    
    test('should handle pagination', async () => {
      const response = await request(app)
        .get('/api/v1/users?page=1&limit=5')
        .set('Authorization', `Bearer ${authToken}`)
        .expect(200);
      
      expect(response.body.pagination.page).toBe(1);
      expect(response.body.pagination.per_page).toBe(5);
    });
  });
  
  describe('POST /api/v1/users', () => {
    test('should create new user', async () => {
      const userData = {
        name: 'Test User',
        email: 'testuser@example.com',
        age: 25
      };
      
      const response = await request(app)
        .post('/api/v1/users')
        .set('Authorization', `Bearer ${authToken}`)
        .send(userData)
        .expect(201);
      
      expect(response.body.data.name).toBe(userData.name);
      expect(response.body.data.email).toBe(userData.email);
    });
    
    test('should return validation error for invalid data', async () => {
      const response = await request(app)
        .post('/api/v1/users')
        .set('Authorization', `Bearer ${authToken}`)
        .send({ name: '' })
        .expect(400);
      
      expect(response.body.error.code).toBe('VALIDATION_ERROR');
    });
  });
  
  describe('Authentication', () => {
    test('should require authentication for protected routes', async () => {
      await request(app)
        .get('/api/v1/users')
        .expect(401);
    });
  });
});
```

### Client Testing (Jest)
```javascript
import { UserAPI } from '../api/user-api.js';

// Mock fetch
global.fetch = jest.fn();

describe('UserAPI', () => {
  let api;
  
  beforeEach(() => {
    api = new UserAPI();
    fetch.mockClear();
  });
  
  test('should fetch users', async () => {
    const mockUsers = {
      data: [{ id: 1, name: 'John Doe' }],
      pagination: { page: 1, total: 1 }
    };
    
    fetch.mockResolvedValueOnce({
      ok: true,
      json: async () => mockUsers
    });
    
    const result = await api.getUsers();
    expect(result).toEqual(mockUsers);
    expect(fetch).toHaveBeenCalledWith(
      'http://localhost:3000/api/v1/users',
      expect.objectContaining({
        method: 'GET',
        headers: expect.objectContaining({
          'Content-Type': 'application/json'
        })
      })
    );
  });
  
  test('should handle API errors', async () => {
    const mockError = {
      error: {
        code: 'NOT_FOUND',
        message: 'User not found'
      }
    };
    
    fetch.mockResolvedValueOnce({
      ok: false,
      status: 404,
      json: async () => mockError
    });
    
    await expect(api.getUser(999)).rejects.toThrow('User not found');
  });
});
```

---

## Performance & Optimization

### Caching
```javascript
import NodeCache from 'node-cache';

const cache = new NodeCache({ stdTTL: 600 }); // 10 minutes

// Cached route handler
export const getUsers = async (req, res) => {
  try {
    const cacheKey = `users:${JSON.stringify(req.query)}`;
    const cached = cache.get(cacheKey);
    
    if (cached) {
      return res.json(cached);
    }
    
    const users = await User.findAll();
    const response = { data: users };
    
    cache.set(cacheKey, response);
    res.json(response);
  } catch (error) {
    res.status(500).json({
      error: {
        code: 'INTERNAL_SERVER_ERROR',
        message: error.message
      }
    });
  }
};
```

### Compression
```javascript
import compression from 'compression';

app.use(compression());
```

### Database Optimization
```javascript
// Use database indexes
await User.sync({
  indexes: [
    {
      fields: ['email'],
      unique: true
    },
    {
      fields: ['created_at']
    }
  ]
});

// Optimize queries
const users = await User.findAll({
  attributes: ['id', 'name', 'email'], // Only select needed fields
  where: { active: true },
  include: [{
    model: Post,
    attributes: ['id', 'title'],
    limit: 5 // Limit related data
  }],
  order: [['created_at', 'DESC']],
  limit: 20,
  offset: 0
});
```

---

## Documentation

### OpenAPI/Swagger
```javascript
import swaggerJsdoc from 'swagger-jsdoc';
import swaggerUi from 'swagger-ui-express';

const options = {
  definition: {
    openapi: '3.0.0',
    info: {
      title: 'User API',
      version: '1.0.0',
      description: 'A simple User API',
    },
    servers: [
      {
        url: 'http://localhost:3000/api/v1',
        description: 'Development server',
      },
    ],
    components: {
      securitySchemes: {
        bearerAuth: {
          type: 'http',
          scheme: 'bearer',
          bearerFormat: 'JWT',
        },
      },
    },
  },
  apis: ['./routes/*.js'], // paths to files containing OpenAPI definitions
};

const specs = swaggerJsdoc(options);
app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(specs));
```

### API Documentation Example
```javascript
/**
 * @swagger
 * /users:
 *   get:
 *     summary: Get all users
 *     tags: [Users]
 *     security:
 *       - bearerAuth: []
 *     parameters:
 *       - in: query
 *         name: page
 *         schema:
 *           type: integer
 *           default: 1
 *         description: Page number
 *       - in: query
 *         name: limit
 *         schema:
 *           type: integer
 *           default: 10
 *         description: Number of users per page
 *     responses:
 *       200:
 *         description: List of users
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 data:
 *                   type: array
 *                   items:
 *                     $ref: '#/components/schemas/User'
 *                 pagination:
 *                   $ref: '#/components/schemas/Pagination'
 *       401:
 *         description: Unauthorized
 */

/**
 * @swagger
 * components:
 *   schemas:
 *     User:
 *       type: object
 *       required:
 *         - name
 *         - email
 *       properties:
 *         id:
 *           type: integer
 *           description: User ID
 *         name:
 *           type: string
 *           description: User name
 *         email:
 *           type: string
 *           format: email
 *           description: User email
 *         age:
 *           type: integer
 *           description: User age
 *         created_at:
 *           type: string
 *           format: date-time
 *         updated_at:
 *           type: string
 *           format: date-time
 */
```

---

## Best Practices Summary

### API Design
- **Use RESTful conventions**: Follow HTTP methods and status codes
- **Consistent naming**: Use nouns, plural forms, lowercase with hyphens
- **Version your API**: Use URL versioning for breaking changes
- **Document everything**: Provide clear API documentation

### Security
- **Authentication**: Implement JWT or API key authentication
- **Authorization**: Check permissions for protected resources
- **Input validation**: Validate and sanitize all inputs
- **Rate limiting**: Prevent abuse with rate limiting
- **HTTPS**: Always use HTTPS in production

### Performance
- **Pagination**: Implement pagination for large datasets
- **Caching**: Cache frequently accessed data
- **Compression**: Use gzip compression
- **Database optimization**: Use indexes and optimize queries

### Error Handling
- **Consistent error format**: Use standard error response structure
- **Proper status codes**: Return appropriate HTTP status codes
- **Detailed error messages**: Provide helpful error information
- **Logging**: Log errors for debugging

### Testing
- **Unit tests**: Test individual functions and methods
- **Integration tests**: Test API endpoints end-to-end
- **Error scenarios**: Test error handling and edge cases
- **Performance tests**: Test API performance under load

REST APIs provide a simple and effective way to build web services. Follow these best practices to create APIs that are secure, performant, and easy to use.
