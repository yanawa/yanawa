# yanawa type system

yanawa uses a strong static type system designed to provide safety without unnecessary verbosity.

The type system should help developers express intent clearly, catch invalid states before execution, and reduce repetition when the compiler already has enough information.

> Type inference should reduce repetition, not uncertainty.

This document describes the current direction of the yanawa type system.

Nothing here should be considered stable or final while the language remains experimental.

## core principles

The yanawa type system should be:

* statically checked
* strongly typed
* predictable
* explicit when ambiguity exists
* concise when the compiler has enough information
* non-nullable by default
* safe against accidental implicit conversions
* understandable without requiring advanced type theory

The compiler should prefer rejecting ambiguous or unsafe code over silently guessing the developer's intent.

## type inference

yanawa may infer types when the value provides enough information.

```yanawa
let name = "Morgan"
let age = 29
let active = true
let price = 24.90
```

The compiler should understand these values as something equivalent to:

```yanawa
let name: str = "Morgan"
let age: int = 29
let active: bool = true
let price: float = 24.90
```

Explicit type declarations remain available.

```yanawa
let retries: int = 3
```

Type inference should never turn a statically typed value into an untyped or dynamically typed value.

## explicit types

Types follow identifiers.

```yanawa
let username: str = "Taylor"
let attempts: int = 0
```

Function parameters follow the same rule.

```yanawa
fn greet(name: str) -> str:
    return "Hello, {name}"
```

Struct fields also follow the same syntax.

```yanawa
struct User:
    id: int
    name: str
```

The language should use one consistent declaration style across different contexts.

## primitive types

The initial primitive type set is still under discussion.

Possible primitive types include:

```text
str
char
bool
int
float
```

Example:

```yanawa
let username: str = "Jordan"
let initial: char = 'J'
let active: bool = true
let age: int = 34
let balance: float = 125.50
```

The exact numeric type model has not yet been decided.

Possible future questions include whether yanawa should provide:

```text
i8
i16
i32
i64
u8
u16
u32
u64
f32
f64
```

or use simpler general-purpose numeric types such as:

```text
int
float
```

This decision should balance simplicity, performance, interoperability, and predictability.

## non-nullable by default

Ordinary yanawa types should not accept the absence of a value.

```yanawa
let name: str = "Casey"
```

This should not be valid:

```yanawa
let name: str = none
```

If absence is valid, the type must explicitly allow it.

```yanawa
let nickname: str? = none
```

This makes:

```text
str
```

and:

```text
str?
```

different types.

The compiler should prevent nullable values from being used as non-nullable values without validation.

## nullable values

A nullable type uses the `?` suffix.

```yanawa
let email: str? = none
```

It may later contain a valid value.

```yanawa
email = "user@example.com"
```

Possible validation:

```yanawa
if email != none:
    print(email)
```

Possible fallback syntax:

```yanawa
let display_name = nickname ?? "Unknown"
```

The exact flow analysis for nullable values is still under design.

A desirable behavior would allow the compiler to understand that a value is non-nullable after a successful check.

```yanawa
if nickname != none:
    print(nickname)
```

Inside that block, `nickname` could be treated as `str` instead of `str?`.

## type compatibility

Values should generally be assignable only when their types are compatible.

```yanawa
let age: int = 30
```

This should be invalid:

```yanawa
age = "thirty"
```

The compiler should report a clear type error instead of attempting an unexpected conversion.

## implicit conversions

yanawa should avoid broad implicit conversions.

For example, converting a string into an integer should require intent.

Instead of silently accepting:

```yanawa
let value: int = "42"
```

yanawa should require an explicit conversion.

Possible syntax:

```yanawa
let value = int("42")
```

or:

```yanawa
let value = "42".to_int()
```

The final conversion API is undecided.

The important principle is that conversions which may fail or change meaning should be visible in the source code.

## numeric conversions

Safe numeric promotion may be allowed in limited situations.

For example:

```yanawa
let count: int = 10
let price: float = 4.50

let total = count * price
```

The compiler might infer `total` as `float`.

However, narrowing conversions should probably require explicit intent.

Possible example:

```yanawa
let value: float = 42.8
let rounded: int = int(value)
```

The exact numeric conversion rules remain undecided.

## collections

Collection types should preserve their element type.

```yanawa
let names: str[] = ["Alex", "Morgan", "Taylor"]
```

The compiler should reject incompatible elements.

```yanawa
let names: str[] = ["Alex", 42]
```

An empty collection requires explicit type information when the compiler cannot infer it.

```yanawa
let names: str[] = []
```

The type of:

```yanawa
let values = [1, 2, 3]
```

should be inferred as:

```text
int[]
```

## function types

Function parameters and return values should be statically checked.

```yanawa
fn add(a: int, b: int) -> int:
    return a + b
```

This call is valid:

```yanawa
add(10, 20)
```

This call should fail at compile time:

```yanawa
add("10", "20")
```

Return values must also match the declared return type.

```yanawa
fn age() -> int:
    return "thirty"
```

The compiler should reject this function.

## inferred return types

yanawa may eventually support inferred function return types.

Possible example:

```yanawa
fn add(a: int, b: int):
    return a + b
```

The compiler could infer:

```text
int
```

However, explicit return types may provide useful documentation and stronger public API contracts.

This remains an open decision.

## structs

Struct fields have fixed types.

```yanawa
struct User:
    id: int
    name: str
    active: bool
```

Creating an instance with incompatible values should fail at compile time.

```yanawa
let user = User(
    id = "one",
    name = "Alex",
    active = true
)
```

The `id` field requires an `int`, so the compiler should reject the instance.

## structural and nominal typing

yanawa has not yet decided whether user-defined types should primarily use nominal typing, structural typing, or a combination of both.

For example:

```yanawa
struct User:
    name: str

struct Customer:
    name: str
```

Should `User` and `Customer` be considered completely different types despite sharing the same structure?

The current direction favors predictable type identity, which may imply nominal typing for declared types.

This requires further exploration.

## type aliases

yanawa may support aliases for improving readability.

Possible syntax:

```yanawa
type UserId = int
type Email = str
```

Example:

```yanawa
let id: UserId = 42
let email: Email = "user@example.com"
```

An open question is whether aliases should create completely new types or simply alternative names for existing types.

Both behaviors may be useful but should not be confused.

## enums

Enums should participate fully in the type system.

```yanawa
enum Status:
    pending
    running
    completed
    failed
```

A variable may explicitly use the enum type.

```yanawa
let status: Status = Status.pending
```

Associated values are also being explored.

```yanawa
enum Result<T, E>:
    ok(T)
    error(E)
```

This could provide the foundation for type-safe error handling.

## generics

yanawa should eventually support generic types and functions.

Possible generic struct:

```yanawa
struct Box<T>:
    value: T
```

Usage:

```yanawa
let number = Box<int>(
    value = 42
)
```

Type inference may allow:

```yanawa
let number = Box(
    value = 42
)
```

where the compiler infers:

```text
Box<int>
```

Generic functions may look like:

```yanawa
fn first<T>(items: T[]) -> T?:
    if items.length == 0:
        return none

    return items[0]
```

The exact generic syntax and constraint system remain undecided.

## generic constraints

Generic types may eventually need constraints.

Possible direction:

```yanawa
fn max<T: Comparable>(a: T, b: T) -> T:
    ...
```

Alternative syntax may be considered later.

Constraints should remain readable and should not turn ordinary generic code into excessive type syntax.

## result types

yanawa is exploring typed results for operations that may fail.

Possible type:

```text
result<T, E>
```

Example:

```yanawa
fn divide(a: int, b: int) -> result<int, str>:
    if b == 0:
        return error("Division by zero")

    return ok(a / b)
```

This makes failure visible in the function signature.

The compiler may require callers to handle or explicitly propagate the error.

The exact error model remains under design.

## type narrowing

The compiler should ideally understand checks that narrow possible types.

Nullable example:

```yanawa
let name: str? = find_name()

if name != none:
    print(name)
```

Inside the `if` block, the compiler could understand `name` as `str`.

Similar narrowing may later apply to enums or other sum types.

## equality

Values of compatible types should support normal equality operators when equality is defined for the type.

```yanawa
if name == "Alex":
    print("Matched")
```

Possible operators:

```text
==
!=
```

The language should avoid requiring special equality methods for common built-in types.

Whether custom structs receive equality automatically remains undecided.

## compile-time guarantees

The type checker should eventually detect problems such as:

* assigning incompatible values
* invalid function arguments
* invalid return values
* unsafe nullable access
* invalid generic usage
* unreachable type cases where detectable
* incompatible collection elements
* invalid operations between types

Compiler diagnostics should explain:

* what type was expected
* what type was received
* where the mismatch occurred
* possible relevant context

Error messages are part of the developer experience and should be treated as a first-class language feature.

## type system boundaries

yanawa should avoid adding advanced type-system features solely because they are theoretically powerful.

Features should justify their complexity through practical improvements to safety, clarity, or expressiveness.

Potential future features such as:

* union types
* intersection types
* dependent types
* higher-kinded types
* variance annotations
* advanced lifetime systems

should not be assumed necessary.

They should only be introduced if real language requirements demonstrate their value.

## open questions

The following areas remain unresolved:

* final primitive type set
* exact integer and floating-point model
* inferred function return types
* implicit numeric promotion rules
* explicit conversion syntax
* nullable flow analysis
* nominal versus structural typing
* type alias semantics
* generic syntax
* generic constraints
* equality for user-defined types
* result type design
* type narrowing
* compile-time constant evaluation
* representation of function types
* whether union types should exist
* whether enums should support associated values

The type system should grow only when the language provides a clear reason for additional complexity.
