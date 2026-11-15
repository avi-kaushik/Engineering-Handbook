# 001 — Variables & Data Types

> **One-line takeaway:** A binding names a value. **Primitives are immutable; objects have
> identity.** Assignment copies a value, so copying an object reference shares the object.

---

## 1. Core Mental Model — Binding ≠ Value

A **binding** associates an identifier with a value within a scope. With dynamic typing,
values have types; a reassignable binding can hold different types over time.

```js
let result = 42;
result = "ready"; // valid: the binding now holds a string
```

```text
Binding → value
          ├── Primitive: immutable value
          └── Object: identity that multiple bindings can share
```

**Reassignment changes a binding; mutation changes object state.** `const` prevents the
former; see
[002 — var, let & const](002-var-let-const.md).

---

## 2. Key Concepts — Seven Primitives + Objects

| Type | Example | Recall point |
|---|---|---|
| **string** | `"hi"`, `"😀"` | Immutable UTF-16 text: `.length` counts code units, not necessarily characters. |
| **number** | `42`, `3.14`, `NaN`, `Infinity` | IEEE 754 binary64; integers and fractions share one type. |
| **bigint** | `9007199254740993n` | Exact integers; see the Number/BigInt comparison below. |
| **boolean** | `true`, `false` | Primitives, unlike wrapper objects: `new Boolean(false)` is truthy. |
| **undefined** | `undefined` | Often an absent value; can also be explicitly assigned. |
| **null** | `null` | Common convention for intentional absence; a distinct primitive. |
| **symbol** | `Symbol("id")` | Each `Symbol()` call creates a unique value; symbol keys are discoverable, not private. |
| **object** | `{}`, `[]`, functions, `Map` | Objects have identity and can be mutable; arrays and functions are not separate language types. |

Reference: [MDN type overview](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Data_structures).

### Symbol — keys without accidental collisions

Symbols give independent code non-colliding property keys. **Descriptions label; they
do not establish identity.**

| Form | Meaning |
|---|---|
| `Symbol("id")` | Fresh unique primitive on every call. |
| `Symbol.for("app.id")` | Creates/retrieves a symbol in the shared registry; repeated keys share identity. |
| `Symbol.iterator` | Well-known key for the iterator-producing method; used by `for...of` and iterable spread (`[...value]`). |
| `Symbol.toPrimitive` | Well-known key for custom object-to-primitive conversion; must return a primitive. |

`Object.keys` and `for...in` omit symbol keys; `Object.getOwnPropertySymbols` and
`Reflect.ownKeys` discover them. **Collision avoidance is not privacy.** Enumerable symbol
properties are copied by object spread. See [MDN: Symbol](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol);
challenge C applies this to plugin metadata.

For iterator consumption, see [005 — Iteration Protocols](005-iterable-iterator-protocols-and-generators.md#2-iterable-vs-iterator).

### `undefined` vs `null` vs an undeclared name

| Situation | Result / meaning |
|---|---|
| `let x;`, a missing argument, a function returning no value | `undefined` |
| Missing ordinary property, e.g. `({}).missing` | `undefined`; an existing property can also hold `undefined` |
| `const x = null` | Explicit null value; APIs can return null too. |
| Reading a name with no binding | `ReferenceError`, not an undefined value |

Both are **nullish**: `null == undefined`, but `null !== undefined`. Use
`Object.hasOwn(obj, key)` to distinguish an absent own property from one set to `undefined`.

---

## 3. Checking Types with `typeof`

For non-null primitives, `typeof` returns the lowercase type name; objects need finer checks.

| Expression | Result | Use instead / implication |
|---|---|---|
| `typeof null` | `"object"` | Legacy behaviour; test `value === null`. Null remains a primitive. |
| `typeof []` | `"object"` | Use `Array.isArray(value)`. |
| `typeof function () {}` | `"function"` | Functions are callable objects, not a ninth language type. |
| `typeof NaN` | `"number"` | Use `Number.isNaN(value)` to detect NaN without coercion. |
| `typeof absentName` (no binding) | `"undefined"` | Special allowance for an unresolvable identifier. |
| `typeof x` before `let x` in its scope | Throws `ReferenceError` | A TDZ binding exists but is uninitialised. |

See [MDN: `typeof`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/typeof)
and the [TDZ explanation](002-var-let-const.md#3-hoisting-initialisation--the-tdz).

### Specialised objects are still the Object language type

Built-ins add behaviours/internal state, not extra language types. Use category checks:

| Category | Primary purpose | Usual check for expected values |
|---|---|---|
| **Array** | Indexed collection | `Array.isArray(value)` |
| **Function** | Callable behaviour | `typeof value === "function"` |
| **Map / Set** | Key–value entries / unique values | `value instanceof Map` / `Set` |
| **Date** | Time value | `value instanceof Date`; invalid dates still pass. |
| **Promise** | Eventual fulfilment/rejection | `value instanceof Promise` for local instances; thenables are broader. |
| **RegExp** | Text pattern matching | `value instanceof RegExp` |

`instanceof` checks prototype relationships; it can be customised and fail across realms
(iframes). `Array.isArray` works across realms. Error, typed arrays, etc. are objects too.
See [MDN: instanceof limitations](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/instanceof#instanceof_and_multiple_realms).

---

## 4. Critical Distinctions — Values, Identity & Copying

| Operation | Primitive | Object |
|---|---|---|
| **Assignment / argument passing** | Copies the primitive value. | Copies a reference to the same object. |
| **Mutation** | Cannot mutate the value; a binding may be reassigned. | Can change state if the object permits it. |
| **Strict equality** | Compares without coercion; watch NaN and signed zero. | Tests identity, not structural contents. |

JavaScript is **pass-by-value** (*call by sharing* for objects): a callee can mutate a
shared object, but rebinding its parameter cannot rebind the caller's variable.

`"hi".length` uses wrapper semantics; it does not make strings mutable. A string index/property
write is ignored in sloppy mode and throws `TypeError` in strict mode. See
[MDN: primitive writes](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode#setting_properties_on_primitive_values).

### Copying boundaries

| Operation | What is shared? |
|---|---|
| `const b = a` | The entire object; `a === b`. |
| `{ ...a }`, `[...a]`, `Object.assign({}, a)` | A new outer container; nested object references remain shared. |
| `structuredClone(a)` | Deep-copies supported data, preserving cycles and internal sharing. |

`structuredClone` is a **host API** with [limitations](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Structured_clone_algorithm):
functions, DOM nodes and symbol values cannot be cloned. Custom prototypes, accessor
descriptors and class private elements are not preserved.

---

## 5. Edge Cases — Numbers & Truthiness

### Number vs BigInt

| Property | Number | BigInt |
|---|---|---|
| **Representation** | IEEE 754 binary64: integers and fractions | Arbitrary-precision integers, within implementation resources |
| **Integer precision** | Safe range: `±(2 ** 53 - 1)`; `Number.MAX_SAFE_INTEGER` is the upper bound | Exact integer arithmetic beyond that range |
| **Division** | `5 / 2` → `2.5`; `1 / 0` → `Infinity` | `5n / 2n` → `2n` (toward zero); division by `0n` throws |
| **Mixed arithmetic** | `1 + 1n` throws `TypeError` | Convert explicitly; Number conversion can lose precision. |
| **JSON** | Serialises; `NaN`/infinities become `null` | `JSON.stringify` throws by default; use a string encoding or explicit replacer/reviver contract. |

`0.1 + 0.2 !== 0.3`: binary rounding; BigInt does not solve decimals. `NaN !== NaN`;
`0 === -0`. `Object.is` distinguishes signed zero and equates NaN with itself.
Money needs a representation and rounding policy. See [MDN: Number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number)
and [BigInt](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/BigInt);
challenge B shows precision loss.

### Falsy is not the same as nullish

The falsy primitives are **`false`, `0`, `-0`, `0n`, `""`, `null`, `undefined`, `NaN`**.
Non-empty strings (`"0"`, `"false"`) and ordinary objects (`[]`, `{}`) are truthy.
Legacy exception: `document.all` is a falsy object with `typeof` → `"undefined"`.
See [MDN: falsy values](https://developer.mozilla.org/en-US/docs/Glossary/Falsy).

`a || fallback` defaults on any falsy value; `a ?? fallback` defaults only on nullish
values: `0 || 10` → `10`, but `0 ?? 10` → `0`. Both short-circuit and return an operand,
not necessarily a boolean. Prefer `??` when `0`, `false` or `""` is valid data.

---

## 6. Active Recall — Predict Before Expanding

### A. Sharing, shallow copying & parameter reassignment

```js
const original = { count: 1, nested: { count: 1 } };
const copy = { ...original };
function update(value) {
  value.nested.count++;
  value = { count: 99 };
}
update(copy);
copy.count++;
console.log(original.count, original.nested.count, copy.count);
console.log(original === copy, original.nested === copy.nested);
```

<details>
<summary>Show answer & reason</summary>

`1 2 2`, then `false true`. Spread creates a new outer object but shares `nested`.
Parameter reassignment stays local; the nested mutation affects both objects.

</details>

### B. Precision beyond the safe integer range

```js
const n = 2 ** 53;
console.log(n === n + 1);
console.log(BigInt(n) + 1n === BigInt(n + 1));
```

<details>
<summary>Show answer & reason</summary>

`true`, then `false`. Number arithmetic rounds `n + 1` back to `n`; BigInt preserves the
extra integer. Converting an already-rounded Number cannot recover lost precision.

</details>

### C. Metadata that does not overwrite application fields

```js
const pluginId = Symbol("id");
const user = { id: "public", [pluginId]: "plugin" };
console.log(user.id, user[Symbol("id")], user[pluginId]);
console.log(Object.keys(user));
console.log(Reflect.ownKeys(user).length);
```

<details>
<summary>Show answer & reason</summary>

`"public" undefined "plugin"`; `["id"]`; `2`. The plugin's symbol key avoids the public
string key. A fresh symbol with the same label is different; reflection still finds the key.

</details>

**Next:** [002 — var, let & const](002-var-let-const.md).

---

## 7. Quick Recall — Interview One-Liners

| Question | Crisp answer |
|---|---|
| **What is dynamically typed?** | Values have types; reassignable bindings can hold different types over time. |
| **How many language types?** | Seven primitives plus object; arrays and functions are objects. |
| **Null, undefined, undeclared?** | Two distinct absence values versus no binding; reading an undeclared name throws. |
| **Why isn't `typeof` enough?** | Null, arrays and NaN need specialised checks; TDZ access still throws. |
| **Is JS pass-by-reference?** | Pass-by-value; copied object references share identity, but parameter reassignment stays local. |
| **Does spread deeply clone?** | No: nested objects remain shared; `structuredClone` supports deep copying only for supported data. |
| **Main numeric traps?** | Binary rounding, unsafe integers, NaN inequality and mixed BigInt/Number arithmetic. |
| **Why use Symbols?** | Collision-resistant keys and language protocols; fresh symbols are unique, registry symbols can be shared, and neither provides privacy. |
| **Are built-ins extra language types?** | No: Array, Map, Date and others are specialised objects; use category checks with realm limitations in mind. |
| **`||` vs `??`?** | Falsy fallback versus nullish fallback; both return operands and short-circuit. |
