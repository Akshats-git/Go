# 10 — Functions as Values: Anonymous Functions & Closures

In Go, **functions are first-class citizens** — they're values like any other. They can be assigned to variables, passed as arguments, and returned from other functions.

---

## Anonymous functions

[▶ 3:34:10](https://www.youtube.com/watch?v=tgGNwG_UxFo&t=12850s)

A function with no name, assigned to a variable:

```go
hypot := func(x, y float64) float64 {
	return math.Sqrt(x*x + y*y)
}

fmt.Println(hypot(5, 12))   // 13
```

Note the shape: no name after `func`, and the whole thing is a value being assigned with `:=`. After that, the variable *is* the function — call it with `hypot(...)`.

---

## Functions as parameters (higher-order functions)

```go
func compute(fn func(float64, float64) float64) float64 {
	return fn(3, 4)
}
```

The parameter type is a **function signature**: `func(float64, float64) float64` — two float64 in, one float64 out. Parameter *names* are omitted in a signature; only types matter.

Anything matching that signature can be passed in:

```go
fmt.Println(compute(hypot))     // 5    — our anonymous function
fmt.Println(compute(math.Pow))  // 81   — a standard library function
```

`math.Pow` has signature `func(x, y float64) float64` — it matches, so it's accepted.

This is the basis of a lot of composition in Go: middleware chains, handler wrapping, callbacks, custom sort comparators.

### Inline anonymous functions

You can also define and use one on the spot without naming it at all — common when spawning goroutines:

```go
go func() {
	// runs concurrently
}()
```

Note the trailing `()` — you're defining *and calling* it. See [14 — Concurrency](14-concurrency.md).

---

## Closures

[▶ 3:37:34](https://www.youtube.com/watch?v=tgGNwG_UxFo&t=13054s)

> **Beginner note from the video:** if you know JavaScript closures, Go's work identically. If closures are new, don't take them too seriously right now. You won't often think "I'll use a closure here" — but they're used implicitly all over libraries, so it's worth knowing the concept exists.

### Definition

A **closure** is a function that references variables from **outside its own body**, and keeps access to them even after the enclosing function has returned. The inner function "closes over" the outer function's variables, giving it **persistent state across calls**.

### The canonical example

```go
func adder() func(int) int {
	sum := 0
	return func(x int) int {
		sum += x
		return sum
	}
}
```

`adder` returns a function. That returned function has access to `sum` — and `sum` survives after `adder` returns.

```go
func main() {
	pos, neg := adder(), adder()
	for i := 0; i < 10; i++ {
		fmt.Println(
			pos(i),
			neg(-2*i),
		)
	}
}
```

### Tracing it

| i | `pos(i)` | running `sum` (pos) | `neg(-2i)` | running `sum` (neg) |
|---|---|---|---|---|
| 0 | 0 | 0 | 0 | 0 |
| 1 | 1 | 1 | −2 | −2 |
| 2 | **3** | 3 | −6 | −6 |
| 3 | 6 | 6 | −12 | −12 |
| 4 | 10 | 10 | −20 | −20 |

The surprising bit at `i = 2`: `sum` **doesn't restart at 0**. It remembers `1` from the previous call, so `1 + 2 = 3`. That memory *is* the closure.

### The other key point: separate instances

`pos` and `neg` come from **two separate calls to `adder()`**, so each has **its own independent `sum`**. They don't interfere. Every call to the outer function produces a fresh closed-over environment.

---

**Next:** [11 — Methods](11-methods.md)
