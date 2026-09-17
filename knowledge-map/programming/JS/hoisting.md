# Hoisting in JavaScript

## What it is

Before executing any code, the JavaScript engine scans the current scope
(function or module) and registers all `var`, `function`, `let`, `const`, and
`class` declarations in memory ahead of time. This is called **hoisting** —
it looks as if declarations were moved ("hoisted") to the top of their scope.
Only the *declaration* is hoisted; any *assignment* stays where it is in the
code.

Different declaration types behave differently once hoisted:

| Declaration | Hoisted? | Initial value | Accessible before the line runs? |
|---|---|---|---|
| `var` | Yes | `undefined` | Yes, reads as `undefined` |
| `function` (declaration) | Yes | the function itself | Yes, fully callable |
| `let` / `const` | Yes | uninitialized (TDZ) | No — throws `ReferenceError` |
| `class` | Yes | uninitialized (TDZ) | No — throws `ReferenceError` |
| function *expression* / arrow assigned to `var`/`let`/`const` | Only the variable is hoisted, not the function body | depends on the variable type above | No, until the assignment line runs |

```js
console.log(a);      // undefined (var hoisted, not yet assigned)
var a = 1;

console.log(b);       // ReferenceError: Cannot access 'b' before initialization
let b = 2;

sayHi();               // "hi" — works, function declarations hoist fully
function sayHi() { console.log('hi'); }

greet();                // TypeError: greet is not a function
var greet = function () { console.log('hello'); };
```

### The Temporal Dead Zone (TDZ)

`let`, `const`, and `class` are hoisted but not initialized. The span between
the start of the scope and the actual declaration line is the **Temporal
Dead Zone** — referencing the variable there throws a `ReferenceError`
instead of silently returning `undefined`. This is a deliberate design
correction over `var`'s looser behavior.

### Scope matters

- `var` is function-scoped (or globally-scoped) — it hoists to the top of
  the nearest function, ignoring blocks (`if`, `for`, `{}`).
- `let`, `const`, `class`, and function declarations in strict mode /
  modules are block-scoped — they hoist only to the top of the nearest
  `{}` block.

```js
if (true) {
  var x = 1;   // leaks out of the block
}
console.log(x); // 1

if (true) {
  let y = 1;   // stays inside the block
}
console.log(y); // ReferenceError: y is not defined
```

## Why it's useful

- **Explains real engine behavior**, so you can read stack traces and
  errors (`ReferenceError` vs `undefined` vs `TypeError`) correctly instead
  of treating them as random.
- **Lets you call functions before their definition appears in the file**,
  which is genuinely convenient for organizing code — e.g. putting a
  `main()` function at the top and helper functions below it, a common and
  readable pattern.
- **Mutual recursion** between function declarations works regardless of
  the order they're written in, because both are fully hoisted before any
  code runs.
- Understanding it is what lets you explain *why* `let`/`const` are safer
  defaults than `var` — the TDZ turns a silent bug (`undefined`) into a
  loud, early error.

## How to write robust code around it

1. **Prefer `const`, then `let`; avoid `var`.** This sidesteps the
   `undefined`-reads-before-assignment trap entirely — you get a clear
   `ReferenceError` instead of a silent bug.
2. **Declare variables at the top of the scope you use them in**, close to
   first use. Even though hoisting makes the order technically not matter
   for `var`, writing code as if it doesn't hoist keeps it readable and
   avoids relying on the mechanism.
3. **Declare functions before calling them** in the reading order of the
   file, even though function declarations don't require it. Treat hoisting
   as an engine implementation detail, not a style to lean on.
4. **Never rely on `var`'s function-scoping to "leak" a variable out of a
   block** (the `if`-block example above). Use block scoping (`let`/`const`)
   so a variable's lifetime matches where it's visibly used.
5. **Use function expressions/arrow functions assigned to `const`** when you
   want to *force* usage-before-definition to fail loudly (TDZ / "not a
   function") rather than degrade into `undefined` bugs.
6. **Turn on `"use strict"` or use ES modules** — both make accidental
   global `var` leaks and other sloppy-mode hoisting quirks fail fast
   instead of silently creating globals.
7. **Lint for it.** ESLint rules like `no-use-before-define` and
   `block-scoped-var` catch code that only works because of hoisting,
   before it becomes a bug someone hits in production.

## One-line mental model

> Declarations are known to the engine before the code runs; only
> assignments happen in the order you wrote them. `var` and function
> declarations get a "safe" default value up front; `let`/`const`/`class`
> get a landmine (the TDZ) instead — write code that never needs to step on it.
