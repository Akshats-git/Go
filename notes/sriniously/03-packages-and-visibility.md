# 03 — Packages & Visibility

[▶ 51:48](https://www.youtube.com/watch?v=tgGNwG_UxFo&t=3108s)

---

## Why packages exist

Write everything in one file and it works — for a small script. At thousands, then millions of lines, it becomes impossible to read and impossible to debug. Every language has a mechanism for this (JavaScript has CommonJS/ESM imports and exports). Go has exactly two levels:

1. **Modules** — one per project, created once with `go mod init`, then never thought about again.
2. **Packages** — the actual code-organization unit.

That's the entire code-management story in Go. Nothing else.

---

## The rules

- **Every `.go` file** begins with a package declaration. The filename itself is irrelevant.
- **One package per directory.** All files in a folder declare the same package.
- **By convention the package name matches the folder name.**
- Test files conventionally use a `_test` suffix on the package name.
- An **executable** must have `package main` containing `func main()`.

```go
package handlers   // file lives in the handlers/ directory
```

---

## Import paths: only the last segment matters

```go
import "math/rand"

func main() {
	fmt.Println(rand.Intn(10))   // referred to as "rand", not "math/rand"
}
```

The prefix is just the path to find it. **The last segment is the identifier you use in code.**

```go
import (
	"crypto"
	"crypto/rsa"
	"crypto/sha1"
	"fmt"
	"net/http"
)
```

Multiple imports go in a parenthesized block, one per line.

---

## Package naming conventions

Two rules, both driven by the fact that you type the package name at every use site:

1. **All lowercase.** `main`, `fmt`, `rand`, `handlers`, `actions`. Never capitalized.
2. **As short as possible.** Not a hard rule, a strong convention. You'll write this name hundreds of times — a ten-letter package name is a tax on every line.

### Real-world check (fider)

| Folder | Package | Notes |
|---|---|---|
| `actions/` | `actions` | matches folder, short, lowercase |
| `handlers/` | `handlers` | every file in the folder declares it |
| `webhooks/` | `webhooks` | same pattern |

Test files in `actions/` use `actions_test`.

---

## Visibility: capitalization is the access modifier

[▶ 57:20 area](https://www.youtube.com/watch?v=tgGNwG_UxFo&t=3440s)

This is Go's encapsulation mechanism, and it applies to **everything** — variables, constants, functions, types, struct fields, methods, interfaces.

| First letter | Visibility |
|---|---|
| **Capital** | **Exported** — accessible from other packages |
| lowercase | **Unexported** — visible only inside its own package |

```go
import "math"

func main() {
	fmt.Println(math.pi)  // ❌ undefined: math.pi
	fmt.Println(math.Pi)  // ✅
}
```

The error surfaces at **compile time** — and in an editor with Go LSP support (VS Code + Go extension, GoLand, Neovim + gopls), you see it inline before you ever run the program. That instant feedback is a real argument for using an editor with first-class Go support.

### Why it matters in practice

If a struct field is lowercase, code in other packages cannot read or write it. The video demonstrates this by lowercasing one exported field of a struct in fider — errors immediately light up across every file in other packages that touched it.

If you come from OOP, map this onto `public` / `private`. But don't try to think in objects when writing Go — it's the same *idea* (hide implementation, expose an interface), not the same model.

---

**Next:** [04 — Functions](04-functions.md)
