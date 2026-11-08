# 002 — var, let & const

> **One-line takeaway:** `var` is the 1995 keyword: **function-scoped** and hoisted as `undefined`, which leads to silent bugs. `let` and `const` (ES2015) are **block-scoped** and locked until declared (the **TDZ**), so mistakes fail loudly. Use **`const` by default, `let` when the value must change, and never `var`.**

> 🧪 **How to use this note:** Answer every **Predict the output** in your head before
> opening it. Run the snippets in DevTools (F12 → Console). Tip: paste each one inside
> `{ }` so `let`/`const` names don't clash between runs.

---

## 1. The Three Keywords at a Glance

```text
var    → 1995 · function-scoped · hoisted as undefined · redeclarable
let    → 2015 · block-scoped    · TDZ                  · reassignable
const  → 2015 · block-scoped    · TDZ                  · NOT reassignable
```

| Feature | `var` | `let` | `const` |
|---|---|---|---|
| **Scope** | Function | Block `{ }` | Block `{ }` |
| **Hoisted?** | ✅ set to `undefined` | ✅ but **locked** (TDZ) | ✅ but **locked** (TDZ) |
| **Use before declaring** | `undefined` 🤫 silent | `ReferenceError` 📢 | `ReferenceError` 📢 |
| **Redeclare in same scope** | ✅ allowed | ❌ SyntaxError | ❌ SyntaxError |
| **Reassign** | ✅ | ✅ | ❌ TypeError |
| **Must initialise** | No | No | **Yes** |
| **Top-level → `window` property** | ✅ | ❌ | ❌ |

> 🧠 Every row in this table answers one question: *"How easy is it to shoot yourself in
> the foot?"* `var` makes it easy. `let`/`const` make it hard.

---

## 2. Scope: Where Can a Variable Be Seen?

> **Scope is the region of code where a variable is visible.**

### 🏢 The office analogy

```text
var        → a notice pinned on the WHOLE FLOOR's board
             visible anywhere inside the FUNCTION it was declared in

let/const  → a sticky note inside ONE MEETING ROOM
             visible only inside the nearest { } block
             (if, for, while, or a bare { })
```

```js
function demo() {
  if (true) {
    var  floorNotice = "everyone sees me";
    let  roomNote    = "only this room";
  }
  console.log(floorNotice);  // "everyone sees me"
  console.log(roomNote);     // ReferenceError: roomNote is not defined
}
```

```text
function demo() { ───────────────────────────── var lives here
│   if (true) { ──────────────────── let lives here
│   │   var floorNotice
│   │   let roomNote
│   } ─────────────────────────────── roomNote dies here
│   floorNotice ✅   roomNote ❌
} ──────────────────────────────────────────── floorNotice dies here
```

> 🧪 **Predict the output**
> ```js
> var x = "outer";
> if (true) {
>   var x = "inner";
> }
> console.log(x);
>
> let y = "outer";
> if (true) {
>   let y = "inner";
> }
> console.log(y);
> ```

<details>
<summary>Show answer</summary>

`"inner"`, then `"outer"`.

`var` ignores the `if` block. Both `var x` lines are the **same** variable, so the inner
one overwrites it. `let y` inside the block is a **new, separate** variable that only
lives in that block (this is called *shadowing*). The outer `y` is untouched.
</details>

### Redeclaring: why `var` is risky

```js
var count = 5;
// … 200 lines later, someone else writes:
var count = 0;     // ✅ no error. The original value is silently wiped out.

let total = 5;
let total = 0;     // ❌ SyntaxError: Identifier 'total' has already been declared
```

---

## 3. Hoisting: What Happens *Before* Your Code Runs

### 🏫 The classroom analogy

A teacher **takes the attendance register first**, writing down every name, *then* starts
the lesson. JavaScript does the same with every scope:

```text
          Your code in a scope
                   │
                   ↓
 ┌──────────────────────────────────────────────────┐
 │ PHASE 1: CREATION  ("take attendance")           │
 │   scan the scope for declarations                │
 │   register every name, before running anything   │
 │     var a       → registered, set to undefined   │
 │     let b       → registered, LOCKED             │
 │     const c     → registered, LOCKED             │
 │     function f  → registered, set to the function│
 └──────────────────────────────────────────────────┘
                   │
                   ↓
 ┌──────────────────────────────────────────────────┐
 │ PHASE 2: EXECUTION  ("start the lesson")         │
 │   run the code line by line                      │
 │   each "= value" happens only when its line runs │
 └──────────────────────────────────────────────────┘
```

> **Hoisting** = every declaration is *known* from the top of its scope, because it was
> registered in Phase 1. **Nothing is physically moved.** And only the **name** is
> registered early, never the `= value` part.

So `var`, `let`, `const` and functions are **all hoisted**. The only difference is what the
name holds *before* its line runs.

### 🪑 The seat analogy

```text
var        → seat has a DUMMY sitting in it (undefined)
             you can "talk" to it early. It answers "undefined", no complaint.

let/const  → seat is ROPED OFF with a "RESERVED" sign
             touching it early → alarm (ReferenceError)
             the rope comes off only when the declaration line runs

function   → the real person is already sitting there
             you can call it before its line
```

### `var`: hoisted with `undefined` 🤫

```js
console.log(a);   // undefined   ← no error!
var a = 10;
console.log(a);   // 10
```

What the engine actually does:

```text
Phase 1:  a = undefined           (name registered, value NOT yet)
Phase 2:  console.log(a)  → undefined
          a = 10                  ← "= 10" happens only now
          console.log(a)  → 10
```

> That silent `undefined` is the problem. Your code keeps running with a wrong value, and
> the bug surfaces somewhere far away.

### `let` / `const`: hoisted but locked (the Temporal Dead Zone) 📢

```js
console.log(b);   // ReferenceError: Cannot access 'b' before initialization
let b = 20;
console.log(b);   // 20
```

The stretch where `b` exists but is still roped off is the **Temporal Dead Zone (TDZ)**:

```text
{ ◄── scope starts: b is registered but LOCKED 🔒
  │
  │   console.log(b);   ✗ ReferenceError   ┐
  │   b = 5;            ✗ ReferenceError   ├─ TEMPORAL DEAD ZONE
  │   typeof b;         ✗ ReferenceError   ┘
  │
  │   let b = 20;       ◄── rope comes off 🔓 (initialised)
  │
  │   console.log(b);   ✓ 20
}
```

> 🔍 **Read the error message closely:** *"Cannot access 'b' before initialization"*, not
> *"b is not defined"*. The engine is admitting it **already knows `b` exists**. That's
> your proof that `let` is hoisted.

### 🧪 Proof that `let` is hoisted

> **Predict the output**
> ```js
> let x = 1;
> {
>   console.log(x);
>   let x = 2;
> }
> ```

<details>
<summary>Show answer</summary>

**`ReferenceError`**, not `1`!

```text
Phase 1 of the inner { } registers its OWN x (locked 🔒).
That inner x hides (shadows) the outer x from the very first line of the block.
So console.log(x) finds the inner x, which is still in the TDZ → error.
```

If `let` were *not* hoisted, the engine would have found the outer `x` and printed `1`.
</details>

### ⏱️ "Temporal" means *time*, not *position*

> **Predict the output**
> ```js
> function greet() {
>   console.log(msg);   // written ABOVE the declaration
> }
> let msg = "hello";
> greet();
> ```

<details>
<summary>Show answer</summary>

**`"hello"`** ✅. The TDZ is about **when the line runs**, not where it is written.
`greet()` is *called* after `let msg = "hello"` has run, so the rope is already off. Move
`greet();` above the `let` line and it throws.
</details>

### Hoisting summary

| Declaration | Hoisted? | Before its line, it holds | Accessing early gives |
|---|---|---|---|
| `var x` | ✅ | `undefined` (dummy) | `undefined` 🤫 |
| `let x` | ✅ | nothing (locked 🔒) | `ReferenceError` 📢 |
| `const x` | ✅ | nothing (locked 🔒) | `ReferenceError` 📢 |
| `function f(){}` | ✅ | the whole function | works ✅ |

> The TDZ is a **feature**: it turns "used too early" from a silent bug into a loud error.

---

## 4. ⚡ The Classic Interview Question: `var` vs `let` in a Loop

> 🧪 **Predict the output** of both loops. Commit to an answer first!
> ```js
> for (var i = 0; i < 3; i++) {
>   setTimeout(() => console.log(i), 0);
> }
>
> for (let j = 0; j < 3; j++) {
>   setTimeout(() => console.log(j), 0);
> }
> ```

<details>
<summary>Show answer</summary>

`var` loop → **`3 3 3`**
`let` loop → **`0 1 2`**

Most people guess `0 1 2` for both. Read on for why.
</details>

You need **two key ideas** to explain it.

### 🔑 Key idea 1: the callbacks run *after* the loop finishes

From note 001: `setTimeout` hands the callback to the host's timer. It runs only when the
call stack is empty, so **the whole loop finishes first**, even with a 0 ms delay.

### 🔑 Key idea 2: a closure remembers the *variable*, not the value

`() => console.log(i)` does **not** take a photo of `i`'s value when created. It keeps a
**live link to the variable `i`** and reads it whenever it finally runs.

```text
A closure is NOT a photo 📸 of the value
A closure IS a window 🪟 onto the variable: it sees whatever is there NOW
```

So the real question is: **how many `i` variables are there?**

### 📋 Case 1: `var`, ONE shared whiteboard

`var` is function-scoped, so the loop has **one** `i`, like a single whiteboard that every
callback will read later.

```text
Step   What happens                          i on the board   Queue
────   ───────────────────────────────────   ──────────────   ───────────────
 1     i = 0 → schedule cb1 (watches i)            0          [cb1]
 2     i++   → schedule cb2 (watches i)            1          [cb1, cb2]
 3     i++   → schedule cb3 (watches i)            2          [cb1, cb2, cb3]
 4     i++   → is 3 < 3? NO → loop ends            3
────   stack is empty → event loop runs the queue ──────────────────────────
 5     cb1 reads the board                         3   → prints 3
 6     cb2 reads the board                         3   → prints 3
 7     cb3 reads the board                         3   → prints 3
```

```text
   cb1 ──┐
   cb2 ──┼──►  i  (ONE variable, final value 3)
   cb3 ──┘
```

> 🤔 *Why 3 and not 2?* The loop only stops when `i++` pushes `i` to 3 and the check
> `3 < 3` fails. So the board's final value is **3**.

### 📒 Case 2: `let`, a NEW notebook every iteration

With `for (let …)`, JavaScript creates a **fresh `j` for every iteration**, copying in the
current value. Each callback watches its own notebook.

```text
Iteration 1:  new j₁ = 0   → cb1 watches j₁
Iteration 2:  new j₂ = 1   → cb2 watches j₂
Iteration 3:  new j₃ = 2   → cb3 watches j₃
(j++ happens on the NEXT iteration's fresh copy, so j₁, j₂, j₃ never change)
```

```text
   cb1 ──►  j₁ = 0
   cb2 ──►  j₂ = 1
   cb3 ──►  j₃ = 2
```

The callbacks *still* run after the loop, but each reads its own `j` → **`0 1 2`**.

### 🛠️ How people fixed it before `let` existed

Wrap each iteration in a function that is called immediately (an **IIFE**). Every call
creates a brand-new variable `k`:

```js
for (var i = 0; i < 3; i++) {
  (function (k) {                         // k = a NEW variable on every call
    setTimeout(() => console.log(k), 0);
  })(i);                                  // pass in the current value of i
}
// 0 1 2
```

> `let` in a `for` loop is basically this fix built into the language.

### 🎤 Say it in 30 seconds

> "The callbacks run after the loop ends, and closures capture **variables**, not values.
> With `var` there's a single function-scoped `i` shared by all three callbacks; it ends at
> 3, so they print 3 3 3. With `let`, each iteration gets a fresh block-scoped binding, so
> each callback sees its own value and prints 0 1 2."

---

## 5. `const` Does NOT Mean "Can't Change"

`const` locks the **arrow** (the binding), **not the value** it points at. (Remember
*arrows, not boxes* from [note 001](001-variables-and-data-types.md).)

> 🧪 **Which lines throw?**
> ```js
> const arr = [1, 2];
> arr.push(3);              // line A
> arr[0] = 99;              // line B
> arr = [7, 8, 9];          // line C
>
> const n = 5;
> n++;                      // line D
> ```

<details>
<summary>Show answer</summary>

Only **C** and **D** throw `TypeError: Assignment to constant variable`.

```text
A, B → change the OBJECT the arrow points at        ✅ allowed
C    → move the arrow to a new array                ❌
D    → n++ means n = n + 1, i.e. move the arrow     ❌
```
</details>

```text
const arr ──🔒──►  [99, 2, 3]
           arrow     contents
           LOCKED    still editable
```

Want the **contents** locked too? Use `Object.freeze`:

```js
const cfg = Object.freeze({ port: 80, db: { host: "x" } });
cfg.port = 3000;       // ignored (TypeError in strict mode)
cfg.db.host = "y";     // ⚠️ WORKS: freeze is SHALLOW, nested objects stay editable
```

---

## 6. Two More `var` Gotchas

### Top-level `var` leaks onto `window`

```js
var  appName = "Shop";
let  version = 2;
window.appName;   // "Shop"     ← polluted the global object
window.version;   // undefined  ← let stays out of it
```

### Forgetting the keyword creates a global by accident

```js
function setTotal() {
  total = 100;      // no var/let/const!
}
setTotal();
console.log(total); // 100 → an accidental GLOBAL variable
```

```text
Sloppy mode → silently creates a global                  🤫
Strict mode → ReferenceError: total is not defined       📢
              ("use strict", ES modules and classes are strict automatically)
```

---

## 7. Which One Should I Use?

```text
                Declaring a variable
                         │
                         ↓
      Will this name ever point to a different value?
                         │
              ┌──────────┴──────────┐
              NO                    YES
              ↓                     ↓
            const                  let
       (the default,         (counters, accumulators,
        ~90% of cases)        values that get reassigned)

                  var → ❌ never in new code
                  (you only need to READ it in legacy code)
```

> 💼 `const` by default also *communicates intent*. A reader instantly knows the binding
> never changes, and only has to watch the `let`s.

---

## 8. Quick Recall — Interview One-Liners

| Question | Crisp answer |
|---|---|
| **`var` vs `let` vs `const`?** | `var`: function-scoped, hoisted as `undefined`, redeclarable. `let`: block-scoped, TDZ, reassignable. `const`: block-scoped, TDZ, no reassignment. |
| **What is scope?** | The region where a variable is visible. `var` → whole function; `let`/`const` → nearest `{ }` block. |
| **What is hoisting?** | In the creation phase, every declaration is registered at the top of its scope before code runs. Only names are hoisted, not assignments. |
| **Are `let` and `const` hoisted?** | Yes, but left uninitialised. The error "Cannot access before initialization" (not "not defined") proves the engine knows they exist. |
| **What is the TDZ?** | The time from scope start until a `let`/`const` declaration runs. Any access in that window throws `ReferenceError`. It's about *time*, not position. |
| **Are function declarations hoisted?** | Yes, fully. You can call them before their line. |
| **`var` loop with `setTimeout` prints?** | `3 3 3`: callbacks run after the loop and share one `i`, which ended at 3. With `let`, `0 1 2`: a fresh binding per iteration. Pre-ES6 fix: IIFE. |
| **Do closures capture values or variables?** | Variables: a live link read when the function runs, not a snapshot. |
| **Does `const` make objects immutable?** | No. It locks the binding, not the object. `Object.freeze` locks the object, but only shallowly. |
| **Can you redeclare `let`?** | Not in the same scope (SyntaxError). `var` allows it silently. |
| **Assign without a keyword?** | Sloppy mode creates an accidental global; strict mode throws `ReferenceError`. |
| **Which should you use?** | `const` by default, `let` when reassignment is needed, never `var`. |

---

## 9. The Mental Model

```text
                          DECLARE A NAME
                                │
          ┌─────────────────────┴─────────────────────┐
          ↓                                           ↓
         var  🤫                                let / const  📢
          │                                           │
  scope: whole FUNCTION 🏢                   scope: nearest { } BLOCK 🚪
          │                                           │
  hoisted → dummy "undefined" 🪑              hoisted → ROPED OFF 🔒 (TDZ)
  early use = silent bug                      early use = loud ReferenceError
          │                                           │
  loop: ONE shared variable 📋                loop: NEW variable per iteration 📒
  → 3 3 3                                     → 0 1 2
          │                                           │
  redeclare ✅  leaks to window ✅             redeclare ❌  window ❌
                                                      │
                                         const = arrow locked 🔒
                                                 contents still editable
```

> **Roll call first, lesson second.** `var` gets a dummy, `let`/`const` get a rope.
> `var` shares one whiteboard; `let` hands out a new notebook every loop.

⬅️ **Previous:** [001 — Variables & Data Types](001-variables-and-data-types.md)
