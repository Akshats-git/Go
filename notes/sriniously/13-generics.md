# 13 — Generics

[▶ 4:46:40](https://www.youtube.com/watch?v=tgGNwG_UxFo&t=17200s)

A relatively new addition — generics arrived in **Go 1.18**, not in the original language.

---

## The problem generics solve

Without generics, you write the same function once per type:

```go
func IndexInt(s []int, x int) int {
	for i, v := range s {
		if v == x {
			return i
		}
	}
	return -1
}

func IndexString(s []string, x string) int {
	for i, v := range s {
		if v == x {
			return i
		}
	}
	return -1
}
```

**The bodies are identical.** The only difference is `int` vs `string`. Static typing forces you to duplicate the implementation — pure boilerplate.

---

## Type parameters

Generics let you pass **types as parameters**, alongside the values you already pass.

```go
func Index[T comparable](s []T, x T) int {
	for i, v := range s {
		if v == x {
			return i
		}
	}
	return -1
}
```

Three changes from the non-generic version:

```
func Index[T comparable](s []T, x T) int
          └─────┬──────┘ └──────┬──────┘
       type parameter list   regular parameters,
       (square brackets)     now using T
```

1. **`[T comparable]`** — the type parameter list, in **square brackets**, before the regular parameters. Declares a type parameter named `T`.
2. **`comparable`** — the **constraint** on `T`. It isn't "any type" — it must be a type supporting `==` and `!=`. Required here because the body does `v == x`.
3. **`s []T, x T`** — the regular parameters now use `T`. `T` gets bound to a concrete type at compile time.

The return type can use `T` too.

---

## Calling it — type inference

```go
si := []int{10, 20, 15, 3}
fmt.Println(Index(si, 15))        // 2

ss := []string{"foo", "bar", "baz"}
fmt.Println(Index(ss, "hello"))   // -1
```

**You don't write the type argument.** The compiler infers `T` from the arguments: `ss` is `[]string` and `"hello"` is a `string`, therefore `T = string`.

Explicit instantiation is available when inference can't figure it out:

```go
Index[string](ss, "hello")
```

**One function now handles both types**, with one implementation to maintain.

---

## Constraints

The constraint is what makes a type parameter useful rather than useless. A constraint is expressed as an interface, and it tells the compiler what operations are legal on `T`.

- **`comparable`** — supports `==` / `!=`. Needed for equality checks and for map keys.
- **`any`** — no constraint at all; you can pass anything around but do almost nothing with it.
- Custom constraints let you list permitted types (e.g. a numeric constraint allowing all int and float kinds so you can use `+` and `<`).

Without a constraint the compiler has no idea whether `v == x` is legal for whatever `T` turns out to be — hence the requirement.

---

## Practical perspective

> "You might not use generics as extensively when you're starting out, but you will definitely see different implementations and use cases of them."

Generics are worth **recognizing** more than reaching for. Most application-level backend code doesn't need them; you'll encounter them in libraries and utility packages (`slices`, `maps`, and similar standard-library packages are built on them).

**What to retain:**
- Type parameters go in square brackets before the value parameters
- They come with constraints
- They can appear in parameter types and return types
- The point is eliminating duplicate implementations that differ only by type
- Use them when two functions have the **same body** and differ only in types

Generics also support **generic types** (e.g. a `List[T]` linked list), which the Tour covers briefly — same idea applied to type declarations rather than functions.

---

**Next:** [14 — Concurrency](14-concurrency.md)
