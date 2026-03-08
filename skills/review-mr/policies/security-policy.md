# Security Review Policy

Правила безопасности для всех языков. Применяй к любому коду независимо от языка.

---

## Injection Vulnerabilities

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| SEC-1 | SQL injection | CRITICAL | String concatenation or f-strings in SQL queries. Use parameterized queries |
| SEC-2 | Command injection | CRITICAL | User input in `os.system()`, `subprocess.call()`, etc. without sanitization |
| SEC-3 | Path traversal | CRITICAL | User input in file paths without validation. Can access sensitive files |
| SEC-4 | LDAP injection | CRITICAL | User input in LDAP queries without escaping |
| SEC-5 | XPath injection | CRITICAL | User input in XPath queries without proper escaping |
| SEC-6 | Template injection | CRITICAL | User input directly in template strings (Jinja2, Mako, etc.) |
| SEC-7 | Code injection | CRITICAL | Using `eval()`, `exec()` with user-controlled input |

## XSS (Cross-Site Scripting)

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| XSS-1 | Reflected XSS | CRITICAL | User input rendered in HTML without escaping |
| XSS-2 | Stored XSS | CRITICAL | Saved user input displayed without sanitization |
| XSS-3 | DOM-based XSS | CRITICAL | Client-side JS using user input in DOM manipulation |
| XSS-4 | Unsafe HTML generation | MAJOR | Manual HTML string building. Use template engine with auto-escaping |

## Authentication & Authorization

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| AUTH-1 | Missing authentication | CRITICAL | Sensitive endpoints without authentication check |
| AUTH-2 | Missing authorization | CRITICAL | No check if user has permission to access resource |
| AUTH-3 | Weak password policy | MAJOR | No password complexity requirements or length validation |
| AUTH-4 | Hardcoded credentials | CRITICAL | Passwords, API keys, tokens in source code |
| AUTH-5 | Credentials in logs | CRITICAL | Logging passwords, tokens, or sensitive data |
| AUTH-6 | Insufficient session timeout | MAJOR | Session tokens without expiration or very long expiration |
| AUTH-7 | Insecure password storage | CRITICAL | Storing passwords as plaintext or with weak hashing (MD5, SHA1) |
| AUTH-8 | Missing rate limiting | MAJOR | Login/API endpoints without rate limiting. Enables brute force |

## Data Exposure

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| EXP-1 | Sensitive data in response | CRITICAL | Passwords, tokens, internal IDs in API responses |
| EXP-2 | Sensitive data in logs | CRITICAL | Logging PII, credentials, credit cards, etc. |
| EXP-3 | Detailed error messages | MAJOR | Stack traces, database errors exposed to users |
| EXP-4 | IDOR vulnerability | CRITICAL | Direct object references without authorization check |
| EXP-5 | Missing data encryption | MAJOR | Sensitive data stored or transmitted without encryption |
| EXP-6 | Weak encryption | MAJOR | Using deprecated algorithms (DES, RC4, MD5) |

## Input Validation

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| VAL-1 | Missing input validation | MAJOR | No validation of user input (length, type, format) |
| VAL-2 | Client-side only validation | MAJOR | Validation only on frontend. Always validate on backend |
| VAL-3 | No size limits | MAJOR | File uploads or requests without size limits. DoS risk |
| VAL-4 | Unsafe deserialization | CRITICAL | Deserializing untrusted data (pickle, YAML with unsafe loader) |
| VAL-5 | XML external entities | CRITICAL | XML parser allowing external entities. Can read local files |
| VAL-6 | Mass assignment | MAJOR | Binding all request fields to model without whitelist |

## Cryptography

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| CRY-1 | Weak random number generation | MAJOR | Using `random` instead of `secrets` for security-sensitive values |
| CRY-2 | Hardcoded secrets | CRITICAL | Encryption keys, salts, IV hardcoded in source |
| CRY-3 | Weak hashing | MAJOR | Using MD5, SHA1 for passwords. Use bcrypt, argon2, or scrypt |
| CRY-4 | No salt for passwords | CRITICAL | Hashing passwords without salt. Vulnerable to rainbow tables |
| CRY-5 | Predictable tokens | MAJOR | Using sequential or timestamp-based IDs for sensitive tokens |

## Miscellaneous

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| MIS-1 | Debug mode in production | CRITICAL | `DEBUG=True` or debug endpoints in production code |
| MIS-2 | CORS misconfiguration | MAJOR | `Access-Control-Allow-Origin: *` for authenticated endpoints |
| MIS-3 | Insecure dependencies | MAJOR | Using libraries with known vulnerabilities |
| MIS-4 | Race conditions | MAJOR | TOCTOU bugs, concurrent access to shared resources |
| MIS-5 | Unsafe redirects | MAJOR | Open redirect using unvalidated user input |
| MIS-6 | Missing CSRF protection | MAJOR | State-changing operations without CSRF token |
| MIS-7 | Directory listing enabled | MINOR | Web server exposing directory contents |
| MIS-8 | Backup files accessible | MINOR | `.bak`, `.old`, `.swp` files in web root |

## Examples

### SEC-1: SQL Injection

**Bad:**
```python
# CRITICAL - SQL injection!
query = f"SELECT * FROM users WHERE id = {user_id}"
cursor.execute(query)

# Also bad
query = "SELECT * FROM users WHERE name = '" + username + "'"
```

**Good:**
```python
# Parameterized query
query = "SELECT * FROM users WHERE id = %s"
cursor.execute(query, (user_id,))

# Or with ORM
user = User.query.filter_by(id=user_id).first()
```

### SEC-2: Command Injection

**Bad:**
```python
# CRITICAL - command injection!
os.system(f"ping {user_ip}")
subprocess.call(f"tar -xzf {filename}", shell=True)
```

**Good:**
```python
# Use list form, no shell
subprocess.run(["ping", "-c", "1", user_ip])
subprocess.run(["tar", "-xzf", filename])

# Or validate/sanitize input
if re.match(r'^[0-9.]+$', user_ip):
    subprocess.run(["ping", "-c", "1", user_ip])
```

### SEC-3: Path Traversal

**Bad:**
```python
# CRITICAL - path traversal!
# user_file could be "../../etc/passwd"
with open(f"/uploads/{user_file}") as f:
    return f.read()
```

**Good:**
```python
from pathlib import Path

base_dir = Path("/uploads")
file_path = (base_dir / user_file).resolve()

# Check path is inside base_dir
if not file_path.is_relative_to(base_dir):
    raise ValueError("Invalid file path")

with open(file_path) as f:
    return f.read()
```

### AUTH-4: Hardcoded Credentials

**Bad:**
```python
# CRITICAL - hardcoded secrets!
API_KEY = "sk-1234567890abcdef"
db_password = "MyP@ssw0rd123"
```

**Good:**
```python
import os
from airflow.models import Variable

# From environment
API_KEY = os.environ["API_KEY"]

# Or from Airflow Variables
db_password = Variable.get("database_password")

# Or from AWS Secrets Manager, HashiCorp Vault, etc.
```

### VAL-4: Unsafe Deserialization

**Bad:**
```python
import pickle
import yaml

# CRITICAL - pickle arbitrary code execution!
data = pickle.loads(user_data)

# CRITICAL - YAML arbitrary code execution!
config = yaml.load(user_config)
```

**Good:**
```python
import json
import yaml

# Use safe formats
data = json.loads(user_data)

# Or safe YAML loader
config = yaml.safe_load(user_config)
```

### CRY-1: Weak Random

**Bad:**
```python
import random

# BAD - predictable for security!
token = random.randint(1000, 9999)
session_id = "".join(random.choices("0123456789", k=16))
```

**Good:**
```python
import secrets

# Cryptographically secure
token = secrets.randbelow(9000) + 1000
session_id = secrets.token_hex(16)
```

### EXP-2: Sensitive Data in Logs

**Bad:**
```python
# CRITICAL - password in logs!
logger.info(f"User login: {username}, password: {password}")
logger.debug(f"API call with token: {api_token}")
```

**Good:**
```python
# Never log secrets
logger.info(f"User login: {username}")
logger.debug(f"API call with token: {api_token[:8]}...")  # Only prefix

# Or use structured logging with redaction
logger.info("User login", extra={"username": username})
```

## Detection Tips

- Search for: `eval(`, `exec(`, `os.system(`, `subprocess.*shell=True`
- Search for: `f"SELECT`, `"SELECT" +`, `.format()` in SQL context
- Search for: password, token, secret, key + `=` + string literal
- Check all places handling user input: request params, body, headers, files
- Check file operations with user-controlled paths
- Check authentication decorators on new endpoints
