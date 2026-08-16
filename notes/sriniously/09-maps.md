# 09 — Maps

[▶ 3:19:30](https://www.youtube.com/watch?v=tgGNwG_UxFo&t=11970s)

Go's hash table. Called dictionaries/hashes/hashmaps elsewhere; in Go it's a `map`.

---

## Shape

```go
map[KeyType]ValueType
```

- **Keys must be comparable** — they must support `==`. Strings, numbers, bools, pointers, structs of comparable fields. Not slices, maps, or functions.
- **Values can be anything.**
- Keys are most often strings, but don't have to be.

---

## Why maps: lookup speed

> "We use maps when we want to find things quickly."

Maps give O(1) average lookup, insert, and delete. That's the whole reason they exist. If you're storing a record, use a struct. If you're going to search for things by key, use a map.

---

## Creating a map

### With `make` (a map is a reference type)

```go
type Vertex struct {
	Lat, Long float64
}

var m map[string]Vertex        // declared — value is nil

func main() {
	m = make(map[string]Vertex)
	m["Bell Labs"] = Vertex{40.68433, -74.39967}
	fmt.Println(m["Bell Labs"])   // {40.68433 -74.39967}
}
```

### With a map literal

```go
var m = map[string]Vertex{
	"Bell Labs": {40.68433, -74.39967},
	"Google":    {37.42202, -122.08408},
}
```

Since the value type is known, the inner `Vertex` can be omitted from each entry.

---

## ⚠️ The `var` trap — writing to a nil map panics

```go
var m map[string]int
m["one"] = 9        // 💥 panic: assignment to entry in nil map
```

`var` gives you the zero value of a map, which is `nil`. **A nil map cannot be written to.**

> **There are exactly two ways to create a usable map: `make`, or a map literal.** Never `var`, unless you know precisely what you're doing.

```go
m := make(map[string]int)        // ✅
m := map[string]int{}            // ✅
```

*(Reading from a nil map is actually safe — it returns the zero value. It's only writes that panic. But the rule "always use `make` or a literal" is the one to internalize.)*

---

## Mutating a map

```go
m := make(map[string]int)

m["Answer"] = 42               // insert
fmt.Println(m["Answer"])       // 42

m["Answer"] = 48               // update
fmt.Println(m["Answer"])       // 48

delete(m, "Answer")            // delete — built-in function
fmt.Println(m["Answer"])       // 0  ← ???
```

---

## The comma-ok idiom

That last `0` is the problem: **is the value actually 0, or is the key absent?** A missing key returns the value type's zero value, so you can't distinguish them from the value alone.

Go's answer — the two-value form of a map read:

```go
v, ok := m["Answer"]
```

| Return | Meaning |
|---|---|
| `v` | the value if present, otherwise the zero value |
| `ok` | `true` if the key exists, `false` if not |

```go
if v, ok := m["Answer"]; ok {
	fmt.Println("present, value is", v)
} else {
	fmt.Println("key not set")
}
```

**Convention: name the second variable `ok`.** Other engineers immediately recognize you're testing existence. (Combines nicely with `if`'s init statement — see [06 — Control flow](06-control-flow.md).)

---

## Iterating a map

Same `range` construct as slices, but it yields **key, value** instead of index, value:

```go
for key, value := range m {
	fmt.Println(key, value)
}

for _, value := range m { ... }   // keys discarded
for key := range m { ... }        // values omitted
```

### ⚠️ Iteration order is deliberately randomized

Go **intentionally randomizes** map iteration order so you can never accidentally depend on it. Two runs of the same program give different orders.

**If you need a stable order, extract the keys into a slice and sort them yourself.**

---

## Maps vs structs — when to use which

[▶ 3:30:40](https://www.youtube.com/watch?v=tgGNwG_UxFo&t=12640s)

The one-line mental model:

> **Structs are for storing data. Maps are for looking data up.**

Oversimplified but a good default. Roughly 70% of the time in backend code, you want a struct.

### The concrete differences

| | Struct | Map |
|---|---|---|
| **Fields known at compile time?** | Yes — fixed, static | No — dynamic, arbitrary keys |
| **Can you `range` over it?** | ❌ No — the fields are already written down in your code, there's nothing to discover | ✅ Yes |
| **Do entries have stable memory addresses?** | ✅ Yes — you can take `&s.Field` | ❌ **No** — you cannot take the address of a map entry |
| **Iteration order** | n/a | Randomized on purpose |

**Why map entries have no fixed address:** maps use hashes and buckets internally, and rehash as they grow. Entries physically move. So `&m["key"]` is illegal — the address wouldn't stay valid. Struct fields sit at fixed offsets and are addressable.

---

## Real-world check (fider / incubator-answer)

[▶ 3:33:10](https://www.youtube.com/watch?v=tgGNwG_UxFo&t=12790s)

The recurring theme: **maps are for data whose shape you don't know at compile time.**

- **HTTP headers** — `map[string]string`. You can't enumerate every header a client might send, so a struct is impossible.
- **Arbitrary JSON** — unmarshal into `map[string]interface{}` when the payload's keys aren't known ahead of time. (If you *do* know them, use a struct with tags — see [07 — Pointers & structs](07-pointers-and-structs.md).)

The decision usually resolves itself: **known, fixed fields → struct. Unknown or arbitrary keys, or you need fast lookup → map.**

---

**Next:** [10 — Functions as values](10-functions-as-values.md)
