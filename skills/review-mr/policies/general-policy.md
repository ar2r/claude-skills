# General Code Quality Policy

Общие правила качества кода для любого языка программирования.

---

## Code Complexity

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| CC-1 | High cyclomatic complexity | MAJOR | Function with >10 decision points. Split into smaller functions |
| CC-2 | Deep nesting | MAJOR | More than 3 levels of nesting (if/for/while). Extract logic |
| CC-3 | Long function | MAJOR | Function >50 lines. Extract helper functions |
| CC-4 | God class | MAJOR | Class with >500 lines or >20 methods. Split responsibilities |
| CC-5 | Long parameter list | MINOR | Function with >5 parameters. Use object or dataclass |
| CC-6 | Too many variables | MINOR | Function with >10 local variables. Extract smaller functions |

## Maintainability

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| MT-1 | Code duplication | MAJOR | Same logic repeated >2 times. Extract to function |
| MT-2 | Magic numbers | MINOR | Numeric literals without explanation. Use named constants |
| MT-3 | Commented code | MINOR | Large blocks of commented code. Remove or explain why kept |
| MT-4 | TODO/FIXME without ticket | MINOR | TODO without issue number or explanation |
| MT-5 | Unclear variable names | MINOR | Names like `x`, `tmp`, `data`. Use domain-specific names |
| MT-6 | Inconsistent naming | MINOR | Mixing naming conventions in same module |

## Error Handling

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| EH-1 | Silent error swallowing | MAJOR | Catch without logging or re-raising. Bugs will be hidden |
| EH-2 | Generic exception catching | MINOR | Catching base Exception class. Be specific |
| EH-3 | Missing error handling | MAJOR | Operations that can fail (IO, network, parsing) without try-catch |
| EH-4 | Error without context | MINOR | Raising exception without helpful message |
| EH-5 | Returning null on error | MAJOR | Returning None/null instead of raising exception. Hides errors |

## Documentation

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| DOC-1 | Missing docstrings | MINOR | Public functions/classes without documentation |
| DOC-2 | Outdated documentation | MINOR | Comments contradicting code behavior |
| DOC-3 | Obvious comments | NIT | Comments stating what code clearly does: `i++  // increment i` |
| DOC-4 | Missing return documentation | MINOR | Function returning non-obvious value without documenting it |
| DOC-5 | Missing parameter documentation | MINOR | Non-obvious parameters without description |

## Code Smells

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| CS-1 | Dead code | MINOR | Unreachable code or unused functions |
| CS-2 | Speculative generality | MINOR | Over-engineering for future that may never come |
| CS-3 | Feature envy | MINOR | Method using data from another class more than its own |
| CS-4 | Primitive obsession | MINOR | Using primitives instead of domain objects (string for email, int for money) |
| CS-5 | Data clumps | MINOR | Same group of parameters passed together. Create object |
| CS-6 | Switch statements | MINOR | Long switch/if-else chains. Use polymorphism or lookup table |

## Concurrency

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| CN-1 | Race condition | CRITICAL | Multiple threads accessing shared mutable state without synchronization |
| CN-2 | Deadlock potential | MAJOR | Multiple locks acquired in inconsistent order |
| CN-3 | Blocking in async | MAJOR | Synchronous IO in async function |
| CN-4 | Thread-unsafe singleton | MAJOR | Singleton initialization not thread-safe |

## Resource Management

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| RM-1 | Resource leak | CRITICAL | File/connection/memory not closed/freed |
| RM-2 | No timeout | MAJOR | Network call or blocking operation without timeout |
| RM-3 | Unbounded growth | MAJOR | Cache or collection growing without limit |
| RM-4 | Expensive operation in loop | MAJOR | Creating connections, compiling regex, etc. inside loop |

## API Design

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| API-1 | Inconsistent API | MINOR | Similar functions with different parameter order or naming |
| API-2 | Boolean trap | MINOR | Function with boolean parameter. Unclear what true/false means |
| API-3 | Breaking API change | CRITICAL | Removing or changing public function signature without versioning |
| API-4 | Mutable return values | MAJOR | Returning internal mutable state. Breaks encapsulation |

## Examples

### CC-2: Deep Nesting

**Bad:**
```python
def process_order(order):
    if order.user:
        if order.user.is_active:
            if order.items:
                for item in order.items:
                    if item.in_stock:
                        if item.price > 0:
                            # Deep nesting hard to follow
                            process_item(item)
```

**Good:**
```python
def process_order(order):
    if not order.user or not order.user.is_active:
        return

    if not order.items:
        return

    valid_items = [
        item for item in order.items
        if item.in_stock and item.price > 0
    ]

    for item in valid_items:
        process_item(item)
```

### MT-1: Code Duplication

**Bad:**
```python
def export_users_csv():
    users = User.query.all()
    with open("users.csv", "w") as f:
        writer = csv.writer(f)
        writer.writerow(["ID", "Name", "Email"])
        for user in users:
            writer.writerow([user.id, user.name, user.email])

def export_orders_csv():
    orders = Order.query.all()
    with open("orders.csv", "w") as f:
        writer = csv.writer(f)
        writer.writerow(["ID", "User", "Total"])
        for order in orders:
            writer.writerow([order.id, order.user_id, order.total])
```

**Good:**
```python
def export_to_csv(filename, headers, rows):
    with open(filename, "w") as f:
        writer = csv.writer(f)
        writer.writerow(headers)
        writer.writerows(rows)

def export_users_csv():
    users = User.query.all()
    rows = [(u.id, u.name, u.email) for u in users]
    export_to_csv("users.csv", ["ID", "Name", "Email"], rows)

def export_orders_csv():
    orders = Order.query.all()
    rows = [(o.id, o.user_id, o.total) for o in orders]
    export_to_csv("orders.csv", ["ID", "User", "Total"], rows)
```

### MT-2: Magic Numbers

**Bad:**
```python
if user.age < 18:
    return False

if len(password) < 8:
    return False

if order.total > 1000000:
    require_approval(order)
```

**Good:**
```python
MIN_AGE = 18
MIN_PASSWORD_LENGTH = 8
APPROVAL_THRESHOLD = 1_000_000

if user.age < MIN_AGE:
    return False

if len(password) < MIN_PASSWORD_LENGTH:
    return False

if order.total > APPROVAL_THRESHOLD:
    require_approval(order)
```

### EH-3: Missing Error Handling

**Bad:**
```python
def load_config():
    # File might not exist, JSON might be invalid
    with open("config.json") as f:
        return json.load(f)

def fetch_user(user_id):
    # Network can fail, API can return error
    response = requests.get(f"{API_URL}/users/{user_id}")
    return response.json()
```

**Good:**
```python
def load_config():
    try:
        with open("config.json") as f:
            return json.load(f)
    except FileNotFoundError:
        logger.error("config.json not found")
        return default_config()
    except json.JSONDecodeError as e:
        logger.error(f"Invalid JSON in config: {e}")
        raise

def fetch_user(user_id):
    try:
        response = requests.get(
            f"{API_URL}/users/{user_id}",
            timeout=5
        )
        response.raise_for_status()
        return response.json()
    except requests.Timeout:
        logger.error(f"Timeout fetching user {user_id}")
        raise
    except requests.HTTPError as e:
        logger.error(f"HTTP error fetching user {user_id}: {e}")
        raise
```

### API-2: Boolean Trap

**Bad:**
```python
# What does True mean here?
send_email(user, True)
process_order(order, False)
```

**Good:**
```python
# Clear what parameters mean
send_email(user, include_attachments=True)
process_order(order, skip_validation=False)

# Or use enum
class ProcessingMode(Enum):
    VALIDATE = 1
    SKIP_VALIDATION = 2

process_order(order, mode=ProcessingMode.VALIDATE)
```

### RM-1: Resource Leak

**Bad:**
```python
def process_file(filename):
    f = open(filename)
    data = f.read()
    # If processing throws exception, file never closed!
    result = expensive_processing(data)
    f.close()
    return result
```

**Good:**
```python
def process_file(filename):
    with open(filename) as f:
        data = f.read()
        return expensive_processing(data)
    # File automatically closed even if exception
```

### CN-1: Race Condition

**Bad:**
```python
class Counter:
    def __init__(self):
        self.value = 0

    def increment(self):
        # Race condition! Multiple threads can read same value
        current = self.value
        self.value = current + 1
```

**Good:**
```python
import threading

class Counter:
    def __init__(self):
        self.value = 0
        self.lock = threading.Lock()

    def increment(self):
        with self.lock:
            self.value += 1

# Or use atomic operations
from threading import atomic

class Counter:
    def __init__(self):
        self.value = atomic.AtomicInteger(0)

    def increment(self):
        self.value.increment()
```

## Severity Guidelines

- **CRITICAL**: Must fix before merge - security, data loss, crashes
- **MAJOR**: Should fix before merge - bugs, poor performance, maintainability issues
- **MINOR**: Nice to fix - code quality improvements
- **NIT**: Optional - style preferences, minor improvements
