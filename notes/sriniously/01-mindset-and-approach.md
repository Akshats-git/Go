# 01 — Mindset & Approach

[▶ 0:00](https://www.youtube.com/watch?v=tgGNwG_UxFo&t=0s)

The first ~28 minutes are not code. They're the framing for the whole series. Worth reading once, then ignoring.

---

## What this video is

- A **complementary resource to [A Tour of Go](https://go.dev/tour/welcome/1)** — not a replacement for it.
- The author copy-pastes each Tour example into a local editor, runs it, and explains it with his own commentary.
- Its distinguishing feature: after each concept, he jumps to a **real open-source production codebase** (fider, incubator-answer) and shows that same concept in the wild. The goal is to collapse the gap between "learn concept" and "see concept used in real code" from years to seconds.

## What this video is *not*

- Not a traditional exhaustive tutorial exploring every corner of the language.
- Not about installing Go or configuring your editor (out of scope — figure it out).
- Scoped specifically to what a **backend / application developer** needs, not a library author.

## Prerequisites

Basic programming literacy — you know what variables, arrays, and loops are. Prior experience in another language helps but isn't required.

---

## The recommended workflow

1. **Pause and go do the entire Tour of Go yourself first.** Read the left pane, run the right pane. 2–3 hours in one sitting.
2. Don't worry about understanding it. The goal of pass one is *feel for the syntax*.
3. **Then** watch this video / read these notes for the explanation and real-world context.

> "Before you understand the concept you should have a feel of the syntax."

---

## The core thesis: programming is pattern matching

Being a good programmer has little to do with genius or talent. It's **pattern matching** — the developed ability to reach for the right tool at the right moment.

> Example: you see a comma-separated string and need an array. Instantly `split` comes to mind — Python, Java, Go, JavaScript, doesn't matter. You don't reason about it. That's pattern matching.

Pattern matching improves with **code velocity** — the more code you write and read, the better it gets. It becomes muscle memory.

**Consequence for how you learn:** understanding should *not* be your priority. Consistency and volume should be. Understanding follows.

---

## Two pieces of advice

### 1. Don't try to understand everything

The worst thing you can do when learning a new language is demand full comprehension up front. It only produces intimidation, and intimidation makes people quit.

- If something doesn't land in this video, it will land later — reading standard library code, reading open-source repos, building projects.
- Trust the process. Keep writing, keep reading.

### 2. Go has "advanced-looking" concepts you can defer

Concepts like interfaces, value-based error handling, concurrency primitives (fan-in/fan-out, mutexes, select, channel directions), and reflection get enormous coverage in tutorials — but for an **application developer**, they're roughly **20% of what you actually use**.

- **Interfaces are the exception** — you will use those a lot.
- Advanced concurrency patterns: you can work productively for *years* without hand-rolling fan-out/fan-in. You'll use them indirectly through libraries.
- Testing is important — but it belongs in the project-building phase, not the basics phase.

Know that these exist, know roughly when they're needed, and move on. Treating them as blockers while learning the basics is what makes people abandon Go as "a complicated systems language."

---

## While reading the real-world code examples

When the video jumps into fider or incubator-answer: **look only at the specific lines being pointed at.** Do not read the surrounding code and get intimidated. Reading whole codebases end to end comes later in the series.

---

## The success criterion (from the outro)

[▶ 5:51:58](https://www.youtube.com/watch?v=tgGNwG_UxFo&t=21118s)

> "My goal with this video was to get you to 10%."

The video explicitly acknowledges it skips things — reflection, directional channel types, and anything not covered by the Tour of Go. The remaining 60% comes from the rest of the series: Effective Go, the modules reference, the language spec, reading standard library source, reading production open-source backends, and finally building projects.

---

**Next:** [02 — Modules & project setup](02-modules-and-project-setup.md)
