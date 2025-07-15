# Golang Code Organization and Data Types
This document describes recommendations and deprecations of Golang programming practices for code organization and data types.

## Recommendations

**Use early returns to reduce nesting**:

Early returns help maintain code readability by keeping the happy path on the left edge of the code. This reduces cognitive load and makes the logical flow clearer.

```go
// NG
func join(s1, s2 string, max int) (string, error) {
    if s1 == "" {
        return "", errors.New("s1 is empty")
    } else {
        if s2 == "" {
            return "", errors.New("s2 is empty")
        } else {
            concat, err := concatenate(s1, s2)
            if err != nil {
                return "", err
            } else {
                if len(concat) > max {
                    return concat[:max], nil
                } else {
                    return concat, nil
                }
            }
        }
    }
}

// OK
func join(s1, s2 string, max int) (string, error) {
    if s1 == "" {
        return "", errors.New("s1 is empty")
    }
    if s2 == "" {
        return "", errors.New("s2 is empty")
    }
    concat, err := concatenate(s1, s2)
    if err != nil {
        return "", err
    }
    if len(concat) > max {
        return concat[:max], nil
    }
    return concat, nil
}
```

**Initialize slices with appropriate capacity when size is known**:

Pre-allocating slice capacity prevents unnecessary backing array reallocations and improves performance significantly.

```go
// NG
func convert(foos []Foo) []Bar {
    bars := make([]Bar, 0)
    for _, foo := range foos {
        bars = append(bars, fooToBar(foo))
    }
    return bars
}

// OK
func convert(foos []Foo) []Bar {
    n := len(foos)
    bars := make([]Bar, n)
    for i, foo := range foos {
        bars[i] = fooToBar(foo)
    }
    return bars
}
```

**Use defer for resource cleanup**:

Defer ensures resources are properly released even when errors occur, making code more robust and maintainable.

```go
// OK
resp, err := http.Get("...")
if err != nil {
    return err
}
defer resp.Body.Close()
```

## Deprecations

**Avoid unintended variable shadowing**:

Variable shadowing occurs when an inner scope declares a variable with the same name as an outer scope variable, potentially causing the outer variable to remain uninitialized.

```go
// NG
var client *http.Client
if tracing {
    client, err := createClientWithTracing() // shadows outer client
    if err != nil { return err }
}
// client is always nil here

// OK
var client *http.Client
var err error
if tracing {
    client, err = createClientWithTracing() // uses assignment
    if err != nil { return err }
}
```

**Avoid misusing init functions**:

Init functions have limited error management capabilities and can complicate application state management and testing. Use them only for dependency initialization that cannot fail.

**Avoid overusing getter and setter methods**:

Excessive use of getters and setters adds unnecessary boilerplate without providing value. Follow Go's simple idioms and use them only when they add clear benefits.

**Avoid premature abstractions**:

Creating interfaces before identifying common behaviors leads to unnecessary complexity. Abstractions should be discovered through actual use cases, not created speculatively.

**Avoid defining interfaces on the producer side**:

Producer-side interfaces force all clients to accept a specific abstraction level. Define interfaces on the consumer side to allow clients to determine their own abstraction needs.

**Avoid returning interfaces from functions**:

Returning interfaces restricts client flexibility. Return concrete types and let clients define their own interfaces for better composability.

**Avoid overusing the any type**:

The any type (interface{}) loses compile-time type safety. Use explicit types in method signatures except where type diversity is inherently required (e.g., encoding/decoding).

**Avoid premature use of generics**:

Generics add complexity. Use them only when concrete benefits like reducing boilerplate are clear and the code becomes more maintainable.

**Avoid ignoring type embedding implications**:

Type embedding can expose unintended fields and methods. Understand promoted fields and methods to prevent accidental API exposure.

**Avoid using outdated option patterns**:

Config structs and Builder patterns have limitations. Use the Functional Options pattern for API-friendly, flexible configuration.

**Avoid poor project organization**:

Follow standard Go project layout conventions (/cmd, /internal, /pkg). Use meaningful package names and avoid unnecessary exports.

**Avoid generic utility packages**:

Package names like utils, common, or base lack meaning. Use descriptive names that indicate the package's specific functionality.

**Avoid variable and package name collisions**:

Name collisions create ambiguity. Use different meaningful names or import aliases to distinguish between variables and packages.

**Avoid missing documentation for exported elements**:

Undocumented exports complicate API usage. Document all exported elements with clear, concise comments.

**Avoid skipping linters**:

Linters catch potential errors and enforce consistency. Use appropriate linters like go vet and shadow in CI/CD pipelines.

**Avoid treating octal literals as decimals**:

Leading zeros denote octal numbers in Go. Use the 0o prefix for clarity when octal numbers are intended.

```go
// NG
sum := 100 + 010 // 010 is octal 8, not decimal 10

// OK
sum := 100 + 0o10 // explicitly octal
```

**Avoid ignoring integer overflow**:

Go silently handles integer overflow. Implement overflow checks for critical operations using math.MaxInt constants.

**Avoid treating floats as exact numbers**:

Floating-point arithmetic is approximate. Use epsilon comparisons and order operations carefully for better precision.

**Avoid confusing slice length and capacity**:

Length is the number of elements; capacity is the backing array size. Understanding this distinction prevents memory leaks and unexpected behavior.

**Avoid confusing nil and empty slices**:

Nil and empty slices behave differently in some contexts (e.g., JSON marshaling). Understand when each is appropriate.

**Avoid checking slice emptiness with nil checks only**:

Empty non-nil slices exist. Use len(slice) != 0 to check if a slice contains elements.

```go
// NG
if operations != nil {
    handle(operations)
}

// OK
if len(operations) != 0 {
    handle(operations)
}
```

**Avoid misusing the copy function**:

The copy function copies min(len(dst), len(src)) elements. Ensure destination slice has sufficient length for complete copies.

**Avoid unexpected side effects with append**:

Appending to slices sharing backing arrays can modify original slices. Use full slice expressions or copies to limit side effects.

**Avoid memory leaks from slice operations**:

Slicing creates references to original backing arrays. Use copy to create independent slices when the original should be garbage collected.

**Avoid inefficient map initialization**:

Provide initial size to make when final element count is known to prevent repeated growth operations.

**Avoid expecting memory release from map deletions**:

Map bucket counts don't decrease after deletions. Consider recreating maps or using pointers for large-scale deletions.

**Avoid using == for incomparable types**:

Slices, maps, and functions cannot be compared with ==. Use reflect.DeepEqual or custom comparison methods.

**Avoid ignoring value copying in range loops**:

Range variables are copies. Modifications don't affect original elements. Use indices or pointer slices for mutations.

```go
// NG
for _, a := range accounts {
    a.balance += 1000 // modifies copy, not original
}

// OK
for i := range accounts {
    accounts[i].balance += 1000
}
```

**Avoid ignoring range expression evaluation**:

Range expressions are evaluated once before iteration. Loop body modifications don't affect iteration count.

**Avoid pointer issues in range loops**:

The range value variable reuses the same address. Create local copies when storing pointers to range elements.

```go
// NG
for _, customer := range customers {
    s.m[customer.ID] = &customer // all point to same address
}

// OK
for _, customer := range customers {
    current := customer
    s.m[current.ID] = &current
}
```

**Avoid assumptions about map iteration**:

Map iteration order is non-deterministic. Don't rely on any specific order or expect added elements to appear in current iteration.

**Avoid misunderstanding break statements**:

Break exits the innermost for, switch, or select. Use labels to break outer loops explicitly.

```go
// NG
for i := 0; i < 5; i++ {
    switch i {
    case 2:
        break // only exits switch, not for loop
    }
}

// OK
loop:
for i := 0; i < 5; i++ {
    switch i {
    case 2:
        break loop // exits for loop
    }
}
```

**Avoid using defer inside loops**:

Deferred functions execute when the enclosing function returns, not at loop iteration end. Extract loop bodies to separate functions for proper cleanup.

```go
// NG
for path := range ch {
    file, err := os.Open(path)
    if err != nil { return err }
    defer file.Close() // accumulates until function returns
}

// OK
for path := range ch {
    err := processFile(path)
    if err != nil { return err }
}

func processFile(path string) error {
    file, err := os.Open(path)
    if err != nil { return err }
    defer file.Close() // executes when processFile returns
    return nil
}
```
