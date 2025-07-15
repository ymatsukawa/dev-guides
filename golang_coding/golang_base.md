# Golang Strings, Functions and Error Management
This document describes recommendations and deprecations of Golang programming practices for strings, functions, and error handling.

## Recommendations

**Use strings.Builder for efficient string concatenation**:

When concatenating multiple strings, strings.Builder provides the most efficient approach by minimizing memory allocations and copies.

```go
// NG
func concat(values []string) string {
    res := ""
    for _, value := range values {
        res += value // creates new string each iteration
    }
    return res
}

// OK
func concat(values []string) string {
    var sb strings.Builder
    for _, value := range values {
        sb.WriteString(value)
    }
    return sb.String()
}
```

**Accept io.Reader for file operations**:

Functions that read files should accept io.Reader interfaces rather than filenames for better testability and reusability.

```go
// NG
func countEmptyLinesInFile(filename string) (int, error) {
    file, err := os.Open(filename)
    if err != nil { return 0, err }
    defer file.Close()
    scanner := bufio.NewScanner(file)
    // ...
}

// OK
func countEmptyLines(reader io.Reader) (int, error) {
    scanner := bufio.NewScanner(reader)
    // ...
}
```

**Use error wrapping for context**:

Wrap errors with additional context using fmt.Errorf and %w to maintain the error chain while adding valuable debugging information.

```go
// OK
if err != nil {
    return fmt.Errorf("failed to process user %d: %w", userID, err)
}
```

**Use errors.Is and errors.As for wrapped errors**:

When checking wrapped errors, use errors.Is for sentinel errors and errors.As for type assertions to properly traverse the error chain.

```go
// NG
if err == sql.ErrNoRows {
    // won't work with wrapped errors
}

// OK
if errors.Is(err, sql.ErrNoRows) {
    // works with wrapped errors
}
```

## Deprecations

**Avoid misunderstanding rune concepts**:

Runes represent Unicode code points (int32), not bytes. String length returns byte count, not character count.

**Avoid incorrect string iteration**:

Iterating strings with indices accesses bytes, not runes. Use range loops with value variables for proper rune access.

```go
// NG
s := "hêllo"
for i := range s {
    fmt.Printf("position %d: %c\n", i, s[i]) // s[i] is a byte
}

// OK
s := "hêllo"
for i, r := range s {
    fmt.Printf("position %d: %c\n", i, r) // r is the rune
}
```

**Avoid confusing trim functions**:

TrimRight/TrimLeft remove any trailing/leading runes from a set, while TrimSuffix/TrimPrefix remove exact strings. Choose appropriately.

**Avoid unnecessary string conversions**:

Converting between []byte and string creates copies. Use the bytes package for []byte operations to avoid conversions.

**Avoid substring memory leaks**:

Substrings may share backing arrays with original strings. Use strings.Clone or deep copies for long-lived substrings.

**Avoid choosing wrong receiver types**:

Use pointer receivers when methods modify state, contain sync types, or deal with large structs. Use value receivers for immutability and small types.

**Avoid ignoring named result parameter benefits**:

Named results can improve API clarity when multiple returns have the same type, but use them judiciously.

**Avoid unintended effects of named results**:

Named result parameters are zero-initialized. Ensure proper assignment to avoid returning unexpected zero values.

**Avoid returning nil pointers as interfaces**:

A nil pointer converted to an interface is not a nil interface. Return explicit nil for interface types.

```go
// NG
func (c Customer) Validate() error {
    var m *MultiError
    if c.Age < 0 {
        m = &MultiError{}
        m.Add(errors.New("age is negative"))
    }
    return m // non-nil interface even when m is nil
}

// OK
func (c Customer) Validate() error {
    var m *MultiError
    if c.Age < 0 {
        m = &MultiError{}
        m.Add(errors.New("age is negative"))
    }
    if m != nil {
        return m
    }
    return nil // explicit nil interface
}
```

**Avoid ignoring defer argument evaluation**:

Defer arguments and receivers are evaluated immediately when defer is called, not when the function executes.

```go
// NG
func processStatus() {
    status := "initial"
    defer notify(status) // captures "initial"
    status = "updated"
}

// OK
func processStatus() {
    status := "initial"
    defer func() {
        notify(status) // captures current value
    }()
    status = "updated"
}
```

**Avoid using panic for non-programmer errors**:

Panic should only be used for unrecoverable programmer errors, not for normal error conditions.

**Avoid ignoring error wrapping**:

Error wrapping adds context and maintains error chains. Use fmt.Errorf with %w for proper error propagation.

**Avoid using type switches for wrapped errors**:

Type switches only check direct types. Use errors.As for type checking through error chains.

```go
// NG
switch err.(type) {
case transientError:
    // misses wrapped errors
}

// OK
var te transientError
if errors.As(err, &te) {
    // finds wrapped errors
}
```

**Avoid using == for wrapped error values**:

Direct equality checks miss wrapped errors. Use errors.Is to check error values through chains.

**Avoid handling errors twice**:

Double handling (e.g., logging then returning) obscures error flow. Handle errors once - either log or return.

**Avoid ignoring errors silently**:

Ignored errors hide problems. Use blank identifier explicitly with comments when ignoring is intentional.

```go
// NG
notify()

// OK
_ = notify() // Best-effort delivery, errors acceptable
```

**Avoid ignoring defer errors**:

Defer errors can indicate resource leaks. Handle them explicitly or document why they're ignored.

```go
// NG
defer rows.Close()

// OK (explicit ignore)
defer func() { _ = rows.Close() }()

// OK (handle error)
defer func() {
    if err := rows.Close(); err != nil {
        log.Printf("failed to close rows: %v", err)
    }
}()
```