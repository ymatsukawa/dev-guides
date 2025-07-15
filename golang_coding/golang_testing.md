# Golang Testing
This document describes recommendations and deprecations for Golang testing practices.

## Recommendations

**Use table-driven tests**:

Table-driven tests reduce duplication, improve readability, and make it easy to add test cases.

```go
// OK
func TestRemoveNewLineSuffix(t *testing.T) {
    tests := []struct {
        name     string
        input    string
        expected string
    }{
        {"empty", "", ""},
        {"with newline", "text\n", "text"},
        {"with CRLF", "text\r\n", "text"},
        {"no newline", "text", "text"},
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got := removeNewLineSuffix(tt.input)
            if got != tt.expected {
                t.Errorf("got %q, want %q", got, tt.expected)
            }
        })
    }
}
```

**Use subtests with t.Run**:

Subtests provide better test organization, allow running specific tests, and enable parallel execution.

```go
// OK
func TestUserService(t *testing.T) {
    t.Run("Create", func(t *testing.T) {
        // test user creation
    })
    
    t.Run("Update", func(t *testing.T) {
        // test user update
    })
    
    t.Run("Delete", func(t *testing.T) {
        // test user deletion
    })
}
```

**Enable race detection**:

Always run tests with race detection enabled for concurrent code.

```bash
# OK
go test -race ./...
```

**Use test helpers**:

Extract common test setup and assertions into helper functions that accept testing.T.

```go
// OK
func setupTestDB(t *testing.T) *sql.DB {
    t.Helper() // marks function as test helper
    
    db, err := sql.Open("sqlite3", ":memory:")
    if err != nil {
        t.Fatalf("failed to open database: %v", err)
    }
    
    t.Cleanup(func() {
        db.Close()
    })
    
    return db
}
```

**Use httptest for HTTP handlers**:

The httptest package provides utilities for testing HTTP handlers without starting real servers.

```go
// OK
func TestHandler(t *testing.T) {
    req := httptest.NewRequest("GET", "/users/123", nil)
    rec := httptest.NewRecorder()
    
    handler := http.HandlerFunc(GetUser)
    handler.ServeHTTP(rec, req)
    
    if rec.Code != http.StatusOK {
        t.Errorf("got status %d, want %d", rec.Code, http.StatusOK)
    }
}
```

**Use iotest for reader/writer testing**:

The iotest package provides utilities for testing I/O implementations.

```go
// OK
func TestCustomReader(t *testing.T) {
    reader := NewCustomReader(strings.NewReader("test data"))
    
    // Test with various error conditions
    err := iotest.TestReader(reader, []byte("test data"))
    if err != nil {
        t.Errorf("reader test failed: %v", err)
    }
}
```

**Categorize tests**:

Use build tags or testing.Short() to separate unit and integration tests.

```go
// OK for integration tests
//go:build integration

func TestDatabaseIntegration(t *testing.T) {
    if testing.Short() {
        t.Skip("skipping integration test in short mode")
    }
    // integration test code
}
```

**Use TestMain for setup/teardown**:

TestMain provides control over test execution and enables global setup/teardown.

```go
// OK
func TestMain(m *testing.M) {
    // setup
    pool := setupTestPool()
    
    // run tests
    code := m.Run()
    
    // teardown
    pool.Close()
    
    os.Exit(code)
}
```

**Use t.Parallel() for independent tests**:

Mark tests that can run concurrently to speed up test execution.

```go
// OK
func TestIndependentFunction(t *testing.T) {
    t.Parallel()
    // test code
}
```

**Write accurate benchmarks**:

Reset timer after setup, prevent compiler optimizations, and use realistic data.

```go
// OK
func BenchmarkProcess(b *testing.B) {
    // setup
    data := generateTestData()
    b.ResetTimer()
    
    // prevent compiler optimization
    var result int
    for i := 0; i < b.N; i++ {
        result = process(data[i%len(data)])
    }
    globalResult = result
}

var globalResult int // prevent optimization
```

## Deprecations

**Avoid duplicate test functions**:

Multiple similar test functions create maintenance burden. Use table-driven tests instead.

```go
// NG
func TestAdd_Positive(t *testing.T) { /* ... */ }
func TestAdd_Negative(t *testing.T) { /* ... */ }
func TestAdd_Zero(t *testing.T) { /* ... */ }

// OK - use table-driven test (see Recommendations)
```

**Avoid time.Sleep in tests**:

time.Sleep makes tests flaky and slow. Use synchronization primitives or polling with timeout.

```go
// NG
go startServer()
time.Sleep(100 * time.Millisecond) // hope server is ready

// OK
serverReady := make(chan struct{})
go func() {
    startServer()
    close(serverReady)
}()

select {
case <-serverReady:
    // server is ready
case <-time.After(5 * time.Second):
    t.Fatal("server failed to start")
}
```

**Avoid direct time dependencies**:

Hardcoded time makes tests brittle. Inject time as a dependency.

```go
// NG
func IsExpired(created time.Time) bool {
    return time.Now().Sub(created) > 24*time.Hour
}

// OK
type TimeProvider interface {
    Now() time.Time
}

func IsExpired(created time.Time, timer TimeProvider) bool {
    return timer.Now().Sub(created) > 24*time.Hour
}
```

**Avoid ignoring test execution modes**:

Not using -shuffle can hide order dependencies. Not using -parallel wastes time.

```bash
# OK
go test -shuffle=on -parallel=4 ./...
```

**Avoid global state in tests**:

Global state causes test interference. Use fresh instances for each test.

```go
// NG
var testDB *sql.DB // global

func TestUserCreate(t *testing.T) {
    // uses global testDB
}

// OK
func TestUserCreate(t *testing.T) {
    db := setupTestDB(t) // fresh instance
    // use db
}
```

**Avoid skipping cleanup**:

Missing cleanup causes resource leaks and test pollution. Use t.Cleanup for reliable cleanup.

```go
// NG
func TestWithTempFile(t *testing.T) {
    f, _ := os.CreateTemp("", "test")
    // missing cleanup
}

// OK
func TestWithTempFile(t *testing.T) {
    f, _ := os.CreateTemp("", "test")
    t.Cleanup(func() {
        os.Remove(f.Name())
        f.Close()
    })
}
```

**Avoid inaccurate benchmarks**:

Benchmarks need careful design to produce meaningful results.

```go
// NG
func BenchmarkBad(b *testing.B) {
    for i := 0; i < b.N; i++ {
        _ = compute() // compiler might optimize away
    }
}

// OK - assign to package variable to prevent optimization
```

**Avoid missing error checks in tests**:

Unchecked errors in tests can hide bugs. Always check errors, even in tests.

```go
// NG
func TestSomething(t *testing.T) {
    db, _ := sql.Open("sqlite3", ":memory:")
    defer db.Close()
}

// OK
func TestSomething(t *testing.T) {
    db, err := sql.Open("sqlite3", ":memory:")
    if err != nil {
        t.Fatalf("failed to open db: %v", err)
    }
    defer db.Close()
}
```

**Avoid testing implementation details**:

Test behavior, not implementation. This makes refactoring easier.

```go
// NG - testing private methods directly
// OK - test through public API
```

**Avoid ignoring code coverage**:

While 100% coverage isn't always necessary, monitor coverage to find untested code.

```bash
# OK
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out
```

**Avoid hardcoded test data paths**:

Use testdata directory or embed test files for portable tests.

```go
// NG
data, _ := os.ReadFile("/home/user/test.json")

// OK
data, _ := os.ReadFile("testdata/test.json")
// or
//go:embed testdata/test.json
var testData []byte
```

**Avoid missing parallel test considerations**:

Parallel tests must not share state or resources.

```go
// NG
func TestParallel(t *testing.T) {
    t.Parallel()
    os.Setenv("KEY", "value") // modifies global state
}

// OK
func TestParallel(t *testing.T) {
    t.Parallel()
    // use local state only
}
```