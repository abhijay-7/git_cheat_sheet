# Go (Golang) Cheatsheet

## CLI Commands

### Project Management
```bash
# Initialize a new module
go mod init <module-name>

# Add missing dependencies and remove unused ones
go mod tidy

# Download dependencies
go mod download

# Verify dependencies
go mod verify

# Create vendor directory
go mod vendor
```

### Running and Building
```bash
# Run a Go program
go run main.go
go run .

# Build executable
go build
go build -o myapp main.go

# Install package
go install

# Cross-compile
GOOS=linux GOARCH=amd64 go build
GOOS=windows GOARCH=amd64 go build
GOOS=darwin GOARCH=amd64 go build
```

### Testing
```bash
# Run tests
go test
go test ./...
go test -v
go test -cover
go test -bench=.
go test -race

# Generate test coverage report
go test -coverprofile=coverage.out
go tool cover -html=coverage.out
```

### Code Quality
```bash
# Format code
go fmt ./...

# Lint code (requires golint)
golint ./...

# Vet code for issues
go vet ./...

# Get dependencies
go get package-name
go get -u package-name  # update
go get package-name@version
```

### Documentation
```bash
# Generate and serve docs
go doc package-name
godoc -http=:6060
```

## Basic Syntax

### Variables and Constants
```go
// Variable declarations
var name string = "John"
var age int = 30
var isActive bool = true

// Short declaration
name := "John"
age := 30

// Multiple variables
var (
    name string = "John"
    age  int    = 30
)

// Constants
const Pi = 3.14159
const (
    StatusOK = 200
    StatusNotFound = 404
)
```

### Data Types
```go
// Basic types
bool
string
int, int8, int16, int32, int64
uint, uint8, uint16, uint32, uint64
float32, float64
complex64, complex128
byte (alias for uint8)
rune (alias for int32)

// Type conversion
var i int = 42
var f float64 = float64(i)
var u uint = uint(f)
```

### Arrays and Slices
```go
// Arrays
var arr [5]int
arr[0] = 1

// Array literal
numbers := [5]int{1, 2, 3, 4, 5}

// Slices
var slice []int
slice = append(slice, 1, 2, 3)

// Make slice
slice := make([]int, 5)      // length 5
slice := make([]int, 0, 10)  // length 0, capacity 10

// Slice operations
slice[1:3]  // elements 1 to 2
slice[:3]   // elements 0 to 2
slice[2:]   // elements 2 to end
slice[:]    // all elements

// Copy slices
copy(dest, src)
```

### Maps
```go
// Create map
var m map[string]int
m = make(map[string]int)

// Map literal
m := map[string]int{
    "apple":  5,
    "banana": 3,
}

// Operations
m["key"] = 42
value := m["key"]
value, ok := m["key"]  // check if key exists
delete(m, "key")

// Iterate over map
for key, value := range m {
    fmt.Println(key, value)
}
```

### Control Flow

#### If-Else
```go
if x > 10 {
    fmt.Println("x is greater than 10")
} else if x < 5 {
    fmt.Println("x is less than 5")
} else {
    fmt.Println("x is between 5 and 10")
}

// If with statement
if err := someFunction(); err != nil {
    return err
}
```

#### Switch
```go
switch day {
case "monday":
    fmt.Println("Start of work week")
case "friday":
    fmt.Println("TGIF!")
default:
    fmt.Println("Regular day")
}

// Switch without expression
switch {
case x < 0:
    fmt.Println("negative")
case x > 0:
    fmt.Println("positive")
default:
    fmt.Println("zero")
}
```

#### Loops
```go
// For loop
for i := 0; i < 10; i++ {
    fmt.Println(i)
}

// While-style loop
for condition {
    // code
}

// Infinite loop
for {
    // code
    break
}

// Range loop
for index, value := range slice {
    fmt.Println(index, value)
}

// Range with map
for key, value := range m {
    fmt.Println(key, value)
}

// Skip index or value
for _, value := range slice {}  // skip index
for key := range m {}           // skip value
```

## Functions

### Basic Functions
```go
func add(x int, y int) int {
    return x + y
}

// Multiple parameters of same type
func add(x, y int) int {
    return x + y
}

// Multiple return values
func divide(x, y float64) (float64, error) {
    if y == 0 {
        return 0, errors.New("division by zero")
    }
    return x / y, nil
}

// Named return values
func split(sum int) (x, y int) {
    x = sum * 4 / 9
    y = sum - x
    return  // naked return
}

// Variadic function
func sum(nums ...int) int {
    total := 0
    for _, num := range nums {
        total += num
    }
    return total
}
```

### Function Types and Closures
```go
// Function as type
type Calculator func(int, int) int

// Function as variable
var calc Calculator = add

// Anonymous function
func() {
    fmt.Println("Anonymous function")
}()

// Closure
func counter() func() int {
    count := 0
    return func() int {
        count++
        return count
    }
}
```

## Structs and Methods

### Structs
```go
type Person struct {
    Name string
    Age  int
}

// Create struct
p := Person{Name: "John", Age: 30}
p := Person{"John", 30}  // positional

// Anonymous struct
p := struct {
    Name string
    Age  int
}{
    Name: "John",
    Age:  30,
}

// Embedded structs
type Employee struct {
    Person
    ID int
}
```

### Methods
```go
// Method with receiver
func (p Person) GetName() string {
    return p.Name
}

// Pointer receiver (can modify)
func (p *Person) SetAge(age int) {
    p.Age = age
}

// Method call
p := Person{Name: "John", Age: 30}
fmt.Println(p.GetName())
p.SetAge(31)
```

## Interfaces

### Interface Definition
```go
type Writer interface {
    Write([]byte) (int, error)
}

type Reader interface {
    Read([]byte) (int, error)
}

// Empty interface
interface{}  // can hold any type
```

### Interface Implementation
```go
type MyWriter struct{}

func (mw MyWriter) Write(data []byte) (int, error) {
    // implementation
    return len(data), nil
}

// Usage
var w Writer = MyWriter{}
```

### Type Assertions and Type Switches
```go
// Type assertion
var i interface{} = "hello"
s := i.(string)
s, ok := i.(string)  // safe assertion

// Type switch
switch v := i.(type) {
case int:
    fmt.Printf("Integer: %d\n", v)
case string:
    fmt.Printf("String: %s\n", v)
default:
    fmt.Printf("Unknown type\n")
}
```

## Error Handling

### Basic Error Handling
```go
import "errors"

// Create error
err := errors.New("something went wrong")

// Custom error type
type MyError struct {
    Msg string
}

func (e MyError) Error() string {
    return e.Msg
}

// Error handling pattern
result, err := someFunction()
if err != nil {
    return err
}
```

### Error Wrapping (Go 1.13+)
```go
import "fmt"

// Wrap error
err := fmt.Errorf("failed to process: %w", originalErr)

// Check for wrapped error
if errors.Is(err, originalErr) {
    // handle
}

// Extract wrapped error
var myErr *MyError
if errors.As(err, &myErr) {
    // handle MyError
}
```

## Goroutines and Channels

### Goroutines
```go
// Start goroutine
go someFunction()
go func() {
    fmt.Println("Anonymous goroutine")
}()

// WaitGroup for synchronization
var wg sync.WaitGroup
wg.Add(1)
go func() {
    defer wg.Done()
    // work
}()
wg.Wait()
```

### Channels
```go
// Create channel
ch := make(chan int)
ch := make(chan int, 10)  // buffered channel

// Send and receive
ch <- 42      // send
value := <-ch // receive

// Close channel
close(ch)

// Range over channel
for value := range ch {
    fmt.Println(value)
}

// Select statement
select {
case msg1 := <-ch1:
    fmt.Println("Received from ch1:", msg1)
case msg2 := <-ch2:
    fmt.Println("Received from ch2:", msg2)
case <-time.After(1 * time.Second):
    fmt.Println("Timeout")
default:
    fmt.Println("No communication ready")
}
```

## Common Packages

### fmt - Formatting
```go
fmt.Print("Hello")
fmt.Println("Hello")
fmt.Printf("Name: %s, Age: %d\n", name, age)
fmt.Sprintf("Formatted: %s", value)

// Scan input
fmt.Scan(&variable)
fmt.Scanf("%d", &number)
```

### strings - String Operations
```go
import "strings"

strings.Contains("hello world", "world")  // true
strings.HasPrefix("hello", "he")          // true
strings.HasSuffix("hello", "lo")          // true
strings.Index("hello", "ll")              // 2
strings.Replace("hello", "l", "x", 2)     // "hexxo"
strings.Split("a,b,c", ",")               // ["a", "b", "c"]
strings.Join([]string{"a", "b"}, ",")     // "a,b"
strings.ToLower("HELLO")                  // "hello"
strings.ToUpper("hello")                  // "HELLO"
strings.TrimSpace("  hello  ")            // "hello"
```

### strconv - String Conversions
```go
import "strconv"

// String to number
i, err := strconv.Atoi("42")
f, err := strconv.ParseFloat("3.14", 64)

// Number to string
s := strconv.Itoa(42)
s := strconv.FormatFloat(3.14, 'f', 2, 64)
```

### time - Time Operations
```go
import "time"

now := time.Now()
fmt.Println(now.Format("2006-01-02 15:04:05"))

// Duration
time.Sleep(2 * time.Second)
duration := 5 * time.Minute

// Timer
timer := time.NewTimer(2 * time.Second)
<-timer.C

// Ticker
ticker := time.NewTicker(1 * time.Second)
for t := range ticker.C {
    fmt.Println("Tick at", t)
}
```

### os - Operating System Interface
```go
import "os"

// Environment variables
os.Getenv("HOME")
os.Setenv("KEY", "value")

// Command line arguments
args := os.Args

// File operations
file, err := os.Open("file.txt")
file, err := os.Create("file.txt")
os.Remove("file.txt")
```

### io/ioutil - I/O Utilities
```go
import "io/ioutil"

// Read file
data, err := ioutil.ReadFile("file.txt")

// Write file
err := ioutil.WriteFile("file.txt", data, 0644)
```

### json - JSON Encoding/Decoding
```go
import "encoding/json"

type Person struct {
    Name string `json:"name"`
    Age  int    `json:"age"`
}

// Marshal (encode)
p := Person{Name: "John", Age: 30}
data, err := json.Marshal(p)

// Unmarshal (decode)
var p Person
err := json.Unmarshal(data, &p)
```

## Memory Management

### Pointers
```go
// Declare pointer
var p *int

// Get address
x := 42
p = &x

// Dereference pointer
fmt.Println(*p)  // prints 42

// New function
p := new(int)  // allocates memory, returns pointer
*p = 42
```

### Make vs New
```go
// make: slices, maps, channels
slice := make([]int, 10)
m := make(map[string]int)
ch := make(chan int)

// new: returns pointer to zero value
p := new(int)  // *int pointing to 0
```

## Testing

### Basic Test
```go
// file: math_test.go
package main

import "testing"

func TestAdd(t *testing.T) {
    result := add(2, 3)
    expected := 5
    
    if result != expected {
        t.Errorf("add(2, 3) = %d; want %d", result, expected)
    }
}

// Table-driven tests
func TestAddTable(t *testing.T) {
    tests := []struct {
        a, b, expected int
    }{
        {2, 3, 5},
        {0, 0, 0},
        {-1, 1, 0},
    }
    
    for _, test := range tests {
        result := add(test.a, test.b)
        if result != test.expected {
            t.Errorf("add(%d, %d) = %d; want %d", 
                test.a, test.b, result, test.expected)
        }
    }
}

// Benchmark
func BenchmarkAdd(b *testing.B) {
    for i := 0; i < b.N; i++ {
        add(2, 3)
    }
}
```

## Best Practices

### Code Organization
- Use `gofmt` to format code
- Use meaningful variable and function names
- Keep functions small and focused
- Use interfaces for abstraction
- Handle errors explicitly
- Use `go vet` and `golint` for code quality

### Performance Tips
- Use pointers for large structs
- Prefer slices over arrays
- Use buffered channels when appropriate
- Profile with `go tool pprof`
- Use `sync.Pool` for object reuse

### Common Patterns
```go
// Constructor pattern
func NewPerson(name string, age int) *Person {
    return &Person{
        Name: name,
        Age:  age,
    }
}

// Options pattern
type Option func(*Server)

func WithPort(port int) Option {
    return func(s *Server) {
        s.port = port
    }
}

func NewServer(opts ...Option) *Server {
    s := &Server{}
    for _, opt := range opts {
        opt(s)
    }
    return s
}
```