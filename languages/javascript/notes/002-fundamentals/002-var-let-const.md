# 002 — var, let & const

> **One-line takeaway:** Separate **scope**, **initialisation** and **reassignment**.
> `var` is initialised early; `let`/`const` have a TDZ. `const` fixes the binding,
> not the object's state. Prefer `const`; use `let` for reassignment.

---

## 1. Core Mental Model — Three Independent Questions

```text
Declaration → Where does the binding exist?       (scope)
            → When can it first be accessed?      (initialisation / TDZ)
            → Can it later hold a different value? (reassignment)
```

A **binding associates an identifier with a value within a scope**; it is distinct from
the value. `user = other` reassigns the binding; `user.name = "Avi"` mutates the referenced
object. Multiple bindings can refer to that same object.

For value semantics, see [001 — Variables & Data Types](001-variables-and-data-types.md).

### Environment Records — the specification's binding model

ECMAScript uses **Environment Records** to model bindings and identifier resolution.
A record tracks binding state and links to an outer record; lookup starts at the current
environment and follows those outer links until it finds the name or reaches the end.

```text
Inner environment { user: uninitialised } → outer { user: object } → … → null
                  ↑ lookup stops here; TDZ does not fall back to the outer user
```

The braces are conceptual. Declarative records model lexical bindings; object records use
object properties; global records combine both. **This does not mandate object storage or
a heap/stack layout.** See [ECMAScript: Environment Records](https://tc39.es/ecma262/multipage/executable-code-and-execution-contexts.html#sec-environment-records).

---

## 2. Key Concepts — Scope & Declaration Rules

| Feature | `var` | `let` | `const` |
|---|---|---|---|
| **Scope** | Function, or top-level script/module; also class static block | Lexical: block, function body, script/module | Same as `let` |
| **Before declaration executes** | New binding already initialised to `undefined` | Binding exists, uninitialised: TDZ | Same as `let` |
| **Reassignment** | Allowed | Allowed | `TypeError` after initialisation |
| **Redeclaration in same scope** | Repeated `var` allowed; lexical conflicts rejected | Rejected | Rejected |
| **Initializer required** | No | No: `let x;` initialises to `undefined` when reached | Yes, except `for...of` / `for...in` declarations |
| **Browser classic-script top level** | Global object property | Global lexical binding | Global lexical binding |

### Scope and shadowing

**Shadowing** creates a separate inner binding; assignment changes whichever binding
name resolution finds. An ordinary `if`/`for` block does not scope `var`.

```js
function demo() {
  var x = 1;
  let y = 1;
  {
    var x = 2; // same function binding
    let y = 2; // distinct block binding
  }
  console.log(x, y); // 2 1
}
```

`var x;` preserves an existing value; `var x = 2` overwrites it. Conflicting lexical/`var`
declarations cause `SyntaxError`; a nested `let` may shadow an outer `var`.
`switch` cases share a block scope; brace cases needing separate
lexical declarations. See [MDN: `var`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/var)
and [MDN: lexical redeclarations](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let#redeclarations).

---

## 3. Hoisting, Initialisation & the TDZ

**Hoisting** describes pre-execution declaration behaviour, not source movement. Binding
creation and initialisation are separate specification operations; engines need not use
a literal two-pass scan.

**Lifecycle:** create binding → initialise → read/access → reassign if mutable.
`var` is initialised during declaration instantiation; lexical bindings wait until their
declarations execute. An uninitialised binding has **no value**, not the value `undefined`.

| Declaration | State before its statement runs | Early use |
|---|---|---|
| `var n = 10` | A newly created `n` already holds `undefined` | Read succeeds; initializer has not run. |
| `let n = 10` / `const n = 10` | Binding exists but is uninitialised | Read or write throws `ReferenceError`. |
| `function f() {}` | Function value available in ordinary function-body scope | Calling before the declaration works. |
| `var f = function () {}` | `f` holds `undefined` | Calling it throws `TypeError`. |

Function expressions follow their binding's rules. Block-level function declarations have
additional strict/sloppy rules; do not generalise function-body behaviour to every block.

### TDZ: the binding exists before its value does

The TDZ lasts from scope entry until **initialisation completes**. The inner binding
shadows an outer binding throughout that scope, including during its own initializer:
`let x = x` reads the uninitialised inner `x` and throws.

`typeof missingName` returns `"undefined"` for an unresolvable name, but `typeof x` in
`x`'s TDZ throws. TDZ depends on **execution time**, not textual position: a function
written above `let x` can safely read it when called after initialisation.

> **Interview phrasing:** “Lexical bindings exist before their declarations execute but
> remain uninitialised.” Whether you call this “hoisted” varies by convention; explain the
> behaviour. See [MDN: hoisting](https://developer.mozilla.org/en-US/docs/Glossary/Hoisting)
> and [TDZ](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let#temporal_dead_zone_tdz).

---

## 4. Critical Distinction — `const` vs Immutability

`const` prevents rebinding, including `count++` because increment writes back to the
binding. Reassignment throws in both strict and sloppy mode.

`const items = [1]; items.push(2);` mutates successfully; `items = []` throws.

`Object.freeze(obj)` is **shallow**: it prevents adding/removing the object's own
properties and makes its own data properties non-writable. It does not recursively freeze
nested objects or guarantee that every kind of internal object state is immutable.

See [MDN: Object.freeze](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze).
Challenge C contrasts nested mutation with a rejected property write.

---

## 5. Edge Cases — Globals, Strict Mode & Loops

### Top-level bindings depend on the execution environment

| Context | Top-level `var` | Top-level `let` / `const` |
|---|---|---|
| **Browser classic `<script>`** | Creates/reuses a global object binding; normally visible via `globalThis` / `window` | Global lexical bindings, not global object properties |
| **ES module** | Module-scoped; no automatic global property | Module-scoped |
| **Node.js CommonJS file** | Scoped to the module wrapper | Scoped to the module wrapper |

Strict mode does not make a classic script module-scoped; REPL/DevTools rules may differ.
See [MDN: script vs module declarations](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/var#description).

Assignment to an **unresolvable name** usually creates a global object property in sloppy
mode; strict mode throws `ReferenceError`. This is assignment, not a `var` declaration.
ES modules and class bodies are automatically strict; ordinary scripts/functions need
`"use strict"` to opt in. See [MDN: strict mode](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode).

### Loop bindings

`for (let i = 0; …)` creates per-iteration bindings. Declaring `let i` **outside** the loop
and using `for (i = 0; …)` shares one binding instead. `for (const item of values)` works
because each iteration initialises a new binding; `for (const i = 0; …; i++)` fails on
increment. These are binding rules, not special timer behaviour; see [MDN: for-loop lexical declarations](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for#lexical_declarations_in_the_initialization_block).

**Modern practice:** default to `const`, use `let` for reassignment, keep scopes narrow.
Review legacy `var` migrations: changing scope can change behaviour.

---

## 6. Active Recall — Predict Before Expanding

Run each challenge independently. Uncaught exceptions stop the remaining statements.

### A. Shadowing + `typeof` in the TDZ

```js
let value = "outer";
{
  console.log(typeof value);
  let value = "inner";
}
```

<details>
<summary>Show answer & reason</summary>

`ReferenceError`; nothing is logged. The inner binding shadows the outer one from block
entry, but is still uninitialised. `typeof` only tolerates an unresolvable name, not a TDZ
binding. Moving the declaration before the log produces `"string"`.

</details>

### B. The classic asynchronous loop

```js
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log("var", i), 0);
}
for (let j = 0; j < 3; j++) {
  setTimeout(() => console.log("let", j), 0);
}
```

<details>
<summary>Show answer & reason</summary>

`var` callbacks print `3, 3, 3`; `let` callbacks print `0, 1, 2`. Both loops finish before
their callbacks run in ordinary browser/Node execution.

Closures retain access to their **lexical environment**, not value snapshots. All `var`
callbacks resolve the same `i` (now 3); each `let` callback resolves its iteration's `j`.
The update expression operates on the next iteration's binding. The old IIFE workaround
created a separate parameter binding per call.

See the [timer refresher](../001-getting-started/001-introduction-to-javascript.md#the-revealing-example)
and [MDN's loop explanation](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for#lexical_declarations_in_the_initialization_block).

</details>

### C. Constant binding vs shallow freezing

```js
"use strict";
const state = Object.freeze({ nested: { n: 1 } });
state.nested.n++;
console.log(state.nested.n);
state.extra = true;
console.log("done");
```

<details>
<summary>Show answer & reason</summary>

Logs `2`, then throws `TypeError`: no `"done"`. Only the outer object is frozen. In sloppy
mode the property write is ignored and `"done"` logs. Rebinding `state` throws in either mode.

</details>

**Previous:** [001 — Variables & Data Types](001-variables-and-data-types.md).
**Next:** [003 — Loops & Control Flow](003-loops-and-control-flow.md).

---

## 7. Quick Recall — Interview One-Liners

| Question | Crisp answer |
|---|---|
| **`var` vs `let` vs `const`?** | `var`: broader scope and early undefined initialisation; `let`/`const`: lexical scope and TDZ; `const` rejects rebinding. |
| **Are `let` and `const` hoisted?** | Bindings exist before execution reaches them, but are uninitialised; describe the behaviour rather than debating the label. |
| **What is an Environment Record?** | An abstract specification model for bindings, with an outer link used for identifier resolution; not a mandated storage layout. |
| **What ends the TDZ?** | Completion of binding initialisation, not merely reaching an earlier reference or assignment. |
| **Why does shadowing sometimes throw?** | Name lookup finds the inner binding even before its initialisation; it does not fall back to the outer value. |
| **Why `3 3 3` vs `0 1 2`?** | One shared `var` binding versus per-iteration `let` bindings, read after the loops finish. |
| **Does `const` mean immutable?** | No: the binding is fixed; object mutation is independent, and `Object.freeze` is shallow. |
| **Does top-level `var` create a global property?** | In a classic script, normally yes; in ES modules and CommonJS files, no. |
| **What does strict mode change here?** | Unresolvable assignment and forbidden property writes throw; TDZ and const-reassignment errors already occur without it. |
