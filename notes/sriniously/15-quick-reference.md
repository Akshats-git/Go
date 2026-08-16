# 15 — Quick Reference

Everything condensed. Use this after the first read-through.

---

## Commands

```bash
go mod init github.com/user/project   # create a module
go get github.com/some/package        # add a dependency to go.mod
go mod download                       # fetch all pinned dependencies
go run main.go                        # compile + run (development)
go build main.go                      # compile to a binary (deployment)
GOOS=windows GOARCH=amd64 go build .  # cross-compile
```

---

## Declarations

```go
var i int              // zero value, function or package level
var i, j int = 1, 2
var v = 10             // type inferred
i := 7                 // short form — INSIDE FUNCTIONS ONLY

const Pi = 3.14        // must be initialized; number/string/bool only
```

**Which to use:** initial value irrelevant → `var`. Initial value known and meaningful → `:=`. Package level → `var` (no choice).

---

## Zero values

| Type | Zero |
|---|---|
| numbers | `0` |
| `bool` | `false` |
| `string` | `""` |
| pointer, slice, map, channel, func | `nil` |
| interface | `nil` |
| array | all elements at their zero value |
| struct | all fields at their zero value |

---

## Type taxonomy

| Category | Types |
|---|---|
| **Basic** | numbers, string, bool |
| **Aggregate** | array (homogeneous), struct (heterogeneous) |
| **Reference** | pointer, slice, map, channel, function |
| **Interface** | interface |

Numeric: `int int8 int16 int32 int64` / `uint uint8 uint16 uint32 uint64` / `float32 float64` / `complex64 complex128`
Aliases: `byte` = `uint8`, `rune` = `int32`

**Default to plain `int`.** Size down only when you know the range.

---

## Conversion & inference

```go
var f float64 = float64(i)   // ALWAYS explicit — no implicit conversion
v := 42                      // inferred as int; type is fixed forever after
```

---

## Functions

```go
func add(x int, y int) int { return x + y }
func add(x, y int) int              // merged same-type params
func swap(x, y string) (string, string)   // multiple returns need parens
func split(sum int) (x, y int) { ...; return }   // named returns — AVOID
```

---

## Control flow

```go
for i := 0; i < 10; i++ { }     // classic
for sum < 1000 { }              // while
for { }                         // infinite
for i, v := range s { }         // range (v is a COPY)

if v := f(); v < lim { } else { }        // init statement, v scoped to if/else

switch os := runtime.GOOS; os {          // no fallthrough by default
case "darwin":
default:
}
switch { case x < 5: ... }               // replaces if/else-if chains
switch v := i.(type) { case int: ... }   // type switch

defer f()   // LIFO stack, runs on return AND on panic
```

---

## Pointers

```go
p := &i    // address of i, type *int
*p = 21    // write through the pointer — i changes
v := *p    // read through the pointer
```

---

## Structs

```go
type Vertex struct { X, Y int }

v1 := Vertex{1, 2}       // positional
v2 := Vertex{X: 1}       // named; Y = 0
v3 := Vertex{}           // all zero
p  := &Vertex{1, 2}      // *Vertex

v.X = 4
p.X = 9                  // auto-dereferenced, no need for (*p).X
```

**Tags:** `` Name string `json:"name"` `` — maps JSON keys to Go fields. Only **exported** (capitalized) fields can be marshaled.

---

## Arrays & slices

```go
var a [2]string          // ARRAY — size is part of the type, cannot resize
s := []int{1, 2, 3}      // SLICE literal
s := make([]int, 5)      // len 5, cap 5
s := make([]int, 0, 5)   // len 0, cap 5  ← the preallocation idiom
s = append(s, x)         // MUST reassign
s[1:4] / s[:4] / s[1:]   // low inclusive, high exclusive
len(s) / cap(s)
```

**Slice = pointer + length + capacity.** Capacity runs from the slice's first element to the **end of the backing array**.

- Slices sharing a backing array **see each other's writes**.
- `s[:n]` keeps capacity; `s[n:]` **reduces** capacity.
- `append` doubles capacity whenever `len == cap` (up to 256 elements, then ~1.25×).
- **Know the max size? `make([]T, 0, n)`** and skip every reallocation.

---

## Maps

```go
m := make(map[string]int)              // ✅
m := map[string]int{"a": 1}            // ✅
var m map[string]int; m["a"] = 1       // 💥 panic: nil map

m["k"] = 9
delete(m, "k")
v, ok := m["k"]                        // comma-ok: does the key exist?
for k, v := range m { }                // order is RANDOMIZED
```

**Keys must be comparable.** Structs to store, maps to look up. Map entries are not addressable.

---

## Functions as values

```go
f := func(x, y float64) float64 { return x + y }   // anonymous
func compute(fn func(float64, float64) float64) {} // function parameter
go func() { ... }()                                // inline + immediately called

func adder() func(int) int {                       // closure
	sum := 0
	return func(x int) int { sum += x; return sum }
}
```

---

## Methods

```go
func (v Vertex) Abs() float64   { }   // value receiver — gets a COPY
func (v *Vertex) Scale(f float64) { } // pointer receiver — can MUTATE
```

**Choosing a receiver — three rules:**
1. Mutating the instance? → **pointer**
2. Large struct you'd rather not copy? → **pointer**
3. Any other method on this type uses a pointer? → **pointer** (never mix)

Otherwise value. Go auto-references/dereferences on method calls, so `v.Scale(10)` works on a value.

---

## Interfaces

```go
type Abser interface { Abs() float64 }   // just method signatures
```

- **Satisfied implicitly** — no `implements` keyword; having the methods is enough.
- An interface value holds a **dynamic type + dynamic value**; zero value is `nil`.
- Calling a method on a nil interface **panics**.
- Pointer-receiver methods are in `*T`'s method set, not `T`'s.

```go
var i any = "hello"        // any = interface{} — every type satisfies it
s := i.(string)            // assertion — PANICS on failure
s, ok := i.(string)        // safe form
switch v := i.(type) { }   // type switch
```

**Two you must know:**

```go
type Stringer interface { String() string }   // controls how fmt prints your type
type error interface { Error() string }       // any type with Error() is an error
```

`if err != nil` works because the zero value of an interface is `nil`.

**Convention:** name interfaces with an `-er` suffix; **declare interfaces on the consumer side**, not the producer side. That's what makes mock-based unit testing work with no mocking library.

---

## Generics

```go
func Index[T comparable](s []T, x T) int { }
Index(si, 15)          // T inferred
Index[string](ss, "a") // explicit
```

Type parameters in `[]` before the value parameters, each with a constraint. Use when two functions share a body and differ only by type.

---

## Concurrency

```go
go f()                    // spawn a goroutine

ch := make(chan int)      // unbuffered — synchronizes
ch := make(chan int, 2)   // buffered — queue

ch <- v                   // send
v := <-ch                 // receive (BLOCKS)
close(ch)                 // sender broadcasts "done"
for v := range ch { }     // receive until closed

select {
case v := <-ch1:
case ch2 <- x:
default:                  // makes select non-blocking
}

var mu sync.Mutex
mu.Lock()
defer mu.Unlock()
```

- Goroutines are scheduler-managed and start at ~2–4 KB (vs 256 KB–1 MB for OS threads).
- **When `main` returns, every goroutine dies.** Main must wait.
- Spawn order and execution order are **never** guaranteed.
- *"Don't communicate by sharing memory; share memory by communicating."* Try channels first, mutexes second.

---

## Visibility

| First letter | Scope |
|---|---|
| **Capital** | exported — visible to other packages |
| lowercase | package-private |

Applies to everything: variables, constants, functions, types, struct fields, methods. Package names are always lowercase and short.

---

## The rules worth memorizing

1. `:=` inside functions only.
2. Nothing is ever uninitialized — everything has a zero value.
3. No implicit type conversion, ever.
4. Capitalization is the access modifier.
5. Arrays are fixed size; use slices.
6. `s = append(s, x)` — always reassign.
7. Never create a map with `var` — use `make` or a literal.
8. Never mix value and pointer receivers on one type.
9. Interfaces are satisfied implicitly, and belong on the consumer side.
10. `main` returning kills every goroutine.
