# 06 — Control Flow: `for`, `if`, `switch`, `defer`

---

## `for` — the only loop

[▶ 1:37:04](https://www.youtube.com/watch?v=tgGNwG_UxFo&t=5824s)

Go has **no `while`, no `do-while`**. Just `for`. Every other loop shape is `for` with parts removed.

```go
sum := 0
for i := 0; i < 10; i++ {
	sum += i
}
fmt.Println(sum)   // 45
```

Three parts, separated by semicolons, **no parentheses around them**:

1. **Init statement** — `i := 0`, runs once before the loop
2. **Condition** — `i < 10`, checked before each iteration
3. **Post statement** — `i++`, runs after each iteration body

The loop variable `i` is **lexically scoped to the loop** — it doesn't exist outside the braces.

### The cycle

```
init → [ condition → body → post ] → condition → body → post → … → condition false → exit
```

The video walks this with the VS Code debugger — worth doing yourself once with a breakpoint inside the loop. Watching `i` and `sum` change per iteration makes the mechanics concrete in a way reading doesn't.

### While loop — drop init and post

```go
sum := 1
for sum < 1000 {
	sum += sum
}
fmt.Println(sum)   // 1024
```

Keep only the condition. That's Go's `while`.

### Infinite loop — drop everything

```go
for {
	// runs forever
}
```

Without a condition the program never exits this loop. To leave it you need an explicit `break` (or `return`):

```go
sum := 1
for {
	sum += sum
	if sum > 100 {
		break
	}
}
```

**Infinite loops are not a bug — they're a pattern.** They're used heavily in concurrency, where you sit in a loop listening for signals from goroutines and break out when a particular signal arrives. See [14 — Concurrency](14-concurrency.md), specifically `select`.

### `range` — iterating collections

Covered in [08 — Arrays & slices](08-arrays-and-slices.md), [09 — Maps](09-maps.md), and [14 — Concurrency](14-concurrency.md) (ranging over a channel).

---

## `if`

[▶ 1:44:15](https://www.youtube.com/watch?v=tgGNwG_UxFo&t=6255s)

```go
if x < 0 {
	return sqrt(-x) + "i"
}
```

No parentheses around the condition. The condition must evaluate to a bool.

### The init statement — Go's signature `if` feature

Like `for`, `if` can run a short statement before the condition:

```go
if v := math.Pow(x, n); v < lim {
	return v
} else {
	return lim
}

// v is NOT accessible here — undefined: v
```

`v` is scoped to the `if`/`else` blocks only.

**Why this matters:** it's the backbone of idiomatic Go error handling. Instead of declaring a variable, calling, then checking, you do all three in one line:

```go
if err := doSomething(); err != nil {
	return err
}
```

This kills a lot of boilerplate and keeps the error variable from leaking into the enclosing scope.

### `else if` / `else`

Standard behavior — conditions evaluated top to bottom, first true branch wins.

---

## `switch`

[▶ 1:47:47](https://www.youtube.com/watch?v=tgGNwG_UxFo&t=6467s)

### Basic form

```go
switch os := runtime.GOOS; os {
case "darwin":
	fmt.Println("OS X.")
case "linux":
	fmt.Println("Linux.")
default:
	fmt.Printf("%s.\n", os)
}
```

- Like `for` and `if`, it takes an **optional init statement** (`os := runtime.GOOS;`). `os` is scoped to the switch block.
- **No fallthrough by default.** The moment a case matches, its body runs and the switch exits. You do *not* write `break` — that's the big difference from C/Java/JavaScript. (An explicit `fallthrough` keyword exists if you ever want the C behavior.)

### Cases can be expressions

Cases aren't limited to constants. They can be arbitrary expressions evaluated at runtime:

```go
today := time.Now().Weekday()
switch time.Saturday {
case today + 0:
	fmt.Println("Today.")
case today + 1:
	fmt.Println("Tomorrow.")
// ...
}
```

### Switch with no condition — the clean `if/else if` chain

```go
t := time.Now()
switch {
case t.Hour() < 12:
	fmt.Println("Good morning!")
case t.Hour() < 17:
	fmt.Println("Good afternoon.")
default:
	fmt.Println("Good evening.")
}
```

An empty `switch` is equivalent to `switch true`. Cases are checked top to bottom; the first true one runs and exits. This is the idiomatic replacement for a long `if / else if / else if / else` ladder — much easier to read.

### Type switch

A special form used with interfaces. See [12 — Interfaces](12-interfaces.md).

---

## `defer`

[▶ 1:50:00](https://www.youtube.com/watch?v=tgGNwG_UxFo&t=6600s)

A concept fairly unique to Go, and an important one.

```go
func main() {
	defer fmt.Println("world")
	fmt.Println("hello")
}
// hello
// world
```

### Mechanics

When you write `defer <expression>`:

1. The expression's **arguments are evaluated immediately**.
2. The **call itself is pushed onto a stack** and not executed.
3. The rest of the function runs normally.
4. When the function **returns**, Go pops the stack and executes every deferred call — **LIFO order, last deferred runs first**.

### The stack demonstration

```go
func main() {
	fmt.Println("counting")
	for i := 0; i < 10; i++ {
		defer fmt.Println(i)
	}
	fmt.Println("done")
}
```

Output:
```
counting
done
9
8
7
6
5
4
3
2
1
0
```

Ten calls pushed in order 0→9, popped in order 9→0. Classic stack.

---

### Why `defer` exists — the two real reasons

#### 1. Humans forget to clean up

Whenever you *start* something you must *stop* it — open a file, close it; open a database connection, close it. Miss the close and you leak memory, especially inside a loop opening many resources.

The naive shape:

```go
f := os.Open("file.txt")
// ... 200 lines of business logic ...
f.Close()          // easy to forget
```

With `defer` the cleanup sits **right next to the acquisition**:

```go
f := os.Open("file.txt")
defer f.Close()    // guaranteed, and visible at the point of acquisition
// ... 200 lines of business logic ...
```

Readability win: anyone scanning the function sees "open, will close" in two adjacent lines.

#### 2. Panics

A **panic** is Go's rough equivalent of an exception — an error that halts normal execution flow. (Go deliberately doesn't have exceptions; it returns errors as values. `panic` is for the genuinely unrecoverable.)

Without `defer`:

```
open file → run logic → 💥 panic → function exits → Close() never ran → leaked resource
```

With `defer`:

```
open file → defer Close() → run logic → 💥 panic → deferred calls run → Close() executes ✅
```

**Deferred functions run when the function returns *and* when it panics.** That's the guarantee. It doesn't matter whether the function succeeded — the cleanup happens.

---

### Real-world check (fider)

A test setup:

```go
func TestSomething(t *testing.T) {
	db := setupDatabase()
	defer db.TearDown()     // second line, immediately after setup
	// ... test body ...
}
```

Regardless of whether the test passes, fails, or panics, the database connection is torn down. The pattern is: **acquire on one line, `defer` the release on the very next line.**

---

**Next:** [07 — Pointers & structs](07-pointers-and-structs.md)
