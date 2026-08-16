# 07 — Pointers & Structs

---

## Reference types recap

[▶ 2:03:27](https://www.youtube.com/watch?v=tgGNwG_UxFo&t=7407s)

Go has **five reference types**:

| Type | Covered in |
|---|---|
| **Pointer** | this file |
| **Slice** | [08 — Arrays & slices](08-arrays-and-slices.md) |
| **Map** | [09 — Maps](09-maps.md) |
| **Channel** | [14 — Concurrency](14-concurrency.md) |
| **Function** | [10 — Functions as values](10-functions-as-values.md) |

They're "reference" types because **they don't contain the data themselves** — they refer to some underlying data working in the background. All five have zero value `nil`.

---

## Pointers

A pointer stores the **memory address** of a variable.

Mental model: your program's memory is a grid of slots. Every variable lives in a slot, and every slot has an address. (In Go it's a *virtual* address — you're abstracted from the kernel by the Go runtime — but reason about it as a physical address.) A pointer is a variable whose value *is* one of those addresses.

### The two operators

| Operator | Name | Does |
|---|---|---|
| `&x` | address-of | gives you the address of `x` |
| `*p` | dereference | gives you the **value** at the address `p` holds |

### Worked example

```go
i, j := 42, 2701

p := &i           // p is of type *int — "pointer to int"
fmt.Println(*p)   // 42 — read i through the pointer
*p = 21           // write to i through the pointer
fmt.Println(i)    // 21  ← i itself changed

p = &j            // point p at j instead
*p = *p / 37
fmt.Println(j)    // 73
```

Output: `42`, `21`, `73`.

### The key points

- `p := &i` makes `p` type `*int`. **`p` is not an int** — it's a pointer that points to an int.
- **Printing `p` directly prints the address** (a garbage-looking hex value), not the data. Print `*p` for the value.
- **Writing through a pointer mutates the original.** `*p = 21` changes `i`.
- Contrast with plain assignment: `p := i` copies the value. Changing `p` afterwards leaves `i` untouched. With a pointer, both names refer to the same storage.

Pointers show up constantly — method receivers, avoiding large copies, letting a function modify its caller's data. More in [11 — Methods](11-methods.md).

---

## Structs

[▶ 2:10:00](https://www.youtube.com/watch?v=tgGNwG_UxFo&t=7800s)

A struct is an **aggregate type**: a collection of **heterogeneous** fields. (An array is the other aggregate type — homogeneous.)

You will use structs constantly. Day-to-day Go backend work is largely structs.

### Declaring

```go
type Vertex struct {
	X int
	Y int
}
```

`type <Name> struct { ... }` — field name, then field type.

Consecutive same-typed fields can be merged:

```go
type Vertex struct {
	X, Y int
}
```

### Initializing — four forms

```go
v1 := Vertex{1, 2}        // positional — X=1, Y=2, order matters
v2 := Vertex{X: 1}        // named fields — Y gets its zero value, 0
v3 := Vertex{}            // everything at zero value — X=0, Y=0
p  := &Vertex{1, 2}       // pointer to a struct — type is *Vertex
```

Printed: `{1 2}`, `&{1 2}`, `{1 0}`, `{0 0}`.

Named-field form is generally the readable choice — it survives field reordering and doesn't require memorizing positions.

### Accessing and mutating fields

```go
v := Vertex{1, 2}
fmt.Println(v.X)   // 1
v.X = 4
fmt.Println(v.X)   // 4
```

### Structs + pointers

This combination is used heavily in real code:

```go
v := Vertex{1, 2}
p := &v
p.X = 1e9            // mutates v itself
fmt.Println(v)       // {1000000000 2}
```

Note you write `p.X`, not `(*p).X`. **Go automatically dereferences pointers to structs for field access** — a convenience that keeps the syntax clean. The same automatic behavior applies to method calls, see [11 — Methods](11-methods.md).

---

## Struct tags

[▶ ~2:16:00](https://www.youtube.com/watch?v=tgGNwG_UxFo&t=8160s)

Real-world structs carry backtick-quoted metadata after the field type:

```go
type BillingSubscription struct {
	PlanID string `json:"planID"`
	Status int    `json:"status"`
}
```

### Why they exist

Unlike JavaScript, **Go cannot use JSON directly**. Incoming JSON arrives as bytes; you must *unmarshal* it into a Go struct before you can work with it.

But naming conventions clash — JSON payloads use `planID` or `plan_id`, Go fields must be `PascalCase` (to be exported, so the JSON package can see them). Struct tags are the **mapping layer**:

```
{"planID": 3}   ──unmarshal──►   BillingSubscription{PlanID: 3}
                  guided by the `json:"planID"` tag
```

Different tag keys serve different libraries — `json:`, `db:`, `validate:`, `yaml:` — so one struct can describe how it maps into several systems at once. Fider's structs carry several depending on which package consumes them.

---

## Exported struct fields

Field capitalization follows the same visibility rule as everything else ([03 — Packages & visibility](03-packages-and-visibility.md)):

```go
type User struct {
	ID    int      // exported — usable from other packages
	Name  string   // exported
	email string   // unexported — invisible outside this package
}
```

This is **not styling**. The video demonstrates the consequence by lowercasing one field of a fider struct that another package imports — errors immediately appear everywhere that field was accessed.

It also matters for JSON: `encoding/json` can only marshal/unmarshal **exported** fields. A lowercase field silently won't appear in your JSON output.

### Real-world shapes (fider)

```go
type User struct {
	ID        int             `json:"id"`
	Name      string          `json:"name"`
	Tenant    *Tenant         `json:"-"`
	Email     string          `json:"-"`
	Role      Role            `json:"role"`
	Providers []*UserProvider `json:"-"`
}
```

Structs hold anything — ints, strings, bools, slices, maps, pointers, other structs. That versatility is why they're everywhere.

---

**Next:** [08 — Arrays & slices](08-arrays-and-slices.md)
