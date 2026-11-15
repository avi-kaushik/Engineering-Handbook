# 004 — Collection Traversal & Iteration

> **One-line takeaway:** **Keys, values and callback results are different contracts.**
> Enumerate properties with `for...in`, consume iterable values with `for...of`, and use
> array methods to express transformations, searches and aggregation.

---

## 1. Core Mental Model — What Are You Traversing?

Choose the data contract first; [003 — Loops & Control Flow](003-loops-and-control-flow.md)
covers manual repetition. `for...of` obtains an iterator and consumes its values; the
mechanism and cleanup belong to [005 — Iteration Protocols](005-iterable-iterator-protocols-and-generators.md).

| Need | Prefer | Why |
|---|---|---|
| Own object properties | `Object.keys/values/entries` + traversal | Explicit ownership boundary |
| Enumerable keys across prototypes | `for...in` | Property enumeration is the intended contract |
| Iterable values with direct control | `for...of` | `break`, `continue`, function `return`, awaited body work |
| Transform / select | `map` / `filter` | Produce a new result array |
| Search / test | `find` / `some` / `every` | Stop once the answer is known |
| Fold values into an accumulator | `reduce` | Express an aggregation, with an explicit seed |

## 2. Property Enumeration vs Value Iteration

| Distinction | `for...in` | `for...of` |
|---|---|---|
| **Produces** | Enumerable **string keys**, including inherited ones | Iterator-produced values |
| **Arrays** | Present index keys as strings, extra properties, inherited enumerable keys | Default iterator reads element indexes, including holes |
| **Plain objects** | No iterator needed | Not iterable by default; throws without a supplied iterator |
| **Order** | For ordinary stable objects: per prototype level, index keys ascending then other strings in creation order | Defined by the iterator |

### `for...in`: property names, including inherited ones

**Enumerable** is a property flag, not a synonym for “exists”. Ordinary assignments create
enumerable properties; `for...in` collects eligible string keys across the prototype chain.

```js
const settings = Object.assign(Object.create({ region: "IN" }), {
  theme: "dark", pageSize: 20,
});
const all = [], own = [];
for (const key in settings) {
  all.push(key);
  if (Object.hasOwn(settings, key)) own.push(key);
}
console.log(all); // ["theme", "pageSize", "region"]
console.log(own); // ["theme", "pageSize"]
```

`region` belongs to the prototype; property lookup can find it, but `Object.hasOwn` rejects
it. Use `for...in` when inherited enumerable properties are deliberately part of the data.
For ordinary records/configuration, own-property traversal usually states the intent better.

**Array trap:** `for...in` produces string indexes, skips absent holes, and may include
extra named/inherited properties. It enumerates the array object's properties, not its
element sequence. Prefer `for...of` for values.

Both `for...in` and the Object methods below omit symbol keys and non-enumerable properties.
A shadowed key is not repeated; even a non-enumerable own property can hide an enumerable
prototype property (challenge A). See [MDN: for...in](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...in)
and [the Symbol recap](001-variables-and-data-types.md#symbol--keys-without-accidental-collisions).

### Object methods: choose keys, values or pairs

All three return **arrays derived from own enumerable string-keyed properties**. Using
the same `settings` object above:

```js
console.log(Object.keys(settings));   // ["theme", "pageSize"]
console.log(Object.values(settings)); // ["dark", 20]
console.log(Object.entries(settings));// [["theme", "dark"], ["pageSize", 20]]
for (const [key, value] of Object.entries(settings)) {
  console.log(`${key}=${value}`); // theme=dark; pageSize=20
}
```

`entries` is useful for serialising/displaying a record: destructuring names each pair's
parts, and inheritance is excluded without a guard inside the loop. Choose `keys` if only
names matter, `values` if only data matters. Order follows own-property ordering: index keys
ascending, then other strings in creation order. Avoid structural mutation during
enumeration; do not assume new keys will be visited.
Reference: [MDN: Object.entries](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/entries).

### `for...of`: values selected by an iterator

`for...of` calls the iterable's **`[Symbol.iterator]()`**. That contract determines values
and order, rather than the object's enumerable properties. Plain objects such as `settings`
have no default iterator, so `for (const value of settings)` throws `TypeError`; converting
to entries supplies an iterable array. Internal stepping and cleanup stay in note 005.

| Iterable | Default values / order |
|---|---|
| **Array** | Elements in index order; extra named properties are ignored. |
| **String** | Unicode code points, not necessarily visible grapheme clusters; indexing uses UTF-16 code units. |
| **Map** | `[key, value]` pairs in insertion order; use `.keys()` / `.values()` for those views. |
| **Set** | Values in insertion order. |

```js
for (const n of [10, 20]) console.log(n);           // 10; 20
for (const ch of "A😀") console.log(ch);           // "A"; "😀"
for (const n of new Set([2, 2, 3])) console.log(n); // 2; 3
const scores = new Map([["Avi", 9], ["Sam", 8]]);
for (const [name, score] of scores) console.log(name, score);
// Avi 9; Sam 8 — destructure each entry, not the Map object
```

Use this form for direct value processing, especially when early exit or sequential
awaited work matters. See [MDN: for...of](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...of).

**Iterable ≠ array-like:** `{ 0: "a", length: 1 }` has indexed shape but no iterator.
`Array.from` accepts either contract; iterable spread requires an iterator. Category checks
are summarised in [Variables & Data Types](001-variables-and-data-types.md#specialised-objects-are-still-the-object-language-type).

## 3. Array Callback Contracts

Pick the **role of the callback result**, not just a familiar method name. Effects act on
the outside world; transformations produce values; predicates decide whether a value qualifies.

| Method | Callback meaning → return value | Early finish | Absent indexes* |
|---|---|---|---|
| **`forEach`** | Effects; callback result ignored → `undefined` | No normal early exit | Skipped |
| **`map`** | Result becomes element → new array | No | Skipped; output keeps holes |
| **`filter`** | Truthy result keeps original element → new array | No | Skipped; selected elements packed |
| **`reduce`** | Return next accumulator → final accumulator | No | Skipped |
| **`find`** | Truthy result selects → first matching value, otherwise `undefined` | First match | Read as `undefined` |
| **`some`** | Any truthy predicate result → boolean | First truthy result | Skipped |
| **`every`** | All predicate results truthy → boolean | First falsy result | Skipped |

**Sparse-array caveat:** assumes no inherited indexed properties. Presence-testing methods can visit inherited
indexes. A stored `undefined` is present, unlike a hole. `find` reads every index until it
matches, including holes. See [MDN: iterative methods](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array#iterative_methods)
and [find](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/find).

Callbacks run synchronously with `(value, index, array)`; `reduce` uses
`(accumulator, value, index, array)`. `return` exits one callback, not the traversal.
Use `for...of` for arbitrary `break`/`continue`; throwing is not a normal search strategy.

### Same input, different intentions

```js
const nums = [1, 2, 3];
const seen = [];
console.log(nums.forEach(n => seen.push(n * 2))); // undefined: effects only
console.log(seen);                              // [2, 4, 6]
console.log(nums.map(n => n * 2));               // [2, 4, 6]: transformation
console.log(nums.filter(n => n > 1));            // [2, 3]: selection
console.log(nums.find(n => n > 1));              // 2: first match
console.log(nums.some(n => n > 2));              // true: any match?
console.log(nums.every(n => n > 1));             // false: all match?
console.log(nums.reduce((sum, n) => sum + n, 0)); // 6: aggregation
```

`map` stores each returned value; `filter` tests the return value but keeps the **original
element**. `forEach` discards it (including the length returned by `push`). `reduce` carries
each callback result into the next call. Choose the method that names the result you need;
choose `for...of` when custom branching or control flow is clearer than a callback.

### Results worth remembering

- `[].some(predicate)` → **false**; `[].every(predicate)` → **true**: no element disproves
  the universal condition. Neither calls the predicate.
- Without a seed, `reduce` uses the first present element; an empty/all-hole array throws
  `TypeError`. An explicit seed defines empty-input behaviour and the initial accumulator.
- `map`/`filter` produce new arrays, **not deep copies**. Shared object elements remain
  shared; prefer the existing [copying recap](001-variables-and-data-types.md#copying-boundaries).

References: [reduce](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce),
[some](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/some),
[every](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/every).

## 4. Mutation & Async Callback Pitfalls

**Array iterators read live length**: appended elements may be visited. These callback
methods capture starting length, but later reads see mutations. Presence-testing methods
skip deleted indexes; `find` still reads them. Neither contract gives a frozen snapshot.

**Promises are callback results, not automatic scheduling instructions:**

- `forEach(async ...)` ignores returned promises: no completion wait or rejection aggregation.
- `map(async ...)` returns promises; use `Promise.all` to aggregate them.
- `filter`/`find`/`some`/`every` with async predicates test a **truthy Promise object**,
  not its eventual boolean. They are not async-aware predicates.

For sequential loops, concurrent starts and async sequences, see
[005 — Async Work](005-iterable-iterator-protocols-and-generators.md#6-sequential-vs-concurrent-work).

---

## 5. Active Recall — Predict Before Expanding

### A. Own, inherited — or hidden?

```js
const user = Object.create({ role: "member", region: "IN" });
user.name = "Avi";
Object.defineProperty(user, "role", { value: "admin" });
const keys = [];
for (const key in user) keys.push(key);
console.log(keys, Object.keys(user));
console.log(user.role, Object.hasOwn(user, "role"));
```

<details>
<summary>Show answer & reason</summary>

`["name", "region"]`, `["name"]`; then `"admin" true`. `defineProperty` defaults
`enumerable` to false. The own `role` exists and shadows the inherited `role`, but neither
is enumerated here. Only `for...in` includes inherited `region`.

</details>

### B. Empty slots vs empty collections

```js
const holes = Array(2);
console.log(holes.some(() => true), holes.every(() => false));
let visits = 0;
holes.find(() => { visits++; return false; });
console.log(visits, holes.reduce((sum, n) => sum + n, 10));
```

<details>
<summary>Show answer & reason</summary>

`false true`; `2 10`. `some`/`every` skip both absent indexes; `find` visits them.
Seeded `reduce` has no present element to process and returns its seed. Without a seed
it would throw, despite the array's nonzero length.

</details>

### C. Async callbacks vs traversal completion

```js
const seen = [];
[1, 2].forEach(async n => {
  await Promise.resolve();
  seen.push(n);
});
console.log(seen.length);
Promise.resolve().then(() => console.log(seen.join(",")));
```

<details>
<summary>Show answer & reason</summary>

`0`, then `"1,2"`. Traversal returns before the callbacks resume; those queued
continuations run before the later `.then`. Make completion explicit with an awaited loop
or aggregated promises, rather than relying on this ordering.

</details>

**Previous:** [003 — Loops & Control Flow](003-loops-and-control-flow.md).
**Next:** [005 — Iterable & Iterator Protocols, Generators](005-iterable-iterator-protocols-and-generators.md).

---

## 6. Quick Recall — Interview One-Liners

| Question | Crisp answer |
|---|---|
| **`for...in` vs `for...of`?** | Enumerable string keys including inherited ones vs iterator-produced values. |
| **Why avoid `for...in` for array values?** | It yields string keys and can include extra/inherited properties, while skipping absent holes. |
| **How traverse own object properties?** | Object.keys/values/entries return arrays of own enumerable string-keyed properties. |
| **What does a Map yield?** | Entry pairs; destructure `[key, value]`, or choose keys()/values(). |
| **`map` vs `filter` vs `reduce`?** | Transform elements vs select original elements vs fold into an accumulator. |
| **Which callbacks short-circuit?** | find on a match, some on truthy, every on falsy; forEach/map/filter/reduce do not. |
| **Sparse-array exception?** | find reads holes; the other listed callback methods skip absent indexes, and map preserves holes. |
| **Empty-input traps?** | some → false, every → true; reduce without a seed and without present elements throws. |
| **Async callback misconception?** | Methods do not await promises; async predicates return truthy objects, not resolved booleans. |
| **Is traversal a snapshot?** | No: callbacks fix initial length but observe later reads; array iterators observe live length too. |
