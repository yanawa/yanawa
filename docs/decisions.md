# yanawa design decisions

This document records design decisions that currently define the direction of yanawa.

A decision listed here represents an intentional language direction, but it may still evolve while yanawa remains experimental.

Syntax experiments and unresolved ideas belong in `syntax.md`, not here.

## D001 — strong static typing

**Status:** accepted

yanawa uses a strong static type system.

Type errors should generally be detected before execution.

The language may use type inference when the compiler has enough information, but inference must not weaken the guarantees of the type system.

```yanawa
let name = "Alex"
let age = 27
```

The compiler should still know that `name` is a `str` and `age` is an `int`.

---

## D002 — indentation-based blocks

**Status:** accepted

yanawa uses indentation to define blocks.

A colon introduces a block.

```yanawa
if active:
    print("Active")
```

Braces are not required for ordinary blocks.

```yanawa
fn greet(name: str):
    print("Hello, {name}")
```

This keeps the language visually lightweight and reduces structural noise.

---

## D003 — no mandatory semicolons

**Status:** accepted

Statements do not require semicolons.

```yanawa
let name = "Alex"
let age = 27

print(name)
```

Semicolons should not be necessary to express ordinary yanawa code.

---

## D004 — `let` for mutable variables

**Status:** accepted

Mutable variables are declared with `let`.

```yanawa
let score = 0

score = 10
```

Explicit types remain available when useful.

```yanawa
let score: int = 0
```

---

## D005 — `const` for immutable constants

**Status:** accepted

Constant values are declared with `const`.

```yanawa
const MAX_RETRIES = 3
```

The syntax `const let` is intentionally avoided because it would express the same concept twice.

---

## D006 — type annotations follow names

**Status:** accepted

When a type is written explicitly, it follows the identifier.

```yanawa
let name: str = "Morgan"

fn greet(name: str) -> str:
    return "Hello, {name}"
```

yanawa does not use declarations such as:

```text
str name
```

This syntax should remain consistent across variables, parameters, fields, and other declarations.

---

## D007 — type inference when obvious

**Status:** accepted

Developers should not need to repeat type information the compiler can safely infer.

```yanawa
let name = "Taylor"
let age = 31
let active = true
```

Explicit typing remains available when it improves clarity or is required by context.

```yanawa
let users: User[] = []
```

---

## D008 — non-nullable by default

**Status:** accepted

Ordinary types do not implicitly accept the absence of a value.

```yanawa
let name: str = "Riley"
```

Nullable values must be represented explicitly.

Current syntax direction:

```yanawa
let nickname: str? = none
```

The exact nullable-value semantics remain under design, but nullability itself should be explicit.

---

## D009 — normal value equality

**Status:** accepted

Ordinary value comparison uses equality operators directly.

```yanawa
if name == "Riley":
    print("Welcome")
```

Common values such as strings should not require methods such as `.equals()` for normal equality checks.

---

## D010 — readable logical operators

**Status:** accepted

yanawa uses word-based logical operators.

```yanawa
if active and verified:
    print("Allowed")

if admin or moderator:
    print("Privileged")

if not banned:
    print("Welcome")
```

The current operators are:

```text
and
or
not
```

---

## D011 — string interpolation

**Status:** accepted

yanawa supports expressions inside strings using braces.

```yanawa
let name = "Casey"

print("Hello, {name}")
```

Interpolation should reduce unnecessary string concatenation while remaining immediately readable.

---

## D012 — `fn` for functions

**Status:** accepted

Functions are declared with `fn`.

```yanawa
fn add(a: int, b: int) -> int:
    return a + b
```

Return types follow `->`.

Whether return statements may eventually be omitted for final expressions remains undecided.

---

## D013 — no mandatory parentheses around conditions

**Status:** accepted

Control-flow conditions do not require surrounding parentheses.

```yanawa
if age >= 18:
    print("Adult")

while running:
    update()
```

Parentheses may still be used inside expressions when needed for precedence or clarity.

---

## D014 — `struct` for simple data structures

**Status:** accepted

yanawa provides `struct` for declaring structured data without requiring boilerplate constructors, getters, setters, or code-generation libraries.

```yanawa
struct User:
    id: int
    name: str
    email: str
```

The broader object model is still under design.

The existence of `struct` does not yet determine whether yanawa will also support classes, traits, inheritance, or other abstractions.

---

## D015 — familiar syntax over unnecessary novelty

**Status:** accepted

yanawa should not invent new syntax simply to look different.

When an existing syntax is already clear, familiar, and compatible with the language's goals, yanawa should prefer familiarity.

Novel syntax should require a meaningful improvement in clarity, safety, consistency, or developer experience.

---

## D016 — clarity over character count

**Status:** accepted

Low verbosity does not mean minimizing the number of characters at any cost.

When a shorter form makes behavior less obvious, yanawa should prefer the clearer form.

The goal is to remove ceremony, not information.

> Code should explain itself before the documentation has to.

---

## unresolved decisions

The following areas are intentionally not decided yet:

* explicit versus implicit returns
* primitive type set
* visibility defaults
* `self` inside methods
* object model
* traits and interfaces
* inheritance
* `switch` and pattern matching
* error handling
* generics
* module system
* package system
* async and concurrency
* memory management
* compiler implementation language
* runtime model
* compilation targets
* standard library boundaries
* HTTP and backend integration

These topics should remain experimental until there is enough evidence to make an intentional decision.
