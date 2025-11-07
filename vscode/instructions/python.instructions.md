---
description: "Updated: 2025-10-11. Python best practices and patterns for modern software development."
applyTo: "**/*.py"
---

# Python Best Practices

Instructions for building high-quality Python applications with modern patterns, type hints, and best practices following PEP standards and official Python documentation.

## Technical Stack

- Programming Language: Python 3.13.7
- Package management: `venv` + `uv`.
- Synchronous web framework: `flask`.
- Asynchronous Web framework: `quart`.
- Event loop: `asyncio`.
- CORS: `flask-cors`, `quart-cors`
- JWT: `flask-jwt-extended` + `quart-flask-patch`
- Database client library: `mysql-connector-aio`.
- Key-value client library: `redis-py`.
- Server-side generator template engine: `jinja2`
- Synchronous HTTP client: `requests`
- Asynchronous HTTP client: `aiohttp`
- HTML parsing: `beautifulsoup4` (or bs4) with `lxml` parser.
- PDF generation: `WeasyPrint`, `PyMuPDF`, `pdfkit`
- Excel file support: `openpyxl`
- YAML parsing: PyYAML 6.0+
- User agent spoofing: `fake-useragent`
- Google Generative AI: `google-genai`, `google-generativeai`
- Code linter: `flake8`
- Code formatter: Black.
- Testing: `pytest`, `pytest-cov`, `pytest-mock`
- CLI: `click`.
- Code style: `PEP8`.
- Documentation: `Sphinx` + `PEP257` (docstring).
- Development tools: pre-commit hooks.

## Development Standards

### Architecture

- Use the `src/`-layout pattern with `src/insert_your_package_name/` for all Python projects. Src-layout provides better package organization, prevents import confusion, and follows modern Python packaging standards.

```py
# Not recommended: Flat structure
my-project/
├── myproject.py
├── models.py
└── views.py

# Good: Src-layout structure
my-project/
├── src/
│   └── my_project/
│       ├── __init__.py
│       ├── models.py
│       └── views.py
├── tests/
├── pyproject.toml
└── README.md
```

- Organize code by feature or domain, not by type. Feature-based organization aligns code structure with business logic, making scaling and refactoring easier as the application grows.

```py
# Bad: Type-based organization
src/my_project/
├── models/
├── views/
└── services/

# Good: Feature-based organization
src/my_project/
├── users/
│   ├── __init__.py
│   ├── models.py
│   ├── views.py
│   └── services.py
├── products/
│   ├── __init__.py
│   ├── models.py
│   └── views.py
└── shared/
    ├── __init__.py
    ├── db.py
    ├── kv.py
    ├── s3.py
    ├── task_queue.py
    └── utils.py
```

- Implement proper separation of concerns with clear boundaries between functions. By doing so, project improves maintainability, testability, and makes the codebase easier to understand and modify.

```py
# Good: Clear separation of concerns
# models.py - Data layer
class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(80))

# user_service.py - Business logic layer
class UserService:
    def create_user(self, name: str) -> User:
        user = User(name=name)
        db.session.add(user)
        db.session.commit()
        return user

# views.py - Presentation layer
@app.route('/users', methods=['POST'])
def create_user():
    data = request.get_json()
    user = UserService().create_user(data['name'])
    return jsonify({'id': user.id, 'name': user.name})
```

- Use Dependency Injection (DI) for better testability and loose coupling. Dependency injection makes code more testable, flexible, and follows the dependency inversion principle.

```py
# Good: Dependency injection
class UserService:
    def __init__(self, db_session: Session):
        self.db_session = db_session

    def create_user(self, name: str) -> User:
        user = User(name=name)
        self.db_session.add(user)
        self.db_session.commit()
        return user

# Usage
user_service = UserService(db.session)
user = user_service.create_user("John Doe")
```

- Implement Factory pattern for application instance initialization. Factory pattern provides better testing isolation, configuration management, and allows for multiple application instances.

```py
# Good: Factory pattern
def create_app(config_name: str = 'default') -> Flask:
    app = Flask(__name__)
    app.config.from_object(config[config_name])

    db.init_app(app)

    from .users import users_bp
    app.register_blueprint(users_bp)

    return app

# Usage
app = create_app('development')
```

### Type Hints

**IMPORTANT:** DO NOT use Python `typing` module for type annotations (e.g. `List`, `Dict`, ...) since they have been depreciated since Python 3.12. Use modern built-in typing syntax `list`, `dict`, ... (no capitalize)

- Use type hints for all function parameters and return values. Type hints improve code clarity, enable better IDE support, catch errors at development time, and make code more maintainable.

```py
# Not recommended: No type hints
def get_user_by_id(user_id):
    return User.query.get(user_id)

# Good: Complete type hints
def get_user_by_id(user_id: int) -> User | None:
    return User.query.get(user_id)

def create_user_list(users: list[dict[str, Any]]) -> list[User]:
    return [User(**user_data) for user_data in users]
```

- Use `Type | None` instead of `Optional[Type]` since the latter has been depreciated since Python 3.12.

```py
# Good: Union syntax (Python 3.10+)
def find_user(email: str) -> User | None:
    return User.query.filter_by(email=email).first()
# Not recommended: Using Optional
from typing import Optional

def find_user(email: str) -> Optional[User]:
    return User.query.filter_by(email=email).first()
```

### Code Style

- Follow Black code formatting. Black provides consistent, opinionated formatting that eliminates style debates and ensures code readability.

```py
# Bad: Inconsistent formatting
def create_user_with_profile(name:str,email:str,profile_data:Dict[str,Any],is_active:bool=True)->User:
    user=User(name=name,email=email,is_active=is_active)
    profile=Profile(**profile_data)
    user.profile=profile
    return user

# Good: Black-formatted code
def create_user_with_profile(
    name: str,
    email: str,
    profile_data: Dict[str, Any],
    is_active: bool = True, # trailing comma break arguments into multiple lines
) -> User:
    user = User(name=name, email=email, is_active=is_active)
    profile = Profile(**profile_data)
    user.profile = profile
    return user

```

- Use `isort` or `basedpyright` import organizing feature for import sorting and organization. Consistent import organization improves code readability and follows PEP 8 standards.

```py
# Good: Organized imports with isort
import os
from typing import Dict, List, Optional

from flask import Flask, jsonify, request
from sqlalchemy import Column, Integer, String
from sqlalchemy.ext.declarative import declarative_base

from .models import User
from .services import UserService
from .utils import validate_email

# Bad: Disorganized imports
from .models import User
from flask import Flask, jsonify, request
import os
from typing import Dict, List, Optional
from .services import UserService
```

- Follow [PEP8 convention](https://peps.python.org/pep-0008/) consistently. Use 4 spaces for each level of indentation and ensure lines do not exceed 79 characters. Consistent naming conventions improve code readability and make the codebase more professional and maintainable.

```py
# Not recommended: PascalCase
def CalculateUserScore(userId: int) -> float:
    userData = GetUserData(userId)
    return userData.score

# Good: PEP 8 naming conventions
# snake_case for functions and variables
def calculate_user_score(user_id: int) -> float:
    user_data = get_user_data(user_id)
    return user_data.score

# PascalCase for classes
class UserProfile:
    def __init__(self, user_id: int):
        self.user_id = user_id

# UPPER_CASE for constants
MAX_RETRY_ATTEMPTS = 3
DEFAULT_TIMEOUT = 30
```

- Use absolute imports over relative imports. Absolute imports are more explicit, less prone to errors during refactoring, and make dependencies clearer.

```py
# Bad: Relative imports (harder to maintain)
from ..models import User
from .services import UserService
from ..utils import validate_email

# Good: Absolute imports
from myproject.models import User
from myproject.services import UserService
from myproject.utils import validate_email
```

- Create `Function/Class` docstring according to [PEP257 convention](https://peps.python.org/pep-0257/).

```py
# One-line Docstrings
def kos_root():
    """Return the pathname of the KOS root directory."""

    global _kos_root
    if _kos_root:
        return _kos_root

# Multi-line Docstrings
def form_complex_number(real=0.0, imag=0.0):
    """Form a complex number.

    Keyword arguments:
    real -- the real part (default 0.0)
    imag -- the imaginary part (default 0.0)
    """

    if imag == 0.0 and real == 0.0:
        return complex_zero
```

### Flask Structure

- Use the Application Factory pattern for application creation. Factory pattern provides better testing isolation, configuration management, and allows for multiple application instances.

```py
# Bad: Global app instance
app = Flask(__name__)

# Good: Factory pattern
def create_app(config_name: str = 'default') -> Flask:
    app = Flask(__name__)
    app.config.from_object(config[config_name])

    # Initialize extensions
    db.init_app(app)
    login_manager.init_app(app)

    # Register blueprints
    from .users import users_bp
    from .products import products_bp
    app.register_blueprint(users_bp, url_prefix='/api/users')
    app.register_blueprint(products_bp, url_prefix='/api/products')

    # Register error handlers
    register_error_handlers(app)

    return app
```

- Organize routes using Blueprints for modularity. Blueprints provide better code organization, reusability, and make large applications more maintainable.

```py
# Good: Blueprint organization
# users/__init__.py
from flask import Blueprint

bp = Blueprint('users', __name__)

from . import views

# users/views.py
from . import bp
from flask import jsonify, request
from ..models import User

@bp.route('/', methods=['GET'])
def get_users():
    users = User.query.all()
    return jsonify([user.to_dict() for user in users])

@bp.route('/', methods=['POST'])
def create_user():
    data = request.get_json()
    user = User(**data)
    db.session.add(user)
    db.session.commit()
    return jsonify(user.to_dict()), 201
```

- Implement proper error handlers for consistent error responses. Consistent error handling improves user experience and makes debugging easier.

```py
# Good: Error handlers
def register_error_handlers(app: Flask) -> None:
    @app.errorhandler(404)
    def not_found(error):
        return jsonify({'error': 'Resource not found'}), 404

    @app.errorhandler(400)
    def bad_request(error):
        return jsonify({'error': 'Bad request'}), 400

    @app.errorhandler(500)
    def internal_error(error):
        db.session.rollback()
        return jsonify({'error': 'Internal server error'}), 500
```

### Database

- Use proper connection pooling for performance. Connection pooling reduces database connection overhead and improves application performance.

```py
# Good: Connection pooling configuration for PostgreSQL
app.config['SQLALCHEMY_DATABASE_URI'] = (
    'postgresql://user:pass@localhost/dbname'
    '?pool_size=10&max_overflow=20&pool_timeout=30'
)

# Good: Connection pooling configuration for MySQL
config = {
    "host": app.config["MYSQL_HOST"],
    "port": int(app.config["MYSQL_PORT"]),
    "user": app.config["MYSQL_USER"],
    "password": app.config["MYSQL_PASSWORD"],
    "database": app.config["MYSQL_DATABASE"],
    "ssl_disabled": app.config["MYSQL_SSL"],
    "autocommit": False,  # set to True to support XA transactions
    "use_pure": True,  # C extension mode, faster
    # TODO: adjust after stress test
    "connection_timeout": int(app.config["MYSQL_CONNECTION_TIMEOUT"]),
    "read_timeout": int(app.config["MYSQL_READ_TIMEOUT"]),
    "write_timeout": int(app.config["MYSQL_WRITE_TIMEOUT"]),
    "pool_name": app.config["MYSQL_POOL_NAME"],
    "pool_size": int(app.config["MYSQL_POOL_SIZE"]),
    "pool_reset_session": True,
}
```

### Authentication

- Use Flask-JWT-Extended for authentication and authorization operations.

```py
# Good: Flask-JWT-Extended integration
from flask_jwt_extended import (
    JWTManager,
    create_access_token,
    jwt_required,
    get_jwt_identity,
)
from werkzeug.security import generate_password_hash, check_password_hash

jwt = JWTManager()

class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    email = db.Column(db.String(120), unique=True, nullable=False)
    password_hash = db.Column(db.String(128))

def set_password(self, password: str) -> None:
    self.password_hash = generate_password_hash(password, method='pbkdf2:sha256')

def check_password(self, password: str) -> bool:
    return check_password_hash(self.password_hash, password)

@app.route('/login', methods=['POST'])
def login():
    data = request.get_json()
    user = User.query.filter_by(email=data['email']).first()
    if user and user.check_password(data['password']):
        access_token = create_access_token(identity=user.id)
        return jsonify({'access_token': access_token}), 200
    return jsonify({'error': 'Invalid credentials'}), 401

@app.route('/protected', methods=['GET'])
@jwt_required()
def protected():
    user_id = get_jwt_identity()
    user = User.query.get(user_id)
    return jsonify({'id': user.id, 'email': user.email})
```

- Hash passwords with `bcrypt` or similar secure hashing. Secure password hashing protects user credentials even if the database is compromised.

```py
# Bad: Plain text passwords
class User(db.Model):
    password = db.Column(db.String(128))  # Never store plain text passwords

# Good: Secure password hashing
from werkzeug.security import generate_password_hash, check_password_hash

class User(db.Model):
    password_hash = db.Column(db.String(128))

    def set_password(self, password: str) -> None:
        self.password_hash = generate_password_hash(password, method='pbkdf2:sha256')

    def check_password(self, password: str) -> bool:
        return check_password_hash(self.password_hash, password)
```

- Implement proper session security. Secure session configuration prevents session hijacking and other security vulnerabilities.

```py
# Good: Secure session configuration
app.config.update(
    SECRET_KEY=os.environ.get('SECRET_KEY'),
    SESSION_COOKIE_SECURE=True,
    SESSION_COOKIE_HTTPONLY=True,
    SESSION_COOKIE_SAMESITE='Lax',
    PERMANENT_SESSION_LIFETIME=timedelta(hours=2)
)
```

### API Design

- Use `Flask-RESTful` for REST API development. `Flask-RESTful` provides structured API development with automatic request parsing and response formatting.

```py
from flask_restful import Api, Resource, reqparse

api = Api()

class UserResource(Resource):
    def __init__(self):
        self.parser = reqparse.RequestParser()
        self.parser.add_argument('name', type=str, required=True)
        self.parser.add_argument('email', type=str, required=True)

    def get(self, user_id: int):
        user = User.query.get_or_404(user_id)
        return user.to_dict()

    def post(self):
        args = self.parser.parse_args()
        user = User(**args)
        db.session.add(user)
        db.session.commit()
        return user.to_dict(), 201

    def put(self, user_id: int):
        user = User.query.get_or_404(user_id)
        args = self.parser.parse_args()
        for key, value in args.items():
            setattr(user, key, value)
        db.session.commit()
        return user.to_dict()

api.add_resource(UserResource, '/users', '/users/<int:user_id>')
```

- Validate data coming from user requests. Request validation prevents invalid data from entering the system and provides clear error messages.

```py
# Good: Request validation
from marshmallow import Schema, fields, ValidationError

class UserSchema(Schema):
    name = fields.Str(required=True, validate=Length(min=1, max=80))
    email = fields.Email(required=True)
    age = fields.Int(validate=Range(min=0, max=120))

@app.route('/users', methods=['POST'])
def create_user():
    try:
        data = UserSchema().load(request.get_json())
        user = User(**data)
        db.session.add(user)
        db.session.commit()
        return jsonify(user.to_dict()), 201
    except ValidationError as e:
        return jsonify({'errors': e.messages}), 400
```

- Use proper HTTP status codes. Correct HTTP status codes provide clear communication about request outcomes and help clients handle responses appropriately.

```py
# Good: Proper HTTP status codes
@app.route('/users/<int:user_id>', methods=['GET'])
def get_user(user_id: int):
    user = User.query.get(user_id)
    if not user:
        return jsonify({'error': 'User not found'}), 404
    return jsonify(user.to_dict()), 200

@app.route('/users', methods=['POST'])
def create_user():
    # ... user creation logic
    return jsonify(user.to_dict()), 201  # Created

@app.route('/users/<int:user_id>', methods=['DELETE'])
def delete_user(user_id: int):
    user = User.query.get_or_404(user_id)
    db.session.delete(user)
    db.session.commit()
    return '', 204  # No Content
```

### Testing

- Use `pytest` for all testing. Pytest provides powerful testing features, better error reporting, and extensive plugin ecosystem.

```py
# Good: Pytest usage
import pytest
from myproject import create_app
from myproject.models import User

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

def test_create_user(client):
    response = client.post('/users', json={
        'name': 'John Doe',
        'email': 'john@example.com'
    })
    assert response.status_code == 201
    assert response.json['name'] == 'John Doe'

def test_get_user_not_found(client):
    response = client.get('/users/999')
    assert response.status_code == 404
```

- Write tests for all routes and business logic. Comprehensive testing ensures code reliability, catches bugs early, and provides confidence for refactoring.

```py
# Good: Comprehensive testing
class TestUserAPI:
    def test_create_user_success(self, client):
        response = client.post('/users', json={
            'name': 'Jane Doe',
            'email': 'jane@example.com'
        })
        assert response.status_code == 201
        assert 'id' in response.json

    def test_create_user_invalid_data(self, client):
        response = client.post('/users', json={
            'name': '',  # Invalid: empty name
            'email': 'invalid-email'  # Invalid email
        })
        assert response.status_code == 400
        assert 'errors' in response.json

    def test_get_user_success(self, client, user):
        response = client.get(f'/users/{user.id}')
        assert response.status_code == 200
        assert response.json['name'] == user.name

    def test_get_user_not_found(self, client):
        response = client.get('/users/999')
        assert response.status_code == 404
```

- Use `pytest-cov` for coverage reporting. Coverage reporting helps identify untested code and ensures adequate test coverage.

```py
# pytest.ini
[tool:pytest]
addopts = --cov=myproject --cov-report=html --cov-report=term-missing
testpaths = tests
python_files = test_*.py
python_classes = Test*
python_functions = test_*
```

- Implement proper fixtures for test data. Fixtures provide reusable test data and setup, making tests more maintainable and reducing duplication.

```py
# Good: Test fixtures
import pytest
from myproject.models import User

@pytest.fixture
def user():
    user = User(name='Test User', email='test@example.com')
    db.session.add(user)
    db.session.commit()
    return user

@pytest.fixture
def admin_user():
    user = User(name='Admin User', email='admin@example.com', is_admin=True)
    db.session.add(user)
    db.session.commit()
    return user

@pytest.fixture
def users():
    users = [
        User(name='User 1', email='user1@example.com'),
        User(name='User 2', email='user2@example.com'),
        User(name='User 3', email='user3@example.com')
    ]
    for user in users:
        db.session.add(user)
    db.session.commit()
    return users
```

- handle edge cases (empty inputs, invalid types, large datasets)
- include comments explaining edge cases and expected behavior
- unit tests with docstrings explaining test cases

```py
def calculate_area(radius: float) -> float:
    """
    Calculate the area of a circle given the radius.
    
    Parameters:
    radius (float): The radius of the circle.
    
    Returns:
    float: The area of the circle, calculated as π * radius^2.
    """
    import math
    return math.pi * radius ** 2
```

### Security

- DO NOT store sensitive data into JWT. JWT can be decoded at client and user can read data from it.

- Use HTTPS in production. HTTPS encrypts data in transit, protecting sensitive information and ensuring secure communication.

```py
# Good: HTTPS configuration
if app.config['ENV'] == 'production':
    app.config['PREFERRED_URL_SCHEME'] = 'https'
```

- Implement proper CORS configuration. Proper CORS configuration prevents unauthorized cross-origin requests while allowing legitimate ones.

```py
# Good: CORS configuration
from flask_cors import CORS

CORS(app, resources={
    r"/api/*": {
        "origins": ["https://myapp.com", "https://admin.myapp.com"],
        "methods": ["GET", "POST", "PUT", "DELETE"],
        "allow_headers": ["Content-Type", "Authorization"]
    }
})
```

- Sanitize all user inputs. Input sanitization prevents injection attacks and ensures data integrity.

```py
# Good: Input sanitization
import bleach
from html import escape

def sanitize_html(text: str) -> str:
    return bleach.clean(text, tags=[], strip=True)

def sanitize_sql_input(value: str) -> str:
    # Use parameterized queries instead of string formatting
    return escape(value)

@app.route('/posts', methods=['POST'])
def create_post():
    data = request.get_json()
    # Sanitize user input
    title = sanitize_html(data['title'])
    content = sanitize_html(data['content'])

    post = Post(title=title, content=content)
    db.session.add(post)
    db.session.commit()
    return jsonify(post.to_dict()), 201
```

- Enable CSRF protection in Flask-JWT-Extended. CSRF protection prevents cross-site request forgery attacks.

### Performance

- Use cache-aside + write through caching strategies. Caching improves application performance and reduces database load.

```py
# Good: Cache-aside via Flask-Caching extension
from flask_caching import Cache

cache = Cache()

@app.route('/users/<int:user_id>', methods=['GET'])
@cache.memoize(timeout=300)  # Cache for 5 minutes
def get_user(user_id: int):
    return User.query.get_or_404(user_id).to_dict()

@app.route('/users', methods=['GET'])
@cache.memoize(timeout=60)  # Cache for 1 minute
def get_users():
    users = User.query.all()
    return jsonify([user.to_dict() for user in users])
```

- Implement database query optimization. Optimized queries improve performance and reduce resource usage.

```py
# Good: Query optimization
# Use eager loading to load all related data under a single query, avoid N+1 queries
users = User.query.options(
    db.joinedload(User.posts),
    db.joinedload(User.profile)
).all()

# Use specific column selection
user_ids = db.session.query(User.id).filter(User.is_active == True).all()

# Use pagination for large datasets
users = User.query.paginate(
    page=page, per_page=20, error_out=False
)
```

- Implement proper pagination for large datasets. Pagination improves performance and user experience when dealing with large datasets.

```py
# Good: Pagination implementation
@app.route('/users', methods=['GET'])
def get_users():
    page = request.args.get('page', 1, type=int)
    per_page = min(request.args.get('per_page', 20, type=int), 100)

    pagination = User.query.paginate(
        page=page, per_page=per_page, error_out=False
    )

    return jsonify({
        'users': [user.to_dict() for user in pagination.items],
        'pagination': {
            'page': page,
            'per_page': per_page,
            'total': pagination.total,
            'pages': pagination.pages,
            'has_next': pagination.has_next,
            'has_prev': pagination.has_prev
        }
    })
```

### Error Handling

- Create custom exception classes. Custom exceptions provide clear error categorization and improve error handling.

```py
# Good: Custom exceptions
class AppError(Exception):
    """Raised when a general error is thrown."""
    pass

class ValidationError(Exception):
    """Raised when data validation fails."""
    pass

class UserNotFoundError(Exception):
    """Raised when a user is not found."""
    pass

class InsufficientPermissionsError(Exception):
    """Raised when user lacks required permissions."""
    pass

# Usage
def get_user(user_id: int) -> User:
    user = User.query.get(user_id)
    if not user:
        raise UserNotFoundError(f"User with id {user_id} not found")
    return user
```

- Wrap try-except blocks around application object initialization and API route logic. Proper exception handling prevents application crashes and provides meaningful error responses.

```py
# Good: Exception handling
@app.route('/users/<int:user_id>', methods=['GET'])
def get_user(user_id: int):
    try:
        user = User.query.get_or_404(user_id)
        return jsonify(user.to_dict())
    except Exception as e:
        app.logger.error(f"Error retrieving user {user_id}: {str(e)}")
        return jsonify({'error': 'Internal server error'}), 500
```

- Implement proper custom logging. Logging provides visibility into application behavior and helps with debugging and monitoring.

```py
# Good: Logging implementation
import logging
from logging.handlers import RotatingFileHandler

def setup_logging(app: Flask) -> None:
    if not app.debug:
        file_handler = RotatingFileHandler(
            'logs/myapp.log', maxBytes=10240, backupCount=10
        )
        file_handler.setFormatter(logging.Formatter(
            '%(asctime)s %(levelname)s: %(message)s [in %(pathname)s:%(lineno)d]'
        ))
        file_handler.setLevel(logging.INFO)
        app.logger.addHandler(file_handler)

        app.logger.setLevel(logging.INFO)
        app.logger.info('MyApp startup')

# Usage in routes
@app.route('/users', methods=['POST'])
def create_user():
    try:
        data = request.get_json()
        user = User(**data)
        db.session.add(user)
        db.session.commit()
        app.logger.info(f'Created user: {user.email}')
        return jsonify(user.to_dict()), 201
    except Exception as e:
        app.logger.error(f'Error creating user: {str(e)}')
        return jsonify({'error': 'Failed to create user'}), 500
```

### Documentation

- Use PEP257 Google-style docstrings. Google-style docstrings are clear, comprehensive, and widely supported by documentation tools.

```py
# Good: Google-style docstrings
def create_user(name: str, email: str, age: Optional[int] = None) -> User:
    """Create a new user in the system.

    Args:
        name: The user's full name.
        email: The user's email address (must be unique).
        age: The user's age in years (optional).

    Returns:
        User: The newly created user object.

    Raises:
        ValidationError: If the email format is invalid.
        DuplicateEmailError: If the email already exists.

    Example:
        >>> user = create_user("John Doe", "john@example.com", 30)
        >>> print(user.name)
        John Doe
    """
    if not is_valid_email(email):
        raise ValidationError("Invalid email format")

    if User.query.filter_by(email=email).first():
        raise DuplicateEmailError("Email already exists")

    user = User(name=name, email=email, age=age)
    db.session.add(user)
    db.session.commit()
    return user
```

- Document all public APIs. API documentation helps other developers understand and use your code correctly.

```py
# Good: API documentation
class UserAPI(Resource):
    """User management API endpoints.

    This class provides RESTful endpoints for user operations including
    creation, retrieval, updating, and deletion of user records.
    """

    def get(self, user_id: int):
        """Retrieve a user by ID.

        Args:
            user_id: The unique identifier of the user.

        Returns:
            dict: User data in JSON format.

        Raises:
            404: If the user is not found.
        """
        user = User.query.get_or_404(user_id)
        return user.to_dict()
```

- Keep `README.md` updated. An updated README provides essential information for project setup and usage.

````md
# MyApp

A Flask-based web application for user management.

## Features

- User registration and authentication
- RESTful API endpoints
- Database integration with SQLAlchemy
- Comprehensive testing with pytest

## Installation

1. Clone the repository:

    ```bash
    git clone https://github.com/username/myapp.git
    cd myapp
    ```

2. Create a virtual environment:

    ```bash
    python -m venv venv
    source venv/bin/activate  # On Windows: venv\Scripts\activate
    ```

3. Install dependencies:

    ```bash
    pip install -r requirements.txt
    ```

4. Set up environment variables:

    ```bash
    cp .env.example .env
    # Edit .env with your configuration
    ```

5. Run database migrations:

    ```bash
    flask db upgrade
    ```

6. Start the application:
    ```bash
    flask run
    ```

## Testing

Run tests with pytest:

    ```bash
    pytest
    ```

Run with coverage:

    ```bash
    pytest --cov=myproject
    ```
````

### Development Workflow

- Use Python virtual environments (i.e. invoke `virtualenv` to create  `.venv`). Virtual environments isolate project dependencies and prevent conflicts between projects.

```bash
# Good: Virtual environment usage
python -m venv venv
source venv/bin/activate[.fish]  # On Windows: venv\Scripts\activate
pip install -r requirements.txt

# Deactivate when done
deactivate
```

- Implement pre-commit hooks. Pre-commit hooks ensure code quality by running checks before commits.

```yaml
# .pre-commit-config.yaml
repos:
- repo: https://github.com/psf/black
    rev: 23.3.0
    hooks:
    - id: black
        language_version: python3

- repo: https://github.com/pycqa/isort
    rev: 5.12.0
    hooks:
    - id: isort

- repo: https://github.com/pycqa/flake8
    rev: 6.0.0
    hooks:
    - id: flake8
```

- Follow semantic versioning. Semantic versioning provides clear communication about changes and helps with dependency management.

```py
# Good: Semantic versioning
__version__ = "1.2.3"  # MAJOR.MINOR.PATCH

# 1.2.3 -> 1.2.4 (patch: bug fixes)
# 1.2.3 -> 1.3.0 (minor: new features, backward compatible)
# 1.2.3 -> 2.0.0 (major: breaking changes)
```

### Dependencies

- Pin dependency versions. Pinned versions ensure reproducible builds and prevent unexpected breaking changes.

```txt
# requirements.txt
Flask==3.0.0
SQLAlchemy==2.0.23
pytest==7.4.3
black==23.11.0
isort==5.12.0
flake8==6.1.0
```

- Separate development dependencies. Separating dev dependencies keeps production environments lean and secure.

```txt
# requirements.txt (production)
Flask==3.0.0
SQLAlchemy==2.0.23
gunicorn==21.2.0
```

```
# requirements-dev.txt (development)
pytest==7.4.3
black==23.11.0
isort==5.12.0
flake8==6.1.0
mypy==1.7.1
```

- Regularly update dependencies. Regular updates ensure security patches and new features are available.

```bash
# Check for outdated packages
pip list --outdated

# Update specific package
pip install --upgrade Flask

# Update all packages (use with caution)
pip install --upgrade -r requirements.txt
```

## Implementation Process

1. Set up project structure with src-layout
2. Configure development environment and tools
3. Implement core models and database schema
4. Create API endpoints with proper validation
5. Add authentication and authorization
6. Implement error handling and logging
7. Write comprehensive tests
8. Add documentation and README
9. Configure deployment and CI/CD
10. Optimize performance and security

## Additional Guidelines

- Always prioritize readability and clarity.
- For algorithm-related code, include explanations of the approach used.
- Write code with good maintainability practices, including comments on why certain design decisions were made.
- Handle edge cases and write clear exception handling.
- For libraries or external dependencies, mention their usage and purpose in comments.
- Use consistent naming conventions and follow language-specific best practices.
- Write concise, efficient, and idiomatic code that is also easily understandable.
- Always include test cases for critical paths of the application.

- Follow PEP 8 style guide consistently.
- Use meaningful variable and function names
- Implement proper logging and monitoring
- Write self-documenting code with clear comments
- Use type hints throughout the codebase
- Implement proper error handling and recovery
- Follow the principle of least privilege for security
- Use environment variables for configuration
- Implement proper backup and recovery procedures
- Monitor application performance and errors

## Patterns

- Factory pattern for application creation
- Repository pattern for data access
- Service layer pattern for business logic
- Dependency injection for loose coupling
- Blueprint pattern for modular applications
- Decorator pattern for cross-cutting concerns
- Strategy pattern for interchangeable algorithms
- Observer pattern for event handling

---

<!-- End of Python Best Practices instructions -->
