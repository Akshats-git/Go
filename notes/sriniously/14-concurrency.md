# 14 — Concurrency

[▶ 4:53:57](https://www.youtube.com/watch?v=tgGNwG_UxFo&t=17637s)

Go's headline feature, and ~58 minutes of the video.

> **Read this first:** the video is explicit that beginners should *not* try to master this section. Get some of it, get none of it — either is fine. It comes with reading more code, more resources, and building projects. Don't let concurrency become the thing that makes you quit Go.

---

## Goroutines

```go
func say(s string) {
	for i := 0; i < 5; i++ {
		time.Sleep(100 * time.Millisecond)
		fmt.Println(s)
	}
}

func main() {
	go say("world")   // new goroutine
	say("hello")      // main goroutine
}
```

Output interleaves unpredictably: `world hello world hello world hello ...`

**`go <function call>` spawns a goroutine** that runs concurrently with the caller.

### What a goroutine is

Roughly analogous to a thread, but importantly different:

| | OS thread | Goroutine |
|---|---|---|
| **Managed by** | the operating system | **the Go scheduler** (part of the runtime) |
| **Stack size** | fixed, ~256 KB – 1 MB | **starts at ~2–4 KB, grows dynamically** |
| **Cost of spawning many** | high — you run out of memory fast | low — hundreds of thousands are routine |

Two takeaways: goroutines are **scheduler-managed, not OS-managed**, and they are **very lightweight**.

### The main goroutine

- **Every Go program starts with the main goroutine** — `func main()` runs in it.
- All other goroutines are spawned from it (and can spawn more themselves).
- **When `main` returns, the program exits and every other goroutine is abandoned.** They're garbage collected; whatever they were doing never finishes and you never see the result.

```
main goroutine ──────────────────────────► return → program ends
       │                                          (all children killed)
       ├──► goroutine A ──►
       └──► goroutine B ──►
```

**So the main goroutine must wait for the goroutines it spawned.** This need for waiting is the reason channels matter.

### Ordering is not guaranteed

The moment you spawn a goroutine, **there is no way to know** whether it starts immediately or whether the next line in the caller runs first. Never write code that depends on it.

---

## Concurrency vs parallelism

A high-level distinction, since the debate is endless:

- **Parallelism ≈ a hardware property.** Tasks genuinely executing at the same instant, on multiple CPU cores.
- **Concurrency ≈ a software property.** Structuring work so tasks are executed in the most efficient *order*, without blocking the system — the order being unspecified.

Four concurrent tasks may run in any order, one at a time, interleaved. If the hardware supports parallelism, some may genuinely overlap. But **concurrency does not mean parallelism.**

> Worth watching: Rob Pike's talks on this (he co-created Go).

---

## The philosophy: CSP

Go follows **CSP — Communicating Sequential Processes**:

> **"Don't communicate by sharing memory; share memory by communicating."**

Most languages implement concurrency via **shared state** protected by locks. Go's position is that shared state is historically where deadlocks, starvation, and race conditions come from, and that using **communication between goroutines** as the primary mechanism avoids a large class of these bugs.

Go still provides mutexes and the classic primitives (see below). But **channels are the recommended first approach** — reach for communication before you reach for locks.

---

## Channels

A **channel is a typed conduit** — a data type that lets two goroutines send values to each other.

```
goroutine A  ──┤  channel  ├──►  goroutine B
```

A channel is a container type — you must say what flows through it:

```go
ch := make(chan int)      // channel of ints
```

Channels are a **reference type**, so they're created with `make` (see [05 — Variables & types](05-variables-and-types.md)).

### Send and receive

```go
ch <- v      // send v INTO the channel
v := <-ch    // receive FROM the channel
```

The arrow points in the direction of data flow. Channel on the left = send; channel on the right = receive.

### Worked example

```go
func sum(s []int, c chan int) {
	sum := 0
	for _, v := range s {
		sum += v
	}
	c <- sum                    // send the result back
}

func main() {
	s := []int{7, 2, 8, -9, 4, 0}

	c := make(chan int)
	go sum(s[:len(s)/2], c)     // first half in one goroutine
	go sum(s[len(s)/2:], c)     // second half in another

	x, y := <-c, <-c            // receive twice

	fmt.Println(x, y, x+y)
}
```

What this does:

1. **Divides the work** — two goroutines, each summing half the slice. Legal because the order of the halves doesn't matter.
2. **Passes the channel to both** so they can report back to main.
3. **Receives twice**, collecting both results.

### The critical property

> **Receiving from a channel blocks.**

`<-c` waits until a value is available. That's what makes this program correct — main can't finish early, because it's parked on those two receives until both goroutines have sent. The channel provides **both communication and synchronization**.

Sending on an unbuffered channel blocks too, until a receiver takes the value.

---

## Buffered vs unbuffered channels

```go
ch := make(chan int)      // unbuffered
ch := make(chan int, 2)   // buffered, capacity 2
```

### Unbuffered — a synchronization point

```
producer ──┤ (no storage) ├── consumer
```

No queue. A send blocks until a receiver is ready, and vice versa. The two goroutines must **meet**.

**Use when:** synchronization is the point — you want the blocking, you want a handoff, you want to know the other side got it.

### Buffered — a queue

```
producer ──► [ _ | _ | _ | _ ] ──► consumer
```

The producer can push values until the buffer is full without blocking, and the consumer drains at its own pace.

```go
ch := make(chan int, 2)
ch <- 1
ch <- 2
fmt.Println(<-ch)   // 1
fmt.Println(<-ch)   // 2
```

**Use when:** you have a producer/consumer relationship whose paces differ, or you know the maximum number of values in advance.

**Neither is "better."** Buffered gives flexibility; unbuffered gives synchronization. Pick based on which you need.

---

## `close` and ranging over a channel

If one goroutine is sending and another receiving, **how does the receiver know when to stop waiting?** That's what `close` is for.

```go
func fibonacci(n int, c chan int) {
	x, y := 0, 1
	for i := 0; i < n; i++ {
		c <- x
		x, y = y, x+y
	}
	close(c)                      // broadcast: no more values coming
}

func main() {
	c := make(chan int, 10)
	go fibonacci(cap(c), c)
	for i := range c {            // receives until the channel is closed
		fmt.Println(i)
	}
}
```

Output: `0 1 1 2 3 5 8 13 21 34`

### `close(ch)`

A built-in that **broadcasts to every receiver**: sending is finished, stop blocking. Called by the **sender**, never the receiver.

### `for i := range ch`

Ranging over a channel does two things:

1. **Receives values sequentially** as the other goroutine sends them — no manual loop, no counting.
2. **Exits automatically when the channel is closed.**

Without it you'd write a `for` loop with a hardcoded count and receive N times manually. `range` removes that boilerplate and, more importantly, removes the need to know N.

---

## `select` — waiting on multiple channels

`select` lets a goroutine wait on several channel operations at once.

```go
func fibonacci(c, quit chan int) {
	x, y := 0, 1
	for {
		select {
		case c <- x:              // if we can SEND on c
			x, y = y, x+y
		case <-quit:              // if we can RECEIVE on quit
			fmt.Println("quit")
			return
		}
	}
}

func main() {
	c := make(chan int)
	quit := make(chan int)

	go func() {                   // anonymous function, see file 10
		for i := 0; i < 10; i++ {
			fmt.Println(<-c)
		}
		quit <- 0                 // signal: we're done
	}()

	fibonacci(c, quit)
}
```

### How it reads

- **`for { ... }`** — an infinite loop (see [06 — Control flow](06-control-flow.md)). This is the pattern the earlier chapter promised: sit in a loop, listen for signals, break out on the right one.
- **`select`** — checks all its cases; whichever channel operation can proceed, that case runs.
- Each case is a **channel operation**, not a boolean.
- `return` inside a case exits the function, which ends the loop.

Trace: the spawned goroutine receives 10 values, so for 10 iterations `c <- x` succeeds and the Fibonacci sequence advances. Then the goroutine sends on `quit`; that case fires, prints "quit", and returns. `main` ends.

Output: `0 1 1 2 3 5 8 13 21 34 quit`

---

## `select` with `default`

A `default` case runs when **no other case is ready** — it makes the select non-blocking.

```go
tick := time.Tick(100 * time.Millisecond)   // fires every 100ms
boom := time.After(500 * time.Millisecond)  // fires once, after 500ms

for {
	select {
	case <-tick:
		fmt.Println("tick.")
	case <-boom:
		fmt.Println("BOOM!")
		return
	default:
		fmt.Println("    .")
		time.Sleep(50 * time.Millisecond)
	}
}
```

Output:
```
    .
    .
tick.
    .
    .
tick.
    .
    .
tick.
...
BOOM!
```

### Tracing it

- `time.Tick` returns a channel that receives a value **every 100 ms**.
- `time.After` returns a channel that receives **once, after 500 ms**.
- Each iteration: is `tick` ready? is `boom` ready? If neither, run `default` — print a dot, sleep 50 ms.
- At 50 ms: neither ready → default. At 100 ms: `tick` fires. At 150 ms: default. At 200 ms: `tick` fires. …
- At 500 ms: `boom` fires → print BOOM → `return` → exit the loop → exit `main` → program ends.

Two dots per tick, because default sleeps 50 ms and tick arrives every 100 ms.

---

## Mutexes — when channels aren't the answer

### The problem: concurrent map writes

```go
type SafeCounter struct {
	v map[string]int
}

func (c *SafeCounter) Inc(key string) {
	c.v[key]++
}

func main() {
	c := SafeCounter{v: make(map[string]int)}
	for i := 0; i < 1000; i++ {
		go c.Inc("somekey")
	}
	time.Sleep(time.Second)
	fmt.Println(c.v["somekey"])
}
```

Expected: `1000`. Actual:

```
fatal error: concurrent map writes
```

1000 goroutines write to the same map in an unspecified order. Go's runtime detects it and kills the program. **This is the classic shared-state bug** that CSP is meant to help you avoid.

> Note the `time.Sleep` — it's a stand-in for waiting on the goroutines. In real code you'd use a channel or a **`sync.WaitGroup`** (covered later in the series). Sleeping is fine for a demo, wrong in production.

### The fix: `sync.Mutex`

A mutex is a **gatekeeper**. It has a `Lock()` and an `Unlock()`, and the region between them is the **critical section** — only one goroutine can be inside at a time, no matter how many are trying.

```
             Lock()
                │
       ┌────────▼────────┐
       │ critical section│  ← only ONE goroutine in here at a time
       └────────┬────────┘
                │
            Unlock()
```

```go
type SafeCounter struct {
	mu sync.Mutex
	v  map[string]int
}

func (c *SafeCounter) Inc(key string) {
	c.mu.Lock()
	c.v[key]++
	c.mu.Unlock()
}

func (c *SafeCounter) Value(key string) int {
	c.mu.Lock()
	defer c.mu.Unlock()      // unlock on return, whatever happens
	return c.v[key]
}
```

Now the result is a reliable `1000`.

**Note the `defer` in `Value`** — the pattern from [06 — Control flow](06-control-flow.md). Rather than carefully placing `Unlock()` before every return path, you defer it once and it's guaranteed to run when the function returns, including on panic. This is the idiomatic way to unlock.

**Reads need the lock too**, not just writes — otherwise a read can observe a half-updated state.

---

## What was skipped

The video acknowledges omissions, since it follows the Tour of Go's scope:

- `sync.WaitGroup` (mentioned, deferred to a later video)
- Directional channel types (`chan<- int` send-only, `<-chan int` receive-only)
- `sync.RWMutex` (read/write locks)
- `context`
- Fan-in / fan-out and other concurrency patterns
- Reflection
- Worker pools

These come later in the series. **You don't need them to write useful Go backends.**

---

## Summary

| Construct | Purpose |
|---|---|
| `go f()` | spawn a goroutine |
| `make(chan T)` | unbuffered channel — synchronization |
| `make(chan T, n)` | buffered channel — queue of capacity n |
| `ch <- v` / `<-ch` | send / receive (receiving blocks) |
| `close(ch)` | broadcast that sending is finished |
| `for v := range ch` | receive until closed |
| `select` | wait on multiple channel operations |
| `default` in select | make the select non-blocking |
| `sync.Mutex` | guard shared state when channels don't fit |

**Goroutines give you concurrency. Channels give you communication and synchronization. Mutexes are the fallback when shared state is genuinely the right model.**

---

**Next:** [15 — Quick reference](15-quick-reference.md)
