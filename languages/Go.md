# Go Architecture

> **Agent load:** Open Project Structure, Principles, Error Handling, Configuration, and Project Prompt / Validation first. Open other sections only when the task needs them. Read project `AGENTS.md` if present. For reviews use `skills/review/audit/SKILL.md` (scope + mode). For naming/comments use `skills/engineering/craft/SKILL.md` (not detector scoring). Prefer extending an existing repo over scaffolding a parallel tree. Discover verify commands from the project; do not invent a toolchain.

Simple, idiomatic structure for Go applications.

---

## Project Structure

```
project/
├── cmd/
│   └── app/
│       └── main.go         # Entry point
├── internal/
│   ├── user/              # Domain logic
│   ├── storage/           # Database, files
│   └── http/              # HTTP handlers
├── pkg/                   # Public libraries (if needed)
├── tests/
└── go.mod
```

---

## Principles

**Keep it simple**
Go is designed for clarity. Don't fight it.

**Small packages**
Each package has one clear purpose.

**Accept interfaces, return structs**
Makes code flexible and testable.

**Handle errors explicitly**
No exceptions. Check every error.

**Avoid globals**
Pass dependencies explicitly.

---

## Package Organization

### Domain Logic

```
internal/user/
├── user.go          # Core types
├── store.go         # Storage interface
└── validator.go     # Validation logic
```

**user.go**
```go
package user

type User struct {
    Name string
    Age  int
}

func New(name string, age int) (*User, error) {
    if name == "" {
        return nil, errors.New("name required")
    }
    
    if !isValidAge(age) {
        return nil, errors.New("invalid age")
    }
    
    return &User{
        Name: name,
        Age:  age,
    }, nil
}

func isValidAge(age int) bool {
    return age >= 0 && age <= 150
}
```

**store.go**
```go
package user

type Store interface {
    Save(user *User) error
    Find(id int) (*User, error)
    Delete(id int) error
}
```

### Storage Implementation

```
internal/storage/
├── memory.go        # In-memory store
└── postgres.go      # Postgres implementation
```

**memory.go**
```go
package storage

import "sync"

type Memory struct {
    users map[int]*user.User
    mu    sync.RWMutex
    next  int
}

func NewMemory() *Memory {
    return &Memory{
        users: make(map[int]*user.User),
    }
}

func (m *Memory) Save(u *user.User) error {
    m.mu.Lock()
    defer m.mu.Unlock()
    
    id := m.next
    m.next++
    m.users[id] = u
    
    return nil
}

func (m *Memory) Find(id int) (*user.User, error) {
    m.mu.RLock()
    defer m.mu.RUnlock()
    
    u, exists := m.users[id]
    if !exists {
        return nil, errors.New("user not found")
    }
    
    return u, nil
}

func (m *Memory) Delete(id int) error {
    m.mu.Lock()
    defer m.mu.Unlock()
    
    delete(m.users, id)
    return nil
}
```

### HTTP Layer

```
internal/http/
├── server.go        # Server setup
└── handler.go       # Route handlers
```

**server.go**
```go
package http

import (
    "net/http"
    "time"
)

type Server struct {
    store user.Store
    mux   *http.ServeMux
}

func NewServer(store user.Store) *Server {
    s := &Server{
        store: store,
        mux:   http.NewServeMux(),
    }
    
    s.routes()
    return s
}

func (s *Server) routes() {
    s.mux.HandleFunc("/users", s.handleUsers)
    s.mux.HandleFunc("/users/", s.handleUser)
}

func (s *Server) Start(addr string) error {
    server := &http.Server{
        Addr:         addr,
        Handler:      s.mux,
        ReadTimeout:  10 * time.Second,
        WriteTimeout: 10 * time.Second,
    }
    
    return server.ListenAndServe()
}
```

**handler.go**
```go
package http

import (
    "encoding/json"
    "net/http"
)

func (s *Server) handleUsers(w http.ResponseWriter, r *http.Request) {
    switch r.Method {
    case http.MethodPost:
        s.createUser(w, r)
    default:
        http.Error(w, "method not allowed", http.StatusMethodNotAllowed)
    }
}

func (s *Server) createUser(w http.ResponseWriter, r *http.Request) {
    var req struct {
        Name string `json:"name"`
        Age  int    `json:"age"`
    }
    
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        http.Error(w, "invalid request", http.StatusBadRequest)
        return
    }
    
    u, err := user.New(req.Name, req.Age)
    if err != nil {
        http.Error(w, err.Error(), http.StatusBadRequest)
        return
    }
    
    if err := s.store.Save(u); err != nil {
        http.Error(w, "save failed", http.StatusInternalServerError)
        return
    }
    
    w.WriteHeader(http.StatusCreated)
    json.NewEncoder(w).Encode(u)
}
```

---

## Main Entry Point

**cmd/app/main.go**
```go
package main

import (
    "log"
    "os"
    
    "project/internal/http"
    "project/internal/storage"
)

func main() {
    if err := run(); err != nil {
        log.Fatal(err)
    }
}

func run() error {
    store := storage.NewMemory()
    server := http.NewServer(store)
    
    port := os.Getenv("PORT")
    if port == "" {
        port = "8080"
    }
    
    log.Printf("Starting server on :%s", port)
    return server.Start(":" + port)
}
```

---

## Error Handling

**Return errors, don't panic**

```go
// Bad
func divide(a, b int) int {
    if b == 0 {
        panic("division by zero")
    }
    return a / b
}

// Good
func divide(a, b int) (int, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}
```

**Check every error**

```go
// Bad
data, _ := os.ReadFile(path)

// Good
data, err := os.ReadFile(path)
if err != nil {
    return fmt.Errorf("read file: %w", err)
}
```

**Wrap errors with context**

```go
func loadConfig(path string) (*Config, error) {
    data, err := os.ReadFile(path)
    if err != nil {
        return nil, fmt.Errorf("load config: %w", err)
    }
    
    var cfg Config
    if err := json.Unmarshal(data, &cfg); err != nil {
        return nil, fmt.Errorf("parse config: %w", err)
    }
    
    return &cfg, nil
}
```

---

## Functions

**Keep them short**

```go
// Bad - too long
func processOrder(order *Order) error {
    // validate
    // calculate total
    // apply discount
    // charge payment
    // update inventory
    // send email
    // 50+ lines
}

// Good - extract steps
func processOrder(order *Order) error {
    if err := validate(order); err != nil {
        return err
    }
    
    total := calculateTotal(order)
    
    if err := charge(order, total); err != nil {
        return err
    }
    
    updateInventory(order)
    sendConfirmation(order)
    
    return nil
}
```

**Do one thing**

```go
// Bad
func saveAndNotify(user *User) error {
    if err := db.Save(user); err != nil {
        return err
    }
    
    if err := email.Send(user); err != nil {
        return err
    }
    
    return nil
}

// Good
func save(user *User) error {
    return db.Save(user)
}

func notify(user *User) error {
    return email.Send(user)
}
```

**No boolean parameters**

```go
// Bad
func save(user *User, shouldLog bool) error {
    if shouldLog {
        log.Println("saving user")
    }
    return db.Save(user)
}

// Good
func save(user *User) error {
    return db.Save(user)
}

func saveWithLog(user *User) error {
    log.Println("saving user")
    return save(user)
}
```

---

## Naming

**Use short names in small scopes**

```go
// Good for small scope
for i, u := range users {
    // i and u are clear here
}

// Good for larger scope
func processUser(currentUser *User) error {
    // currentUser is clear
}
```

**Package names are singular**

```go
// Good
package user
package storage

// Bad
package users
package storages
```

**Avoid stutter**

```go
// Bad
user.UserStore
user.NewUser()

// Good
user.Store
user.New()
```

**Getters don't use "Get"**

```go
// Bad
user.GetName()

// Good
user.Name()
```

---

## Interfaces

**Keep them small**

```go
// Bad - too large
type UserService interface {
    Create(*User) error
    Update(*User) error
    Delete(int) error
    Find(int) (*User, error)
    List() ([]*User, error)
    Validate(*User) error
    SendEmail(*User) error
}

// Good - split
type Store interface {
    Save(*User) error
    Find(int) (*User, error)
    Delete(int) error
}

type Notifier interface {
    Send(*User) error
}
```

**Accept interfaces, return structs**

```go
// Good
func NewService(store Store) *Service {
    return &Service{store: store}
}

// Bad - harder to test
func NewService(db *sql.DB) *Service {
    return &Service{db: db}
}
```

---

## Concurrency

**Use channels to communicate**

```go
func process(items []Item) {
    results := make(chan Result)
    
    for _, item := range items {
        go func(i Item) {
            results <- processItem(i)
        }(item)
    }
    
    for range items {
        result := <-results
        handleResult(result)
    }
}
```

**Protect shared state with mutexes**

```go
type Cache struct {
    data map[string]string
    mu   sync.RWMutex
}

func (c *Cache) Get(key string) (string, bool) {
    c.mu.RLock()
    defer c.mu.RUnlock()
    
    val, ok := c.data[key]
    return val, ok
}

func (c *Cache) Set(key, val string) {
    c.mu.Lock()
    defer c.mu.Unlock()
    
    c.data[key] = val
}
```

---

## Testing

**Table-driven tests**

```go
func TestValidateAge(t *testing.T) {
    tests := []struct {
        age   int
        valid bool
    }{
        {30, true},
        {0, true},
        {150, true},
        {-1, false},
        {151, false},
    }
    
    for _, tt := range tests {
        got := isValidAge(tt.age)
        if got != tt.valid {
            t.Errorf("isValidAge(%d) = %v, want %v", tt.age, got, tt.valid)
        }
    }
}
```

**Use interfaces for mocks**

```go
type mockStore struct {
    saveFunc func(*User) error
}

func (m *mockStore) Save(u *User) error {
    if m.saveFunc != nil {
        return m.saveFunc(u)
    }
    return nil
}

func TestService(t *testing.T) {
    mock := &mockStore{
        saveFunc: func(u *User) error {
            return errors.New("save failed")
        },
    }
    
    svc := NewService(mock)
    err := svc.CreateUser("Alice", 30)
    
    if err == nil {
        t.Error("expected error, got nil")
    }
}
```

---

## Configuration

Load config once at startup into a typed struct, validate it, and pass it down via
dependency injection, no `os.Getenv` calls scattered through the packages
(`../standards/Principles.md` §15.1). Fail fast on a missing required var.

```go
// internal/config/config.go
package config

import (
    "fmt"
    "os"
)

type Config struct {
    Port        string
    DatabaseURL string
    JWTSecret   string
}

func Load() (Config, error) {
    cfg := Config{
        Port:        getenv("PORT", "3000"),
        DatabaseURL: os.Getenv("DATABASE_URL"),
        JWTSecret:   os.Getenv("JWT_SECRET"),
    }
    if cfg.DatabaseURL == "" || cfg.JWTSecret == "" {
        return Config{}, fmt.Errorf("missing required env: DATABASE_URL and JWT_SECRET")
    }
    return cfg, nil
}

func getenv(key, fallback string) string {
    if v := os.Getenv(key); v != "" {
        return v
    }
    return fallback
}
```

`main.go` calls `config.Load()` first and exits non-zero if it errors. Secrets come
from the environment, never from committed files.

---

## Summary

Keep packages small and focused.
Handle every error explicitly.
Use short, clear names.
Accept interfaces, return structs.
Write table-driven tests.
Keep functions short and simple.

---

## Project Prompt

Write Go against the structure and rules above. Where they disagree with your defaults,
this file wins.

Read `../standards/Principles.md` alongside this file before starting.

**Error Handling**
- Check every error explicitly (no `_` ignoring)
- Wrap errors with context: `fmt.Errorf("context: %w", err)`
- Return errors, don't panic
- Custom error types for domain errors

**Interfaces**
- Small interfaces (1-3 methods)
- Define at point of use, not implementation
- No god interfaces

**Concurrency**
- Goroutines for I/O, not CPU
- Always provide context for cancellation
- Close channels from sender
- Use mutexes or channels for shared state

**Testing**
- Table-driven tests
- Use interfaces for mocking
- Test all error paths

### Setup

```bash
go mod init projectname
mkdir -p cmd/api internal/{handlers,services,repository,models}
```

### Deliverables

1. Complete project following architecture structure above
2. HTTP handlers with proper routing
3. Service layer with business logic
4. Repository layer for data access
5. Dependency injection in main.go
6. Context-aware operations
7. Every error checked and wrapped with context
8. README with setup instructions
9. Test files for all packages

### Validation Checklist

- [ ] Verify commands from project AGENTS.md / README run (or honest manual checks listed)
- [ ] No secrets committed; env examples use placeholders only

- [ ] Functions are small and single-purpose; extract when a second concern appears (see Principles / skills/engineering/craft/SKILL.md)
- [ ] All errors checked (no `_` ignoring)
- [ ] Errors wrapped with context
- [ ] Interfaces at point of use
- [ ] No stuttering in names
- [ ] Context passed to long operations
- [ ] Proper resource cleanup (defer)
- [ ] Names match domain and local convention (skills/engineering/craft/SKILL.md)
- [ ] `go vet` and `golint` clean

### Pre-Delivery

```bash
go build ./...        # compiles clean
go vet ./...          # no vet warnings
go test -race ./...   # all tests pass under the race detector
gofmt -l .            # prints nothing (everything formatted)
```
