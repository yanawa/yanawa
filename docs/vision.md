# yanawa vision

yanawa is an experimental strongly typed programming language focused on simplicity, readability, and low verbosity.

The project explores whether a modern programming language can provide strong guarantees and expressive features without requiring excessive syntax, boilerplate, configuration, or framework-driven development.

yanawa should feel familiar to developers coming from languages such as Java, Python, TypeScript, and Rust while developing its own identity around clarity and minimal ceremony.

## core idea

> Familiar syntax, strong typing, almost no ceremony.

yanawa should prefer simple and predictable language features over implicit magic.

The goal is not to write the smallest amount of code possible.

The goal is to remove code that does not meaningfully express the developer's intent.

## goals

* strong static typing
* simple and readable syntax
* low verbosity
* sensible defaults
* minimal boilerplate
* explicit nullability
* type inference when the type is obvious
* few annotations or no annotations
* clear and predictable language behavior
* first-class tooling
* clear compiler errors
* a small and coherent core language
* productive application development without requiring large frameworks

## non-goals

yanawa is not intended to:

* replicate Java with different syntax
* optimize for code golf
* hide complex behavior behind excessive magic
* include features only because other languages have them
* become a large ecosystem before the core language is well designed
* sacrifice readability only to reduce the number of characters written

## design principles

When two approaches are equally clear, yanawa should prefer the simpler one.

When shorter syntax makes behavior harder to understand, yanawa should prefer clarity.

Features should exist because they solve a real problem, not because they are common in other languages.

Defaults should reduce ceremony without hiding important behavior.

> Code should explain itself before the documentation has to.

## current status

yanawa is currently in the language design and experimentation stage.

The syntax, compiler architecture, runtime model, execution targets, standard library, package system, and tooling are still being explored.

Nothing described at this stage should be considered stable or final.
