# Python Review Policy

Правила ревью для Python кода. Применяй все подходящие правила к коду.

---

## Best Practices

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| BP-1 | Type hints missing | MINOR | Public functions and methods should have type hints for parameters and return values |
| BP-2 | Docstring missing | MINOR | Public classes and functions should have docstrings (Google or NumPy style) |
| BP-3 | Magic numbers | MINOR | Numeric constants should be named constants or config values |
| BP-4 | Deep nesting | MAJOR | More than 3 levels of nesting. Extract to separate functions |
| BP-5 | Long function | MAJOR | Function >50 lines. Split into smaller functions |
| BP-6 | Too many parameters | MINOR | Function with >5 parameters. Use dataclass or dict |
| BP-7 | Mutable default arguments | CRITICAL | Using mutable defaults (`def foo(x=[]):`) causes shared state bugs |
| BP-8 | Global variables | MAJOR | Global mutable state. Use classes, closures, or dependency injection |

## Correctness

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| CR-1 | Bare except | MAJOR | `except:` catches all exceptions including KeyboardInterrupt. Use specific exceptions |
| CR-2 | Exception swallowing | MAJOR | Catch block without logging or re-raising. Bugs will be hidden |
| CR-3 | LBYL instead of EAFP | MINOR | Checking before accessing (Java style) instead of try-except (Python style) |
| CR-4 | Incorrect comparison | CRITICAL | Using `is` for value comparison instead of `==`. Only use `is` for None/True/False |
| CR-5 | Missing None check | MAJOR | Using Optional[T] result without None check |
| CR-6 | Incorrect dict access | MINOR | Using `dict[key]` when key might not exist. Use `.get()` or check first |
| CR-7 | List modification during iteration | CRITICAL | Modifying list while iterating. Use list comprehension or iterate over copy |
| CR-8 | Closure variable capture | MAJOR | Lambda in loop capturing wrong variable. Use default argument or comprehension |

## Performance

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| PF-1 | N+1 queries | CRITICAL | Database queries in loop. Use bulk fetch or join |
| PF-2 | Inefficient list operations | MAJOR | Using `+=` or `append()` in loop. Use list comprehension or pre-allocate |
| PF-3 | Repeated string concatenation | MINOR | Using `+` for strings in loop. Use `"".join()` or f-string with list |
| PF-4 | Unnecessary list creation | MINOR | Creating full list when generator would work: `[x for x in ...]` → `(x for x in ...)` |
| PF-5 | Pandas anti-patterns | MAJOR | `iterrows()`, `apply()` with lambda, or row-by-row operations. Use vectorized operations |
| PF-6 | Polars DataFrame copy | MINOR | Unnecessary `.clone()` or operations that don't use lazy evaluation |
| PF-7 | Re-compiling regex | MINOR | `re.search()` in loop. Pre-compile with `re.compile()` |
| PF-8 | Synchronous IO in loop | CRITICAL | Sync API calls in loop. Use async or batch requests |

## Data Processing

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| DP-1 | Missing data validation | MAJOR | No validation of external data (API, file, database) |
| DP-2 | Unsafe DataFrame operations | MAJOR | No check for empty DataFrame before operations that can fail |
| DP-3 | Missing null handling | MAJOR | No handling of null/None values in data processing |
| DP-4 | Incorrect aggregation | MAJOR | Aggregation without grouping or wrong aggregation function |
| DP-5 | Memory inefficiency | MAJOR | Loading entire dataset into memory when streaming would work |
| DP-6 | Missing encoding specification | MINOR | Reading files without explicit encoding (default is system-dependent) |

## Testing

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| TS-1 | No tests for new code | MAJOR | New public functions without tests |
| TS-2 | Test naming | NIT | Test names should describe behavior: `test_user_creation_with_invalid_email_raises_error` |
| TS-3 | Missing edge cases | MAJOR | No tests for empty input, None, boundary values |
| TS-4 | No assertions | CRITICAL | Test without assertions or expected exceptions |
| TS-5 | Test interdependence | MAJOR | Tests depend on execution order or shared state |
| TS-6 | Mocking internal methods | MINOR | Mocking private methods instead of dependencies. Tests become brittle |
| TS-7 | Missing cleanup | MAJOR | No cleanup of created files, database records, or resources |

## Code Style

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| CS-1 | PEP 8 violations | NIT | Line length >120, inconsistent spacing, wrong quotes |
| CS-2 | Poor naming | MINOR | Non-descriptive names: `data`, `tmp`, `x`, `foo`. Use domain terms |
| CS-3 | Inconsistent naming | MINOR | Mixing `snake_case` and `camelCase` in Python code |
| CS-4 | Complex comprehension | MINOR | Nested or multi-line comprehensions. Use regular loops for readability |
| CS-5 | Import organization | NIT | Imports not organized: stdlib, third-party, local. Use isort |
| CS-6 | Wildcard imports | MAJOR | `from module import *`. Makes dependencies unclear |
| CS-7 | Commented code | MINOR | Large blocks of commented code. Remove or use feature flags |

## Idiomatic Python

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| PY-1 | Not using context managers | MAJOR | Manual file/connection close. Use `with` statement |
| PY-2 | Not using pathlib | MINOR | String manipulation for paths. Use `pathlib.Path` |
| PY-3 | Not using enumerate | NIT | Manual index tracking: `for i in range(len(lst))`. Use `enumerate()` |
| PY-4 | Not using zip | NIT | Iterating with indices to access parallel lists. Use `zip()` |
| PY-5 | Not using dataclasses | MINOR | Class with just `__init__` and attributes. Use `@dataclass` |
| PY-6 | Dict/list construction | NIT | Manual dict building in loop. Use dict/list comprehension |
| PY-7 | Boolean comparison | NIT | `if x == True:`. Just use `if x:` |
| PY-8 | Useless else after return | NIT | `else` after `return`. Remove else, dedent code |

## Project Structure

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| PS-1 | Utils/Common/Helpers package | MAJOR | Creating `utils/`, `common/`, `helpers/` packages. Code dump without clear responsibility |
| PS-2 | Constants file chaos | MAJOR | Single `constants.py` with unrelated constants. Split by domain: `validation_rules.py`, `timeout_config.py` |
| PS-3 | Wrong file naming | NIT | Files not in snake_case.py format. Use `user_service.py`, not `UserService.py` |
| PS-4 | Missing __init__.py | MINOR | Package without `__init__.py`. Makes imports unclear |
| PS-5 | Circular imports | MAJOR | Modules importing each other. Refactor to remove circular dependency |

## Imports

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| IM-1 | Wildcard imports | MAJOR | `from module import *`. Makes dependencies unclear, name conflicts |
| IM-2 | Relative imports overuse | MINOR | Using relative imports (`from .module`) when absolute is clearer |
| IM-3 | Import order wrong | NIT | Imports not grouped: stdlib, external, local. Use isort |
| IM-4 | Unused imports | NIT | Import statements for unused modules. Remove with autoflake |
| IM-5 | Multiple imports per line | NIT | `import os, sys` instead of separate lines (except `from typing import A, B`) |

## Airflow Specific

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| AF-1 | Dynamic DAG generation issues | CRITICAL | DAG parsing code that requires imports not available in scheduler environment |
| AF-2 | Heavy imports in DAG file | MAJOR | Importing heavy libraries (pandas, numba) at module level. Use lazy imports |
| AF-3 | Global variables in tasks | MAJOR | Using global variables that change between task runs |
| AF-4 | Missing idempotency | CRITICAL | Task not idempotent - reruns produce different results or fail |
| AF-5 | No error handling in tasks | MAJOR | Task without proper exception handling and logging |
| AF-6 | Incorrect dependencies | MAJOR | Tasks with missing or wrong dependencies (`>>` operator) |
| AF-7 | Secrets in code | CRITICAL | Hardcoded credentials instead of Airflow Variables/Connections |

## Examples

### BP-7: Mutable default arguments

**Bad:**
```python
def add_item(item, items=[]):
    items.append(item)
    return items

# Bug: same list reused
add_item(1)  # [1]
add_item(2)  # [1, 2] - not [2]!
```

**Good:**
```python
def add_item(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items
```

### PF-1: N+1 queries

**Bad:**
```python
users = User.query.all()
for user in users:
    # N+1 queries!
    orders = Order.query.filter_by(user_id=user.id).all()
    process(orders)
```

**Good:**
```python
users = User.query.options(
    joinedload(User.orders)
).all()
for user in users:
    process(user.orders)
```

### PS-1: Utils Package

**Bad:**
```python
# utils/
#   string_utils.py
#   date_utils.py
#   validation_utils.py
# ... code dump

# utils/string_utils.py
def format_name(s):
    ...

def validate_email(s):
    ...

def truncate(s, length):
    ...
```

**Good:**
```python
# formatting/
#   name_formatter.py
#   text_truncator.py
# validation/
#   email_validator.py

# formatting/name_formatter.py
class NameFormatter:
    @staticmethod
    def format(name: str) -> str:
        ...

# validation/email_validator.py
class EmailValidator:
    @staticmethod
    def validate(email: str) -> bool:
        ...
```

### IM-1: Wildcard Imports

**Bad:**
```python
from models import *  # What does this import?
from utils import *   # Name conflicts possible

user = User()  # Where is User from?
```

**Good:**
```python
from models import User, Order
from utils import format_date, validate_email

user = User()  # Clear where it comes from
```

### AF-2: Heavy imports

**Bad:**
```python
import pandas as pd
import numba

def my_dag():
    # This runs during DAG parsing!
    ...
```

**Good:**
```python
def my_task():
    # Import inside task
    import pandas as pd
    import numba
    ...
```