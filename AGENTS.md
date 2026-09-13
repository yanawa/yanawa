# AGENTS.md

This repository contains the source code, documentation, tooling, and long-term development of the yanawa programming language.

yanawa is experimental.

Agents working in this repository must treat language design decisions as deliberate engineering decisions rather than implementation details.

## project identity

yanawa is a strongly typed programming language focused on:

* strong static typing
* low ceremony
* readable and familiar syntax
* predictable behavior
* application-oriented development
* first-party tooling
* minimal framework dependence
* explicit semantics without unnecessary verbosity

The project should avoid novelty for novelty's sake.

A feature should not be introduced merely because another programming language provides it.

The main design question is:

> Does this make yanawa simpler, clearer, safer, or meaningfully better for the programs it is intended to build?

## branding

The project name is always written as:

```text
yanawa
```

Use lowercase `yanawa` in:

* documentation
* prose
* repository metadata
* examples
* comments
* tooling output

Do not capitalize it as `Yanawa` unless required by an external system or identifier convention.

## language status

yanawa is currently in the language-design and early implementation phase.

Many areas remain intentionally unresolved.

Do not treat exploratory documentation as a finalized specification.

Before implementing behavior, inspect:

```text
docs/
```

especially documents related to the area being changed.

Important design decisions should be consistent with the existing documentation.

## source of truth

The current documentation under `docs/` is the primary source of design context.

Relevant documents may include:

```text
docs/vision.md
docs/philosophy.md
docs/syntax.md
docs/decisions.md
docs/type-system.md
docs/errors.md
docs/modules.md
docs/object-model.md
docs/memory-model.md
docs/concurrency.md
docs/standard-library.md
docs/package-system.md
docs/tooling.md
docs/compiler-architecture.md
```

Not every statement in exploratory documents represents a final decision.

When documents conflict, prefer:

1. explicit accepted decisions
2. more recent intentional decisions
3. clearly stated current direction
4. unresolved questions remaining unresolved

Do not silently resolve open design questions.

## implementation philosophy

Prefer:

* correctness before optimization
* clear architecture before abstraction
* small changes
* explicit behavior
* strong diagnostics
* testable compiler stages
* simple solutions
* evidence from prototypes

Avoid:

* premature abstraction
* speculative extensibility
* unnecessary dependencies
* large framework-style internal architecture
* implementing future milestones early
* copying another language's behavior without justification

The first implementation should help validate the language design.

It does not need to solve every future problem.

## language design rules

Do not introduce new syntax, semantics, keywords, type-system behavior, runtime behavior, or public APIs without checking whether the decision is already documented.

If the behavior is unresolved:

* do not invent a permanent solution
* identify the unresolved decision
* prefer the smallest reversible implementation where necessary
* document assumptions explicitly

Do not make the language different merely to make it look original.

Familiar syntax is preferred when familiar syntax already expresses the concept clearly.

Originality should come from better semantics and developer experience, not arbitrary punctuation.

## current language direction

The current design direction includes:

* strong static typing
* local type inference
* non-nullability by default
* explicit nullable types
* indentation-based blocks
* no mandatory semicolons
* `let` for variables
* `const` for constants
* `fn` for functions
* `struct` as the primary structured-data abstraction
* enums for closed variants
* traits for behavior contracts
* composition preferred over inheritance
* typed error handling
* first-party tooling
* application development without requiring a large framework

These are directions, not permission to invent missing semantics.

Consult the documentation before relying on details.

## object model

Do not assume traditional class-based object orientation.

In particular, do not introduce concepts such as:

```text
class
extends
abstract class
```

without an explicit language-design decision.

The current direction favors:

```text
struct
enum
trait
impl
composition
```

## concurrency

The final concurrency model is not decided.

Do not assume conventional `async` / `await` will necessarily become the permanent model.

Concurrency work should be treated as experimental until the relevant milestone.

## memory model

The final memory strategy is unresolved.

Do not assume:

* garbage collection
* ownership
* borrowing
* reference counting
* manual memory management

unless a specific implementation experiment explicitly requires one.

Temporary implementation choices must not silently become language semantics.

## compiler architecture

Keep the language frontend as independent from the final execution backend as practical.

Conceptually, compiler work may evolve through stages such as:

```text
source
→ lexer
→ parser
→ syntax representation
→ name resolution
→ type checking
→ semantic representation
→ lowering
→ intermediate representation
→ backend
```

The first implementation may use fewer stages.

Do not create unnecessary architecture merely to match this conceptual pipeline.

## execution target

No production execution target has been selected.

Do not assume:

* LLVM
* JVM bytecode
* WebAssembly
* JavaScript
* native machine code
* custom bytecode

The first executable implementation may use an interpreter to validate language semantics.

## implementation language

The compiler implementation language is not yet a permanent project decision.

Do not change or establish the implementation language without an explicit decision.

## tooling philosophy

The intended user-facing toolchain revolves around a single primary command:

```text
yanawa
```

Possible future workflows include:

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

Do not create separate user-facing tools unless there is a concrete reason.

Internal binaries may exist if technically useful.

## standard library philosophy

The standard library should be:

* coherent
* relatively small
* predictable
* useful for real applications

Do not place functionality in the language core merely because it is commonly used.

Prefer:

```text
language feature
```

only when semantics genuinely belong to the language.

Otherwise prefer:

```text
standard library
```

or:

```text
first-party package
```

depending on scope.

## application orientation

A major yanawa goal is exploring whether modern application development can require less framework ceremony.

Important areas include:

* HTTP
* JSON
* typed routes
* configuration
* databases
* application boundaries
* compiler/tooling awareness of application semantics

Do not prematurely turn these into language syntax.

A library or first-party package should be preferred when it can provide equivalent clarity.

## repository changes

Before changing files:

1. inspect the relevant existing documentation and code
2. understand the current milestone or issue
3. keep the change within that scope
4. avoid unrelated refactors

Do not modify unrelated files for cleanup unless required by the task.

Do not implement future roadmap work opportunistically.

## commits

Repository-facing Git content must be written in English.

Commit messages should be concise and conventional.

Preferred style:

```text
type: concise description
```

Examples:

```text
docs: define compiler architecture direction
feat: add integer tokenization
fix: preserve source span for string literals
test: add lexer indentation cases
refactor: simplify parser expression handling
chore: initialize compiler workspace
```

Use lowercase commit subjects.

Do not add punctuation at the end of commit subjects.

One meaningful technical change should generally correspond to one focused commit.

Do not create commits unless the task explicitly asks for repository changes or the established workflow requires it.

Do not push unless explicitly instructed.

## documentation

All repository documentation must be written in English unless explicitly requested otherwise.

Documentation should:

* explain intent
* distinguish decisions from exploration
* record meaningful open questions
* avoid presenting speculation as finalized behavior
* use simple fictitious names in examples

Do not include personal information in public examples.

Prefer neutral example names such as:

```text
Alex
Morgan
Riley
Taylor
Jordan
Casey
```

## code style

Until implementation-specific conventions are established:

* prioritize readability
* avoid clever code
* keep functions focused
* use explicit names
* minimize hidden behavior
* keep dependencies intentional

Follow the formatter and linter once first-party tooling exists.

## dependencies

Do not add dependencies casually.

Before introducing a dependency, consider:

* what problem it solves
* whether the standard library is sufficient
* maintenance cost
* security implications
* portability
* whether implementing the small required subset internally is simpler

Compiler infrastructure should remain understandable.

## testing

New compiler behavior should be tested at the narrowest useful layer.

Potential test categories include:

* lexer tests
* parser tests
* semantic tests
* type-system tests
* diagnostic tests
* integration tests

A bug fix should preferably include a regression test.

Do not rely only on end-to-end tests when a smaller test can identify the behavior precisely.

## diagnostics

Compiler diagnostics are a first-class part of yanawa.

Prefer diagnostics that explain:

* what happened
* where it happened
* what was expected
* what was found
* how the developer may fix it

Avoid compiler jargon when plain language is sufficient.

Invalid user code must not crash the compiler.

## formatting generated changes

Do not manually reformat unrelated code.

Run established formatting tools when they exist.

Avoid commits dominated by unrelated formatting noise.

## security

Treat source code, dependencies, manifests, and package metadata as potentially untrusted input.

Do not introduce automatic execution of project code during:

* dependency discovery
* editor startup
* formatting
* static analysis
* documentation discovery

Build hooks or plugins should not be introduced without explicit design consideration.

## compatibility

yanawa is experimental and breaking changes are currently acceptable.

However, breaking changes should still be intentional.

Do not change behavior accidentally merely because compatibility is not yet guaranteed.

## roadmap discipline

The roadmap is directional.

Later milestones should not constrain earlier experiments unnecessarily.

Do not implement a later milestone just because its eventual design seems obvious.

Evidence from earlier compiler and language prototypes should influence later decisions.

## when uncertain

When a task intersects an unresolved language decision:

1. inspect the relevant documentation
2. identify what is decided and what remains open
3. avoid silently choosing a permanent semantic
4. prefer a reversible implementation
5. surface the decision clearly

The goal is not to make progress at any cost.

The goal is to make yanawa coherent.
