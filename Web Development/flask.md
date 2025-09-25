# Flask - Python Web Framework

Flask is a lightweight and flexible Python web framework that provides the tools, libraries, and technologies needed to build web applications. It's known for its simplicity, flexibility, and "micro" approach to web development.

---

## What is Flask?

Flask is a:
- **Micro Framework**: Minimal core with extensible architecture
- **Werkzeug Based**: Built on WSGI toolkit and Jinja2 templating
- **Flexible**: No specific project structure or ORM required
- **Lightweight**: Small codebase with minimal dependencies
- **Extensible**: Rich ecosystem of extensions

### Key Features
- **Routing**: URL routing with decorators
- **Templates**: Jinja2 templating engine
- **Request Handling**: Easy access to request data
- **Session Management**: Client-side sessions
- **Extensions**: Rich ecosystem (SQLAlchemy, Flask-Login, etc.)
- **Testing**: Built-in testing support

---

## Installation & Setup

### Basic Installation
```bash
# Install Flask
pip install flask

# Create virtual environment
python -m venv flask_env
source flask_env/bin/activate  # Linux/Mac
# or
flask_env\Scripts\activate     # Windows

# Install with requirements
pip install -r requirements.txt
```

### Project Structure
```
my_flask_app/
├── app.py                 # Main application file
├── requirements.txt       # Dependencies
├── config.py             # Configuration
├── models/               # Database models
│   ├── __init__.py
│   └── user.py
├── views/                # View functions
│   ├── __init__.py
│   ├── auth.py
│   └── main.py
├── templates/            # HTML templates
│   ├── base.html
│   ├── index.html
│   └── auth/
│       ├── login.html
│       └── register.html
├── static/               # Static files
│   ├── css/
│   ├── js/
│   └── images/
└── tests/                # Test files
    ├── __init__.py
    └── test_app.py
```

---

## Basic Flask Application

### Hello World
```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello, World!'

if __name__ == '__main__':
    app.run(debug=True)
```

### Application Factory Pattern
```python
from flask import Flask

def create_app(config_name='development'):
    app = Flask(__name__)
    
    # Configuration
    if config_name == 'development':
        app.config['DEBUG'] = True
        app.config['SECRET_KEY'] = 'dev-secret-key'
    elif config_name == 'production':
        app.config['DEBUG'] = False
        app.config['SECRET_KEY'] = 'prod-secret-key'
    
    # Register blueprints
    from views.main import main_bp
    from views.auth import auth_bp
    
    app.register_blueprint(main_bp)
    app.register_blueprint(auth_bp, url_prefix='/auth')
    
    return app

if __name__ == '__main__':
    app = create_app()
    app.run()
```

---

## Routing

### Basic Routes
```python
from flask import Flask

app = Flask(__name__)

# Basic route
@app.route('/')
def index():
    return 'Home Page'

# Route with parameters
@app.route('/user/<username>')
def show_user(username):
    return f'User: {username}'

# Route with type converters
@app.route('/post/<int:post_id>')
def show_post(post_id):
    return f'Post ID: {post_id}'

# Multiple routes for same function
@app.route('/about')
@app.route('/info')
def about():
    return 'About Page'

# HTTP methods
@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        return 'Login POST'
    return 'Login GET'
```

### Advanced Routing
```python
from flask import Flask, request, jsonify

app = Flask(__name__)

# URL building
@app.route('/user/<username>')
def profile(username):
    return f'Profile for {username}'

# Query parameters
@app.route('/search')
def search():
    query = request.args.get('q', '')
    page = request.args.get('page', 1, type=int)
    return f'Search: {query}, Page: {page}'

# JSON responses
@app.route('/api/users')
def api_users():
    users = [
        {'id': 1, 'name': 'John'},
        {'id': 2, 'name': 'Jane'}
    ]
    return jsonify(users)

# Error handlers
@app.errorhandler(404)
def not_found(error):
    return 'Page not found', 404

@app.errorhandler(500)
def internal_error(error):
    return 'Internal server error', 500
```

---

## Templates & Jinja2

### Basic Templates
```html
<!-- templates/base.html -->
<!DOCTYPE html>
<html>
<head>
    <title>{% block title %}My App{% endblock %}</title>
    <link rel="stylesheet" href="{{ url_for('static', filename='css/style.css') }}">
</head>
<body>
    <nav>
        <a href="{{ url_for('index') }}">Home</a>
        <a href="{{ url_for('about') }}">About</a>
    </nav>
    
    <main>
        {% block content %}{% endblock %}
    </main>
    
    <footer>
        {% block footer %}{% endblock %}
    </footer>
</body>
</html>
```

```html
<!-- templates/index.html -->
{% extends "base.html" %}

{% block title %}Home - My App{% endblock %}

{% block content %}
<h1>Welcome to My App</h1>
<p>Hello, {{ name }}!</p>

{% if users %}
    <h2>Users:</h2>
    <ul>
    {% for user in users %}
        <li>{{ user.name }} - {{ user.email }}</li>
    {% endfor %}
    </ul>
{% else %}
    <p>No users found.</p>
{% endif %}
{% endblock %}
```

### Template Rendering
```python
from flask import Flask, render_template, request

app = Flask(__name__)

@app.route('/')
def index():
    name = request.args.get('name', 'Guest')
    users = [
        {'name': 'John', 'email': 'john@example.com'},
        {'name': 'Jane', 'email': 'jane@example.com'}
    ]
    return render_template('index.html', name=name, users=users)

@app.route('/user/<username>')
def user_profile(username):
    return render_template('user.html', username=username)
```

### Jinja2 Filters & Functions
```python
from flask import Flask, render_template

app = Flask(__name__)

# Custom filter
@app.template_filter('reverse')
def reverse_filter(s):
    return s[::-1]

# Custom function
@app.template_global()
def get_current_year():
    from datetime import datetime
    return datetime.now().year

# Template context processor
@app.context_processor
def inject_user():
    # This will be available in all templates
    return dict(current_user=get_current_user())
```

---

## Request Handling

### Request Data
```python
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route('/api/data', methods=['POST'])
def handle_data():
    # Form data
    name = request.form.get('name')
    email = request.form.get('email')
    
    # JSON data
    if request.is_json:
        data = request.get_json()
        name = data.get('name')
        email = data.get('email')
    
    # Query parameters
    page = request.args.get('page', 1, type=int)
    limit = request.args.get('limit', 10, type=int)
    
    # Headers
    user_agent = request.headers.get('User-Agent')
    content_type = request.content_type
    
    # Files
    if 'file' in request.files:
        file = request.files['file']
        filename = file.filename
        file.save(f'uploads/{filename}')
    
    return jsonify({
        'name': name,
        'email': email,
        'page': page,
        'limit': limit
    })
```

### Request Validation
```python
from flask import Flask, request, jsonify, abort
from werkzeug.exceptions import BadRequest

app = Flask(__name__)

@app.route('/api/user', methods=['POST'])
def create_user():
    data = request.get_json()
    
    if not data:
        abort(400, 'JSON data required')
    
    # Validate required fields
    required_fields = ['name', 'email']
    for field in required_fields:
        if field not in data:
            abort(400, f'{field} is required')
    
    # Validate email format
    import re
    email_pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    if not re.match(email_pattern, data['email']):
        abort(400, 'Invalid email format')
    
    # Process data
    user = {
        'id': 1,
        'name': data['name'],
        'email': data['email']
    }
    
    return jsonify(user), 201
```

---

## Sessions & Cookies

### Session Management
```python
from flask import Flask, session, request, redirect, url_for

app = Flask(__name__)
app.secret_key = 'your-secret-key'

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        username = request.form['username']
        password = request.form['password']
        
        # Validate credentials
        if username == 'admin' and password == 'password':
            session['username'] = username
            session['logged_in'] = True
            return redirect(url_for('dashboard'))
        else:
            return 'Invalid credentials'
    
    return '''
    <form method="post">
        <input type="text" name="username" placeholder="Username" required>
        <input type="password" name="password" placeholder="Password" required>
        <button type="submit">Login</button>
    </form>
    '''

@app.route('/dashboard')
def dashboard():
    if not session.get('logged_in'):
        return redirect(url_for('login'))
    
    return f'Welcome, {session["username"]}!'

@app.route('/logout')
def logout():
    session.clear()
    return redirect(url_for('login'))
```

### Cookies
```python
from flask import Flask, request, make_response

app = Flask(__name__)

@app.route('/set-cookie')
def set_cookie():
    resp = make_response('Cookie set')
    resp.set_cookie('username', 'john', max_age=60*60*24*7)  # 7 days
    return resp

@app.route('/get-cookie')
def get_cookie():
    username = request.cookies.get('username')
    return f'Username: {username}'

@app.route('/delete-cookie')
def delete_cookie():
    resp = make_response('Cookie deleted')
    resp.set_cookie('username', '', expires=0)
    return resp
```

---

## Database Integration

### SQLAlchemy Setup
```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from flask_migrate import Migrate

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///app.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

db = SQLAlchemy(app)
migrate = Migrate(app, db)

# Models
class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)
    posts = db.relationship('Post', backref='author', lazy=True)
    
    def __repr__(self):
        return f'<User {self.username}>'

class Post(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(100), nullable=False)
    content = db.Column(db.Text, nullable=False)
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'), nullable=False)
    
    def __repr__(self):
        return f'<Post {self.title}>'

# Create tables
with app.app_context():
    db.create_all()
```

### Database Operations
```python
from flask import Flask, request, jsonify
from models import db, User, Post

@app.route('/api/users', methods=['GET'])
def get_users():
    users = User.query.all()
    return jsonify([{
        'id': user.id,
        'username': user.username,
        'email': user.email
    } for user in users])

@app.route('/api/users', methods=['POST'])
def create_user():
    data = request.get_json()
    
    user = User(
        username=data['username'],
        email=data['email']
    )
    
    db.session.add(user)
    db.session.commit()
    
    return jsonify({
        'id': user.id,
        'username': user.username,
        'email': user.email
    }), 201

@app.route('/api/users/<int:user_id>', methods=['PUT'])
def update_user(user_id):
    user = User.query.get_or_404(user_id)
    data = request.get_json()
    
    user.username = data.get('username', user.username)
    user.email = data.get('email', user.email)
    
    db.session.commit()
    
    return jsonify({
        'id': user.id,
        'username': user.username,
        'email': user.email
    })

@app.route('/api/users/<int:user_id>', methods=['DELETE'])
def delete_user(user_id):
    user = User.query.get_or_404(user_id)
    db.session.delete(user)
    db.session.commit()
    
    return '', 204
```

---

## Authentication & Authorization

### Flask-Login
```python
from flask import Flask, render_template, request, redirect, url_for, flash
from flask_login import LoginManager, UserMixin, login_user, logout_user, login_required, current_user
from werkzeug.security import generate_password_hash, check_password_hash

app = Flask(__name__)
app.secret_key = 'your-secret-key'

login_manager = LoginManager()
login_manager.init_app(app)
login_manager.login_view = 'login'

@login_manager.user_loader
def load_user(user_id):
    return User.query.get(int(user_id))

class User(UserMixin, db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)
    password_hash = db.Column(db.String(128))
    
    def set_password(self, password):
        self.password_hash = generate_password_hash(password)
    
    def check_password(self, password):
        return check_password_hash(self.password_hash, password)

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        username = request.form['username']
        password = request.form['password']
        
        user = User.query.filter_by(username=username).first()
        
        if user and user.check_password(password):
            login_user(user)
            return redirect(url_for('dashboard'))
        else:
            flash('Invalid username or password')
    
    return render_template('login.html')

@app.route('/logout')
@login_required
def logout():
    logout_user()
    return redirect(url_for('index'))

@app.route('/dashboard')
@login_required
def dashboard():
    return render_template('dashboard.html', user=current_user)
```

### JWT Authentication
```python
from flask import Flask, request, jsonify
from flask_jwt_extended import JWTManager, jwt_required, create_access_token, get_jwt_identity
from datetime import timedelta

app = Flask(__name__)
app.config['JWT_SECRET_KEY'] = 'your-secret-key'
app.config['JWT_ACCESS_TOKEN_EXPIRES'] = timedelta(hours=1)

jwt = JWTManager(app)

@app.route('/api/login', methods=['POST'])
def login():
    data = request.get_json()
    username = data.get('username')
    password = data.get('password')
    
    # Validate credentials
    user = User.query.filter_by(username=username).first()
    if user and user.check_password(password):
        access_token = create_access_token(identity=user.id)
        return jsonify({
            'access_token': access_token,
            'user': {
                'id': user.id,
                'username': user.username,
                'email': user.email
            }
        })
    
    return jsonify({'error': 'Invalid credentials'}), 401

@app.route('/api/protected')
@jwt_required()
def protected():
    current_user_id = get_jwt_identity()
    user = User.query.get(current_user_id)
    return jsonify({
        'message': 'Protected endpoint',
        'user': {
            'id': user.id,
            'username': user.username
        }
    })
```

---

## Blueprints & Modular Applications

### Blueprint Structure
```python
# views/main.py
from flask import Blueprint, render_template

main_bp = Blueprint('main', __name__)

@main_bp.route('/')
def index():
    return render_template('index.html')

@main_bp.route('/about')
def about():
    return render_template('about.html')
```

```python
# views/auth.py
from flask import Blueprint, render_template, request, redirect, url_for, flash
from flask_login import login_user, logout_user, login_required

auth_bp = Blueprint('auth', __name__)

@auth_bp.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        # Login logic
        pass
    return render_template('auth/login.html')

@auth_bp.route('/logout')
@login_required
def logout():
    logout_user()
    return redirect(url_for('main.index'))
```

```python
# app.py
from flask import Flask
from views.main import main_bp
from views.auth import auth_bp

def create_app():
    app = Flask(__name__)
    
    # Register blueprints
    app.register_blueprint(main_bp)
    app.register_blueprint(auth_bp, url_prefix='/auth')
    
    return app
```

---

## API Development

### RESTful API
```python
from flask import Flask, request, jsonify
from flask_restful import Api, Resource, reqparse
from models import db, User

app = Flask(__name__)
api = Api(app)

class UserResource(Resource):
    def get(self, user_id):
        user = User.query.get_or_404(user_id)
        return {
            'id': user.id,
            'username': user.username,
            'email': user.email
        }
    
    def put(self, user_id):
        user = User.query.get_or_404(user_id)
        parser = reqparse.RequestParser()
        parser.add_argument('username', type=str)
        parser.add_argument('email', type=str)
        args = parser.parse_args()
        
        if args['username']:
            user.username = args['username']
        if args['email']:
            user.email = args['email']
        
        db.session.commit()
        return {
            'id': user.id,
            'username': user.username,
            'email': user.email
        }
    
    def delete(self, user_id):
        user = User.query.get_or_404(user_id)
        db.session.delete(user)
        db.session.commit()
        return '', 204

class UserListResource(Resource):
    def get(self):
        users = User.query.all()
        return [{
            'id': user.id,
            'username': user.username,
            'email': user.email
        } for user in users]
    
    def post(self):
        parser = reqparse.RequestParser()
        parser.add_argument('username', type=str, required=True)
        parser.add_argument('email', type=str, required=True)
        args = parser.parse_args()
        
        user = User(username=args['username'], email=args['email'])
        db.session.add(user)
        db.session.commit()
        
        return {
            'id': user.id,
            'username': user.username,
            'email': user.email
        }, 201

# Register resources
api.add_resource(UserListResource, '/api/users')
api.add_resource(UserResource, '/api/users/<int:user_id>')
```

---

## Testing

### Unit Testing
```python
import unittest
from app import create_app, db
from models import User

class TestApp(unittest.TestCase):
    def setUp(self):
        self.app = create_app('testing')
        self.app_context = self.app.app_context()
        self.app_context.push()
        self.client = self.app.test_client()
        db.create_all()
    
    def tearDown(self):
        db.session.remove()
        db.drop_all()
        self.app_context.pop()
    
    def test_index(self):
        response = self.client.get('/')
        self.assertEqual(response.status_code, 200)
    
    def test_user_creation(self):
        response = self.client.post('/api/users', json={
            'username': 'testuser',
            'email': 'test@example.com'
        })
        self.assertEqual(response.status_code, 201)
        
        user = User.query.filter_by(username='testuser').first()
        self.assertIsNotNone(user)
        self.assertEqual(user.email, 'test@example.com')

if __name__ == '__main__':
    unittest.main()
```

### Integration Testing
```python
import pytest
from app import create_app, db
from models import User

@pytest.fixture
def app():
    app = create_app('testing')
    with app.app_context():
        db.create_all()
        yield app
        db.drop_all()

@pytest.fixture
def client(app):
    return app.test_client()

@pytest.fixture
def user(app):
    user = User(username='testuser', email='test@example.com')
    db.session.add(user)
    db.session.commit()
    return user

def test_user_endpoint(client, user):
    response = client.get(f'/api/users/{user.id}')
    assert response.status_code == 200
    assert response.json['username'] == 'testuser'

def test_user_creation(client):
    response = client.post('/api/users', json={
        'username': 'newuser',
        'email': 'new@example.com'
    })
    assert response.status_code == 201
    assert response.json['username'] == 'newuser'
```

---

## Configuration & Environment

### Configuration Classes
```python
import os
from flask import Flask

class Config:
    SECRET_KEY = os.environ.get('SECRET_KEY') or 'dev-secret-key'
    SQLALCHEMY_TRACK_MODIFICATIONS = False

class DevelopmentConfig(Config):
    DEBUG = True
    SQLALCHEMY_DATABASE_URI = os.environ.get('DEV_DATABASE_URL') or 'sqlite:///dev.db'

class ProductionConfig(Config):
    DEBUG = False
    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL') or 'sqlite:///prod.db'

class TestingConfig(Config):
    TESTING = True
    SQLALCHEMY_DATABASE_URI = 'sqlite:///:memory:'

config = {
    'development': DevelopmentConfig,
    'production': ProductionConfig,
    'testing': TestingConfig,
    'default': DevelopmentConfig
}

def create_app(config_name='default'):
    app = Flask(__name__)
    app.config.from_object(config[config_name])
    return app
```

### Environment Variables
```bash
# .env file
FLASK_APP=app.py
FLASK_ENV=development
SECRET_KEY=your-secret-key
DATABASE_URL=postgresql://user:password@localhost/dbname
```

```python
# Load environment variables
from dotenv import load_dotenv
load_dotenv()

# Use in application
import os
app.config['SECRET_KEY'] = os.environ.get('SECRET_KEY')
```

---

## Deployment

### Production Server (Gunicorn)
```bash
# Install Gunicorn
pip install gunicorn

# Run with Gunicorn
gunicorn -w 4 -b 0.0.0.0:8000 app:app

# With configuration file
gunicorn -c gunicorn.conf.py app:app
```

```python
# gunicorn.conf.py
bind = "0.0.0.0:8000"
workers = 4
worker_class = "sync"
worker_connections = 1000
timeout = 30
keepalive = 2
max_requests = 1000
max_requests_jitter = 100
```

### Docker Deployment
```dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["gunicorn", "-w", "4", "-b", "0.0.0.0:8000", "app:app"]
```

### Nginx Configuration
```nginx
server {
    listen 80;
    server_name your-domain.com;
    
    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    
    location /static {
        alias /path/to/your/app/static;
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

---

## Best Practices

### 1. Application Structure
- **Use Blueprints**: Organize code into modules
- **Application Factory**: Use factory pattern for flexibility
- **Configuration Management**: Separate configs for different environments
- **Error Handling**: Implement proper error handlers

### 2. Security
- **Secret Key**: Use strong, random secret keys
- **Input Validation**: Validate all user inputs
- **SQL Injection**: Use ORM or parameterized queries
- **CSRF Protection**: Enable CSRF protection for forms
- **HTTPS**: Use HTTPS in production

### 3. Performance
- **Database Optimization**: Use proper indexing and queries
- **Caching**: Implement caching for expensive operations
- **Static Files**: Serve static files efficiently
- **Database Connections**: Use connection pooling

### 4. Development
- **Virtual Environment**: Always use virtual environments
- **Requirements**: Pin dependency versions
- **Testing**: Write comprehensive tests
- **Logging**: Implement proper logging
- **Code Quality**: Use linting and formatting tools

Flask is a powerful and flexible framework for building web applications. Start with simple applications and gradually add complexity as you become more familiar with the framework and its ecosystem.
