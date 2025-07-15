# Golang Standard Library and Optimization
This document describes recommendations and deprecations for Golang standard library usage and performance optimization.

## Recommendations

**Use time constants for duration values**:

Always use time package constants to make duration units explicit and prevent unit confusion.

```go
// NG
time.NewTicker(1000) // 1000 nanoseconds

// OK
time.NewTicker(1 * time.Second)
```

**Use prepared statements for repeated queries**:

Prepared statements prevent SQL injection and improve performance by reusing query plans.

```go
// OK
stmt, err := db.Prepare("SELECT name FROM users WHERE id = ?")
if err != nil {
    return err
}
defer stmt.Close()

for _, id := range userIDs {
    err := stmt.QueryRow(id).Scan(&name)
    // handle error
}
```

**Profile before optimizing**:

Use pprof to identify actual bottlenecks before optimizing. Never optimize based on assumptions.

```go
// OK
import _ "net/http/pprof"

func main() {
    go func() {
        log.Println(http.ListenAndServe("localhost:6060", nil))
    }()
    // main application
}
```

**Design data structures with cache locality**:

Group related data together to improve CPU cache utilization and reduce cache misses.

**Use sync.Pool for temporary objects**:

Reduce garbage collection pressure by reusing temporary objects with sync.Pool.

```go
// OK
var bufferPool = sync.Pool{
    New: func() interface{} {
        return new(bytes.Buffer)
    },
}

func process() {
    buf := bufferPool.Get().(*bytes.Buffer)
    defer func() {
        buf.Reset()
        bufferPool.Put(buf)
    }()
    // use buffer
}
```

## Deprecations

**Avoid raw integer durations**:

Raw integers are interpreted as nanoseconds, leading to confusion and bugs.

**Avoid repeated time.After in loops**:

time.After creates new timers that aren't garbage collected until expiry. Use time.NewTimer with Reset.

```go
// NG
for {
    select {
    case <-time.After(timeout): // creates new timer each iteration
        return ErrTimeout
    case result := <-ch:
        return result
    }
}

// OK
timer := time.NewTimer(timeout)
defer timer.Stop()
for {
    select {
    case <-timer.C:
        return ErrTimeout
    case result := <-ch:
        return result
    }
    timer.Reset(timeout)
}
```

**Avoid unexpected JSON marshaling with embedded types**:

Embedded types may implement json.Marshaler unexpectedly. Name embedded fields or implement custom marshaling.

**Avoid comparing time.Time with ==**:

time.Time contains monotonic clock data lost in JSON marshaling. Use Equal method for comparisons.

```go
// NG
if t1 == t2 { // may fail after JSON round-trip
    // ...
}

// OK
if t1.Equal(t2) {
    // ...
}
```

**Avoid assuming JSON number types**:

JSON unmarshaling to interface{} converts numbers to float64. Handle type conversions explicitly.

```go
// NG
data := result["count"].(int) // panics, data is float64

// OK
data := result["count"].(float64)
count := int(data)
```

**Avoid expecting sql.Open to connect**:

sql.Open only validates arguments. Use Ping to verify connectivity.

```go
// OK
db, err := sql.Open("postgres", dsn)
if err != nil {
    return err
}
if err := db.Ping(); err != nil {
    return err
}
```

**Avoid ignoring connection pool settings**:

Default pool settings may not suit your workload. Configure explicitly.

```go
// OK
db.SetMaxOpenConns(25)
db.SetMaxIdleConns(5)
db.SetConnMaxLifetime(5 * time.Minute)
```

**Avoid ignoring NULL handling**:

Use sql.NullString, sql.NullInt64, or pointers for nullable columns.

```go
// NG
var name string
err := db.QueryRow("SELECT name FROM users WHERE id = ?", id).Scan(&name)
// fails if name is NULL

// OK
var name sql.NullString
err := db.QueryRow("SELECT name FROM users WHERE id = ?", id).Scan(&name)
if name.Valid {
    // use name.String
}
```

**Avoid missing rows.Err() checks**:

Check rows.Err() after iteration to catch errors that occurred during scanning.

```go
// OK
for rows.Next() {
    // scan row
}
if err := rows.Err(); err != nil {
    return err
}
```

**Avoid resource leaks**:

Always close HTTP response bodies, SQL rows, and files.

```go
// OK
resp, err := http.Get(url)
if err != nil {
    return err
}
defer resp.Body.Close()
```

**Avoid missing returns after http.Error**:

http.Error doesn't stop execution. Always return after sending errors.

```go
// NG
http.Error(w, "error", 500)
w.Write([]byte("success")) // still executes

// OK
http.Error(w, "error", 500)
return
```

**Avoid default HTTP client in production**:

Configure timeouts explicitly for production robustness.

```go
// OK
client := &http.Client{
    Timeout: 30 * time.Second,
    Transport: &http.Transport{
        DialContext: (&net.Dialer{
            Timeout: 5 * time.Second,
        }).DialContext,
        TLSHandshakeTimeout: 5 * time.Second,
        ResponseHeaderTimeout: 5 * time.Second,
    },
}
```

**Avoid ignoring CPU cache effects**:

Poor memory access patterns cause cache misses. Design with spatial and temporal locality in mind.

**Avoid false sharing in concurrent code**:

Pad structures to prevent different goroutines from modifying data in the same cache line.

```go
// NG
type Counter struct {
    count1 int64
    count2 int64 // same cache line as count1
}

// OK
type Counter struct {
    count1 int64
    _      [7]int64 // padding to separate cache lines
    count2 int64
}
```

**Avoid ignoring instruction-level parallelism**:

Reduce data dependencies between instructions to enable CPU pipelining.

```go
// NG
sum := a + b
result := sum + c // depends on previous line

// OK
sum1 := a + b
sum2 := c + d // independent, can execute in parallel
result := sum1 + sum2
```

**Avoid poor struct field ordering**:

Order fields by decreasing size to minimize padding and improve memory usage.

```go
// NG
type Poor struct {
    flag bool    // 1 byte + 7 padding
    counter int64 // 8 bytes
    active bool   // 1 byte + 7 padding
} // total: 24 bytes

// OK
type Better struct {
    counter int64 // 8 bytes
    flag bool     // 1 byte
    active bool   // 1 byte + 6 padding
} // total: 16 bytes
```

**Avoid unnecessary heap allocations**:

Understand escape analysis. Keep variables on stack when possible.

```go
// NG
func getValue() *int {
    val := 42
    return &val // escapes to heap
}

// OK
func getValue() int {
    val := 42
    return val // stays on stack
}
```

**Avoid ignoring GC impact**:

Minimize heap allocations and tune GOGC if needed. Monitor GC metrics in production.

**Avoid container resource mismatches**:

Ensure Go runtime recognizes container CPU and memory limits.

```go
// OK
import _ "go.uber.org/automaxprocs"
// Automatically sets GOMAXPROCS to match container CPU quota
```