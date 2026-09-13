# yanawa

**yanawa** is an experimental strongly typed programming language focused on building applications with less ceremony.

Familiar syntax. Strong guarantees. First-party tooling. Less framework-shaped code.

```yanawa
struct User:
    id: int
    name: str

fn greet(user: User) -> str:
    return "Hello, {user.name}"

fn main():
    let user = User(
        id = 1,
        name = "Morgan"
    )

    print(greet(user))
```

## why yanawa

Modern application development often requires a surprising amount of code that does not directly express application behavior.

Configuration, annotations, framework conventions, generated code, dependency wiring, serialization metadata, build tooling, and infrastructure abstractions can become a significant part of the codebase.

yanawa explores a different direction:

> What would application development look like if strong typing, useful tooling, and common application semantics were designed together from the beginning?

The goal is not to create the shortest possible language.

The goal is to remove ceremony that does not communicate developer intent.

## direction

yanawa is currently exploring:

* strong static typing
* local type inference
* non-nullability by default
* explicit nullable types
* indentation-based blocks
* structs for structured data
* enums for closed variants
* traits for behavior contracts
* composition over traditional inheritance
* typed error handling
* structured and predictable concurrency
* a coherent standard library
* integrated first-party tooling
* application development without requiring large frameworks

The syntax should remain familiar where familiar syntax is already good enough.

yanawa should not be different for the sake of being different.

## application-oriented

One of the central ideas behind yanawa is that application development should not require an enormous abstraction layer between the language and the application.

Areas under exploration include:

* HTTP
* JSON
* typed routes
* configuration
* databases
* application boundaries
* compiler-aware tooling

This does not mean these concepts must become language syntax.

The project is exploring how much can be achieved through a careful combination of language semantics, standard library design, first-party packages, and compiler tooling.

## philosophy

A few principles guide the project:

**Clarity over cleverness.**

Code should be understandable before it is impressive.

**Strong typing without excessive syntax.**

The type system should provide useful guarantees without forcing developers to repeat information the compiler already knows.

**Familiar before novel.**

New syntax should exist because it solves a problem better, not because it looks unique.

**Safe by default.**

Common code should naturally lead developers toward predictable and safe behavior.

**Tooling is part of the language.**

Formatting, testing, diagnostics, dependency management, documentation, and editor support should feel like one coherent system.

**Frameworks should be optional architecture, not mandatory infrastructure.**

Ordinary applications should not require a large framework simply to become productive.

## tooling direction

The intended developer experience revolves around a single toolchain:

```text
yanawa run
yanawa build
yanawa check
yanawa test
yanawa fmt
yanawa lint
yanawa add
yanawa remove
yanawa docs
```

These commands describe the intended direction of the project.

They are not all implemented yet.

## current status

yanawa is in the language-design and early compiler phase.

The repository currently focuses on defining:

* language semantics
* syntax
* type-system direction
* object model
* error handling
* modules
* memory and concurrency models
* standard library boundaries
* package management
* developer tooling
* compiler architecture

Implementation decisions such as the final compiler language, memory strategy, runtime model, and production backend are intentionally still open.

The first implementation is expected to prioritize language experimentation and correctness before optimization.

## documentation

The design documents live in [`docs/`](docs/).

They describe both accepted directions and unresolved questions.

yanawa is experimental, so syntax and semantics are expected to evolve significantly.

## project status

This project is not production-ready.

There is currently no stable compiler, runtime, package ecosystem, or compatibility guarantee.

That is intentional.

The project is still answering a more important question first:

> What should yanawa become before we decide how aggressively to build it?

---

**Strong typing, less ceremony.**
