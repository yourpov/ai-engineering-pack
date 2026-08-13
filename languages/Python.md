# Python Architecture

> **Agent load:** Open Project Structure, Principles, Error Handling, Configuration, and Project Prompt / Validation first. Open other sections only when the task needs them. Read project `AGENTS.md` if present. For reviews use `skills/review/audit/SKILL.md` (scope + mode). For naming/comments use `skills/engineering/craft/SKILL.md` (not detector scoring). Prefer extending an existing repo over scaffolding a parallel tree. Discover verify commands from the project; do not invent a toolchain.

Clean, Pythonic structure for maintainable applications.

---

## Project Structure

```
project/
├── src/
│   ├── core/           # Business logic
│   ├── storage/        # Database, files
│   ├── api/            # HTTP/API layer
│   └── utils/          # Helpers
├── tests/
│   ├── unit/
│   └── integration/
├── requirements.txt
└── pyproject.toml
```

---

## Principles

**Explicit is better than implicit**
Don't hide behavior. Make it obvious.

**Simple is better than complex**
If it's hard to explain, it's a bad idea.

**Flat is better than nested**
Avoid deep nesting. Extract functions.

**Readability counts**
Code is read far more than written.

---

## Module Organization

### Domain Logic

```
src/core/
├── __init__.py
├── user.py
└── validator.py
```

**user.py**
```python
from dataclasses import dataclass

@dataclass
class User:
    name: str
    age: int
    
    def __post_init__(self):
        if not self.name:
            raise ValueError("Name required")
        
        if not is_valid_age(self.age):
            raise ValueError("Invalid age")

def is_valid_age(age: int) -> bool:
    return 0 <= age <= 150
```

### Storage Layer

```
src/storage/
├── __init__.py
├── base.py          # Abstract interface
└── memory.py        # In-memory implementation
```

**base.py**
```python
from abc import ABC, abstractmethod
from typing import Optional

class UserStore(ABC):
    @abstractmethod
    def save(self, user: User) -> None:
        pass
    
    @abstractmethod
    def find(self, user_id: int) -> Optional[User]:
        pass
    
    @abstractmethod
    def delete(self, user_id: int) -> None:
        pass
```

**memory.py**
```python
from typing import Optional, Dict

class MemoryStore(UserStore):
    def __init__(self):
        self._users: Dict[int, User] = {}
        self._next_id = 1
    
    def save(self, user: User) -> None:
        user_id = self._next_id
        self._next_id += 1
        self._users[user_id] = user
    
    def find(self, user_id: int) -> Optional[User]:
        return self._users.get(user_id)
    
    def delete(self, user_id: int) -> None:
        self._users.pop(user_id, None)
```

### API Layer

```
src/api/
├── __init__.py
├── app.py           # Flask/FastAPI app
└── routes.py        # Route handlers
```

**app.py (Flask)**
```python
from flask import Flask

def create_app(store: UserStore) -> Flask:
    app = Flask(__name__)
    app.config['store'] = store
    
    register_routes(app)
    
    return app
```

**routes.py**
```python
from flask import request, jsonify

def register_routes(app: Flask) -> None:
    @app.route('/users', methods=['POST'])
    def create_user():
        data = request.json
        
        try:
            user = User(
                name=data['name'],
                age=data['age']
            )
        except (KeyError, ValueError) as e:
            return jsonify({'error': str(e)}), 400
        
        store = app.config['store']
        store.save(user)
        
        return jsonify({'name': user.name, 'age': user.age}), 201
```

---

## Functions

**Keep them short**

```python
# Bad - too long
def process_order(order):
    # validate order
    # calculate total
    # apply discount
    # charge payment
    # update inventory
    # send email
    # 50+ lines

# Good - extract steps
def process_order(order):
    validate(order)
    total = calculate_total(order)
    charge(order, total)
    update_inventory(order)
    send_confirmation(order)
```

**Do one thing**

```python
# Bad
def save_and_notify(user):
    db.save(user)
    email.send(user)

# Good
def save(user):
    db.save(user)

def notify(user):
    email.send(user)
```

**No flag parameters**

```python
# Bad
def save(user, should_log=False):
    if should_log:
        print(f"Saving {user.name}")
    db.save(user)

# Good
def save(user):
    db.save(user)

def save_with_log(user):
    print(f"Saving {user.name}")
    save(user)
```

**Return early**

```python
# Bad
def process(data):
    if data:
        if is_valid(data):
            result = transform(data)
            return result
    return None

# Good
def process(data):
    if not data:
        return None
    
    if not is_valid(data):
        return None
    
    return transform(data)
```

---

## Error Handling

**Use exceptions, not error codes**

```python
# Bad
def divide(a, b):
    if b == 0:
        return None
    return a / b

result = divide(10, 0)
if result is None:
    # handle error

# Good
def divide(a, b):
    if b == 0:
        raise ValueError("Division by zero")
    return a / b

try:
    result = divide(10, 0)
except ValueError as e:
    # handle error
```

**Be specific with exceptions**

```python
# Bad
try:
    process()
except Exception:
    pass

# Good
try:
    process()
except FileNotFoundError:
    handle_missing_file()
except PermissionError:
    handle_permissions()
```

**Use context managers for cleanup**

```python
# Bad
file = open('data.txt')
data = file.read()
file.close()

# Good
with open('data.txt') as file:
    data = file.read()
```

---

## Naming

**Use descriptive names**

```python
# Bad
def calc(x, y):
    return x + y

# Good
def calculate_total(price, tax):
    return price + tax
```

**Functions are verbs**

```python
def save_user(user):
    pass

def validate_age(age):
    pass

def find_by_id(user_id):
    pass
```

**Variables are nouns**

```python
total_count = 0
file_path = 'data.txt'
active_users = []
```

**Constants are UPPER_CASE**

```python
MAX_RETRIES = 3
DEFAULT_TIMEOUT = 30
API_KEY = 'secret'
```

**Private members use underscore**

```python
class User:
    def __init__(self, name):
        self._name = name  # private
    
    @property
    def name(self):
        return self._name
```

---

## Classes

**Single responsibility**

```python
# Bad
class User:
    def save(self):
        # persistence logic
    
    def validate(self):
        # validation logic
    
    def send_email(self):
        # email logic

# Good
class User:
    def __init__(self, name, age):
        self.name = name
        self.age = age

class UserRepository:
    def save(self, user):
        # persistence

class UserValidator:
    def is_valid(self, user):
        # validation

class UserNotifier:
    def send_welcome(self, user):
        # email
```

**Use dataclasses for simple data**

```python
from dataclasses import dataclass

@dataclass
class Point:
    x: int
    y: int

@dataclass
class User:
    name: str
    age: int
    email: str = ""  # default value
```

**Composition over inheritance**

```python
# Bad
class FileLogger(Logger):
    pass

class DatabaseLogger(Logger):
    pass

# Good
class Logger:
    def __init__(self, writer):
        self.writer = writer
    
    def log(self, message):
        self.writer.write(message)

file_logger = Logger(FileWriter())
db_logger = Logger(DatabaseWriter())
```

---

## Type Hints

**Use them consistently**

```python
from typing import List, Optional, Dict

def find_users(ids: List[int]) -> List[User]:
    return [find_user(id) for id in ids]

def find_user(user_id: int) -> Optional[User]:
    return db.get(user_id)

def count_by_age(users: List[User]) -> Dict[int, int]:
    counts = {}
    for user in users:
        counts[user.age] = counts.get(user.age, 0) + 1
    return counts
```

---

## List Comprehensions

**Keep them simple**

```python
# Good - simple
numbers = [x * 2 for x in range(10)]

# Good - with filter
evens = [x for x in numbers if x % 2 == 0]

# Bad - too complex
result = [
    transform(x)
    for x in data
    if validate(x) and check(x)
    for y in process(x)
]

# Better - extract to function
def process_data(data):
    valid = [x for x in data if is_valid(x)]
    return [transform(x) for x in valid]
```

---

## Testing

**Use pytest**

```python
# tests/test_user.py
import pytest
from src.core.user import User, is_valid_age

def test_create_user_valid():
    user = User("Alice", 30)
    assert user.name == "Alice"
    assert user.age == 30

def test_create_user_no_name():
    with pytest.raises(ValueError):
        User("", 30)

def test_create_user_invalid_age():
    with pytest.raises(ValueError):
        User("Bob", -5)

@pytest.mark.parametrize("age,expected", [
    (0, True),
    (30, True),
    (150, True),
    (-1, False),
    (151, False),
])
def test_is_valid_age(age, expected):
    assert is_valid_age(age) == expected
```

**Use fixtures for setup**

```python
import pytest

@pytest.fixture
def user():
    return User("Alice", 30)

@pytest.fixture
def store():
    return MemoryStore()

def test_save_user(store, user):
    store.save(user)
    found = store.find(1)
    assert found.name == "Alice"
```

**Mock external dependencies**

```python
from unittest.mock import Mock

def test_notify_user():
    email_service = Mock()
    notifier = UserNotifier(email_service)
    
    user = User("Alice", 30)
    notifier.send_welcome(user)
    
    email_service.send.assert_called_once()
```

---

## Project Setup

**requirements.txt**
```
flask==3.0.0
pytest==7.4.0
```

**pyproject.toml**
```toml
[project]
name = "app"
version = "1.0.0"
requires-python = ">=3.10"

[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = ["test_*.py"]
```

---

## Configuration

Load and validate config once with `pydantic-settings`, export a single typed
`settings`, and import it everywhere, no `os.environ` reads scattered through the code
(`../standards/Principles.md` §15.1). A missing or wrong-typed var raises at startup.

```python
# app/config.py
from pydantic_settings import BaseSettings, SettingsConfigDict


class Settings(BaseSettings):
    model_config = SettingsConfigDict(env_file=".env")

    database_url: str
    jwt_secret: str
    port: int = 3000
    debug: bool = False


settings = Settings()  # raises ValidationError on a missing required var
```

**.env** (gitignored; commit a `.env.example` with the keys and dummy values)

```
DATABASE_URL=postgresql://localhost/mydb
JWT_SECRET=your-secret-key
PORT=3000
DEBUG=false
```

Secrets come from the environment, never from committed source.

---

## Summary

Write short, focused functions.
Use type hints for clarity.
Handle errors with exceptions.
Keep classes small and single-purpose.
Test everything with pytest.
Follow PEP 8 for style.

---

## Project Prompt

Write Python against the structure and rules above. Where they disagree with your
defaults, this file wins.

Read `../standards/Principles.md` alongside this file before starting.

**Type Safety**
- Type hints on all function signatures
- Dataclasses for data containers
- Use Protocol for interfaces
- Annotate return types

**Error Handling**
- Specific exceptions, not bare `except:`
- Custom exception classes for domain errors
- Context managers for resource cleanup
- Never swallow exceptions silently

**Testing**
- pytest for all tests
- Test all error paths
- Use fixtures for setup
- Mock external dependencies

### Setup

```bash
python -m venv venv
source venv/bin/activate
pip install fastapi uvicorn sqlalchemy pydantic pytest
```

### Deliverables

1. Complete project following architecture structure above
2. FastAPI routes with dependency injection
3. Service layer with business logic
4. Repository layer for data access
5. Type hints throughout
6. Custom exception classes
7. Environment configuration
8. requirements.txt
9. README with setup instructions
10. pytest test suite

### Validation Checklist

- [ ] Verify commands from project AGENTS.md / README run (or honest manual checks listed)
- [ ] No secrets committed; env examples use placeholders only

- [ ] Functions are small and single-purpose; extract when a second concern appears (see Principles / skills/engineering/craft/SKILL.md)
- [ ] Type hints on all functions
- [ ] No bare `except:` clauses
- [ ] Context managers for resources
- [ ] Dataclasses for data containers
- [ ] Names match domain and local convention (skills/engineering/craft/SKILL.md)
- [ ] All tests pass
- [ ] mypy type checking passes

### Pre-Delivery

```bash
pytest
mypy .
black .
flake8
```
