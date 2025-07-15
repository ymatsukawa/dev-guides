# Golang Concurrency
This document describes recommendations and deprecations of Golang concurrency practices.

## Recommendations

**Use channels for communication between goroutines**:

Channels provide safe communication and synchronization. They express intent clearly and prevent data races by design.

```go
// OK
ch := make(chan Result)
go func() {
    ch <- computeResult()
}()
result := <-ch
```

**Use sync.WaitGroup for goroutine coordination**:

WaitGroup provides clean synchronization when waiting for multiple goroutines to complete.

```go
// OK
var wg sync.WaitGroup
for _, item := range items {
    wg.Add(1)
    go func(item Item) {
        defer wg.Done()
        process(item)
    }(item)
}
wg.Wait()
```

**Use context for cancellation and deadlines**:

Context provides a standard way to propagate cancellation signals and deadlines across API boundaries.

```go
// OK
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

select {
case result := <-compute(ctx):
    return result
case <-ctx.Done():
    return ctx.Err()
}
```

**Use worker pools for CPU-bound tasks**:

For CPU-bound work, limit goroutines to GOMAXPROCS to avoid excessive context switching.

```go
// OK
workers := runtime.GOMAXPROCS(0)
jobs := make(chan Job, workers)
var wg sync.WaitGroup

for i := 0; i < workers; i++ {
    wg.Add(1)
    go func() {
        defer wg.Done()
        for job := range jobs {
            process(job)
        }
    }()
}
```

## Deprecations

**Avoid confusing concurrency with parallelism**:

Concurrency is about structure (dealing with many things), parallelism is about execution (doing many things). They are related but distinct concepts.

**Avoid assuming concurrency is always faster**:

Goroutine overhead, context switching, and synchronization costs can make concurrent code slower for small workloads. Profile before optimizing.

**Avoid forcing channels for all concurrent problems**:

Channels are for communication; mutexes are for protecting shared state. Use the right tool for each problem.

**Avoid confusing data races with race conditions**:

Data races are concurrent memory access conflicts. Race conditions are timing-dependent behaviors. Both need different solutions.

**Avoid ignoring workload types**:

CPU-bound tasks benefit from limiting goroutines to CPU cores. I/O-bound tasks can use many more goroutines effectively.

**Avoid misunderstanding context**:

Context carries deadlines, cancellation signals, and request-scoped values. Don't use it for optional parameters.

**Avoid propagating inappropriate contexts**:

Detach context when async operations should outlive parent requests. Create custom contexts to control cancellation scope.

**Avoid starting goroutines without clear stop conditions**:

Every goroutine needs a termination plan. Use channels, contexts, or WaitGroups to manage lifecycle.

**Avoid capturing loop variables in goroutines**:

Loop variables are reused across iterations. Capture them explicitly to avoid races.

```go
// NG
for _, val := range values {
    go func() {
        process(val) // captures loop variable
    }()
}

// OK
for _, val := range values {
    val := val // create local copy
    go func() {
        process(val)
    }()
}

// OK (alternative)
for _, val := range values {
    go func(v string) {
        process(v)
    }(val)
}
```

**Avoid expecting deterministic select behavior**:

When multiple channels are ready, select chooses randomly. Don't assume any order.

**Avoid using chan bool for signaling**:

Use chan struct{} for signal-only channels. It's clearer and uses no memory.

```go
// NG
done := make(chan bool)

// OK
done := make(chan struct{})
```

**Avoid ignoring nil channel utility**:

Nil channels block forever. Use them to disable select cases dynamically.

```go
// OK
var ticker <-chan time.Time
if enablePolling {
    ticker = time.NewTicker(interval).C
}

select {
case <-ticker: // blocks forever if ticker is nil
    poll()
case <-done:
    return
}
```

**Avoid confusion about channel buffer sizes**:

Unbuffered channels provide synchronization. Buffered channels decouple senders and receivers. Choose based on needs.

**Avoid string formatting side effects in concurrent code**:

String formatting can trigger method calls that acquire locks, potentially causing deadlocks.

**Avoid concurrent append to shared slices**:

Append isn't thread-safe. Protect with mutexes or use separate slices and merge results.

**Avoid incorrect mutex scope with slices/maps**:

Slice and map assignment shares backing data. Protect all operations or make deep copies.

**Avoid calling WaitGroup.Add inside goroutines**:

Add must be called before starting goroutines to prevent races with Wait.

```go
// NG
go func() {
    wg.Add(1) // race with wg.Wait()
    defer wg.Done()
    work()
}()

// OK
wg.Add(1)
go func() {
    defer wg.Done()
    work()
}()
```

**Avoid forgetting sync.Cond for broadcasts**:

sync.Cond enables broadcasting to multiple waiting goroutines, which channels cannot do efficiently.

**Avoid manual error group implementations**:

Use errgroup.Group for concurrent operations with error handling and context propagation.

```go
// OK
g, ctx := errgroup.WithContext(context.Background())
for _, url := range urls {
    url := url
    g.Go(func() error {
        return fetch(ctx, url)
    })
}
if err := g.Wait(); err != nil {
    return err
}
```

**Avoid copying sync types**:

Copying sync.Mutex, sync.WaitGroup, etc. creates independent instances that break synchronization.

```go
// NG
type Counter struct {
    mu sync.Mutex
    count int
}
func (c Counter) Inc() { // copies mutex
    c.mu.Lock()
    c.count++
    c.mu.Unlock()
}

// OK
func (c *Counter) Inc() { // doesn't copy
    c.mu.Lock()
    c.count++
    c.mu.Unlock()
}
```