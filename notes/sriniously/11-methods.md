# 11 — Methods

[▶ 3:42:21](https://www.youtube.com/watch?v=tgGNwG_UxFo&t=13341s)

A **method** is a function attached to a type. Go has no classes, but methods give types behavior.

**You can define a method on any type — struct, int, string, slice, anything — as long as the type is defined in the same package.** That's the only restriction.

---

## Syntax: the receiver

```go
type Vertex struct {
	X, Y float64
}

func (v Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}
```

The difference from a plain function is the extra parameter list **before the name**:

```
func (v Vertex) Abs() float64
     └────┬────┘
      the receiver
```

The receiver's type is what the method attaches to. This declares "`Abs` is a method on `Vertex`."

Inside the body, `v` is a **concrete instance** of the struct, and you access its fields through it.

### Calling

```go
v := Vertex{3, 4}
fmt.Println(v.Abs())   // 5
```

Calling `v.Abs()` passes `v` into the receiver.

**Convention:** receiver names are short (`v`, `u`, `s`) because you type them constantly. Not important enough to fight over.

---

## Methods on non-struct types

```go
type MyFloat float64

func (f MyFloat) Abs() float64 {
	if f < 0 {
		return float64(-f)
	}
	return float64(f)
}

f := MyFloat(-math.Sqrt2)
fmt.Println(f.Abs())
```

`type MyFloat float64` defines a **custom (user-defined) type** with `float64` as its underlying type. You can hang methods on it. This is the mechanism behind things like `type Role int` with `Role.String()`.

---

## Value receiver vs pointer receiver

```go
func (v Vertex) Abs() float64    { ... }   // VALUE receiver
func (v *Vertex) Scale(f float64) { ... }  // POINTER receiver
```

### The difference

| Receiver | What gets passed | Mutations visible to caller? |
|---|---|---|
| **Value** `(v Vertex)` | a **copy** of the instance | ❌ No |
| **Pointer** `(v *Vertex)` | the **address** of the instance | ✅ Yes |

```go
func (v *Vertex) Scale(f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}

v := Vertex{3, 4}
v.Scale(10)
fmt.Println(v.Abs())   // 50, not 5 — Scale actually mutated v
```

Swap `*Vertex` for `Vertex` in `Scale` and the result goes back to `5` — the mutation happened on a copy and was thrown away.

---

## Automatic referencing/dereferencing — a real advantage of methods

With a **function**, types must match exactly:

```go
func ScaleFunc(v *Vertex, f float64) { ... }

v := Vertex{3, 4}
ScaleFunc(v, 10)    // ❌ cannot use v (Vertex) as *Vertex
ScaleFunc(&v, 10)   // ✅ you must write the &
```

With a **method**, Go inserts it for you:

```go
v := Vertex{3, 4}
v.Scale(10)   // ✅ works — Go rewrites this as (&v).Scale(10)
```

It works both directions: a pointer variable calling a value-receiver method gets automatically dereferenced (`p.Abs()` → `(*p).Abs()`).

The payoff is readability. `p.Scale(10)` instead of `(&p).Scale(10)` scattered through your code. You stop having to think about whether the receiver signature matches what you're holding.

---

## Choosing a receiver — three rules

> These three rules are the practical takeaway of the whole section.

### 1. Are you mutating? → **pointer receiver**

If the method changes the instance and you want that change to persist, you need a pointer. A value receiver mutates a copy that's immediately discarded.

### 2. Is the struct large? → **pointer receiver**

Every call with a value receiver **copies the entire struct**. For a struct with 50–100 fields — slices, nested structs, maps — that copying is real work on every call. Not catastrophic, but avoidable. A pointer copies one address.

### 3. Did you already use a pointer receiver anywhere on this type? → **all of them are pointers**

**Do not mix value and pointer receivers on the same type.** Pick one and apply it to every method of that type.

Mixing produces subtle bugs (and, as [12 — Interfaces](12-interfaces.md) shows, breaks interface satisfaction in confusing ways). If even one method needs to mutate, make all of them pointer receivers.

**Default when none of these apply:** value receiver.

---

## Why methods instead of functions

[▶ ~3:57:00](https://www.youtube.com/watch?v=tgGNwG_UxFo&t=14220s)

The functional result is identical — so why bother?

### The real-world argument (fider)

fider defines a `User` struct with methods:

```go
func (u *User) HasProvider(provider string) bool {
	for _, p := range u.Providers {
		if p.Name == provider {
			return true
		}
	}
	return false
}

func (u *User) IsCollaborator() bool  { ... }
func (u *User) IsAdministrator() bool { ... }
```

Now compare the alternative — free functions:

```go
func UserHasProvider(u *User, provider string) bool
func IsUserCollaborator(u *User) bool
func IsUserAdministrator(u *User) bool
```

Notice what you're forced to do: **encode "user" into every function name**, because there's no other way to signal what the function is for. A bare `HasProvider()` floating in a package is meaningless to someone new — no way to know its intended scope.

Methods give you that scope for free:

```go
u.HasProvider("google")
u.IsCollaborator()
u.IsAdministrator()
```

The type is right there. The names get shorter, the scope is unambiguous, and the set of methods reads as **documentation of what a `User` can do**.

### The other benefit

If you come from OOP, this maps onto class methods. Methods let you declare "this type has these behaviors/capabilities" as a coherent group. And crucially — **methods are what satisfy interfaces**, which is where Go's real abstraction power lives. See [12 — Interfaces](12-interfaces.md).

### A note on that fider example

`HasProvider` doesn't mutate anything, yet uses a pointer receiver. Why? We can't know without reading the whole codebase, but the likely reasons are rule 2 (`User` is a large struct from a database) or rule 3 (some *other* method on `User` mutates, so consistency forces pointers everywhere).

---

**Next:** [12 — Interfaces](12-interfaces.md)
