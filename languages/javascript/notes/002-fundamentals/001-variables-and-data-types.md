# 001 — Variables & Data Types

> **One-line takeaway:** A variable is a **name tag with an arrow** pointing at a value. JavaScript has **8 types**: 7 **primitives** (simple, unchangeable, *photocopied* when you copy them) and **objects** (changeable, *shared like a Google Doc link*).

> 🧪 **How to use this note:** Whenever you see **Predict the output**, stop and answer in
> your head *before* opening the answer. Open DevTools (F12 → Console) and run the snippets
> yourself. You remember best what you get wrong first.

---

## 1. What Is a Variable?

> **A variable is a name that points to a value, so your code can use that value later.**

### ❌ The "box" picture (misleading)

```text
┌─────┐
│  5  │  a       "a is a box with 5 inside"
└─────┘
```

### ✅ The "arrow" picture (accurate)

```text
  a  ───────►  5          "a is a name tag; its arrow points at the value 5"
```

Why does it matter? Watch what happens with objects:

```js
const user  = { name: "Avi" };
const admin = user;              // copy the variable…
admin.name  = "Sam";
console.log(user.name);          // ???
```

```text
  user  ──┐
          ├──►  { name: "Sam" }     ONE object, TWO arrows
  admin ──┘
```

`admin = user` copied the **arrow**, not the object. Both names point at the same thing,
so the answer is **`"Sam"`**. In the box picture this makes no sense. In the arrow picture
it is obvious.

> 🧠 **Remember:** *Arrows, not boxes.* Almost every "weird" JavaScript bug about copying,
> `const` or `===` disappears once you think this way.

### Declaring a variable

```js
const name = "Avi";   // can't point the arrow elsewhere later → your default
let   age  = 25;      // can point the arrow elsewhere later
var   old  = true;    // the 1995 way → avoid in new code
```

> The full story of `var`, `let` and `const` (scope, hoisting, the TDZ and the famous loop
> question) has its own note: [002 — var, let & const](002-var-let-const.md).

### Naming rules

```text
✅ Can start with   a letter, $ or _          userName, $el, _id
❌ Cannot start with a digit                   2fast
❌ Cannot be         a reserved word            class, return, let
⚠️ Case-sensitive    age and Age are DIFFERENT variables
```

| Convention | Used for | Example |
|---|---|---|
| `camelCase` | variables & functions | `userName`, `getTotal` |
| `UPPER_SNAKE_CASE` | fixed config constants | `MAX_RETRIES`, `API_URL` |
| `PascalCase` | classes & React components | `UserProfile` |
| `is` / `has` / `can` prefix | booleans | `isLoggedIn`, `hasAccess` |

---

## 2. Data Types: The 8 Types

> **In JavaScript, types belong to VALUES, not to variables.** A variable can point to a
> number now and a string later. That's what *dynamically typed* means.

```js
let v = 42;          // v → a number
v = "forty-two";     // v → a string   (totally legal)
```

### The family tree

```text
                        JavaScript types (8)
                                │
           ┌────────────────────┴────────────────────┐
           ↓                                         ↓
     PRIMITIVES (7)                             OBJECT (1)
  simple · immutable · photocopied     complex · mutable · shared by link
           │                                         │
  string   number   bigint                {} plain objects   [] arrays
  boolean  undefined  null                functions          Date, Map, Set
  symbol                                  RegExp, Promise, Error …
```

> 🧠 **Memory hook for the 7 primitives:** **"S**even **N**ice **B**unnies **B**ounce
> **U**nder **N**ew **S**ky" → **S**tring, **N**umber, **B**igint, **B**oolean,
> **U**ndefined, **N**ull, **S**ymbol. Everything else is an **object**, including arrays
> and functions.

### Meet the primitives

| Type | Example | In one line |
|---|---|---|
| **string** | `"hi"`, `'hi'`, `` `Hi ${name}` `` | Text. Backticks allow `${}` and multiple lines |
| **number** | `42`, `3.14`, `-7`, `NaN`, `Infinity` | **One** type for ints *and* decimals (64-bit float) |
| **bigint** | `9007199254740993n` | Huge whole numbers; note the `n` at the end |
| **boolean** | `true`, `false` | Yes / no |
| **undefined** | `let x;` | "Nothing has been put here **yet**" |
| **null** | `let x = null;` | "I **deliberately** set this to empty" |
| **symbol** | `Symbol("id")` | A guaranteed-unique value, used as hidden object keys |

### `undefined` vs `null`: the parking spot analogy

```text
undefined → an empty parking spot nobody has used yet
            (the ENGINE's default: unset variable, missing property,
             missing argument, function with no return)

null      → a spot with a "RESERVED, intentionally empty" cone
            (YOU put it there on purpose)
```

> 🧪 **Predict the output**
> ```js
> let a;
> const obj = {};
> function f() {}
> console.log(a, obj.color, f());
> ```

<details>
<summary>Show answer</summary>

`undefined undefined undefined`: an unassigned variable, a missing property and a function
with no `return` all give `undefined`. You never get `null` unless someone wrote it.
</details>

### Number: one type, a few surprises

> 🧪 **Predict the output** of each line before opening:
> ```js
> 0.1 + 0.2 === 0.3
> typeof NaN
> NaN === NaN
> 1 / 0
> 9007199254740992 === 9007199254740993
> ```

<details>
<summary>Show answers</summary>

| Code | Result | Why |
|---|---|---|
| `0.1 + 0.2 === 0.3` | `false` | Binary floats can't store 0.1 exactly → `0.30000000000000004` |
| `typeof NaN` | `"number"` | "Not-a-Number" is a special *number* value |
| `NaN === NaN` | `false` | NaN equals nothing, not even itself → use `Number.isNaN(x)` |
| `1 / 0` | `Infinity` | No crash, just a special number |
| `2⁵³ === 2⁵³ + 1` | `true` | Past `Number.MAX_SAFE_INTEGER` (2⁵³ − 1), precision is lost → use `bigint` |
</details>

> 💼 **Real-world rule:** never store money as `19.99`. Store **paise/cents** as integers
> (`1999`) to dodge float errors.

### Primitives can't be changed, only replaced

```js
let s = "hello";
s[0] = "H";            // silently ignored
console.log(s);        // "hello"

s = s.toUpperCase();   // makes a NEW string and moves the arrow to it
console.log(s);        // "HELLO"
```

```text
before:  s ──► "hello"
after:   s ──► "HELLO"        "hello" itself was never modified
```

> 🤔 *But wait, how does `"hi".length` work if a string isn't an object?* JS briefly wraps
> it in a temporary `String` object to find the property, then throws the wrapper away.
> This is called **autoboxing**.

---

## 3. Checking Types with `typeof`

`typeof` tells you a value's type as a string. Mostly reliable, with **three traps**:

| Expression | Result | |
|---|---|---|
| `typeof "a"` | `"string"` | ✅ |
| `typeof 1` | `"number"` | ✅ |
| `typeof 1n` | `"bigint"` | ✅ |
| `typeof true` | `"boolean"` | ✅ |
| `typeof undefined` | `"undefined"` | ✅ |
| `typeof Symbol()` | `"symbol"` | ✅ |
| `typeof {}` | `"object"` | ✅ |
| `typeof function(){}` | `"function"` | ✅ functions get their own label |
| `typeof null` | `"object"` | ⚠️ **Trap 1**: legacy behaviour preserved for compatibility ([origins](../001-getting-started/002-origins-netscape-sun-and-the-10-day-prototype.md#4-four-quirks-worth-explaining)) → check `x === null` |
| `typeof []` | `"object"` | ⚠️ **Trap 2**: arrays are objects → use `Array.isArray(x)` |
| `typeof NaN` | `"number"` | ⚠️ **Trap 3** → use `Number.isNaN(x)` |

---

## 4. Primitive vs Reference: Photocopy vs Shared Link

This is the **most important idea** in the note.

```text
PRIMITIVE  = a PHOTOCOPY
             Copy it, scribble on your copy → the original is untouched.

OBJECT     = a SHARED GOOGLE DOC LINK
             Copy the link, edit the doc → everyone with the link sees the change.
```

### See it in code

```js
// 📄 Primitive: photocopy
let a = 10;
let b = a;       // b gets its own copy of 10
b = 20;
console.log(a);  // 10 ✅ untouched

// 🔗 Object: shared link
const p1 = { name: "Avi" };
const p2 = p1;   // p2 gets a copy of the LINK
p2.name = "Sam";
console.log(p1.name);  // "Sam" ⚠️ same document
```

```text
Primitives:   a ──► 10          b ──► 20          two separate values

Objects:      p1 ──┐
                   ├──►  { name: "Sam" }          one object, two links
              p2 ──┘
```

### `===` compares links, not contents

> 🧪 **Predict the output**
> ```js
> console.log("ab" === "ab");
> console.log({} === {});
> console.log([1, 2] === [1, 2]);
> const o = {};
> console.log(o === o);
> ```

<details>
<summary>Show answer</summary>

`true`, `false`, `false`, `true`.

Primitives compare by **value**. Objects compare by **identity**: "are these two arrows
pointing at the *same* object?" Two separately written `{}` are two different objects,
even if they look identical.
</details>

### Making a real copy

```text
Shallow copy   { ...obj }   [...arr]   Object.assign({}, obj)
               → top level copied, NESTED objects still shared

Deep copy      structuredClone(obj)
               → everything copied (except functions)
```

> 🧪 **Predict the output**
> ```js
> const a = { name: "Avi", address: { city: "Delhi" } };
> const b = { ...a };
> b.name = "Sam";
> b.address.city = "Pune";
> console.log(a.name, a.address.city);
> ```

<details>
<summary>Show answer</summary>

`"Avi" "Pune"`. The spread copied the top level (so `name` is separate) but
`address` is a nested object, so only its **link** was copied. Both `a` and `b` share
the same address object. Use `structuredClone(a)` for a full copy.
</details>

### Passing to functions: "call by sharing"

JavaScript **always passes a copy**. For an object, the copy is a copy of the *link*:

```js
function rename(u)  { u.name = "X"; }        // edits the shared doc → caller sees it ✅
function replace(u) { u = { name: "Y" }; }   // only repoints the local link → caller unaffected ❌
```

---

## 5. Truthy & Falsy

In an `if`, `&&`, `||` or `!`, every value is treated as either true-ish or false-ish.

### The 8 falsy values (the complete list)

```text
false    0    -0    0n    ""    null    undefined    NaN
```

**Everything else is truthy.**

> 🧪 **Truthy or falsy?** Decide for each: `"0"`, `"false"`, `" "`, `[]`, `{}`, `-1`

<details>
<summary>Show answer</summary>

**All truthy.** `"0"`, `"false"` and `" "` are non-empty strings. `[]` and `{}` are
objects, and *every* object is truthy, even an empty one. `-1` is a non-zero number.
</details>

### `||` vs `??`: a bug you will actually meet

```js
const itemsInCart = 0;

itemsInCart || 10    // 10 ❌  0 is falsy, so a REAL value is thrown away
itemsInCart ?? 10    // 0  ✅  ?? only falls back on null / undefined
```

> 💼 Use `??` for defaults whenever `0`, `""` or `false` are valid values.

---

## 6. Quick Recall — Interview One-Liners

| Question | Crisp answer |
|---|---|
| **What is a variable?** | A name (binding) that points to a value: an arrow, not a box. |
| **How many data types?** | 8: string, number, bigint, boolean, undefined, null, symbol (primitives) + object. |
| **Are arrays and functions separate types?** | No, both are objects. `typeof []` is `"object"`; `typeof fn` is `"function"`. |
| **What does dynamically typed mean?** | Types belong to values, so one variable can hold different types over time. |
| **`null` vs `undefined`?** | `undefined` = nothing assigned yet (engine). `null` = intentionally empty (programmer). `null == undefined` but not `===`. |
| **Why is `typeof null` `"object"`?** | A bug from the first implementation, kept for backward compatibility. |
| **How to check for an array? For NaN?** | `Array.isArray(x)`; `Number.isNaN(x)`. |
| **Primitive vs reference?** | Primitives are immutable and copied by value (photocopy). Objects are mutable and copying copies the reference (shared link). |
| **Why is `{} === {}` false?** | Objects compare by identity, and those are two different objects. |
| **Shallow vs deep copy?** | Shallow (`{...obj}`) copies the top level; nested objects stay shared. Deep (`structuredClone`) copies everything. |
| **Is JS pass-by-reference?** | No. Always pass-by-value; for objects the value is a reference ("call by sharing"). |
| **Why `0.1 + 0.2 !== 0.3`?** | All numbers are 64-bit floats; 0.1 can't be represented exactly in binary. |
| **Falsy values?** | `false, 0, -0, 0n, "", null, undefined, NaN`. Everything else, including `[]`, `{}` and `"0"`, is truthy. |
| **`||` vs `??`?** | `||` falls back on any falsy value; `??` only on `null`/`undefined`. |

---

## 7. The Mental Model

```text
                     NAME  ──── arrow ────►  VALUE
                                               │
                 ┌─────────────────────────────┴───────────────┐
                 ↓                                             ↓
            PRIMITIVE  📄                                  OBJECT  🔗
   string number bigint boolean                   {}, [], functions,
   undefined null symbol                          Date, Map, Set …
                 │                                             │
         immutable (replace, never edit)          mutable (edit in place)
         copy  = photocopy                        copy  = share the link
         ===   compares VALUE                     ===   compares IDENTITY
```

> **Arrows, not boxes. Photocopies vs shared links.** Hold onto those two images and the
> rest of this note rebuilds itself.

➡️ **Next:** [002 — var, let & const](002-var-let-const.md)
