# Tests Review Policy

Правила для ревью тестового кода. Применяй к файлам с тестами (test_*.py, *_test.py).

---

## Test Coverage

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| TC-1 | Missing tests for new code | MAJOR | New public functions/methods without corresponding tests |
| TC-2 | Missing edge cases | MAJOR | No tests for: empty input, None, 0, negative numbers, boundary values |
| TC-3 | Only happy path tested | MAJOR | Tests only for successful scenarios, no error cases |
| TC-4 | Missing integration tests | MINOR | Only unit tests, no tests for component interaction |
| TC-5 | Incomplete mock coverage | MINOR | External dependencies not mocked (DB, API, file system) |

## Test Quality

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| TQ-1 | No assertions | CRITICAL | Test function without assert statements or expected exceptions |
| TQ-2 | Weak assertions | MAJOR | Generic assertions like `assert result` instead of specific checks |
| TQ-3 | Multiple concepts per test | MINOR | Test checking multiple unrelated things. Split into separate tests |
| TQ-4 | Poor test naming | NIT | Names like `test_1` or `test_function`. Use descriptive names |
| TQ-5 | Unclear test intent | MINOR | Hard to understand what behavior is being tested |
| TQ-6 | Testing implementation details | MAJOR | Test knows too much about internals. Test behavior, not implementation |
| TQ-7 | Fragile tests | MAJOR | Tests break on any refactoring even if behavior unchanged |

## Test Structure

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| TS-1 | Missing AAA structure | MINOR | Tests without clear Arrange-Act-Assert sections |
| TS-2 | Setup in test body | MINOR | Repeated setup code in each test. Use fixtures/setUp |
| TS-3 | Magic numbers in tests | MINOR | Unclear test data. Use named constants or descriptive variables |
| TS-4 | Excessive mocking | MAJOR | Mocking everything including standard library. Tests become meaningless |
| TS-5 | God test | MAJOR | Single test >50 lines. Split into smaller tests |

## Test Independence

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| TI-1 | Test interdependence | CRITICAL | Tests depend on execution order or state from previous tests |
| TI-2 | Shared mutable state | MAJOR | Tests modifying global/class variables that affect other tests |
| TI-3 | Missing cleanup | MAJOR | Tests creating files, DB records, etc. without cleanup |
| TI-4 | Time-dependent tests | MAJOR | Tests using current time/date. Use freezegun or mock time |
| TI-5 | Random data in tests | MAJOR | Tests with random values. Use fixed seeds or deterministic data |

## Test Fixtures & Mocks

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| TF-1 | Fixture overuse | MINOR | Complex fixture hierarchies. Keep fixtures simple |
| TF-2 | Fixture scope issues | MAJOR | Session/module fixtures with mutable state. Use function scope |
| TF-3 | Mock without assertions | MAJOR | Creating mocks but not verifying they were called correctly |
| TF-4 | Overly specific mocks | MINOR | Mocks tied to exact call signatures. Tests become brittle |
| TF-5 | Mock return None | MAJOR | Mock not configured to return expected values |

## Pytest Specific

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| PT-1 | Not using parametrize | MINOR | Copy-pasted tests with different data. Use `@pytest.mark.parametrize` |
| PT-2 | Incorrect fixture scope | MINOR | Function fixture that could be module/session for performance |
| PT-3 | Not using pytest idioms | NIT | `self.assertEqual` instead of `assert`. Use plain assert with pytest |
| PT-4 | Missing fixture cleanup | MAJOR | Fixture without proper yield/finalizer for cleanup |
| PT-5 | Fixture side effects | MAJOR | Fixture modifying global state or files without restoration |

## Data Testing (Pandas/Polars)

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| DT-1 | Not testing empty DataFrames | MAJOR | No tests for empty input data |
| DT-2 | Not testing schema | MAJOR | No validation of column names and types in result |
| DT-3 | Float comparison without tolerance | MAJOR | `assert df["col"] == 0.1` fails due to float precision. Use `np.isclose` |
| DT-4 | Not testing null handling | MAJOR | No tests for None/NaN values in data |
| DT-5 | Ignoring row order | MINOR | Comparing DataFrames without sorting. Tests flaky if order varies |

## Examples

### TQ-1: No Assertions

**Bad:**
```python
def test_user_creation():
    user = create_user("John", "john@example.com")
    # No assertion!
```

**Good:**
```python
def test_user_creation():
    user = create_user("John", "john@example.com")
    assert user.name == "John"
    assert user.email == "john@example.com"
    assert user.id is not None
```

### TC-2: Missing Edge Cases

**Bad:**
```python
def test_divide():
    assert divide(10, 2) == 5
```

**Good:**
```python
def test_divide_normal():
    assert divide(10, 2) == 5

def test_divide_by_zero():
    with pytest.raises(ZeroDivisionError):
        divide(10, 0)

def test_divide_negative():
    assert divide(-10, 2) == -5

def test_divide_floats():
    assert divide(10, 3) == pytest.approx(3.333, rel=1e-3)
```

### TI-1: Test Interdependence

**Bad:**
```python
# Global state shared between tests!
user = None

def test_create_user():
    global user
    user = create_user("John")
    assert user.name == "John"

def test_update_user():
    # Depends on test_create_user running first!
    global user
    update_user(user, name="Jane")
    assert user.name == "Jane"
```

**Good:**
```python
@pytest.fixture
def user():
    return create_user("John")

def test_create_user():
    user = create_user("John")
    assert user.name == "John"

def test_update_user(user):
    update_user(user, name="Jane")
    assert user.name == "Jane"
```

### TQ-4: Poor Test Naming

**Bad:**
```python
def test_1():
    ...

def test_user():
    ...

def test_case_2():
    ...
```

**Good:**
```python
def test_user_creation_with_valid_email_succeeds():
    ...

def test_user_creation_with_invalid_email_raises_validation_error():
    ...

def test_user_update_changes_name_field():
    ...
```

### PT-1: Not Using Parametrize

**Bad:**
```python
def test_add_positive():
    assert add(2, 3) == 5

def test_add_negative():
    assert add(-2, -3) == -5

def test_add_zero():
    assert add(0, 5) == 5
```

**Good:**
```python
@pytest.mark.parametrize("a,b,expected", [
    (2, 3, 5),
    (-2, -3, -5),
    (0, 5, 5),
    (10, -10, 0),
])
def test_add(a, b, expected):
    assert add(a, b) == expected
```

### DT-3: Float Comparison

**Bad:**
```python
def test_score_calculation():
    df = calculate_scores(data)
    assert df["score"][0] == 0.123456789  # Fails due to precision!
```

**Good:**
```python
import numpy as np

def test_score_calculation():
    df = calculate_scores(data)
    assert np.isclose(df["score"][0], 0.123456789)
    # Or with pytest
    assert df["score"][0] == pytest.approx(0.123456789)
```

### TI-3: Missing Cleanup

**Bad:**
```python
def test_file_processing():
    with open("test_output.txt", "w") as f:
        f.write("data")
    result = process_file("test_output.txt")
    assert result == "processed"
    # File left behind!
```

**Good:**
```python
import tempfile
import os

def test_file_processing():
    with tempfile.NamedTemporaryFile(mode="w", delete=False) as f:
        f.write("data")
        filepath = f.name

    try:
        result = process_file(filepath)
        assert result == "processed"
    finally:
        os.unlink(filepath)

# Or use pytest tmp_path fixture
def test_file_processing(tmp_path):
    filepath = tmp_path / "test_output.txt"
    filepath.write_text("data")
    result = process_file(filepath)
    assert result == "processed"
    # Cleanup automatic
```

### DT-1: Not Testing Empty DataFrames

**Bad:**
```python
def test_filter_users():
    df = pl.DataFrame({"name": ["Alice", "Bob"], "age": [25, 30]})
    result = filter_users(df, min_age=20)
    assert len(result) == 2
```

**Good:**
```python
def test_filter_users_normal():
    df = pl.DataFrame({"name": ["Alice", "Bob"], "age": [25, 30]})
    result = filter_users(df, min_age=20)
    assert len(result) == 2

def test_filter_users_empty_input():
    df = pl.DataFrame({"name": [], "age": []})
    result = filter_users(df, min_age=20)
    assert result.is_empty()

def test_filter_users_no_matches():
    df = pl.DataFrame({"name": ["Alice"], "age": [18]})
    result = filter_users(df, min_age=20)
    assert result.is_empty()
```

## Best Practices

1. **One assertion per test** (or assertions testing same concept)
2. **Test behavior, not implementation** - don't test private methods
3. **Tests should be fast** - use mocks for slow operations
4. **Tests should be deterministic** - no randomness, no time dependencies
5. **Tests should be independent** - can run in any order
6. **Test names describe behavior** - should read like documentation
7. **AAA pattern** - Arrange, Act, Assert clearly separated
