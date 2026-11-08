# 001 — Introduction to JavaScript

> **One-line takeaway:** JavaScript is the language every browser speaks. It was hacked together in **10 days in 1995** and now runs on servers, phones, desktops and edge networks. The trick is that it is **one language with many hosts**: swap the host and the same JavaScript gains new powers.

---

## 1. What Is JavaScript?

> **JavaScript is a high-level, dynamically typed, garbage-collected, multi-paradigm
> programming language. It runs on a single thread with an event loop, and it is
> standardised as ECMAScript.**

It was born to make web pages interactive. Today it is a general-purpose language.

```text
JavaScript
      │
      ├── High-level          → no manual memory management (garbage collector)
      ├── Dynamically typed   → types live on VALUES, not variables
      ├── Weakly typed        → converts types implicitly ("coercion")
      ├── Multi-paradigm      → procedural + OOP (prototypes) + functional
      ├── JIT-compiled        → interpreted first, hot code compiled to machine code
      └── Single-threaded     → one call stack + an event loop for async work
```

### The headline traits, in practice

| Trait | What it means |
|---|---|
| **Dynamically typed** | `let x = 5; x = "hi";` is legal. Type errors show up at *runtime*. |
| **Weakly typed** | `"5" - 1 === 4` but `"5" + 1 === "51"`. The `+` operator prefers strings. |
| **Prototype-based OOP** | Objects inherit directly from other objects. `class` (ES2015) is syntax sugar over prototypes. |
| **First-class functions** | Functions are values: store them, pass them, return them. This gives you closures and callbacks. |
| **JIT-compiled** | Not "just interpreted". V8 runs bytecode in **Ignition**, then optimises hot paths with **TurboFan**. |
| **Single thread + event loop** | One thing runs at a time. Slow work (network, timers, disk) is handed off, and its callback is queued. |
| **Standardised** | The spec is **ECMAScript**, written by committee **TC39** and published by **Ecma International**. New edition every year since **ES2015 (ES6)**. |

### JavaScript ≠ Java

```text
Java        → statically typed, class-based, compiled to JVM bytecode
JavaScript  → dynamically typed, prototype-based, runs in a JS engine
```

> *"Java is to JavaScript as car is to carpet."* The shared name is pure marketing (§2).

---

## 2. The Core Model: Language + Engine + Host

This is the single most useful idea in the whole note.

```text
 ┌──────────────────────────── RUNTIME ─────────────────────────────┐
 │                                                                  │
 │   ┌────────── ENGINE ──────────┐    ┌──────── HOST APIs ───────┐ │
 │   │ V8 (Chrome, Node, Deno)    │    │ Browser: DOM, fetch,     │ │
 │   │ SpiderMonkey (Firefox)     │ +  │   setTimeout, events     │ │
 │   │ JavaScriptCore (Safari,Bun)│    │ Node: fs, http, process  │ │
 │   │   → runs the LANGUAGE      │    │   → gives it POWERS      │ │
 │   └────────────────────────────┘    └──────────────────────────┘ │
 │                single thread  +  event loop                      │
 └──────────────────────────────────────────────────────────────────┘
```

```text
Language  → variables, functions, objects, promises       (ECMAScript spec)
Engine    → parses, compiles and executes the language    (V8, SpiderMonkey…)
Host      → everything that touches the outside world     (browser, Node…)
```

> `setTimeout`, `document`, `fetch` and `fs` are **not part of JavaScript the language**.
> They are host APIs. Once you see this, "JS runs on servers, phones and desktops" stops
> sounding like magic. You only swap the host.

### The revealing example

```js
console.log("1");
setTimeout(() => console.log("2"), 0);   // 0 ms delay!
console.log("3");

// Output: 1  3  2
```

```text
1. log "1"                        → runs immediately on the call stack
2. setTimeout(cb, 0)              → engine hands cb to the HOST's timer
3. log "3"                        → stack keeps going, nothing waits
4. stack empty → event loop       → pushes cb back in → log "2"
```

> One snippet covers four ideas: **single thread, host APIs, the event loop, and
> non-blocking execution.** Even a 0 ms timer waits for the stack to clear.

---

## 3. Origin — Who, Why & How

### Who

**Brendan Eich**, at **Netscape Communications**, in **May 1995**.

### Why

```text
The web in 1995   → static HTML documents
Every check       → "is this field empty?" needed a full round trip to the server
Netscape wanted   → a lightweight "glue" language that designers and part-time
                    programmers could write straight inside HTML
Java's role       → heavy, compiled components
JS's role         → the easy-going scripting companion
```

### How

The first prototype was built in about **10 days**, to hit the Netscape Navigator 2.0
deadline. That rush explains many of the quirks we still live with.

It took its design DNA from three languages:

```text
                 JavaScript
                     │
     ┌───────────────┼────────────────┐
     ↓               ↓                ↓
    Java           Scheme            Self
  (syntax)    (first-class funcs,  (prototype-based
  curly braces    closures)          inheritance)
```

### Three names in one year

```text
Mocha  →  LiveScript  →  JavaScript  (Dec 1995)
                              ↑
              Netscape–Sun deal, timed to ride Java's hype
```

The "JavaScript" trademark now belongs to **Oracle** (which bought Sun). That's one reason
the standard is called **ECMAScript**.

### Timeline

| Year | Event |
|---|---|
| **1995** | Created at Netscape by Brendan Eich; ships in Navigator 2.0 |
| **1996** | Microsoft reverse-engineers it as **JScript** for IE3. Browser wars, "works only in one browser" pain |
| **1997** | Netscape hands it to Ecma → **ECMAScript 1**, to stop the fragmentation |
| **1999** | **ES3**, stable for a decade (the ambitious ES4 was abandoned) |
| **2005** | **AJAX** (Gmail, Google Maps): fetch data without reloading. Web *apps* are born |
| **2008** | Google's **V8** engine (Chrome) makes JS *fast* via JIT compilation |
| **2009** | **ES5** + **Node.js** (Ryan Dahl puts V8 on the server). JS escapes the browser |
| **2015** | **ES6 / ES2015**: `let`/`const`, arrows, classes, modules, promises. Modern JS starts here |
| **2016+** | Yearly releases: `async/await`, `?.`, `??`, and more |

> **Interview soundbite:** "Brendan Eich built JavaScript in 10 days at Netscape in 1995 to
> make web pages interactive. It took Java's syntax, Scheme's functions and Self's
> prototypes. It was standardised as ECMAScript in 1997, and V8 plus Node.js in 2008–09
> turned it from a browser toy into a full-stack language."

---

## 4. Advantages of JavaScript

### 🌍 Runs everywhere, nothing to install

Every browser on every device ships a JS engine. It is the **only** language browsers run
natively. (WebAssembly is a compile target that runs *alongside* JS, not a replacement.)

### 🔁 One language, full stack

```text
Frontend   → React / Vue / Angular
Backend    → Node.js / Deno / Bun
Mobile     → React Native
Desktop    → Electron
             ↓
Share validation logic, types and utilities across all of them
```

### 📦 Huge ecosystem

**npm** is the world's largest package registry. There is a library or framework for
almost anything.

### ⚡ Great for I/O-heavy work

The event loop handles thousands of concurrent connections (APIs, chat, streaming) without
a thread per request.

### 🚀 Fast enough for most things

JIT engines compile hot code paths to optimised machine code.

### 🧪 Instant feedback

Press F12, type into the DevTools console, see the result. No compiler, no setup.

### 🎨 Rich interactivity

Direct access to the DOM, events, animation, canvas, audio and video in the browser.

### 🔗 JSON is native

The web's data format *is* JavaScript object syntax, so frontend and backend exchange data
without effort.

### 🧱 Never breaks old code

New features arrive every year, and code from 1999 still runs.

---

## 5. Disadvantages — and How Engineers Handle Them

> Never list a weakness without its fix. That pairing shows engineering judgement, not
> memorisation.

| Weakness | Example | Mitigation |
|---|---|---|
| **Type coercion surprises** | `[] + []` → `""`, `"5" + 1` → `"51"`, `"5" - 1` → `4` | Always use `===`; ESLint |
| **Dynamic typing → runtime errors** | A `user.adress.city` typo crashes in production, not at build time | **TypeScript** |
| **Single-threaded for CPU work** | A heavy loop freezes the UI or blocks every Node request | Web Workers, `worker_threads`, offload to another service |
| **"Don't break the web" baggage** | `typeof null === "object"` is a 1995 bug that can **never** be fixed | Learn the quirks; strict mode; modern syntax |
| **Client-side code is public** | Anyone can read or modify your frontend JS | Never trust the client: validate on the server, keep secrets server-side |
| **Security surface (XSS)** | Injected scripts run with your page's privileges | Sanitise input, CSP headers, avoid `innerHTML` with user data |
| **Number precision** | `0.1 + 0.2 !== 0.3` (every number is a 64-bit float) | Integers (paise/cents), `BigInt`, decimal libraries |
| **Ecosystem churn & dependency risk** | `left-pad` (2016): an 11-line package removed, thousands of builds broke | Lockfiles, `npm audit`, fewer dependencies |
| **Browser inconsistencies** (historical) | The same API behaving differently across browsers | Standards, Babel, polyfills |

### ⚠️ Why the quirks can't be fixed

```text
Billions of pages depend on today's behaviour
        ↓
Changing it would break them
        ↓
Rule of the web: "DON'T BREAK THE WEB"
        ↓
Mistakes are permanent; features are only ever ADDED
  (e.g. === added alongside ==, let/const alongside var)
```

---

## 6. Applications — Where JavaScript Runs

> **Atwood's Law (2007):** *"Any application that can be written in JavaScript, will
> eventually be written in JavaScript."*

Same language, different host:

| Domain | Host / Tooling | Real examples |
|---|---|---|
| **Web frontend** | Browser + React, Angular, Vue, Svelte, Next.js | Gmail, Facebook, Netflix UI |
| **Web servers / APIs** | **Node.js** (also Deno, Bun) + Express, NestJS, Fastify | LinkedIn, PayPal, parts of Netflix |
| **Mobile apps** | **React Native**, Ionic / Capacitor, NativeScript | Instagram, Shopify, Discord (parts) |
| **Desktop apps** | **Electron** (Chromium + Node bundled) | **VS Code**, Slack, Discord, Figma desktop |
| **Serverless / edge** | AWS Lambda, Cloudflare Workers, Vercel Functions | APIs that scale to zero |
| **Real-time** | WebSockets, Socket.IO | Chat, multiplayer, live dashboards |
| **Games & 3D** | Canvas, WebGL, Three.js, Phaser, Babylon.js | Browser games, 3D product viewers |
| **Machine learning** | TensorFlow.js, ONNX Runtime Web | In-browser face detection |
| **CLI & dev tools** | Node.js | npm, ESLint, Prettier, webpack, Vite |
| **Browser extensions** | Chrome / Firefox extension APIs | Ad blockers, Grammarly |
| **IoT & hardware** | Johnny-Five, Espruino | Robots, microcontrollers |
| **Databases** | MongoDB shell, CouchDB views | Queries and map-reduce in JS |

### Electron vs React Native

```text
Electron      → bundles a whole Chromium browser + Node.js
                your app IS a web page with OS access
                easy, but memory-heavy                       (desktop)

React Native  → JS runs in an engine (Hermes) and drives
                REAL native UI widgets, not a WebView        (mobile)
```

> Both exist for the same business reason: **one team, one language, every platform.**

---

## 7. Quick Recall — Interview One-Liners

| Question | Crisp answer |
|---|---|
| **What is JavaScript?** | A high-level, dynamically typed, garbage-collected, multi-paradigm language that runs on a single thread with an event loop; standardised as ECMAScript. |
| **JavaScript vs ECMAScript?** | ECMAScript is the *specification* (TC39 / Ecma International); JavaScript is the language that *implements* it, plus host APIs. |
| **Interpreted or compiled?** | JIT-compiled: engines interpret bytecode first, then compile hot code to optimised machine code (and deoptimise if assumptions break). |
| **Who created it, when, and why?** | Brendan Eich at Netscape, May 1995, in ~10 days, as a lightweight glue language to make web pages interactive. |
| **Why is it called JavaScript?** | Marketing: a Netscape–Sun deal renamed LiveScript to ride Java's hype. Only the syntax resembles Java. |
| **Its three influences?** | Java → syntax; Scheme → first-class functions; Self → prototypes. |
| **Output of `log 1; setTimeout(log 2, 0); log 3`?** | `1, 3, 2`. The callback waits in the queue until the call stack is empty. |
| **Single-threaded, so how does Node serve 10,000 requests?** | I/O is delegated to the OS / libuv; JS only runs the callbacks. The thread never sits waiting. (CPU-heavy work *does* block it.) |
| **Is `setTimeout` part of JavaScript?** | No. It is a host API (browser / Node). The language itself has no timers, DOM or file system. |
| **Main advantages?** | Runs everywhere, one language full-stack, npm ecosystem, efficient I/O concurrency, fast JIT, instant feedback. |
| **Main disadvantages?** | Coercion quirks, runtime type errors, single thread for CPU work, public client code, permanent legacy quirks. |
| **Why can't `typeof null === "object"` be fixed?** | Backward compatibility: "don't break the web". Existing pages depend on it. |
| **Electron vs React Native?** | Electron bundles Chromium + Node (desktop, web UI). React Native drives real native widgets from JS (mobile). |

---

## 8. The Mental Model

```text
                     ONE LANGUAGE
                (ECMAScript, yearly spec)
                          │
                          ↓
                       ENGINE
           (V8 / SpiderMonkey / JavaScriptCore)
        JIT-compiles · single thread · event loop
                          │
     ┌──────────┬─────────┼──────────┬───────────┐
     ↓          ↓         ↓          ↓           ↓
  Browser     Node.js   Electron  React Native  Edge
  DOM, fetch  fs, http  desktop   native UI     Workers
     │          │         │          │           │
  frontend   backend   VS Code,   Instagram   serverless
                        Slack
```

> **Built in 10 days, never allowed to break, and now running everywhere.** Learn it once
> and you can build for every screen. Welcome to JavaScript.
