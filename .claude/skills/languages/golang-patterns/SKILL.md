---
name: golang-patterns
description: Comprehensive Go skill covering project structure, idiomatic patterns, error handling, concurrency, interfaces, testing, benchmarks, and modules.
---

# Go Development Patterns

Idiomatic Go for building robust, efficient, maintainable applications — from project layout to production-ready testing.

---

## 1. Project Structure

Standard layout for Go projects:

```text
myproject/
├── cmd/myapp/main.go       # Entry point
├── internal/
│   ├── handler/            # HTTP handlers
│   ├── service/            # Business logic
│   ├── repository/         # Data access
│   └── config/             # Configuration
├── pkg/client/             # Public API client
├── api/v1/                 # Proto / OpenAPI definitions
├── testdata/               # Test fixtures
├── go.mod
├── go.sum
└── Makefile
```

Use short, lowercase, no-underscore package names (`user`, not `userService`). Inject dependencies via struct fields, never use package-level globals.

**Rule:** Place interfaces in the consumer package, not the provider package.

---

## 2. Error Handling

Wrap errors with `fmt.Errorf` and `%w` so callers can inspect with `errors.Is`/`errors.As`.

```go
func LoadConfig(path string) (*Config, error) {
    data, err := os.ReadFile(path)
    if err != nil {
        return nil, fmt.Errorf("load config %s: %w", path, err)
    }
    var cfg Config
    if err := json.Unmarshal(data, &cfg); err != nil {
        return nil, fmt.Errorf("parse config %s: %w", path, err)
    }
    return &cfg, nil
}
```

Define sentinel errors and custom types for domain errors:

```go
var ErrNotFound = errors.New("resource not found")

type ValidationError struct{ Field, Message string }
func (e *ValidationError) Error() string {
    return fmt.Sprintf("validation failed on %s: %s", e.Field, e.Message)
}

func HandleError(err error) {
    if errors.Is(err, ErrNotFound) { /* ... */ }
    var ve *ValidationError
    if errors.As(err, &ve) { /* use ve.Field */ }
}
```

**Rule:** Never use `_` to discard errors. Never use `panic` for control flow. Always pass `context.Context` as the first parameter.

---

## 3. Interfaces and Struct Design

Accept interfaces, return concrete types. Keep interfaces small (1–3 methods).

```go
// Functional options pattern
type Option func(*Server)

func WithTimeout(d time.Duration) Option {
    return func(s *Server) { s.timeout = d }
}

func NewServer(addr string, opts ...Option) *Server {
    s := &Server{addr: addr, timeout: 30 * time.Second}
    for _, opt := range opts { opt(s) }
    return s
}
```

Use type assertions for optional behavior:

```go
func WriteAndFlush(w io.Writer, data []byte) error {
    if _, err := w.Write(data); err != nil { return err }
    if f, ok := w.(interface{ Flush() error }); ok { return f.Flush() }
    return nil
}
```

**Rule:** Design zero values to be immediately usable without explicit initialization.

---

## 4. Goroutines and Channels

Use `errgroup` for coordinated goroutines and buffered channels to prevent leaks:

```go
import "golang.org/x/sync/errgroup"

func FetchAll(ctx context.Context, urls []string) ([][]byte, error) {
    g, ctx := errgroup.WithContext(ctx)
    results := make([][]byte, len(urls))
    for i, url := range urls {
        i, url := i, url
        g.Go(func() error {
            data, err := fetchWithTimeout(ctx, url)
            if err != nil { return err }
            results[i] = data
            return nil
        })
    }
    if err := g.Wait(); err != nil { return nil, err }
    return results, nil
}
```

Avoid goroutine leaks by using buffered channels or `select` with `ctx.Done()`:

```go
func safeFetch(ctx context.Context, url string) <-chan []byte {
    ch := make(chan []byte, 1)
    go func() {
        data, err := fetch(url)
        if err != nil { return }
        select {
        case ch <- data:
        case <-ctx.Done():
        }
    }()
    return ch
}
```

**Rule:** Never use `GlobalScope`-equivalent patterns. Always cancel contexts and close channels you own.

---

## 5. Context Propagation and Defer

Always propagate context and use `defer cancel()` immediately after `WithTimeout`/`WithCancel`:

```go
func FetchWithTimeout(ctx context.Context, url string) ([]byte, error) {
    ctx, cancel := context.WithTimeout(ctx, 5*time.Second)
    defer cancel()
    req, err := http.NewRequestWithContext(ctx, "GET", url, nil)
    if err != nil { return nil, fmt.Errorf("create request: %w", err) }
    resp, err := http.DefaultClient.Do(req)
    if err != nil { return nil, fmt.Errorf("fetch %s: %w", url, err) }
    defer resp.Body.Close()
    return io.ReadAll(resp.Body)
}
```

For graceful shutdown:

```go
func GracefulShutdown(server *http.Server) {
    quit := make(chan os.Signal, 1)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
    <-quit
    ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
    defer cancel()
    server.Shutdown(ctx)
}
```

---

## 6. Memory and Performance

Preallocate slices when length is known; use `strings.Builder` for string joins; use `sync.Pool` for frequent allocations:

```go
results := make([]Result, 0, len(items)) // single allocation

var bufferPool = sync.Pool{New: func() interface{} { return new(bytes.Buffer) }}
func ProcessRequest(data []byte) []byte {
    buf := bufferPool.Get().(*bytes.Buffer)
    defer func() { buf.Reset(); bufferPool.Put(buf) }()
    buf.Write(data)
    return buf.Bytes()
}
```

---

## 7. Table-Driven Tests and TDD

Follow RED-GREEN-REFACTOR. Use table-driven tests as the standard pattern:

```go
func TestAdd(t *testing.T) {
    tests := []struct {
        name     string
        a, b     int
        expected int
    }{
        {"positive", 2, 3, 5},
        {"negative", -1, -2, -3},
        {"zero", 0, 0, 0},
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got := Add(tt.a, tt.b)
            if got != tt.expected {
                t.Errorf("Add(%d,%d) = %d; want %d", tt.a, tt.b, got, tt.expected)
            }
        })
    }
}
```

Use `t.Helper()` in helpers, `t.Cleanup()` for teardown, `t.TempDir()` for temp files, `t.Parallel()` for independent subtests.

**Rule:** Test behavior through public APIs, not private functions.

---

## 8. Mocks and HTTP Testing

Define mock structs with function fields for interface-based mocking:

```go
type MockUserRepository struct {
    GetUserFunc  func(id string) (*User, error)
    SaveUserFunc func(user *User) error
}
func (m *MockUserRepository) GetUser(id string) (*User, error) { return m.GetUserFunc(id) }
func (m *MockUserRepository) SaveUser(user *User) error        { return m.SaveUserFunc(user) }
```

Test HTTP handlers with `httptest`:

```go
func TestHealthHandler(t *testing.T) {
    req := httptest.NewRequest(http.MethodGet, "/health", nil)
    w   := httptest.NewRecorder()
    HealthHandler(w, req)
    resp := w.Result()
    defer resp.Body.Close()
    if resp.StatusCode != http.StatusOK {
        t.Errorf("got %d; want %d", resp.StatusCode, http.StatusOK)
    }
}
```

---

## 9. Benchmarks and Coverage

```go
func BenchmarkProcess(b *testing.B) {
    data := generateTestData(1000)
    b.ResetTimer()
    for i := 0; i < b.N; i++ { Process(data) }
}
// go test -bench=. -benchmem ./...
```

Coverage targets: 100% for critical business logic, 90%+ for public APIs, 80%+ overall.

```bash
go test -race -coverprofile=coverage.out ./...
go tool cover -html=coverage.out
```

---

## 10. Go Modules and Tooling

```bash
go mod tidy && go mod verify
go vet ./...
golangci-lint run
gofmt -w . && goimports -w .
```

Recommended linters in `.golangci.yml`: `errcheck`, `govet`, `staticcheck`, `unused`, `gofmt`, `goimports`, `misspell`.

**Rule:** Run `go test -race ./...` in CI. Always format with `gofmt`/`goimports` before committing.
