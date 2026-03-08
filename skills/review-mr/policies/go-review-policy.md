# Go Review Policy

Правила ревью для Go кода. Основано на Google Go Style Guide и best practices.

---

## Code Organization

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| CO-1 | Utils/Common/Helpers package | MAJOR | Creating `utils/`, `common/`, `helpers/` packages. Code dump without clear responsibility |
| CO-2 | Models package | MAJOR | Creating generic `models/` package. Types should be in functional packages |
| CO-3 | Wrong package naming | MINOR | Package names not lowercase single word. Use `repomap`, not `repoMap` or `repo_map` |
| CO-4 | Package name mismatch | MINOR | Package name doesn't match directory name |
| CO-5 | Too many files in package | MINOR | Package with >15 files. Consider splitting by subdomain |

## Naming Conventions

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| NM-1 | Wrong export naming | MAJOR | Exported names not PascalCase. Use `GenerateDigest`, not `generateDigest` |
| NM-2 | Wrong private naming | MAJOR | Private names not camelCase. Use `extractSymbols`, not `ExtractSymbols` |
| NM-3 | Acronym inconsistency | MINOR | Acronyms mixed case. Use `URL` or `url`, not `Url` |
| NM-4 | Interface naming | NIT | Interface not ending in `-er`. Use `Parser`, `Generator`, not `IParser` |
| NM-5 | Receiver naming | NIT | Receiver name too long or not consistent. Use single/double letter like `p *Parser` |
| NM-6 | Getters with Get prefix | NIT | Getter methods with `Get` prefix. Use `Name()` not `GetName()` |

## Error Handling

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| EH-1 | Ignored errors | CRITICAL | Ignoring function return error with `_`. Must handle or explicitly ignore with comment |
| EH-2 | panic for normal errors | CRITICAL | Using `panic()` for business logic errors. Reserve for invariants only |
| EH-3 | Error not wrapped | MAJOR | Returning error without context. Use `fmt.Errorf("context: %w", err)` |
| EH-4 | Error string capitalized | NIT | Error messages start with capital letter. Use lowercase unless proper noun |
| EH-5 | Error string ends with punctuation | NIT | Error messages ending with period. Don't use punctuation |
| EH-6 | Missing error check | CRITICAL | Function call that returns error without checking return value |

## Correctness

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| CR-1 | Global mutable variables | CRITICAL | Global variables that are modified. Use parameters or config structs |
| CR-2 | Goroutine leak | CRITICAL | Starting goroutines without way to stop them. Use context cancellation |
| CR-3 | Race condition | CRITICAL | Shared mutable state without synchronization (mutex, channels, atomic) |
| CR-4 | Defer in loop | MAJOR | Using `defer` inside loop. Resources not freed until function returns |
| CR-5 | Shadowing err | MAJOR | Using `:=` when `=` needed, shadowing error variable |
| CR-6 | nil pointer dereference | CRITICAL | Dereferencing pointer without nil check |
| CR-7 | Type assertion without check | MAJOR | Type assertion without ok check: `v.(Type)`. Use `v, ok := v.(Type)` |

## Imports

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| IM-1 | Dot imports | MAJOR | Using `.` import except for testing. Makes code origin unclear |
| IM-2 | Unused imports | NIT | Import not used in file. Run `goimports` |
| IM-3 | Import grouping wrong | NIT | Imports not grouped: stdlib, external, local. Use `goimports` |
| IM-4 | Named imports without reason | MINOR | Aliasing imports without name conflict or clarity reason |

## Performance

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| PF-1 | Growing slice in loop | MINOR | Appending to slice in loop without capacity. Pre-allocate with `make([]T, 0, expectedSize)` |
| PF-2 | String concatenation in loop | MAJOR | Using `+` for strings in loop. Use `strings.Builder` or `bytes.Buffer` |
| PF-3 | Inefficient map operations | MINOR | Checking key existence then accessing. Use `v, ok := m[key]` |
| PF-4 | Copying large structs | MINOR | Passing large structs by value. Pass by pointer |
| PF-5 | Unnecessary conversions | NIT | Converting types back and forth. Keep single type |

## Concurrency

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| CC-1 | goroutine without context | MAJOR | Starting goroutine without context for cancellation |
| CC-2 | WaitGroup not passed by pointer | CRITICAL | Passing `sync.WaitGroup` by value. Always use pointer |
| CC-3 | Mutex copied | CRITICAL | Copying struct with `sync.Mutex`. Mutexes must not be copied |
| CC-4 | Channel not closed | MAJOR | Creating channel but never closing it. Close when done sending |
| CC-5 | Select without default | MINOR | Select statement that can block forever. Consider timeout or default |
| CC-6 | Unbuffered channel in goroutine | MAJOR | Sending to unbuffered channel without receiver. Deadlock risk |

## Testing

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| TS-1 | Test function naming | NIT | Test not named `TestXxx`. Must start with `Test` for `go test` to find |
| TS-2 | Table tests not used | MINOR | Multiple similar tests. Use table-driven tests |
| TS-3 | t.Error vs t.Fatal | MINOR | Using `t.Error` when should fail immediately. Use `t.Fatal` |
| TS-4 | Test helpers without t.Helper | MINOR | Test helper function without `t.Helper()`. Stack traces point to wrong line |
| TS-5 | Parallel tests race | MAJOR | Running tests in parallel with shared state. Use `t.Parallel()` carefully |

## Code Style

| ID | Rule | Severity | Description |
|----|------|----------|-------------|
| CS-1 | Not using gofmt | MAJOR | Code not formatted with `gofmt`. Run before commit |
| CS-2 | Not using goimports | MINOR | Imports not organized with `goimports`. Run before commit |
| CS-3 | Line too long | NIT | Lines >120 characters. Break for readability |
| CS-4 | Magic numbers | MINOR | Numeric literals without meaning. Use named constants |
| CS-5 | Naked returns | MINOR | Named returns with naked `return`. Unclear what's returned |
| CS-6 | Unnecessary else | NIT | `else` after `return/continue/break`. Remove else, dedent |

## Examples

### CO-1: Utils Package

**Bad:**
```go
// internal/utils/
//   string_utils.go
//   file_utils.go
//   ...

package utils

func FormatName(s string) string { ... }
func ValidateEmail(s string) bool { ... }
func ReadFile(path string) ([]byte, error) { ... }
```

**Good:**
```go
// internal/formatting/formatter.go
package formatting

func FormatName(name string) string { ... }

// internal/validation/validator.go
package validation

func ValidateEmail(email string) bool { ... }
```

### EH-3: Error Not Wrapped

**Bad:**
```go
func processFile(path string) error {
    data, err := os.ReadFile(path)
    if err != nil {
        return err  // Lost context - which file?
    }
    // ...
}
```

**Good:**
```go
func processFile(path string) error {
    data, err := os.ReadFile(path)
    if err != nil {
        return fmt.Errorf("failed to read file %s: %w", path, err)
    }
    // ...
}
```

### CR-2: Goroutine Leak

**Bad:**
```go
func startWorker() {
    go func() {
        for {
            // No way to stop this goroutine!
            work := getWork()
            process(work)
        }
    }()
}
```

**Good:**
```go
func startWorker(ctx context.Context) {
    go func() {
        for {
            select {
            case <-ctx.Done():
                return  // Graceful shutdown
            default:
                work := getWork()
                process(work)
            }
        }
    }()
}
```

### CR-4: Defer in Loop

**Bad:**
```go
for _, file := range files {
    f, err := os.Open(file)
    if err != nil {
        continue
    }
    defer f.Close()  // Only runs at function end, not each iteration!
    processFile(f)
}
// All files stay open until function returns
```

**Good:**
```go
for _, file := range files {
    func() {
        f, err := os.Open(file)
        if err != nil {
            return
        }
        defer f.Close()  // Runs at end of anonymous function
        processFile(f)
    }()
}
```

### PF-2: String Concatenation in Loop

**Bad:**
```go
var result string
for _, s := range strings {
    result += s + "\n"  // Creates new string each iteration
}
```

**Good:**
```go
var builder strings.Builder
for _, s := range strings {
    builder.WriteString(s)
    builder.WriteString("\n")
}
result := builder.String()
```

### CC-2: WaitGroup Not Passed by Pointer

**Bad:**
```go
func processItems(items []Item, wg sync.WaitGroup) {  // Copied!
    for _, item := range items {
        wg.Add(1)  // Modifying copy, not original
        go func(i Item) {
            defer wg.Done()
            process(i)
        }(item)
    }
}
```

**Good:**
```go
func processItems(items []Item, wg *sync.WaitGroup) {  // Pointer
    for _, item := range items {
        wg.Add(1)
        go func(i Item) {
            defer wg.Done()
            process(i)
        }(item)
    }
}
```

### TS-2: Table Tests

**Bad:**
```go
func TestAdd(t *testing.T) {
    if add(1, 2) != 3 {
        t.Error("...")
    }
    if add(0, 0) != 0 {
        t.Error("...")
    }
    // ... 10 more similar tests
}
```

**Good:**
```go
func TestAdd(t *testing.T) {
    tests := []struct {
        name string
        a, b int
        want int
    }{
        {"positive", 1, 2, 3},
        {"zeros", 0, 0, 0},
        {"negative", -1, -1, -2},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got := add(tt.a, tt.b)
            if got != tt.want {
                t.Errorf("add(%d, %d) = %d, want %d", tt.a, tt.b, got, tt.want)
            }
        })
    }
}
```

## Detection Tips

- Search for: `panic(`, `_ =`, `defer` in loops
- Check error returns: functions returning errors must be checked
- Look for: `utils/`, `common/`, `helpers/` packages
- Find global mutable variables (non-const globals)
- Check goroutines: must have way to stop (context, done channel)
- Verify: WaitGroup, Mutex passed by pointer not value
- Look for: string concatenation in loops (`+=`)
- Check: imports with `.` or unused imports
