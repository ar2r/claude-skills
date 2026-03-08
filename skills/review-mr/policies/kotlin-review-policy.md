# Kotlin Review Policy

Правила ревью для Kotlin кода. Основано на Kotlin best practices и idiomatic Kotlin.

---

## Null Safety

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| NS-1 | No `!!` (non-null assertion) | MAJOR | `!!` throws `NullPointerException` at runtime. Use safe call (`?.`), elvis operator (`?:`), or early return instead |
| NS-2 | Nullable parameters must be checked | MAJOR | Nullable parameters used without null-check, safe call, or contract |
| NS-3 | Platform types need explicit nullability | MINOR | Types from Java interop should have explicit `?` or non-null annotation. Unmarked platform types hide potential NPEs |
| NS-4 | `lateinit` misuse | MAJOR | `lateinit` on a type that may legitimately be null at access time. Consider nullable type with `?` instead |
| NS-5 | Unnecessary null checks | NIT | Checking non-null type for null: `if (nonNullVar != null)`. Remove redundant check |
| NS-6 | Safe call on non-null | NIT | Using `?.` on non-nullable type. Either make type nullable or use regular call |

## Correctness

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| CR-1 | Unsynchronized mutable state | CRITICAL | Mutable state accessed from multiple threads without synchronization (`@Volatile`, `Mutex`, `synchronized`, atomic) |
| CR-2 | `==` vs `===` confusion | MAJOR | Using referential equality (`===`) where structural equality (`==`) is intended, or vice versa |
| CR-3 | Scope function misuse | MINOR | Wrong scope function choice leading to confusing code (e.g., `apply` with side effects that should use `also`, `let` where `run` is clearer) |
| CR-4 | Unchecked cast | MAJOR | Using `as` instead of `as?` when the cast can fail. Leads to `ClassCastException` |
| CR-5 | Swallowed exceptions | MAJOR | Catch block that discards the exception without logging or rethrowing. Hides bugs |
| CR-6 | Smart cast impossible | MINOR | Code structure prevents smart cast. Refactor to enable smart casting |
| CR-7 | Sealed class without else | MAJOR | `when` on sealed class without exhaustive branches or `else`. Crashes on new subclass addition |
| CR-8 | Coroutine scope leak | CRITICAL | Launching coroutines without proper scope. Memory leaks and crashes |

## Idiomatics

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| ID-1 | Java-style code | NIT | Explicit getters/setters instead of properties; `if (x != null)` instead of `x?.let`; manual `equals`/`hashCode` instead of data class |
| ID-2 | `var` where `val` suffices | MINOR | Variable declared as `var` but never reassigned. Use `val` for immutability |
| ID-3 | Mutable collection where immutable suffices | MINOR | `MutableList`/`MutableMap`/`MutableSet` used but never mutated after creation. Use `List`/`Map`/`Set` |
| ID-4 | `if-else` chain instead of `when` | NIT | Three or more `if-else` branches on the same subject. Use `when` expression |
| ID-5 | Missing data class | NIT | Class used as DTO or value object without `data` modifier — missing `equals`, `hashCode`, `copy`, `toString` |
| ID-6 | Named arguments missing | MINOR | Function call with >3 parameters or boolean parameters without named arguments. Hard to understand |
| ID-7 | String templates not used | NIT | String concatenation with `+` instead of string templates: `"Hello $name"` |
| ID-8 | Explicit type where inference works | NIT | `val name: String = "John"` instead of `val name = "John"`. Let compiler infer obvious types |

## Performance

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| PF-1 | Object allocation in hot path | MINOR | Creating objects inside tight loops or frequently called functions. Consider reuse or pre-allocation |
| PF-2 | Unnecessary collection copies | MINOR | Calling `.toList()`, `.toMap()`, etc. when the original collection is already the right type and not mutated |
| PF-3 | String concatenation in loop | MINOR | Using `+` for string building inside a loop. Use `StringBuilder` or `buildString` |
| PF-4 | Sequence not used | MINOR | Chain of collection operations creating intermediate collections. Use `.asSequence()` for lazy evaluation |
| PF-5 | Inline class not used | MINOR | Wrapper class for primitive type without `@JvmInline value class`. Boxing overhead |
| PF-6 | Coroutine blocking call | MAJOR | Blocking call inside coroutine (Thread.sleep, blocking IO). Use suspending alternatives |

## Error Handling

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| EH-1 | Empty catch block | MAJOR | Catch block with no body. Exceptions are silently swallowed. At minimum, log the error |
| EH-2 | Overly broad catch | MINOR | Catching `Exception` or `Throwable` when a more specific type is appropriate. Masks unrelated errors |
| EH-3 | Missing error handling at boundary | MAJOR | No error handling around IO, network, or external API calls. These can fail and must be handled |
| EH-4 | Result type not used | MINOR | Functions that can fail returning nullable instead of `Result<T>`. Less clear error handling |
| EH-5 | Checked exceptions from Java | MAJOR | Not handling checked exceptions from Java interop. Will compile but crash at runtime |

## Coroutines

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| CO-1 | GlobalScope usage | CRITICAL | Using `GlobalScope.launch`. Coroutines never cancelled, memory leaks. Use structured concurrency |
| CO-2 | Missing coroutine scope | MAJOR | Suspending function called without proper scope. Use `coroutineScope` or structured concurrency |
| CO-3 | Blocking in coroutine | MAJOR | `Thread.sleep()`, blocking IO in suspending function. Use `delay()`, non-blocking IO |
| CO-4 | Missing Dispatchers | MINOR | CPU-intensive work on default dispatcher. Use `Dispatchers.Default` for CPU work, `Dispatchers.IO` for IO |
| CO-5 | Flow vs suspend function | MINOR | Suspending function returning single value where Flow makes more sense for multiple values |
| CO-6 | StateFlow initial value misuse | MINOR | StateFlow with nullable type when non-null with proper initial value is better |
| CO-7 | runBlocking in production | CRITICAL | Using `runBlocking` outside tests or `main()`. Blocks threads, defeats coroutines purpose |

## Collections

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| CL-1 | Wrong collection type | MINOR | Using List where Set is appropriate (unique values), or vice versa |
| CL-2 | Collection operations order | MINOR | `.map().filter()` instead of `.filter().map()`. Unnecessary transformations |
| CL-3 | Finding element inefficiently | MINOR | `.filter { }.firstOrNull()` instead of `.find { }`. Extra iteration |
| CL-4 | Mutable collection exposed | MAJOR | Returning mutable collection from public API. Breaks encapsulation |
| CL-5 | Collection builder not used | NIT | Manual collection creation with add. Use `buildList`, `buildMap`, etc. |

## Code Style

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| CS-1 | Naming conventions violated | NIT | camelCase for classes, snake_case for variables, SCREAMING_SNAKE for constants |
| CS-2 | Expression body not used | NIT | Single-expression function with explicit return. Use `= expression` |
| CS-3 | Trailing commas missing | NIT | Multi-line argument/parameter list without trailing comma. Harder diffs |
| CS-4 | Wildcard imports | MINOR | `import package.*`. Makes dependencies unclear |
| CS-5 | Unused imports | NIT | Import statements for unused classes. Code cleanup needed |
| CS-6 | Magic numbers | MINOR | Numeric literals without explanation. Use named constants |
| CS-7 | Constants import wrong | MAJOR | Importing constant holder class instead of constant directly. Import constant with `import pkg.Constants.MAX_RETRIES` |
| CS-8 | Static method import | MINOR | Importing static method instead of calling with class name. Makes code origin unclear |
| CS-9 | Wrong file naming | NIT | Files not in PascalCase.kt format. Use `UserService.kt`, not `user_service.kt` |

## Project Structure

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| PS-1 | Utils/Common/Helpers package | MAJOR | Creating `util/`, `common/`, `helper/` packages. Code dump without clear responsibility |
| PS-2 | Utils/Helper class suffix | MAJOR | Classes with `.Util`, `.Helper`, `.Common` suffix. Use specific domain names |
| PS-3 | Wrong service naming | MINOR | Service not named `DomainService`. Use `UserService`, not `UserManager` or `UserHelper` |
| PS-4 | Wrong client naming | MINOR | External client not named `SystemClient`. Use `PaymentGatewayClient`, not `PaymentGatewayService` |
| PS-5 | Extensions not in separate file | MINOR | Extension functions not in `[ClassName]Extensions.kt` file. Keep extensions organized |
| PS-6 | Constants file chaos | MAJOR | Single `Constants.kt` file with unrelated constants. Split by domain: `ValidationRules.kt`, `TimeoutConfig.kt` |
| PS-7 | Enum in external API model | MAJOR | Enum for values controlled externally. Use String or sealed class with Unknown variant |

## Examples

### NS-1: No `!!`

**Bad:**
```kotlin
fun processUser(user: User?) {
    val name = user!!.name  // NullPointerException if user is null
    println(name)
}
```

**Good:**
```kotlin
fun processUser(user: User?) {
    val name = user?.name ?: return  // Early return
    println(name)
}

// Or
fun processUser(user: User?) {
    user?.let { u ->
        println(u.name)
    }
}
```

### CR-3: Scope Function Misuse

**Bad:**
```kotlin
val user = User().apply {
    name = "John"
    age = 30
    save()  // Side effect, should use also
}

user?.let {
    calculate(it.age, it.height)  // Should use run
}
```

**Good:**
```kotlin
val user = User().apply {
    name = "John"
    age = 30
}.also { it.save() }

user?.run {
    calculate(age, height)  // Direct access to properties
}
```

### ID-2: var vs val

**Bad:**
```kotlin
var name = "John"  // Never reassigned
var age = 30       // Never reassigned
```

**Good:**
```kotlin
val name = "John"
val age = 30
```

### ID-5: Missing Data Class

**Bad:**
```kotlin
class User(val name: String, val email: String) {
    override fun equals(other: Any?): Boolean {
        // Manual equals implementation
    }
    override fun hashCode(): Int {
        // Manual hashCode implementation
    }
}
```

**Good:**
```kotlin
data class User(val name: String, val email: String)
// Gets equals, hashCode, copy, toString for free
```

### CO-1: GlobalScope Usage

**Bad:**
```kotlin
fun loadData() {
    GlobalScope.launch {
        // Never cancelled, memory leak
        val data = repository.fetchData()
        updateUI(data)
    }
}
```

**Good:**
```kotlin
// Use structured concurrency
class DataLoader(private val scope: CoroutineScope) {
    fun loadData() {
        scope.launch {
            // Cancelled when scope is cancelled
            val data = repository.fetchData()
            updateUI(data)
        }
    }
}

// Or define your own scope
class MyService {
    private val scope = CoroutineScope(SupervisorJob() + Dispatchers.Default)

    fun loadData() {
        scope.launch {
            val data = repository.fetchData()
            updateUI(data)
        }
    }

    fun cleanup() {
        scope.cancel()  // Cancel all coroutines
    }
}
```

### PF-4: Sequence Not Used

**Bad:**
```kotlin
val result = list
    .filter { it > 0 }      // Creates intermediate list
    .map { it * 2 }         // Creates another intermediate list
    .take(10)               // Creates yet another list
```

**Good:**
```kotlin
val result = list
    .asSequence()           // Lazy evaluation
    .filter { it > 0 }
    .map { it * 2 }
    .take(10)
    .toList()               // Only materialize at the end
```

### CL-4: Mutable Collection Exposed

**Bad:**
```kotlin
class UserRepository {
    private val users = mutableListOf<User>()

    fun getUsers(): MutableList<User> = users  // Callers can modify!
}
```

**Good:**
```kotlin
class UserRepository {
    private val _users = mutableListOf<User>()

    fun getUsers(): List<User> = _users  // Immutable view
}
```

### EH-4: Result Type

**Bad:**
```kotlin
suspend fun fetchUser(id: String): User? {
    return try {
        api.getUser(id)
    } catch (e: Exception) {
        null  // Lost error information
    }
}
```

**Good:**
```kotlin
suspend fun fetchUser(id: String): Result<User> {
    return try {
        Result.success(api.getUser(id))
    } catch (e: Exception) {
        Result.failure(e)  // Error information preserved
    }
}

// Usage
when (val result = fetchUser(id)) {
    is Result.Success -> handleUser(result.value)
    is Result.Failure -> handleError(result.error)
}
```

### CS-7: Constants Import Wrong

**Bad:**
```kotlin
import com.example.Constants  // Importing holder class

fun process() {
    if (retries > Constants.MAX_RETRIES) {  // Unclear where MAX_RETRIES from
        ...
    }
}
```

**Good:**
```kotlin
import com.example.Constants.MAX_RETRIES  // Direct import

fun process() {
    if (retries > MAX_RETRIES) {  // Clear constant usage
        ...
    }
}
```

### PS-1: Utils Package

**Bad:**
```kotlin
// src/main/kotlin/ru/ttech/loyalty/util/
//   - StringUtils.kt
//   - DateUtils.kt
//   - ValidationUtils.kt
//   ... code dump

object StringUtils {
    fun formatName(s: String) = ...
    fun validateEmail(s: String) = ...
    fun truncate(s: String, len: Int) = ...
}
```

**Good:**
```kotlin
// src/main/kotlin/ru/ttech/loyalty/formatting/
//   - NameFormatter.kt
//   - TextTruncator.kt
// src/main/kotlin/ru/ttech/loyalty/validation/
//   - EmailValidator.kt

// Specific, single-purpose classes
class NameFormatter {
    fun format(name: String): String = ...
}

class EmailValidator {
    fun validate(email: String): Boolean = ...
}
```

### PS-7: Enum in External API

**Bad:**
```kotlin
// External API can add new statuses - enum breaks
enum class PaymentStatus {
    SUCCESS, FAILED, PENDING
}

data class PaymentResponse(
    val status: PaymentStatus  // Crashes on unknown status!
)
```

**Good:**
```kotlin
// String or sealed class with Unknown
data class PaymentResponse(
    val status: String  // Safe for unknown values
)

// Or sealed class
sealed class PaymentStatus {
    object Success : PaymentStatus()
    object Failed : PaymentStatus()
    object Pending : PaymentStatus()
    data class Unknown(val value: String) : PaymentStatus()
}
```

## Scope Functions Quick Reference

| Function | Context Object | Return Value | Use Case |
|----------|---------------|--------------|----------|
| `let` | `it` | Lambda result | Null safety, chaining |
| `run` | `this` | Lambda result | Object config + result |
| `with` | `this` | Lambda result | Multiple calls on object |
| `apply` | `this` | Context object | Object configuration |
| `also` | `it` | Context object | Side effects |

**Rule of thumb:**
- Need result? `let`, `run`, `with`
- Need object? `apply`, `also`
- Want `this`? `run`, `apply`, `with`
- Want `it`? `let`, `also`

## Detection Tips

- Search for: `!!`, `GlobalScope`, `runBlocking`
- Check nullable types without null handling
- Look for `var` that could be `val`
- Find classes without `data` modifier that should have it
- Check coroutine launches for proper scope
- Verify collections: mutable where immutable works
- Find packages: `util/`, `common/`, `helper/`
- Check class names: `*Util`, `*Helper`, `*Common`
- Verify imports: importing constant holder classes instead of constants directly
- Look for single `Constants.kt` file with many unrelated constants
- Check enum usage in API models for external data
- Verify service naming: should be `*Service`, not `*Manager` or `*Helper`
- Check extensions: should be in `[ClassName]Extensions.kt` files
