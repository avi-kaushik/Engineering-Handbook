# 002 — Origins: Netscape, Sun & the 10-Day Prototype

> **One-line takeaway:** JavaScript was designed as Java's lightweight scripting
> companion, not a version of Java. **Ten days describes the first prototype**;
> deliberate design choices and later compatibility constraints explain more than haste alone.

---

## 1. Why Netscape & Sun Collaborated

In 1995, **Netscape** wanted an interactive browser platform; **Sun Microsystems** wanted
distribution for Java. Netscape agreed to integrate Java into Navigator. A smaller
scripting language would let page authors glue components together without Java's
class/type ceremony. This also supported a cross-platform web alternative to Microsoft's
Windows ecosystem.

The intended pairing was **Java for components; scripting for page-level glue**.
The shared branding promoted that partnership. See the participant-authored history,
[JavaScript: The First 20 Years, §§2–3](https://wirfs-brock.com/allen/jshopl.pdf).

### The naming sequence

```text
Mocha                    LiveScript                  JavaScript
internal prototype   →   Navigator 2 beta name   →   joint Netscape–Sun announcement
May 1995                 September 1995               4 December 1995
```

These were names for the evolving language, not three separate languages.
Navigator 2's **final release was in 1996**; the 1995 public versions were betas.
The same [historical account, timeline and original announcement](https://wirfs-brock.com/allen/jshopl.pdf)
document these dates.

---

## 2. Why JavaScript Is Not Java

Java influenced the surface syntax and some library conventions, including early `Date`
behaviour. Shared naming does **not** imply shared execution or object models.

| Dimension | Java | JavaScript |
|---|---|---|
| **Typing** | Static typing, checked before execution | Dynamic typing; values carry types |
| **Object model** | Class-based | Prototype-based, including when using `class` syntax |
| **Execution** | Typically compiled to JVM bytecode, then interpreted/JIT-compiled by a JVM | Executed by a JS engine; modern engines commonly interpret and JIT-compile |
| **Functions** | Methods and functional interfaces / lambdas | First-class function values |
| **Shared appearance** | Braces, loops, `if`, familiar operators | Similar syntax, different semantics |

> **Avoid two shortcuts:** “JS is simplified Java” is false. “They share absolutely
> nothing except the name” ignores syntax and library influence. Both can use JIT;
> “Java is compiled, JS is interpreted” is not a useful distinction.

Eich confirms the early Java `Date` influence in his
[first-person engine history](https://brendaneich.com/2011/06/new-javascript-engine-module-owner/).

---

## 3. What “Built in 10 Days” Actually Means

**Brendan Eich built the first engine/language prototype at Netscape in May 1995** for a
feasibility demonstration. The constraint was to prove a lightweight, Java-looking
scripting companion could work, not to finish today's language in ten days.

| Constraint / influence | Design consequence |
|---|---|
| **Approachable page scripting** | Dynamic values and automatic conversions reduced explicit type declarations/conversions. |
| **Java-like appearance** | Familiar braces and control-flow syntax, rather than Scheme's surface syntax. |
| **Scheme influence** | First-class functions and the foundation for functional programming idioms. |
| **Self influence** | Prototype-based objects rather than Java's class model. |
| **Short demonstration deadline** | A minimal implementation first; refinement and integration continued afterward. |

The [history's design discussion (§2)](https://wirfs-brock.com/allen/jshopl.pdf) describes
these constraints. Eich also states that the prototype had a **bytecode compiler and
interpreter from the start**; browser integration occupied the rest of the year in his
[first-person account](https://brendaneich.com/2011/06/new-javascript-engine-module-owner/).

> **Historical fact vs oversimplification:** the ten-day sprint is real. “Every quirk
> was an accidental mistake made in those ten days” is not. Coercion and permissive
> syntax reflect design tradeoffs; bugs, later revisions and web compatibility also matter.

---

## 4. Four Quirks Worth Explaining

### `typeof null`: legacy behaviour, not a type lesson

```js
typeof null;       // "object"
null === null;     // true — test null directly
```

`null` is a **primitive**, despite this result. An early tagged-value representation
caused it to be reported as an object; compatibility preserved the observable behaviour.
See the [type-history explanation](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/typeof#typeof_null).
For the full type table, use [Variables & Data Types](../002-fundamentals/001-variables-and-data-types.md#3-checking-types-with-typeof).

### Coercion: the operator determines conversion

```js
"5" + 1;           // "51": + concatenates if a converted operand is a string
"5" - 1;           // 4: subtraction performs numeric conversion
```

This is specified behaviour, not random engine inconsistency. Dynamic typing means
values can have different types; **coercion** means an operation converts those values.
They are separate ideas. Prefer explicit conversion at input boundaries.

### Loose equality: not the same operation as truthiness

```js
0 == false;        // true: false becomes 0
0 === false;       // false: different types
null == undefined; // true: special nullish equality rule
null == 0;         // false: null does not become 0 in this comparison
```

`==` follows specific conversion rules; it does not simply convert both sides to booleans.
`===` avoids type coercion. Prefer it by default; an intentional `value == null` checks
both `null` and `undefined` in ordinary code. Strict equality still has rules such as
`NaN !== NaN` and object comparison by identity. These operators are defined in the
[ECMAScript equality semantics](https://tc39.es/ecma262/#sec-equality-operators).

### ASI: a newline can change the parse

**Automatic Semicolon Insertion (ASI)** applies grammar rules at certain boundaries;
it does not append a semicolon after every newline.

```js
function getConfig() {
  return
  { enabled: true };
}
getConfig(); // undefined: parsed as return; followed by a separate block
```

Keep the expression on the same line: `return { enabled: true };`.
Even writing a semicolon after the object cannot undo the line break after `return`.
See the [ASI rules and examples](https://tc39.es/ecma262/multipage/ecmascript-language-lexical-grammar.html#sec-automatic-semicolon-insertion).

---

## 5. Why Compatibility Keeps Old Behaviour Alive

The web has no coordinated upgrade day. Changing established semantics can break pages
whose authors no longer maintain them. Newer alternatives therefore often coexist with
older behaviour: `let`/`const` alongside `var`, and stricter rules in opt-in strict mode.

```text
Old behaviour → deployed pages depend on it → changing it risks breakage
                                          → prefer compatible additions / opt-in changes
```

**“Don't break the web” is a strong constraint, not a claim that changes are impossible.**
Strict mode changes some semantics, but it does not fix `typeof null`, disable coercion,
replace `==`, or remove ASI. The [strict variant's scope](https://tc39.es/ecma262/multipage/overview.html#sec-the-strict-variant-of-ecmascript)
is deliberately limited.

For standardisation and JScript, see [004 — JavaScript vs ECMAScript](004-javascript-vs-ecmascript-engines-and-runtimes.md#1-javascript-vs-ecmascript--why-a-standard).

---

## 6. Quick Recall — Interview One-Liners

| Question | Crisp answer |
|---|---|
| **Why Netscape + Sun?** | Browser distribution for Java and a richer cross-platform web; JS supplied lightweight page scripting. |
| **Why the name JavaScript?** | Partnership branding positioned it as Java's scripting companion. |
| **Naming order?** | Mocha → LiveScript → JavaScript, all in 1995. |
| **What took ten days?** | Eich's May 1995 prototype; integration and language evolution continued. |
| **Three key influences?** | Java-like syntax, Scheme-inspired functions, Self-inspired prototypes. |
| **Is JS a Java subset?** | No. The typing, object and execution models differ. |
| **Why `typeof null` → `"object"`?** | An implementation legacy preserved for compatibility; null is a primitive. |
| **Why coercion surprises?** | Operators have different conversion rules; dynamic typing and coercion are distinct. |
| **Why prefer `===`?** | It avoids equality's implicit type conversions. |
| **ASI trap?** | A newline after `return` ends the return before the intended expression. |
| **Why retain quirks?** | Existing pages depend on observable behaviour; strict mode changes only selected rules. |
