# 04 — Functions

[▶ 1:00:25](https://www.youtube.com/watch?v=tgGNwG_UxFo&t=3625s)

---

## Anatomy

```go
func add(x int, y int) int {
	return x + y
}
```

| Part | Rule |
|---|---|
| `func` | the keyword every function starts with |
| `add` | the name |
| `(x int, y int)` | parameters — **name first, type second**, comma separated |
| `int` | the return type — **mandatory** to declare |
| `{ ... }` | opening brace on the **same line** |

The name-then-type ordering is the reverse of C/Java and is consistent everywhere in Go (variables, struct fields, parameters).

### Static typing is enforced here

```go
add("hello", 7)
// cannot use "hello" (untyped string constant) as int value in argument to add
```

Go is statically typed — argument types are verified at compile time, not runtime.

---

## Shortening consecutive same-type parameters

```go
func add(x, y int) int { ... }   // equivalent to (x int, y int)
```

If consecutive parameters share a type, you can drop the type from all but the last.

> **The video's preference:** use the explicit long form `(x int, y int)`. On first read it states outright that x is an int and y is an int — slightly more readable, less room for misreading. Personal preference, not a rule.

---

## Multiple return values

Go functions can return more than one value. This is used *constantly* — it's the foundation of Go's error handling.

```go
func swap(x, y string) (string, string) {
	return y, x
}

func main() {
	a, b := swap("hello", "world")
	fmt.Println(a, b)   // world hello
}
```

- More than one return value ⇒ **parentheses required** around the return types.
- Exactly one return value ⇒ no parentheses (`func f() string`).
- Receive them with a comma-separated list of variables, positionally.

---

## Named return values (and why to avoid them)

Go lets you name the return values in the signature:

```go
func split(sum int) (x, y int) {
	x = sum * 4 / 9
	y = sum - x
	return              // "naked return" — implicitly returns x, y
}
```

What happens:

- `x` and `y` are **declared and zero-initialized** at the top of the function body, before the first statement runs.
- That's why the body **assigns** (`x =`) rather than declares (`x :=`).
- A bare `return` returns whatever `x` and `y` currently hold.

### The recommendation: don't use this

> "I have never used it. I don't want to use it."

The problem is readability at scale. In a long function, you reach a bare `return` at the bottom and have no idea what it returns. You have to scroll to the signature, find the names, then trace those variables through the whole body. It makes things more complicated than they need to be, and it's error-prone.

**Prefer the explicit form:**

```go
func split(sum int) (int, int) {
	x := sum * 4 / 9
	y := sum - x
	return x, y
}
```

Same result, and the return statement tells you exactly what comes back.

**Know that naked returns exist** (you'll read them in other people's code) — just don't write them.

---

## Related

- Functions are **first-class values** in Go — assignable to variables, passable as arguments, returnable. See [10 — Functions as values](10-functions-as-values.md).
- Functions attached to a type are **methods**. See [11 — Methods](11-methods.md).
- A function's *signature* is what satisfies an **interface**. See [12 — Interfaces](12-interfaces.md).
- Variadic functions (`func f(args ...int)`) accept any number of arguments — `append`, `fmt.Println` and friends are variadic. Covered in [08 — Arrays & slices](08-arrays-and-slices.md).

---

**Next:** [05 — Variables & types](05-variables-and-types.md)
