# 004 — JavaScript vs ECMAScript: Standards, Engines & Runtimes

> **One-line takeaway:** ECMAScript defines the language rules, an engine executes them,
> and a runtime combines the engine with host APIs and scheduling.

This companion to [001 — Introduction to JavaScript](001-introduction-to-javascript.md)
expands the standardisation and execution-environment distinctions for interviews.

---

## 1. JavaScript vs ECMAScript — Why a Standard?

Netscape introduced JavaScript; Microsoft implemented **JScript in Internet Explorer 3
(1996)**. Competing implementations and extensions threatened portability: developers
could not assume the same script would behave identically across browsers.

Netscape brought the language to Ecma in **1996**; the first **ECMA-262** edition was
adopted in **June 1997**. The goal was a common language contract across vendors, rather
than one browser deciding the rules. See [ECMA-262's introduction and edition history](https://ecma-international.org/publications-and-standards/standards/ecma-262/).

| Name | Role | Common confusion |
|---|---|---|
| **JavaScript** | Everyday name for the language used by developers. | Not an engine or the browser's entire API surface. |
| **ECMAScript** | Formal name of the standardised language; ECMA-262 defines syntax and semantics. | Not a competing language to learn separately. |
| **Ecma International** | Standards organisation that publishes ECMA-262. | Not the committee or an implementation. |
| **TC39** | Ecma technical committee that maintains and evolves ECMAScript. | Does not ship V8 or define the DOM. |
| **ES6 / ES2015** | Two names for the sixth edition; annual editions followed. | New editions do not imply immediate support in every engine. |

[TC39](https://tc39.es/) develops proposals with implementers and the community;
completed features enter the specification. Engines implement those rules independently.

> **Standardisation has a boundary:** ECMAScript aligns language behaviour, but historical
> DOM differences needed separate web standards. A language standard alone cannot make
> every browser API identical.

---

## 2. The Core Model: Specification → Engine → Runtime

| Layer | What it does | Examples |
|---|---|---|
| **Specification** | Defines observable language behaviour: syntax, types, operators, objects, promises. | ECMAScript / ECMA-262 |
| **Engine** | Parses and executes code, compiles where appropriate, manages memory. | V8, SpiderMonkey, JavaScriptCore |
| **Runtime / host environment** | Embeds an engine and supplies APIs, I/O and scheduling. | Browser, Node.js |

```text
ECMAScript rules
       ↓ implemented by
Engine + host APIs + scheduling = runtime environment
       ├── Browser: DOM, events, fetch, timers
       └── Node.js: fs, process, networking, timers
```

**Chrome and Node.js both embed V8**, but Node does not provide the browser DOM.
Sharing an engine does not imply sharing APIs. Other engines can implement the same
language rules. The [specification's host model](https://tc39.es/ecma262/multipage/overview.html#sec-hosts-and-implementations)
explicitly separates the language from its environment.

```js
Promise.resolve(42);       // ECMAScript built-in
setTimeout(() => {}, 0);   // host API: browser or Node
// document.querySelector("h1"); // browser DOM; absent in plain Node
```

> **Interview trap:** `fetch`, `console`, `setTimeout`, the DOM and Node's `fs` are
> host APIs. JavaScript does not mean “ECMAScript + DOM”.

### One example ties the model together

```js
console.log("1");
setTimeout(() => console.log("2"), 0);
console.log("3");
// Output: 1  3  2
```

The host arranges the timer; synchronous code continues. The callback becomes eligible
for a later turn, after the current work finishes. **Zero delay is not immediate
execution or an exact timing guarantee.** Async I/O enables concurrency; a long
synchronous loop still blocks the thread running it.

> **Single-threaded needs context:** workers can run JavaScript in parallel, and engines
> and hosts use other threads internally. One JS thread running synchronous code does
> not mean the entire runtime operates on one thread.

---

## 3. Quick Recall — Interview One-Liners

| Question | Crisp answer |
|---|---|
| **Why standardise?** | Netscape JavaScript and Microsoft's JScript needed shared rules to reduce vendor fragmentation. |
| **JavaScript vs ECMAScript?** | JavaScript is the common language name; ECMAScript is its formal standardised definition. |
| **Ecma vs TC39?** | Ecma is the standards organisation; TC39 is its committee responsible for ECMAScript. |
| **Spec vs engine vs runtime?** | Rules vs executor vs executor plus host APIs and scheduling. |
| **Is Node.js an engine?** | No: it is a runtime that embeds V8. |
| **Does ECMAScript require JIT?** | No. It specifies behaviour, not an execution pipeline. |
| **Single-threaded means no parallelism?** | No. Workers and host operations can run concurrently or in parallel. |
| **Zero-delay timer output above?** | `1, 3, 2`: the timer callback runs on a later turn. |
| **What makes JS portable?** | Shared language semantics; portability of host APIs must be checked separately. |
