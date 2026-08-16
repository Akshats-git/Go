# 08 — Arrays & Slices

The longest section of the video (~55 minutes) and the most important data structure in day-to-day Go.

---

## Arrays

[▶ 2:22:34](https://www.youtube.com/watch?v=tgGNwG_UxFo&t=8554s)

An array is a collection of **homogeneous** data — every element the same type.

```go
var a [2]string
a[0] = "Hello"
a[1] = "World"
fmt.Println(a[0], a[1])   // Hello World
fmt.Println(a)            // [Hello World]

primes := [6]int{2, 3, 5, 7, 11, 13}
```

### The one thing to remember: **size is part of the type**

`[2]string` and `[3]string` are **different types**. Combined with static typing, this means:

> **Arrays cannot be resized.** You can't grow one, shrink one, or append to one. Whatever length you declare is the length forever.

That's why arrays are comparatively rare in real Go code. You'll use one when you know a fixed count up front. The rest of the time you use a **slice**.

---

## Slices

[▶ 2:25:34](https://www.youtube.com/watch?v=tgGNwG_UxFo&t=9334s)

A slice is a dynamically-sized, flexible view into an array. It's a **reference type** — it doesn't hold data, it points at a **backing array**.

### Creating a slice from an array

```go
primes := [6]int{2, 3, 5, 7, 11, 13}
s := primes[1:4]
fmt.Println(s)   // [3 5 7]
```

`[low:high]` — **low is inclusive, high is exclusive**. `[1:4]` gives indices 1, 2, 3.

---

## Slice internals — the three fields

This is the model that makes everything else make sense. A slice is a small struct with three fields:

```
┌──────────────────────────────────┐
│ SLICE                            │
│   ptr  → first element it covers │
│   len  → number of elements      │
│   cap  → from ptr to the END of  │
│          the backing array       │
└──────────────────────────────────┘
             │
             ▼
   backing array: [ 2 | 3 | 5 | 7 | 11 | 13 ]
                    0   1   2   3    4    5
```

| Field | Definition |
|---|---|
| **pointer** | address of the first element the slice covers |
| **length** | how many elements the slice currently exposes |
| **capacity** | count from the slice's first element **to the end of the backing array** — *not* to the end of the slice |

That capacity definition is the one people get wrong. Capacity is measured against the **backing array**, not the slice.

---

## Slices share the backing array — the aliasing gotcha

```go
names := [4]string{"John", "Paul", "George", "Ringo"}

a := names[0:2]   // len 2, cap 4, points at "John"
b := names[1:3]   // len 2, cap 3, points at "Paul"

fmt.Println(a, b)   // [John Paul] [Paul George]

b[0] = "XXX"        // mutate through b

fmt.Println(a, b)   // [John XXX] [XXX George]
fmt.Println(names)  // [John XXX George Ringo]
```

**One write through `b` changed `a` and `names` too.** All three view the same underlying storage.

This is the practical meaning of "reference type": modify a slice element and every slice sharing that backing array sees the change. It's the single biggest source of surprise bugs with slices.

---

## Slice literals — creating without an array

You don't need an array first. Declare a slice directly and Go creates the backing array for you:

```go
q := []int{2, 3, 5, 7, 11, 13}
r := []bool{true, false, true, true, false, true}

s := []struct {
	i int
	b bool
}{
	{2, true},
	{3, false},
	{5, true},
}
```

**How to tell an array from a slice at a glance:** an array type has a number in the brackets (`[6]int`); a slice has empty brackets (`[]int`).

For a slice literal, `len == cap == number of elements`.

---

## Slice bounds are optional

```go
s := []int{2, 3, 5, 7, 11, 13}

s = s[1:4]   // [3 5 7]
s = s[:2]    // low defaults to 0
s = s[1:]    // high defaults to len(s)
s = s[:]     // both — the whole thing
```

You can also slice **an existing slice**, not just an array.

---

## `len` and `cap` in action

```go
func printSlice(s []int) {
	fmt.Printf("len=%d cap=%d %v\n", len(s), cap(s), s)
}

s := []int{2, 3, 5, 7, 11, 13}
printSlice(s)     // len=6 cap=6 [2 3 5 7 11 13]

s = s[:0]         // slice it down to zero length
printSlice(s)     // len=0 cap=6 []

s = s[:4]         // extend it back out — capacity allows it
printSlice(s)     // len=4 cap=6 [2 3 5 7]

s = s[2:]         // drop the first two
printSlice(s)     // len=2 cap=4 [5 7]
```

### Reading that trace

- `s[:0]` — length 0, but **capacity stays 6**. The backing array is untouched; you just narrowed the window. This is why you can extend it again.
- `s[:4]` — re-widens to length 4. Legal because cap ≥ 4.
- `s[2:]` — you moved the **start pointer** forward by 2, so capacity drops from 6 to 4.

### The capacity rule

> **Capacity only shrinks when you re-slice from an index greater than 0.** Slicing off the *end* (`s[:n]`) doesn't touch capacity, because capacity is measured from the start pointer to the end of the backing array.

---

## Zero value of a slice is `nil`

```go
var s []int
printSlice(s)             // len=0 cap=0 []
if s == nil {
	fmt.Println("nil!")   // prints
}
```

A `var`-declared slice is `nil` with len 0 and cap 0. It prints as `[]` but it is genuinely `nil`, and comparing to `nil` is how you test it.

Unlike a nil map, **a nil slice is safe to `append` to** — `append` allocates for you.

---

## Creating slices with `make`

`make` is a built-in function for initializing **reference types** — slices, maps, and channels.

```go
make([]int, 5)      // type, length          → cap defaults to len = 5
make([]int, 0, 5)   // type, length, capacity → len 0, cap 5
```

| Argument | Meaning |
|---|---|
| 1st | the type (`[]int` — must be explicit since `make` also builds maps/channels) |
| 2nd | length |
| 3rd (optional) | capacity — defaults to length |

**Capacity can never be less than length.** That's an error.

### Worked example

```go
a := make([]int, 5)      // len=5 cap=5 [0 0 0 0 0]
b := make([]int, 0, 5)   // len=0 cap=5 []
c := b[:2]               // len=2 cap=5 [0 0]
d := c[2:5]              // len=3 cap=3 [0 0 0]
```

- `a` — length 5 means five slots, each holding the zero value of `int`.
- `b` — length 0 so it prints empty, but has room for 5.
- `c` — sliced from index 0, so cap stays 5.
- `d` — sliced starting at index 2, so cap drops from 5 to 3. You can slice up to `cap`, not just `len`.

---

## Two-dimensional slices

```go
board := [][]string{
	{"_", "_", "_"},
	{"_", "_", "_"},
	{"_", "_", "_"},
}

board[0][0] = "X"
board[2][2] = "O"
```

A slice whose element type is another slice. Works exactly as intuition suggests.

---

## `append`

[▶ 2:59:15](https://www.youtube.com/watch?v=tgGNwG_UxFo&t=10755s)

```go
var s []int
s = append(s, 0)          // len=1 cap=1 [0]
s = append(s, 1)          // len=2 cap=2 [0 1]
s = append(s, 2, 3, 4)    // len=5 cap=6 [0 1 2 3 4]
```

- **`append` is variadic** — it takes the slice, then any number of elements.
- **You must reassign the result** (`s = append(s, x)`). `append` may allocate a new backing array, so it returns a (possibly different) slice.

### What `append` does under the hood

1. Checks whether the backing array has spare capacity (`len < cap`).
2. **If yes** — writes into the existing array, bumps length. Cheap.
3. **If no** — allocates a **new, larger backing array**, copies everything over, points the returned slice at it. Expensive.

### The growth algorithm

Trace it with a loop:

```go
var s []int
for i := 0; i < 20; i++ {
	s = append(s, i)
	printSlice(s)
}
```

```
len=1  cap=1
len=2  cap=2
len=3  cap=4     ← doubled
len=4  cap=4
len=5  cap=8     ← doubled
len=6  cap=8
len=7  cap=8
len=8  cap=8
len=9  cap=16    ← doubled
...
```

**The pattern: every time `len == cap`, the next append doubles the capacity** and reallocates.

The doubling holds up to a threshold of **256 elements**; past that Go grows by roughly **1.25×** instead of 2× (there's a `nextslicecap` function in the runtime source implementing this). Internal detail — the point is that growth is amortized, not linear.

---

## The one optimization you should actually apply

Every reallocation means: find new memory, allocate a bigger array, copy every element. If you know the final size in advance, you can skip all of it.

```go
// ❌ 3 reallocations to reach 5 elements: cap 0→1→2→4→8
var s []int
for i := 0; i < 5; i++ {
	s = append(s, i)
}
```

```go
// ✅ zero reallocations: capacity is right from the start
s := make([]int, 0, 5)
for i := 0; i < 5; i++ {
	s = append(s, i)
}
// cap stays 5 through every iteration
```

**Note `make([]int, 0, 5)` — length 0, capacity 5.** Not `make([]int, 5, 5)`, which would pre-fill five zeros that you'd then append *after*.

> **Rule of thumb:** whenever you know the maximum number of elements ahead of time — a database result set, a fixed transformation of an existing slice — preallocate with `make([]T, 0, n)`.

This is the entire practical reason for understanding capacity. You will otherwise never think about it: `append` handles everything.

---

## `range` — looping over slices

[▶ 3:17:33](https://www.youtube.com/watch?v=tgGNwG_UxFo&t=11853s)

The traditional way works:

```go
for i := 0; i < len(pow); i++ {
	fmt.Printf("%d = %d\n", i, pow[i])
}
```

But `range` is the idiom:

```go
for i, v := range pow {
	fmt.Printf("2**%d = %d\n", i, v)
}
```

Each iteration yields **two values: the index, and a *copy* of the element.**

> ⚠️ **`v` is a copy.** Assigning to `v` inside the loop does **not** modify the slice. To mutate, index directly: `pow[i] = ...`.

### Skipping either value

```go
for _, v := range pow { ... }   // index discarded
for i := range pow { ... }      // value omitted entirely
```

`_` is the blank identifier — "I know there's a value here, I don't want it."

---

## Real-world check (fider)

[▶ 3:18:40](https://www.youtube.com/watch?v=tgGNwG_UxFo&t=11920s)

All three patterns appear constantly:

```go
tags := []string{"bug", "feature"}          // slice literal
titles := make([]string, 31)                 // make with known size
for _, title := range titles { ... }         // range, index discarded
```

Slices are among the most-used types in any Go codebase.

---

**Next:** [09 — Maps](09-maps.md)
