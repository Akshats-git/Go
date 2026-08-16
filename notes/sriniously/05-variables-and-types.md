# 05 — Variables & Types

[▶ 1:09:12](https://www.youtube.com/watch?v=tgGNwG_UxFo&t=4152s)

---

## Declaring variables

### `var` — works anywhere

```go
var c, python, java bool         // package level or function level
var i int                        // zero-initialized to 0
var i, j int = 1, 2              // with initializers
var v = 10                       // type inferred from initializer
```

If several variables share a type, one type annotation at the end covers them all. Different types ⇒ annotate each.

### `:=` — short declaration, functions only

```go
func main() {
	i := 7                // declare + initialize, type inferred
	name := "akshat"
}
```

```go
k := 8   // ❌ at package level: "expected declaration, found k"
```

**`:=` only works inside a function.** Package-level declarations must use `var`.

`var v = 10` and `v := 10` are the same thing — `:=` is syntactic sugar for it.

---

## Which form to use — the decision rule

Three cases, and only three:

| Situation | Use | Why |
|---|---|---|
| Initial value **doesn't matter** — will be set later in the program | `var i int` | Signals to the reader: "starts at zero value, real value assigned later" |
| Initial value **is known and matters** | `i := 7` | Most common by far |
| **Outside a function** (package level) | `var c bool` | No choice — `:=` is illegal here |

### Real-world check (fider)

- Package-level variables: all `var` — forced.
- Inside functions: `var err error` before a block where the error gets assigned later by a function call. Initial value is meaningless, so `var` communicates that.
- `:=` everywhere else — global search shows it's the default. Most of the time you know the initial value.

---

## Zero values — the core guarantee

**A Go variable is never uninitialized, undefined, or null.** Every declaration produces a value.

| Type | Zero value |
|---|---|
| All numeric types (`int`, `float64`, `uint8`, …) | `0` |
| `bool` | `false` |
| `string` | `""` |
| All **reference types** (pointer, slice, map, channel, func) | `nil` |
| `interface` | `nil` |
| **Array** | the array with every element at its zero value — `[4]int{}` → `[0 0 0 0]` |
| **Struct** | the struct with every field at its zero value |

```go
type T struct {
	A int
	B string
}
var t T   // T{A: 0, B: ""}
```

This is why Go has no null-pointer-exception epidemic for value types, and why `if err != nil` works — see [12 — Interfaces](12-interfaces.md).

---

## The four categories of Go types

The video's taxonomy — a useful mental map:

### 1. Basic
- **Numbers** — integers, unsigned integers, floats, complex, plus `byte` and `rune`
- **String**
- **Bool**

### 2. Aggregate
Types built out of other types:
- **Array** — collection of **homogeneous** data (all one type)
- **Struct** — collection of **heterogeneous** data (mixed types)

### 3. Reference
They don't hold the data themselves; they *refer* to underlying data working in the background. **Five of them:**
- **Pointer**
- **Slice**
- **Map**
- **Channel**
- **Function**

All have zero value `nil`.

### 4. Interface
Its own category. See [12 — Interfaces](12-interfaces.md).

---

## Numeric types in detail

```
int  int8  int16  int32  int64
uint uint8 uint16 uint32 uint64 uintptr
float32 float64
complex64 complex128
byte = uint8      rune = int32
```

### What the number means

The number is the **bit width**, which determines the range.

- `int8` → 8 bits → 2⁸ = 256 distinct values, signed → **−128 to 127**
- `uint8` → 8 bits → 256 values, unsigned → **0 to 255**
- Same logic scales to 16, 32, 64.

> ⚠️ **Correction:** the video says `int8` ranges "−127 to +128." It's **−128 to +127** (one extra negative slot because zero occupies a positive-side value). `uint8` = 0–255 as stated, correct.

### Plain `int` / `uint`

Sized by your platform architecture: 32 bits on a 32-bit system, 64 bits on a 64-bit system.

**The official recommendation: default to `int` / `uint`.** Reach for a sized type only when you genuinely know the range.

### When a sized type is worth it — real example

fider defines log levels as `uint8`:

```go
type Level uint8

const (
	DEBUG Level = iota + 1
	INFO
	WARN
	ERROR
	// ...
)
```

The range is 1 to maybe 5 (worst case 10 or 12). On a 64-bit machine `uint` would allocate 64 bits per value for something that never exceeds a handful. `uint8` (0–255) is plenty and saves memory across many instances.

**Rule of thumb:** if you know the maximum value in advance, size the type to it. If you're handling user data or values of unknown magnitude, use plain `int` — the performance difference is negligible and the docs recommend it.

### `byte` and `rune`

- `byte` = alias for `uint8` — used to represent the raw bytes of strings
- `rune` = alias for `int32` — used to represent a single UTF-8 character

### Floats and complex

- Floats: only `float32` and `float64`. No platform-sized `float`.
- Complex: `complex64`, `complex128`. Exist; you will likely never use them.

---

## Printing verbs

```go
fmt.Printf("Type: %T  Value: %v\n", x, x)
```

| Verb | Prints |
|---|---|
| `%v` | the **value** in Go's default format |
| `%T` | the **type** |
| `%s` | a string |
| `%q` | a string **with quotes** |

```go
var s string
fmt.Printf("%v\n", s)   // (prints nothing — empty string)
fmt.Printf("%q\n", s)   // ""
```

Don't memorize the verbs. You'll absorb the handful you use.

---

## Type conversion — always explicit

Go performs **no implicit conversion**, ever. Not even between numeric types that "obviously" fit.

```go
var k int64 = 32
var v int32 = 32

v = k              // ❌ compile error — different types
v = int32(k)       // ✅ explicit conversion
```

```go
var x, y int = 3, 4
f := math.Sqrt(float64(x*x + y*y))
z := uint(f)
fmt.Println(x, y, z)   // 3 4 5
```

Syntax is `TypeName(value)`.

You rarely have to *worry* about this: these are compile-time errors, so your editor's LSP flags them as you type. You fix them while writing, not while debugging.

---

## Type inference

```go
v := 42        // int
v := 42.7      // float64
v := "hello"   // string
```

The compiler derives the type from the initializer. **It's inferred exactly once, and it's permanent:**

```go
v := "hello"
v = 9   // ❌ cannot use 9 (untyped int constant) as string value
```

The type is fixed for the variable's whole lifetime. That's what "statically typed" means.

---

## Constants

```go
const Pi = 3.14
const Truth = true
const World = "世界"
```

**Properties:**

1. **Must be initialized at declaration.** No `const x int` — that's an error.
2. **Cannot be reassigned.** `Truth = false` → *cannot assign to Truth (neither addressable nor a map index expression)*, a compile-time error.
3. **Evaluated at compile time**, not runtime. That's *why* an initializer is mandatory — the compiler has to know the value.
4. **Only three types allowed:** number, string, bool.

### When to use a constant

Any time you have a **hardcoded value**: a port number, a URL, an allowed set of values to validate against, a magic number.

Naming it as a constant does two things: gives it **semantic meaning** for the reader, and gives it a **compile-time guarantee** that it can never change.

### Real-world check (fider)

Constants are used to implement an **enum pattern** — Go has no native enum:

```go
const (
	RoleVisitor       Role = 1
	RoleCollaborator  Role = 2
	RoleAdministrator Role = 3
)
```

Three otherwise-meaningless integers get meaningful names. (The full idiomatic enum pattern with `iota` and a `String()` method comes later in the series.)

---

**Next:** [06 — Control flow](06-control-flow.md)
