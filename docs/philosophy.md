# yanawa language philosophy

yanawa is designed around a simple idea:

> Familiar syntax, strong typing, almost no ceremony.

The language should feel approachable to developers who already know modern programming languages while avoiding unnecessary syntax, boilerplate, and configuration.

Simplicity in yanawa does not mean hiding behavior.

It means removing ceremony that does not help express intent.

## clarity over cleverness

yanawa should prefer code that is easy to understand over code that is merely short.

Shorter syntax is valuable only when it preserves or improves clarity.

```yanawa
let name = "Bryan"
```

is preferable to additional syntax when the compiler can safely infer the type.

However, explicit types should remain available whenever they improve readability or communicate intent.

```yanawa
let age: int = 22
```

The language should avoid features that save a few characters while making behavior difficult to predict.

## strong typing without excessive syntax

Static typing should be one of yanawa's core guarantees.

The compiler should detect as many invalid states and type errors as reasonably possible before execution.

Strong typing should not require developers to repeat information the compiler already knows.

```yanawa
let name = "Bryan"
let age = 22
let active = true
```

Type inference should complement the type system, not weaken it.

## sensible defaults

Common behavior should require little or no configuration.

Defaults should represent the safest or most common behavior whenever possible.

Examples of this philosophy may include:

* values being non-nullable by default
* type inference when the type is obvious
* simple visibility rules
* no mandatory semicolons
* no mandatory parentheses around conditions
* predictable project conventions

Defaults should reduce ceremony without introducing hidden magic.

## explicit when it matters

Important behavior should remain visible in the source code.

yanawa should avoid relying heavily on annotations, reflection, code generation, or external configuration to determine how ordinary code behaves.

A developer reading a file should be able to understand most of its behavior from the file itself.

This does not mean everything must be explicit.

It means hidden behavior should justify its existence.

## minimal boilerplate

Boilerplate is code that exists primarily to satisfy a tool, framework, or language requirement rather than to express application behavior.

yanawa should actively reduce this kind of code.

A simple data structure should be simple to declare.

```yanawa
struct User:
    id: int
    name: str
    email: str
```

Developers should not need constructors, getters, setters, annotations, or code-generation libraries for ordinary data structures unless those behaviors are specifically required.

## familiar before novel

yanawa should not invent new syntax only to appear different.

When an existing concept is already widely understood and works well, yanawa should prefer a familiar representation.

Innovation should focus on removing friction, improving safety, or enabling simpler programming models.

A developer should be able to read basic yanawa code before learning every detail of the language.

## consistency

Similar concepts should use similar syntax.

The language should avoid multiple ways of expressing the same basic idea unless there is a strong reason for them to coexist.

For example, yanawa should not introduce several competing styles for declaring variables, functions, or collections.

Consistency is more valuable than providing every possible preference.

## safe by default

Where practical, the language should make unsafe or error-prone behavior explicit.

Possible examples include:

* non-nullable values by default
* explicit nullable types
* bounds-aware collections
* explicit error handling
* compiler validation of unreachable or invalid states

Safety features should remain understandable and should not require excessive ceremony.

## application development without framework dependence

yanawa should explore how much common application infrastructure can be provided through the language, standard library, or first-party tooling.

Developers should not need a large framework merely to perform common tasks such as:

* serving HTTP
* parsing and producing JSON
* validating data
* accessing databases
* configuring applications
* writing tests

This does not mean every feature belongs directly in the language syntax.

The boundary between the language, standard library, and external packages should remain intentional.

## tooling is part of the language

A modern language is more than its grammar and compiler.

yanawa should eventually provide a coherent first-party developer experience around:

* compilation
* execution
* formatting
* testing
* linting
* dependency management
* documentation
* editor integration

These tools should share conventions and feel like parts of one system rather than unrelated utilities.

## performance without obsession

yanawa should aim for predictable and reasonable performance.

Performance matters, but it should not automatically take priority over readability, safety, or developer experience.

Optimization decisions should be based on real requirements and measurements rather than assumptions.

## evolution with restraint

New language features should have a clear reason to exist.

Before introducing a feature, yanawa should ask:

* What problem does this solve?
* Can the existing language already express this clearly?
* Does this introduce another way to do something we already support?
* Does the benefit justify the additional complexity?
* Can the feature remain understandable five years from now?

The language should prefer a small coherent feature set over a large collection of loosely connected ideas.

## guiding questions

When evaluating a language design decision, ask:

1. Is it clear?
2. Is it predictable?
3. Is it safe?
4. Does it reduce unnecessary ceremony?
5. Does it remain familiar where familiarity helps?
6. Does it introduce hidden behavior?
7. Is the additional complexity justified?
8. Will this still make sense as the language grows?

When these principles conflict, yanawa should generally prefer clarity and predictability.

> Code should explain itself before the documentation has to.
