# 12 — Interfaces

[▶ 4:02:14](https://www.youtube.com/watch?v=tgGNwG_UxFo&t=14534s)

The deepest idea in Go's design, and the one you'll meet everywhere in the standard library. ~45 minutes of the video.

> The video's advice: don't try to fully master interfaces on first contact. Get the basics, move on, and let understanding accumulate as you read and write more code.

---

## What an interface is

A type whose definition is a **set of method signatures**:

```go
type Abser interface {
	Abs() float64
}
```

That's the whole syntax. Interfaces contain **method signatures, nothing else**.

An interface describes **behavior**: "anything with these methods."

---

## Implicit satisfaction — the defining feature

```go
type MyFloat float64
func (f MyFloat) Abs() float64 { ... }

type Vertex struct{ X, Y float64 }
func (v *Vertex) Abs() float64 { ... }

func main() {
	var a Abser

	f := MyFloat(-math.Sqrt2)
	a = f          // ✅ MyFloat has Abs() float64

	v := Vertex{3, 4}
	a = &v         // ✅ *Vertex has Abs() float64

	a = v          // ❌ compile error
	// Vertex does not implement Abser
	// (method Abs has pointer receiver)
}
```

**No `implements` keyword. No explicit declaration. No registration.** A type satisfies an interface *automatically* if it has the required methods with matching signatures.

This is the difference from Java/C#/TypeScript, where you declare intent. In Go the relationship is discovered by the compiler. A type can satisfy an interface that was written years later by someone who'd never seen it.

Recall from [05 — Variables & types](05-variables-and-types.md) that Go normally refuses to assign one type to another without explicit conversion. Interfaces are the exception — but only because the type genuinely provides the required behavior.

### ⚠️ The receiver gotcha

`a = v` fails while `a = &v` succeeds. Why? Because `Abs` was declared with a **pointer receiver** `(v *Vertex)`. Only `*Vertex` has that method in its method set — a plain `Vertex` value does not.

This is a concrete reason for the rule in [11 — Methods](11-methods.md): **don't mix value and pointer receivers on one type.** The mixture produces exactly this class of confusing error.

---

## What an interface value actually holds

An interface value is a **two-field container**:

```
┌─────────────────────────┐
│ INTERFACE VALUE         │
│   dynamic type          │
│   dynamic value         │
└─────────────────────────┘
```

```go
type I interface{ M() }

type T struct{ S string }
func (t T) M() { fmt.Println(t.S) }

var i I           // both fields empty → i is nil
i = T{"hello"}    // dynamic type = T, dynamic value = T{"hello"}
i.M()             // hello
```

Before assignment the container is **empty — the zero value of an interface is `nil`**. The compiler will happily let you *write* `i.M()` (the interface declares the method), but at runtime:

```go
var i I
i.M()   // 💥 panic: runtime error: invalid memory address or nil pointer dereference
```

**An interface gives you behavior, not data.** It must be given a concrete implementation before it's usable. Call it before that and the program panics.

---

## The empty interface

```go
interface{}
```

Zero methods. Since satisfaction means "has all the required methods," and there are none required, **every type in Go satisfies the empty interface.**

```go
var i interface{}
i = 42
i = "hello"
i = []int{1, 2, 3}   // all fine
```

### `any`

Go 1.18 added an alias:

```go
type any = interface{}
```

Purely a readability shortcut — identical meaning.

### Where you've already been using it

```go
func Println(a ...any) (n int, err error)
```

`fmt.Println` and friends are **variadic functions taking `...any`** — any number of arguments of any type. That's how they accept anything you throw at them.

---

## Type assertion — getting the concrete type back

Once a value is inside an empty interface, Go's static type system won't let you use it. You can't do arithmetic on it, take its length, or anything else:

```go
var i any = "hello"
i = i + 2      // ❌ invalid operation: mismatched types interface{} and int
len(i)         // ❌ invalid argument: i (variable of type interface{}) for len
```

You must **assert** it back to a concrete type:

```go
s := i.(string)
fmt.Println(s)   // hello
```

### The two-value (safe) form

```go
s, ok := i.(string)    // "hello", true
f, ok := i.(float64)   // 0, false — no panic
```

`ok` reports whether the assertion succeeded. The value is the zero value on failure.

### The one-value (unsafe) form

```go
f := i.(float64)   // 💥 panic: interface conversion: interface {} is string, not float64
```

**Without the `ok` variable, a failed assertion panics at runtime.** Not a compile error — a runtime crash.

> **Use the two-value form** unless you're certain of the type.

---

## Type switch

Asserting one type at a time gets tedious. The type switch handles a list:

```go
func do(i any) {
	switch v := i.(type) {
	case int:
		fmt.Printf("Twice %v is %v\n", v, v*2)
	case string:
		fmt.Printf("%q is %v bytes long\n", v, len(v))
	default:
		fmt.Printf("I don't know about type %T!\n", v)
	}
}

do(21)      // Twice 21 is 42
do("hello") // "hello" is 5 bytes long
do(true)    // I don't know about type bool!
```

The magic syntax is **`i.(type)`**, legal only inside a switch header. In each case, `v` already has that case's concrete type — so `v*2` works in the `int` case and `len(v)` works in the `string` case.

You'll see this pattern anywhere `any` is involved.

---

## `Stringer` — a standard-library interface you should know

```go
// package fmt
type Stringer interface {
	String() string
}
```

**Any type with a `String() string` method is a Stringer.** That's the whole contract.

```go
type Person struct {
	Name string
	Age  int
}

func (p Person) String() string {
	return fmt.Sprintf("%v (%v years)", p.Name, p.Age)
}

a := Person{"Arthur Dent", 42}
fmt.Println(a)   // Arthur Dent (42 years)
```

Comment out the `String()` method and the output reverts to Go's default struct format `{Arthur Dent 42}`.

### How it works

`Println`/`Printf`/`Sprintf` take `...any`. Internally they check whether each argument satisfies `Stringer` — i.e. has a `String()` method. If yes, they call it. If no, they fall back to the default representation.

This is implicit satisfaction paying off: you control how your type prints, everywhere, by adding one method. No registration, no inheritance.

> **Naming convention:** interfaces are conventionally named with an **`-er` suffix** — `Stringer`, `Reader`, `Writer`, `Handler`. Interfaces define *behavior*, and the `-er` suffix conveys "the thing that does X." Note `fmt.Sprintf` in the example — it **formats and returns** a string rather than printing it.

---

## `error` — the interface behind all Go error handling

```go
// builtin
type error interface {
	Error() string
}
```

**Any type with an `Error() string` method is an error.** Same mechanism as `Stringer`.

```go
type MyError struct {
	When time.Time
	What string
}

func (e *MyError) Error() string {
	return fmt.Sprintf("at %v, %s", e.When, e.What)
}

func run() error {                       // returns the INTERFACE
	return &MyError{
		time.Now(),
		"it didn't work",
	}                                    // returns a CONCRETE type
}

func main() {
	if err := run(); err != nil {
		fmt.Println(err)
	}
}
```

### Why return `error` and not `*MyError`

This is the crux of `if err != nil`:

- The zero value of an interface is **`nil`**.
- So "no error" is expressible simply as returning `nil`.
- And "an error occurred" is any concrete type satisfying `error`.

The caller doesn't need to know *which* error type came back. It only checks:

```go
if err != nil {
	// failure path
}
// success path
```

One check works against every error type in existence. That's the entire foundation of Go's explicit, value-based error handling — no exceptions, no try/catch, errors are ordinary return values.

---

## Interfaces in real backends: dependency injection and testing

[▶ ~4:32:00](https://www.youtube.com/watch?v=tgGNwG_UxFo&t=16320s)

The single most valuable practical use of interfaces in backend Go.

### The typical layering

```
┌──────────────────┐
│   Controller     │  HTTP: parse body/query params, validate,
│                  │  call service, write response + status code
├──────────────────┤
│    Service       │  Business logic: orchestrates repository calls,
│                  │  sends emails, schedules tasks
├──────────────────┤
│   Repository     │  Database: queries, inserts, updates, deletes
└──────────────────┘
```

(Some systems skip the service layer and let controllers call repositories. Terminology varies; three layers is typical.)

### The pattern (incubator-answer)

```go
// declared in the SERVICE package
type AuthRepo interface {
	GetUserCacheInfo(ctx context.Context, accessToken string) (*entity.UserCacheInfo, error)
	SetUserCacheInfo(ctx context.Context, ...) error
	// ...
}

type AuthService struct {
	authRepo AuthRepo
}

func NewAuthService(authRepo AuthRepo) *AuthService {
	return &AuthService{authRepo: authRepo}
}
```

The service **does not import a concrete repository struct**. It takes an **interface** as a constructor parameter. That's **dependency injection**.

### The convention: interfaces belong on the consumer side

> **Declare the interface where it's *used*, not where it's implemented.**

The consumer (the service) defines what it needs. The producer (the repository) just happens to satisfy it. Not the other way around — the producer should *not* define and export an interface for its consumers.

There are valid exceptions, but this is the rule of thumb that holds most of the time. It's the opposite of the Java instinct.

### Why this matters: testing without a database

The payoff shows up when you write **unit tests**. You don't want to spin up a Docker container and hit a real database just to test business logic — too slow, too fragile.

Because `AuthService` accepts an interface:

```
Production code path:
    NewAuthService(realAuthRepo)   → actual SQL queries

Test code path:
    NewAuthService(mockAuthRepo)   → returns hardcoded values / in-memory slices
```

Both satisfy `AuthRepo`, so both are accepted. The compiler doesn't care which — `AuthService` only cares that the methods exist.

Your mock is just a struct with the same method set, returning canned data. **No mocking library needed** — Go's implicit interface satisfaction gives you this natively. That's a substantial reduction in test boilerplate compared to languages that require mock frameworks.

---

## Interfaces elsewhere in the standard library

- `io.Reader` / `io.Writer` — the composition backbone of all I/O
- `os.File`, `http.Handler`, `sort.Interface`, and many more

Because satisfaction is implicit, an enormous amount of composition falls out for free — any type with a `Read([]byte) (int, error)` method works everywhere an `io.Reader` is expected.

---

**Next:** [13 — Generics](13-generics.md)
