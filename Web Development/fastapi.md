# FastAPI - Modern Python Web Framework

FastAPI is a modern, fast (high-performance) web framework for building APIs with Python 3.7+ based on standard Python type hints. It's designed to be easy to use and learn, fast to code, and ready for production.

---

## What is FastAPI?

FastAPI is a:
- **High Performance**: One of the fastest Python frameworks available
- **Type Hints**: Built on Python type hints for better IDE support
- **Automatic Documentation**: Interactive API docs with Swagger UI
- **Modern Python**: Uses async/await for better concurrency
- **Standards Based**: Based on OpenAPI and JSON Schema standards
- **Production Ready**: Built for production with automatic validation

### Key Features
- **Automatic API Documentation**: Interactive docs with Swagger UI and ReDoc
- **Type Safety**: Full type hints support with automatic validation
- **Async Support**: Native async/await support for high performance
- **Data Validation**: Automatic request/response validation with Pydantic
- **Dependency Injection**: Powerful dependency injection system
- **OpenAPI**: Automatic OpenAPI schema generation

---

## Installation & Setup

### Basic Installation
```bash
# Install FastAPI
pip install fastapi

# Install ASGI server
pip install uvicorn[standard]

# For development
pip install fastapi uvicorn[standard] python-multipart
```

### Project Structure
```
my_fastapi_app/
├── main.py                 # Main application file
├── requirements.txt        # Dependencies
├── app/
│   ├── __init__.py
│   ├── api/               # API routes
│   │   ├── __init__.py
│   │   ├── auth.py
│   │   └── users.py
│   ├── core/              # Core functionality
│   │   ├── __init__.py
│   │   ├── config.py
│   │   └── security.py
│   ├── models/            # Database models
│   │   ├── __init__.py
│   │   └── user.py
│   ├── schemas/           # Pydantic schemas
│   │   ├── __init__.py
│   │   └── user.py
│   └── database.py        # Database configuration
├── tests/                 # Test files
│   ├── __init__.py
│   └── test_main.py
└── static/                # Static files
    └── style.css
```

---

## Basic FastAPI Application

### Hello World
```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"Hello": "World"}

@app.get("/items/{item_id}")
def read_item(item_id: int, q: str = None):
    return {"item_id": item_id, "q": q}

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

### Running the Application
```bash
# Development server
uvicorn main:app --reload

# Production server
uvicorn main:app --host 0.0.0.0 --port 8000

# With specific workers
uvicorn main:app --workers 4
```

---

## Path Parameters & Query Parameters

### Path Parameters
```python
from fastapi import FastAPI

app = FastAPI()

# Basic path parameter
@app.get("/items/{item_id}")
def read_item(item_id: int):
    return {"item_id": item_id}

# Path parameter with type conversion
@app.get("/users/{user_id}")
def get_user(user_id: int):
    return {"user_id": user_id, "type": type(user_id).__name__}

# Multiple path parameters
@app.get("/users/{user_id}/items/{item_id}")
def read_user_item(user_id: int, item_id: int):
    return {"user_id": user_id, "item_id": item_id}

# Path parameter with validation
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    description: str = None
    price: float
    tax: float = None

@app.put("/items/{item_id}")
def update_item(item_id: int, item: Item):
    return {"item_id": item_id, **item.dict()}
```

### Query Parameters
```python
from fastapi import FastAPI, Query
from typing import Optional, List

app = FastAPI()

# Basic query parameters
@app.get("/items/")
def read_items(skip: int = 0, limit: int = 10):
    return {"skip": skip, "limit": limit}

# Optional query parameters
@app.get("/users/")
def read_users(q: Optional[str] = None):
    if q:
        return {"users": f"Searching for: {q}"}
    return {"users": "All users"}

# Query parameter validation
@app.get("/items/")
def read_items(
    q: Optional[str] = Query(None, min_length=3, max_length=50),
    skip: int = Query(0, ge=0),
    limit: int = Query(10, ge=1, le=100)
):
    return {"q": q, "skip": skip, "limit": limit}

# Multiple query parameters
@app.get("/search/")
def search_items(
    q: str = Query(..., description="Search query"),
    category: Optional[str] = Query(None, description="Item category"),
    tags: List[str] = Query([], description="Item tags")
):
    return {"q": q, "category": category, "tags": tags}
```

---

## Request Body & Pydantic Models

### Basic Request Body
```python
from fastapi import FastAPI
from pydantic import BaseModel
from typing import Optional

app = FastAPI()

class Item(BaseModel):
    name: str
    description: Optional[str] = None
    price: float
    tax: Optional[float] = None

@app.post("/items/")
def create_item(item: Item):
    return item

@app.put("/items/{item_id}")
def update_item(item_id: int, item: Item):
    return {"item_id": item_id, **item.dict()}

# Nested models
class Address(BaseModel):
    street: str
    city: str
    country: str

class User(BaseModel):
    name: str
    email: str
    address: Address

@app.post("/users/")
def create_user(user: User):
    return user
```

### Advanced Pydantic Models
```python
from pydantic import BaseModel, Field, validator
from typing import Optional, List
from datetime import datetime
from enum import Enum

class UserRole(str, Enum):
    ADMIN = "admin"
    USER = "user"
    MODERATOR = "moderator"

class User(BaseModel):
    id: int
    name: str = Field(..., min_length=1, max_length=100)
    email: str = Field(..., regex=r'^[\w\.-]+@[\w\.-]+\.\w+$')
    age: int = Field(..., ge=0, le=120)
    role: UserRole = UserRole.USER
    created_at: datetime = Field(default_factory=datetime.now)
    tags: List[str] = Field(default_factory=list)
    
    @validator('name')
    def name_must_not_be_empty(cls, v):
        if not v.strip():
            raise ValueError('Name cannot be empty')
        return v.strip()
    
    @validator('age')
    def age_must_be_positive(cls, v):
        if v < 0:
            raise ValueError('Age must be positive')
        return v
    
    class Config:
        schema_extra = {
            "example": {
                "id": 1,
                "name": "John Doe",
                "email": "john@example.com",
                "age": 30,
                "role": "user",
                "tags": ["developer", "python"]
            }
        }

@app.post("/users/", response_model=User)
def create_user(user: User):
    return user
```

---

## Response Models & Status Codes

### Response Models
```python
from fastapi import FastAPI, status
from pydantic import BaseModel
from typing import List, Optional

app = FastAPI()

class ItemBase(BaseModel):
    name: str
    description: Optional[str] = None
    price: float

class ItemCreate(ItemBase):
    pass

class Item(ItemBase):
    id: int
    is_active: bool = True
    
    class Config:
        orm_mode = True

class ItemResponse(BaseModel):
    success: bool = True
    data: Item
    message: str = "Item created successfully"

# Using response models
@app.post("/items/", response_model=ItemResponse, status_code=status.HTTP_201_CREATED)
def create_item(item: ItemCreate):
    # Simulate item creation
    created_item = Item(id=1, **item.dict())
    return ItemResponse(data=created_item)

@app.get("/items/", response_model=List[Item])
def read_items():
    # Simulate returning items
    return [
        Item(id=1, name="Item 1", price=10.0),
        Item(id=2, name="Item 2", price=20.0)
    ]

@app.get("/items/{item_id}", response_model=Item)
def read_item(item_id: int):
    return Item(id=item_id, name=f"Item {item_id}", price=10.0)
```

### Custom Status Codes
```python
from fastapi import FastAPI, status, HTTPException

app = FastAPI()

@app.post("/items/", status_code=status.HTTP_201_CREATED)
def create_item(item: ItemCreate):
    return {"message": "Item created", "item": item}

@app.put("/items/{item_id}", status_code=status.HTTP_200_OK)
def update_item(item_id: int, item: ItemCreate):
    return {"message": "Item updated", "item_id": item_id, "item": item}

@app.delete("/items/{item_id}", status_code=status.HTTP_204_NO_CONTENT)
def delete_item(item_id: int):
    # Return None for 204 No Content
    return None

# Custom error responses
@app.get("/items/{item_id}")
def read_item(item_id: int):
    if item_id < 1:
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail="Item ID must be positive"
        )
    if item_id > 100:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="Item not found"
        )
    return {"item_id": item_id}
```

---

## Dependency Injection

### Basic Dependencies
```python
from fastapi import FastAPI, Depends
from typing import Optional

app = FastAPI()

# Simple dependency
def get_common_parameters(q: Optional[str] = None, skip: int = 0, limit: int = 100):
    return {"q": q, "skip": skip, "limit": limit}

@app.get("/items/")
def read_items(commons: dict = Depends(get_common_parameters)):
    return commons

# Class-based dependency
class CommonQueryParams:
    def __init__(self, q: Optional[str] = None, skip: int = 0, limit: int = 100):
        self.q = q
        self.skip = skip
        self.limit = limit

@app.get("/items/")
def read_items(commons: CommonQueryParams = Depends()):
    return commons
```

### Database Dependencies
```python
from fastapi import FastAPI, Depends
from sqlalchemy.orm import Session
from database import SessionLocal, engine
from models import User

app = FastAPI()

# Database dependency
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

@app.get("/users/{user_id}")
def read_user(user_id: int, db: Session = Depends(get_db)):
    user = db.query(User).filter(User.id == user_id).first()
    if user is None:
        raise HTTPException(status_code=404, detail="User not found")
    return user

@app.post("/users/")
def create_user(user: UserCreate, db: Session = Depends(get_db)):
    db_user = User(**user.dict())
    db.add(db_user)
    db.commit()
    db.refresh(db_user)
    return db_user
```

### Authentication Dependencies
```python
from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from jose import JWTError, jwt
from passlib.context import CryptContext

app = FastAPI()
security = HTTPBearer()
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

# Authentication dependency
def get_current_user(credentials: HTTPAuthorizationCredentials = Depends(security)):
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": "Bearer"},
    )
    
    try:
        payload = jwt.decode(credentials.credentials, SECRET_KEY, algorithms=[ALGORITHM])
        username: str = payload.get("sub")
        if username is None:
            raise credentials_exception
    except JWTError:
        raise credentials_exception
    
    user = get_user_by_username(username)
    if user is None:
        raise credentials_exception
    return user

# Protected endpoint
@app.get("/users/me/")
def read_users_me(current_user: User = Depends(get_current_user)):
    return current_user

# Optional authentication
def get_current_user_optional(credentials: Optional[HTTPAuthorizationCredentials] = Depends(security)):
    if credentials is None:
        return None
    return get_current_user(credentials)

@app.get("/items/")
def read_items(current_user: Optional[User] = Depends(get_current_user_optional)):
    if current_user:
        return {"items": "user items", "user": current_user.username}
    return {"items": "public items"}
```

---

## Async Support

### Async Endpoints
```python
import asyncio
import httpx
from fastapi import FastAPI

app = FastAPI()

# Async endpoint
@app.get("/async-endpoint")
async def read_async():
    # Simulate async operation
    await asyncio.sleep(1)
    return {"message": "Async response"}

# Async with external API calls
@app.get("/external-data")
async def get_external_data():
    async with httpx.AsyncClient() as client:
        response = await client.get("https://api.github.com/users/octocat")
        return response.json()

# Mixed sync/async
@app.get("/mixed")
async def mixed_endpoint():
    # Async operation
    async_result = await some_async_operation()
    
    # Sync operation
    sync_result = some_sync_operation()
    
    return {"async": async_result, "sync": sync_result}

async def some_async_operation():
    await asyncio.sleep(0.1)
    return "async result"

def some_sync_operation():
    return "sync result"
```

### Background Tasks
```python
from fastapi import FastAPI, BackgroundTasks
from typing import Dict

app = FastAPI()

def write_log(message: str):
    with open("log.txt", "a") as f:
        f.write(f"{message}\n")

def send_email(email: str, message: str):
    # Simulate sending email
    print(f"Sending email to {email}: {message}")

@app.post("/send-notification/{email}")
async def send_notification(email: str, background_tasks: BackgroundTasks):
    message = "Notification sent"
    background_tasks.add_task(send_email, email, message)
    background_tasks.add_task(write_log, f"Email sent to {email}")
    return {"message": "Notification will be sent in the background"}
```

---

## File Upload & Static Files

### File Upload
```python
from fastapi import FastAPI, File, UploadFile, Form
from typing import List
import shutil
import os

app = FastAPI()

# Single file upload
@app.post("/upload/")
async def upload_file(file: UploadFile = File(...)):
    # Save file
    with open(f"uploads/{file.filename}", "wb") as buffer:
        shutil.copyfileobj(file.file, buffer)
    
    return {"filename": file.filename, "content_type": file.content_type}

# Multiple files upload
@app.post("/upload-multiple/")
async def upload_multiple_files(files: List[UploadFile] = File(...)):
    uploaded_files = []
    for file in files:
        with open(f"uploads/{file.filename}", "wb") as buffer:
            shutil.copyfileobj(file.file, buffer)
        uploaded_files.append({
            "filename": file.filename,
            "content_type": file.content_type
        })
    return {"uploaded_files": uploaded_files}

# File upload with form data
@app.post("/upload-with-form/")
async def upload_file_with_form(
    file: UploadFile = File(...),
    name: str = Form(...),
    description: str = Form(None)
):
    return {
        "filename": file.filename,
        "name": name,
        "description": description
    }
```

### Static Files
```python
from fastapi import FastAPI
from fastapi.staticfiles import StaticFiles

app = FastAPI()

# Mount static files
app.mount("/static", StaticFiles(directory="static"), name="static")

# Serve files from different directories
app.mount("/images", StaticFiles(directory="images"), name="images")
app.mount("/css", StaticFiles(directory="css"), name="css")
```

---

## Database Integration

### SQLAlchemy Setup
```python
from fastapi import FastAPI, Depends
from sqlalchemy import create_engine, Column, Integer, String, Boolean
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker, Session
from pydantic import BaseModel

# Database setup
SQLALCHEMY_DATABASE_URL = "sqlite:///./test.db"
engine = create_engine(SQLALCHEMY_DATABASE_URL, connect_args={"check_same_thread": False})
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()

# Models
class User(Base):
    __tablename__ = "users"
    
    id = Column(Integer, primary_key=True, index=True)
    name = Column(String, index=True)
    email = Column(String, unique=True, index=True)
    is_active = Column(Boolean, default=True)

# Pydantic schemas
class UserBase(BaseModel):
    name: str
    email: str

class UserCreate(UserBase):
    pass

class User(UserBase):
    id: int
    is_active: bool
    
    class Config:
        orm_mode = True

# Create tables
Base.metadata.create_all(bind=engine)

# Dependency
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

app = FastAPI()

# CRUD operations
@app.post("/users/", response_model=User)
def create_user(user: UserCreate, db: Session = Depends(get_db)):
    db_user = User(**user.dict())
    db.add(db_user)
    db.commit()
    db.refresh(db_user)
    return db_user

@app.get("/users/{user_id}", response_model=User)
def read_user(user_id: int, db: Session = Depends(get_db)):
    user = db.query(User).filter(User.id == user_id).first()
    if user is None:
        raise HTTPException(status_code=404, detail="User not found")
    return user

@app.get("/users/", response_model=List[User])
def read_users(skip: int = 0, limit: int = 100, db: Session = Depends(get_db)):
    users = db.query(User).offset(skip).limit(limit).all()
    return users
```

---

## Authentication & Security

### JWT Authentication
```python
from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from jose import JWTError, jwt
from passlib.context import CryptContext
from datetime import datetime, timedelta
from typing import Optional

app = FastAPI()
security = HTTPBearer()

# Configuration
SECRET_KEY = "your-secret-key"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

# Password hashing
def verify_password(plain_password, hashed_password):
    return pwd_context.verify(plain_password, hashed_password)

def get_password_hash(password):
    return pwd_context.hash(password)

# JWT functions
def create_access_token(data: dict, expires_delta: Optional[timedelta] = None):
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(minutes=15)
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt

def verify_token(token: str):
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        username: str = payload.get("sub")
        if username is None:
            raise HTTPException(
                status_code=status.HTTP_401_UNAUTHORIZED,
                detail="Invalid authentication credentials"
            )
        return username
    except JWTError:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid authentication credentials"
        )

# Authentication dependency
def get_current_user(credentials: HTTPAuthorizationCredentials = Depends(security)):
    return verify_token(credentials.credentials)

# Login endpoint
@app.post("/login")
def login(username: str, password: str):
    # Verify user credentials
    if not verify_password(password, get_user_password(username)):
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Incorrect username or password"
        )
    
    access_token_expires = timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    access_token = create_access_token(
        data={"sub": username}, expires_delta=access_token_expires
    )
    return {"access_token": access_token, "token_type": "bearer"}

# Protected endpoint
@app.get("/protected")
def protected_route(current_user: str = Depends(get_current_user)):
    return {"message": f"Hello {current_user}"}
```

---

## API Documentation

### Custom Documentation
```python
from fastapi import FastAPI
from fastapi.openapi.utils import get_openapi

app = FastAPI(
    title="My API",
    description="A simple API built with FastAPI",
    version="1.0.0",
    docs_url="/docs",
    redoc_url="/redoc"
)

def custom_openapi():
    if app.openapi_schema:
        return app.openapi_schema
    
    openapi_schema = get_openapi(
        title="My Custom API",
        version="2.0.0",
        description="This is a custom OpenAPI schema",
        routes=app.routes,
    )
    
    # Custom modifications
    openapi_schema["info"]["x-logo"] = {
        "url": "https://fastapi.tiangolo.com/img/logo-margin/logo-teal.png"
    }
    
    app.openapi_schema = openapi_schema
    return app.openapi_schema

app.openapi = custom_openapi

@app.get("/")
def read_root():
    return {"Hello": "World"}
```

### API Tags and Metadata
```python
from fastapi import FastAPI, APIRouter

app = FastAPI(
    title="My API",
    description="A comprehensive API built with FastAPI",
    version="1.0.0",
    contact={
        "name": "API Support",
        "email": "support@example.com",
    },
    license_info={
        "name": "MIT",
        "url": "https://opensource.org/licenses/MIT",
    },
)

# Router with tags
users_router = APIRouter(prefix="/users", tags=["users"])
items_router = APIRouter(prefix="/items", tags=["items"])

@users_router.get("/", summary="Get all users", description="Retrieve a list of all users")
def get_users():
    return {"users": []}

@users_router.post("/", summary="Create user", description="Create a new user")
def create_user():
    return {"message": "User created"}

@items_router.get("/", summary="Get all items", description="Retrieve a list of all items")
def get_items():
    return {"items": []}

app.include_router(users_router)
app.include_router(items_router)
```

---

## Testing

### Unit Testing
```python
from fastapi.testclient import TestClient
from main import app

client = TestClient(app)

def test_read_main():
    response = client.get("/")
    assert response.status_code == 200
    assert response.json() == {"Hello": "World"}

def test_create_user():
    response = client.post(
        "/users/",
        json={"name": "John Doe", "email": "john@example.com"}
    )
    assert response.status_code == 200
    assert response.json()["name"] == "John Doe"

def test_read_user():
    response = client.get("/users/1")
    assert response.status_code == 200
    assert "id" in response.json()

def test_read_nonexistent_user():
    response = client.get("/users/999")
    assert response.status_code == 404
```

### Async Testing
```python
import pytest
from httpx import AsyncClient
from main import app

@pytest.mark.asyncio
async def test_async_endpoint():
    async with AsyncClient(app=app, base_url="http://test") as ac:
        response = await ac.get("/async-endpoint")
    assert response.status_code == 200
    assert response.json() == {"message": "Async response"}

@pytest.mark.asyncio
async def test_create_user_async():
    async with AsyncClient(app=app, base_url="http://test") as ac:
        response = await ac.post(
            "/users/",
            json={"name": "Jane Doe", "email": "jane@example.com"}
        )
    assert response.status_code == 200
    assert response.json()["name"] == "Jane Doe"
```

---

## Deployment

### Production Server
```bash
# Install production server
pip install gunicorn

# Run with Gunicorn
gunicorn main:app -w 4 -k uvicorn.workers.UvicornWorker

# With configuration
gunicorn main:app -c gunicorn.conf.py
```

### Docker Deployment
```dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Docker Compose
```yaml
version: '3.8'
services:
  api:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://user:password@db:5432/dbname
    depends_on:
      - db
  
  db:
    image: postgres:13
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=dbname
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

---

## Best Practices

### 1. Project Structure
- **Organize by features**: Group related functionality together
- **Use routers**: Split large applications into modules
- **Separate concerns**: Keep models, schemas, and routes separate
- **Configuration management**: Use environment variables for config

### 2. Performance
- **Use async/await**: For I/O bound operations
- **Database optimization**: Use connection pooling and proper indexing
- **Caching**: Implement caching for expensive operations
- **Response compression**: Enable gzip compression

### 3. Security
- **Input validation**: Use Pydantic models for validation
- **Authentication**: Implement proper JWT authentication
- **HTTPS**: Use HTTPS in production
- **Rate limiting**: Implement rate limiting for API endpoints

### 4. Development
- **Type hints**: Use type hints throughout your code
- **Testing**: Write comprehensive tests
- **Documentation**: Use docstrings and OpenAPI descriptions
- **Error handling**: Implement proper error handling and logging

FastAPI is a powerful and modern framework for building APIs. Its automatic documentation, type safety, and async support make it an excellent choice for both simple and complex applications. Start with basic endpoints and gradually add more advanced features as you become familiar with the framework.
