# 005 — Iterable & Iterator Protocols, Generators

> **One-line takeaway:** An **iterable supplies a traversal; an iterator supplies its next
> result**. Generators preserve suspended execution to produce values lazily; async
> iteration awaits each step and supports awaited cleanup.

---

## 1. Core Mental Model — Producer Meets Consumer

`for...of`, iterable spread and other consumers speak a protocol instead of requiring
arrays. The producer controls values and order; the consumer controls how many are needed.
For collection choices and callback methods, see [004 — Collection Traversal](004-collection-traversal-and-iteration.md).

```text
iterable[Symbol.iterator]() → iterator
                                │ next()
                                ├── { value, done: false } → consume → next()
                                └── { value?, done: true } → stop; no loop value
```

## 2. Iterable vs Iterator

| Contract | Requirement | Recall point |
|---|---|---|
| **Iterable** | Callable `[Symbol.iterator]()` returning an iterator | Can start a traversal; may return a fresh or shared iterator. |
| **Iterator** | `next()` returning an object with `value`/`done` semantics | Maintains progress; need not itself be iterable. |
| **Iterable iterator** | Both; `[Symbol.iterator]()` returns `this` | Consumable directly; subsequent loops continue its current state. |

Conceptually, `for...of` obtains the iterator, repeatedly calls `next()`, checks `done`,
and binds `value` for the body. Falsy `done` means another value; truthy `done` means stop.
A terminal result's `value` is **not** a yielded loop value. A non-object result is invalid.
This describes specification behaviour, not a required engine implementation.
Reference: [MDN: iteration protocols](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols).

### Custom iterable: independent traversal state

```js
const range = {
  [Symbol.iterator]() {
    let n = 1;
    return {
      next() {
        return n <= 3 ? { value: n++, done: false } : { done: true };
      },
    };
  },
};
console.log([...range], [...range]); // [1, 2, 3] twice: fresh iterators
```

The returned iterator need not expose `[Symbol.iterator]`: the consumer already has it.
The outer `range` is reusable because each call creates new state. An iterator whose state
is exhausted is not reset by starting another `for...of`.

## 3. Iterator Closing — Request Cleanup on Early Exit

| Event in `for...of` | Calls optional iterator `return()`? |
|---|---|
| `break`, enclosing-function `return`, error in the consumer/body | Yes: early consumer exit requests cleanup. |
| `continue` targeting the same loop | No: traversal continues. |
| Normal exhaustion (`done: true`) | No: the producer is assumed finished. |
| `next()` itself throws / violates the result contract | No automatic closing through this path. |

When called, sync `return()` must return an object. A custom iterator can release resources;
a generator's `return()` enters its `finally` cleanup. **Closing requests cleanup; it does
not mandate how a custom iterator manages resources.** A generator that yields inside
`finally` can suspend again; keep cleanup straightforward. See
[ECMAScript: IteratorClose](https://tc39.es/ecma262/multipage/abstract-operations.html#sec-iteratorclose)
and [MDN: iterator error handling](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols#error_handling).

## 4. Generators — Lazy, Suspended Execution

Calling **`function*`** returns a generator object without running its body. `next()`
starts/resumes it; `yield` produces a result and suspends with local state retained.
`return` completes it with `done: true`. Generator objects are both iterable and iterator,
and one-shot: call the generator function again for fresh traversal.

Laziness means computing only requested values: useful for ranges, tree traversal and
pipelines without materialising intermediate arrays. It does **not** imply asynchronous
execution or parallelism. Iterable spread materialises everything; do not spread an
unbounded generator. See [MDN: function*](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function*).

## 5. Asynchronous Iteration

| Contract / tool | What changes |
|---|---|
| **Async iterable** | `[Symbol.asyncIterator]()` supplies an async iterator. |
| **Async iterator** | `next()` supplies an awaitable result object. |
| **`for await...of`** | Awaits each step, then executes the body; early consumer exit calls and awaits optional `return()`. |
| **`async function*`** | Lazy generator with promise-returning iterator methods; can await production. |

`for await...of` requires an async function or module top level. It prefers the async
protocol; if absent, it wraps a sync iterator and also resolves promise-valued elements.
It does not independently unwrap a promise-valued `value` from an arbitrary native async
iterator; async generators themselves resolve yielded promises.
See [MDN: for await...of](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for-await...of).

```js
async function* pages() {
  yield await Promise.resolve("page 1");
  yield "page 2";
}
(async () => {
  for await (const page of pages()) console.log(page);
})();
```

Node.js readable streams are async iterable: `for await (const chunk of readable)` consumes
chunks. With the default iterator, early exit destroys the stream; cleanup is meaningful
resource behaviour, not just loop syntax. See [Node: readable async iteration](https://nodejs.org/api/stream.html#readablesymbolasynciterator).

## 6. Sequential vs Concurrent Work

```js
async function run() {
  const work = async n => n * 2;
  for (const n of [1, 2]) await work(n); // next call starts after previous completion
  const results = await Promise.all([1, 2].map(work)); // start both, aggregate completion
  console.log(results); // [2, 4]: input order, not completion order
}
run();
```

`for await...of` consumes steps sequentially; it does not cancel or defer work already
started, such as promises created before entering the loop. `Promise.all` aggregates
independent work, rejects on a rejection, and does not cancel the other operations.
Concurrency limits, retries and cancellation require separate policies.
See [MDN: Promise.all](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all).

`forEach` ignores async callback promises; `map` returns them for aggregation. The full
[callback pitfalls](004-collection-traversal-and-iteration.md#4-mutation--async-callback-pitfalls)
belong in the traversal note. Per-iteration binding capture remains in
[002 — Loop Bindings](002-var-let-const.md#b-the-classic-asynchronous-loop).

---

## 7. Active Recall — Predict Before Expanding

### A. Yielded value vs terminal value

```js
function* numbers() { yield 1; return 9; }
const it = numbers();
console.log(it[Symbol.iterator]() === it);
console.log([...it], [...it]);
console.log(numbers().next());
```

<details>
<summary>Show answer & reason</summary>

`true`; `[1] []`; `{ value: 1, done: false }`. Spread consumes the same iterable iterator
to exhaustion; terminal `9` is not yielded. Calling `numbers()` creates fresh state.

</details>

### B. Resume, close, then resume again?

```js
function* values() {
  try { yield 1; yield 2; }
  finally { console.log("cleanup"); }
}
const it = values();
console.log(it.next().value);
for (const n of it) { console.log(n); break; }
console.log(it.next());
```

<details>
<summary>Show answer & reason</summary>

`1`, `2`, `"cleanup"`, then `{ value: undefined, done: true }`. The loop resumes the
existing iterator; `break` invokes its `return()`, runs `finally`, and completes it.

</details>

### C. A sync iterable containing promises

```js
async function inspect() {
  const values = [Promise.resolve(3)];
  for (const value of values) console.log(value instanceof Promise);
  for await (const value of values) console.log(value);
}
inspect();
```

<details>
<summary>Show answer & reason</summary>

`true`, then `3`. Ordinary iteration yields the stored promise; the async-from-sync
adapter used by `for await...of` resolves that promise-valued element.

</details>

**Previous:** [004 — Collection Traversal & Iteration](004-collection-traversal-and-iteration.md).

---

## 8. Quick Recall — Interview One-Liners

| Question | Crisp answer |
|---|---|
| **Iterable vs iterator?** | Supplies a traversal vs supplies next() result objects; an object can implement both. |
| **How does `for...of` consume values?** | Obtain iterator, call next(), test done, consume value, repeat; handle early-exit cleanup. |
| **Does `done: true` contribute a loop value?** | No, even if the terminal result has a value. |
| **Reusable iterable vs one-shot iterator?** | Fresh state per iterator call enables repetition; reusing an exhausted iterator does not reset it. |
| **When is `return()` called?** | Early consumer exit, not normal exhaustion or a failed next() step; same-loop continue keeps iterating. |
| **Why are generators lazy?** | Calling creates an object; resumption executes code until yield/completion. |
| **Does laziness imply async execution?** | No; async generators add awaitable production to suspension. |
| **What does `for await...of` await?** | Iterator result steps and early-exit cleanup; the sync fallback also resolves promise-valued elements. |
| **How do streams fit?** | Async iterables of chunks, with resource cleanup on early exit according to the stream iterator's contract. |
| **Sequential vs concurrent async work?** | Await each operation before starting the next, or start independent operations and aggregate with Promise.all. |
