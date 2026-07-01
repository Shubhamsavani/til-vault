# GO NOTES
> Based on: *Learn GO Fast: Full Tutorial*

---

## 1. Key Features of Go

### Statically Typed
Variables must have a declared or inferred type, and that type cannot change.

```go
var myVariable string        // explicit type declaration
var myVariable = "SHUBHAM"   // type inferred as string
```

---

### Strongly Typed
Operations depend on the variable's type. You **cannot** mix types in operations.

```go
var a = 1
var b = "STRING"
var c = a + b   // ❌ NOT ALLOWED — can't add int + string
```
> Unlike weakly typed languages like JavaScript, Go enforces this strictly.

---

### Compiled Language
Go compiles to a binary (machine code) `.exe`/binary file — much faster than interpreted languages like Python.

**Speed comparison — counting to 100 million:**

| Language | Time       |
|----------|------------|
| Go       | ~50 ms     |
| Python   | ~6 seconds |

> **~120x faster** — compiled binary vs. interpreter overhead.

---

### Fast Compilation
Go compiles quickly, so you can write → compile → test in a tight feedback loop.

---

### Built-in Concurrency
No special packages needed. Concurrency is handled natively using **goroutines**.

```go
go myFunction()   // launches myFunction as a goroutine
```

---

### Simplicity
- Concise syntax
- Built-in **garbage collection** (automatic memory management)
- Less boilerplate, more readable code

---

## 2. Packages vs Modules

| Concept | Definition |
|---------|------------|
| **Package** | A folder containing one or more `.go` files |
| **Module** | A collection of packages; initialized when you start a project |

```bash
go mod init github.com/yourname/projectname
```
This creates a `go.mod` file — your module's config file, listing the module name, Go version, and any external dependencies.

---

## 3. Project Setup

```bash
mkdir go-tutorials
cd go-tutorials
go mod init goat-tutorials
```

**Folder structure:**
```
go-tutorials/
├── go.mod
└── cmd/
    └── tutorial1/
        └── main.go
```

Every Go file declares which package it belongs to at the top:
```go
package main
```

The `main` package is special — the compiler looks for a `main()` function here as the program entry point.

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello, World!")
}
```

**Run options:**
```bash
go build cmd/tutorial1/main.go   # compiles to binary
go run cmd/tutorial1/main.go     # compiles + runs in one step
```

---

## 4. Variables & Data Types

### Declaring Variables

```go
var intNum int                  // declared, default value = 0
var intNum int = 10             // declared with value
var intNum = 10                 // type inferred
intNum := 10                    // shorthand (most common)
```

> ⚠️ Every declared variable **must be used**, or the compiler throws an error.

---

### Integer Types

| Type   | Bits | Max Positive Value |
|--------|------|--------------------|
| int8   | 8    | 127                |
| int16  | 16   | 32,767             |
| int32  | 32   | ~2.1 billion       |
| int64  | 64   | very large         |
| int    | 32/64| depends on system  |

**Unsigned integers** (positive only, double the range):
```go
var u uint8 = 255
```

> ⚠️ Overflow errors only caught at compile time, **not** at runtime — choose types carefully!

---

### Float Types

```go
var f32 float32 = 3.14   // less precise
var f64 float64 = 3.14   // more precise (recommended)
```

> Go requires specifying `float32` or `float64` — there is no plain `float`.

---

### Arithmetic Notes

```go
// Cannot mix types:
var a int = 5
var b float32 = 3.2
// a + b  ❌ — must cast first:
float32(a) + b   // ✅

// Integer division truncates:
3 / 2   // = 1 (not 1.5)
3 % 2   // = 1 (remainder)
```

---

### String Type

```go
var s1 string = "Hello"          // double quotes — single line
var s2 string = `Hello
World`                           // backticks — multi-line, raw

s3 := s1 + " World"             // concatenation

len(s1)                          // length in BYTES (not characters!)
```

> For non-ASCII characters (e.g. accented letters, emojis), use:
```go
import "unicode/utf8"
utf8.RuneCountInString(s)   // true character count
```

---

### Booleans

```go
var b bool = true
var b bool = false
// default value = false
```

---

### Default Values

| Type            | Default |
|-----------------|---------|
| int, float, rune | 0      |
| string          | `""`    |
| bool            | `false` |

---

### Constants

```go
const Pi float64 = 3.14159   // cannot be changed after declaration
// const x int               // ❌ must initialize at declaration
```

---

## 5. Functions

```go
func printMe(printValue string) {
    fmt.Println(printValue)
}
```

### Returning Values

```go
func intDivision(numerator int, denominator int) int {
    return numerator / denominator
}
```

### Multiple Return Values

```go
func intDivision(numerator int, denominator int) (int, int, error) {
    if denominator == 0 {
        return 0, 0, errors.New("cannot divide by zero")
    }
    return numerator / denominator, numerator % denominator, nil
}
```

### Calling with Multiple Returns

```go
result, remainder, err := intDivision(10, 3)
if err != nil {
    fmt.Println(err.Error())
}
```

> Returning an `error` type is the standard Go pattern for error handling.

---

## 6. Control Structures

### If / Else If / Else

```go
if err != nil {
    fmt.Println("error:", err.Error())
} else if remainder == 0 {
    fmt.Println("divides evenly")
} else {
    fmt.Println("has remainder")
}
```

**Logical operators:**
```go
&&   // AND — both must be true
||   // OR  — at least one must be true
!=   // not equal
==   // equal
```

---

### Switch Statement

```go
switch {
case err != nil:
    fmt.Println(err.Error())
case remainder == 0:
    fmt.Println("No remainder")
default:
    fmt.Println("Has remainder")
}
```

**Conditional switch (value-based):**
```go
switch remainder {
case 0:
    fmt.Println("zero")
case 1, 2:
    fmt.Println("one or two")
default:
    fmt.Println("other")
}
```

> In Go, `break` is **implied** — no need to write it explicitly.

---

## 7. Arrays, Slices & Maps

### Arrays (Fixed Length)

```go
var arr [3]int32                        // [0, 0, 0]
arr[0] = 5                              // indexing
arr := [3]int32{1, 2, 3}               // initialize with values
arr := [...]int32{1, 2, 3}             // compiler infers length
```

- Zero-indexed
- Stored in **contiguous memory**
- Length **cannot change**

---

### Slices (Dynamic)

```go
var s []int32                       // slice (no length in brackets)
s = append(s, 4)                   // add element

s2 := make([]int32, 3, 10)         // length=3, capacity=10 (pre-allocated)
```

**Key concepts:**
- `len(s)` — number of elements
- `cap(s)` — size of underlying array
- Pre-allocating capacity avoids expensive reallocations (~3x faster)

```go
// Append multiple values:
s = append(s, otherSlice...)
```

> ⚠️ Slices **share** the underlying array when copied — modifying one affects the other!

---

### Maps (Key-Value)

```go
m := make(map[string]uint8)
m["Adam"] = 30

// Initialize with values:
m := map[string]uint8{"Adam": 30, "Eve": 28}

// Lookup:
age := m["Adam"]

// Check if key exists:
age, ok := m["Adam"]
if ok { fmt.Println(age) }

// Delete:
delete(m, "Adam")
```

> ⚠️ Missing keys return the **default value** of the type, not an error.

---

## 8. Loops

```go
// Range over slice/array:
for i, v := range mySlice {
    fmt.Println(i, v)
}

// Range over map:
for name, age := range myMap {
    fmt.Println(name, age)
}

// While-style loop:
for i < 10 {
    i++
}

// Break-based loop:
for {
    if i >= 10 { break }
    i++
}

// Classic C-style loop:
for i := 0; i < 10; i++ {
    fmt.Println(i)
}
```

> Go has **no `while` keyword** — use `for` instead.

---

## 9. Strings, Runes & Bytes

Go strings are stored as **UTF-8 encoded byte arrays**.

```go
s := "résumé"
len(s)           // bytes (NOT characters!)

// Indexing returns bytes:
s[0]             // uint8 (byte value)

// Range gives runes (decoded characters):
for i, v := range s {
    fmt.Println(i, v)   // v is the rune (Unicode code point)
}

// Cast to rune slice for proper indexing:
r := []rune(s)
r[1]   // = 233 (é character)
```

**Rune** = alias for `int32`, represents a Unicode code point. Use single quotes:
```go
var r rune = 'é'
```

### String Building (Efficient)

```go
import "strings"

var sb strings.Builder
sb.WriteString("Hello")
sb.WriteString(" World")
result := sb.String()
```

> ⚠️ Strings are **immutable** — using `+` creates a new string every time (slow in loops).

---

## 10. Structs

Define your own types with mixed fields:

```go
type GasEngine struct {
    MilesPerGallon uint8
    Gallons        uint8
}

// Creating instances:
e1 := GasEngine{MilesPerGallon: 25, Gallons: 10}
e2 := GasEngine{25, 10}     // positional
var e3 GasEngine             // zero-valued

// Access fields:
e1.MilesPerGallon
```

### Methods on Structs

```go
func (e GasEngine) MilesLeft() uint8 {
    return e.MilesPerGallon * e.Gallons
}

// Call it:
e1.MilesLeft()
```

### Embedded Structs

```go
type Owner struct {
    Name string
}

type GasEngine struct {
    MilesPerGallon uint8
    Gallons        uint8
    Owner                   // embedded — fields promoted
}

e.Name   // directly accessible
```

---

## 11. Interfaces

Define a set of method signatures that a type must implement:

```go
type Engine interface {
    MilesLeft() uint8
}

func canMakeIt(e Engine, miles int) bool {
    return int(e.MilesLeft()) >= miles
}
```

Now `canMakeIt` accepts **any type** that has a `MilesLeft() uint8` method — both `GasEngine` and `ElectricEngine`.

> Go uses **implicit interface satisfaction** — no `implements` keyword needed.

---

## 12. Pointers

Variables that store **memory addresses** rather than values.

```go
var p *int32             // pointer to int32, default = nil
p = new(int32)           // allocate memory
*p = 10                  // set value at address
fmt.Println(*p)          // dereference — get value at address

// Get address of existing variable:
i := 5
p = &i                   // p now points to i
*p = 20                  // i is now 20
```

**Star (`*`) has two roles:**
1. Declare a pointer: `var p *int32`
2. Dereference a pointer: `*p = 10`

### Pointers in Functions

```go
func square(arr *[5]float64) {
    for i, v := range arr {
        arr[i] = v * v
    }
}
square(&myArr)   // pass address — no copy made, saves memory
```

> ⚠️ Slices already contain pointers internally — copying a slice does **not** copy the underlying data.

---

## 13. Goroutines

Launch functions **concurrently** with the `go` keyword:

```go
go myFunction()
```

**Concurrency vs Parallelism:**
- **Concurrency** — multiple tasks *in progress* at the same time (may interleave on one core)
- **Parallelism** — multiple tasks executing *simultaneously* on multiple cores

### WaitGroups (sync)

```go
import "sync"

var wg sync.WaitGroup

for i := 0; i < 5; i++ {
    wg.Add(1)
    go func() {
        defer wg.Done()
        dbCall()
    }()
}
wg.Wait()   // blocks until all goroutines finish
```

### Mutex (Mutual Exclusion)

Prevents data races when multiple goroutines write to shared memory:

```go
var mu sync.Mutex

mu.Lock()
results = append(results, val)   // safe write
mu.Unlock()
```

### RWMutex (Read-Write Mutex)

Allows **multiple concurrent reads**, but exclusive writes:

```go
var mu sync.RWMutex

mu.RLock()            // multiple goroutines can read at once
// read data
mu.RUnlock()

mu.Lock()             // exclusive write lock
// write data
mu.Unlock()
```

> ⚠️ Place locks **only around shared memory access**, not around the whole concurrent operation — or you destroy the performance benefit.

---

## 14. Channels

A thread-safe way for goroutines to pass data to each other.

```go
ch := make(chan int)       // unbuffered — holds 1 value

// Send:
ch <- 5

// Receive:
val := <-ch
```

### With Goroutines

```go
func process(ch chan int) {
    ch <- 42
    close(ch)   // signal that no more values will be sent
}

ch := make(chan int)
go process(ch)

for v := range ch {
    fmt.Println(v)
}
```

### Buffered Channel

```go
ch := make(chan int, 5)   // can hold up to 5 values without blocking
```

> Sender doesn't block until the buffer is full.

### Select Statement

Listen on multiple channels:

```go
select {
case website := <-chickenCh:
    sendText(website)
case item := <-tofuCh:
    sendEmail(item)
}
```

---

## 15. Generics

Write functions/types that work with multiple types:

```go
func sumSlice[T int | float32 | float64](s []T) T {
    var sum T
    for _, v := range s {
        sum += v
    }
    return sum
}

sumSlice([]int{1, 2, 3})        // T inferred as int
sumSlice([]float32{1.1, 2.2})   // T inferred as float32
```

### `any` type

Use when the function works for all types (no type-specific operations):

```go
func isEmpty[T any](s []T) bool {
    return len(s) == 0
}
```

### Generics with Structs

```go
type Car[T GasEngine | ElectricEngine] struct {
    Make   string
    Model  string
    Engine T
}

gasCar := Car[GasEngine]{Engine: GasEngine{25, 10}}
```

---

## 16. Building an API

### Project Structure

```
project/
├── go.mod
├── api/
│   └── api.go          # request/response structs
├── cmd/
│   └── api/
│       └── main.go     # entry point
└── internal/
    ├── handlers/
    │   ├── api.go              # Handler() + route setup
    │   └── getCoinBalance.go   # endpoint logic
    ├── middleware/
    │   └── authorization.go    # auth middleware
    └── tools/
        ├── database.go         # DB interface + types
        └── mockdb.go           # mock implementation
```

### Key Concepts

**External packages** (install with):
```bash
go mod tidy
```

**Public vs Private** — capitalization controls visibility:
```go
func Handler(...)  // ✅ exported — other packages can use it
func handler(...)  // 🔒 private — only within this package
```

**Middleware** — runs before the main handler:
```go
r.Use(middleware.StripSlashes)          // global
r.Route("/account", func(r chi.Router) {
    r.Use(middleware.Authorization)     // route-specific
    r.Get("/coins", handlers.GetCoinBalance)
})
```

**Response Writer & Request:**
```go
func myHandler(w http.ResponseWriter, r *http.Request) {
    // r = incoming request (headers, body, params)
    // w = outgoing response (write status code, body)
}
```

**Start the server:**
```go
http.ListenAndServe(":8000", router)
```

---

## Quick Reference Cheat Sheet

```go
// Variable declaration
x := 10
var x int = 10

// Function
func add(a, b int) int { return a + b }

// Goroutine
go myFunc()

// Channel
ch := make(chan int)
ch <- val      // send
val = <-ch     // receive

// Defer (runs on function exit)
defer wg.Done()

// Error handling pattern
result, err := someFunc()
if err != nil { log.Fatal(err) }

// Struct + method
type Dog struct { Name string }
func (d Dog) Bark() { fmt.Println("Woof!") }

// Interface
type Animal interface { Bark() }
```