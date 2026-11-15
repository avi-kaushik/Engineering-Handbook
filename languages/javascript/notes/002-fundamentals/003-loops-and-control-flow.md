# 003 — Loops & Control Flow

> **One-line takeaway:** Choose a loop by **when it tests, how it advances, and where
> control goes next**. `break` ends a loop; `continue` advances it; neither exits a function.

---

## 1. Core Mental Model — Test, Work, Progress

A condition-controlled loop needs a stopping condition and progress toward it. JavaScript
converts each evaluated condition to a boolean; a truthy condition permits another iteration.

```text
for:       initialise → test → body → update → test …
while:                  test → body          → test …
do...while:                    body → test   → body …
```

Conditions are re-evaluated, not captured once. Synchronous loops do not yield control
automatically; a long or infinite loop blocks the thread running it.

## 2. Loop Selection Cheat Sheet

| Tool | Why / when | When NOT |
|---|---|---|
| **`for`** | Explicit index, stride, reverse traversal or coordinated counters | Only iterable values matter; prefer `for...of`. |
| **`while`** | Unknown count; condition determines whether work starts | The body must run at least once. |
| **`do...while`** | Work once, then decide whether to repeat | Zero iterations must be possible. |
| **`for...of`** | Values with direct loop control | Need property keys or a transformation result. |
| **Array methods** | Express mapping, selection, search or aggregation | Need arbitrary `break`/`continue` or sequential async body work. |

The last two belong to [004 — Collection Traversal & Iteration](004-collection-traversal-and-iteration.md).
Use [005 — Async Iteration](005-iterable-iterator-protocols-and-generators.md#5-asynchronous-iteration)
for asynchronous sequences and sequential/concurrent work.

## 3. Execution & Control Transfers

| Statement | Destination / implication |
|---|---|
| **`break`** | Leaves the nearest enclosing loop or `switch`. In a switch inside a loop, it leaves only the switch. |
| **`continue` in `for`** | Skips the remaining body, **runs the update**, then tests again. |
| **`continue` in `while`** | Goes directly to the condition; a skipped body update can cause an infinite loop. |
| **`continue` in `do...while`** | Goes to the trailing condition. |
| **`return`** | Exits the current function, including its enclosing loops. |
| **Labelled control** | `break label` exits the labelled statement; `continue label` targets an enclosing labelled loop. |

A label is not an arbitrary jump target. These statements cannot jump across a function
boundary; returning from a callback does not control the surrounding caller's loop.
See [MDN: continue](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/continue).

## 4. Index-Based Traversal & Pitfalls

```js
const values = [10, 20, 30, 40];
for (let i = values.length - 1; i >= 0; i -= 2) {
  console.log(i, values[i]); // 3 40; 1 20
}
```

- **Bounds:** `i < array.length` excludes the out-of-range index `length`; backward
  traversal usually starts at `length - 1`. Empty arrays then perform zero iterations.
- **Live vs fixed length:** `i < array.length` observes current length. Caching length
  fixes the upper bound, not the values; insertions/deletions can shift or skip elements.
- **Missing condition:** `for (;;)` has no terminating test; use an explicit exit.
  A trailing semicolon in `for (...);` or `while (...);` creates an empty body.
- **Post-test syntax:** write `do { ... } while (condition);`. A false initial condition
  still permits the first body execution.
- **Bindings:** per-iteration lexical binding rules are explained in
  [002 — Loop Bindings](002-var-let-const.md#loop-bindings); do not infer them from timing.

Reference: [MDN: loops and iteration](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Loops_and_iteration).

---

## 5. Active Recall — Predict Before Expanding

### A. Does `continue` skip the update?

```js
const seen = [];
let i = 0;
for (; i < 4; i++) {
  if (i === 1) continue;
  if (i === 3) break;
  seen.push(i);
}
console.log(seen, i);
```

<details>
<summary>Show answer & reason</summary>

`[0, 2]`, `3`. `continue` at 1 still runs `i++`; `break` at 3 exits without the update.
Moving the update into the body after `continue` would change the progress guarantee.

</details>

### B. Post-test execution + labelled continuation

```js
let count = 0;
do { count++; } while (false);
const pairs = [];
outer: for (let i = 0; i < 2; i++) {
  for (let j = 0; j < 3; j++) {
    if (j === 1) continue outer;
    pairs.push(`${i}:${j}`);
  }
}
console.log(count, pairs);
```

<details>
<summary>Show answer & reason</summary>

`1`, `["0:0", "1:0"]`. The post-test loop executes once. `continue outer` abandons the
inner iteration and advances the outer `for`, including its update expression.

</details>

**Previous:** [002 — var, let & const](002-var-let-const.md).
**Next:** [004 — Collection Traversal & Iteration](004-collection-traversal-and-iteration.md).

---

## 6. Quick Recall — Interview One-Liners

| Question | Crisp answer |
|---|---|
| **`for` vs `while`?** | Same pre-test capability; `for` makes initialisation and update explicit, while `while` emphasises the condition. |
| **When use `do...while`?** | When work must happen before the first stopping decision. |
| **What follows `continue`?** | `for`: update then test; `while`/`do...while`: test. |
| **What follows `break`?** | Execution after the targeted loop/switch; no final loop update. |
| **Does `return` merely skip an iteration?** | No: it exits the current function. A callback is a different function. |
| **Why can length mutation surprise an indexed loop?** | Its condition may read live length, and shifting indexes can skip or revisit values. |
| **Do labels act like `goto`?** | No: they only target enclosing labelled statements/loops under the control-statement rules. |
| **When choose traversal methods instead?** | When the data contract is values, keys, search or transformation rather than manual progress. |
