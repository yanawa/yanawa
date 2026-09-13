# Language Identity

This document defines the durable identity of yanawa.

It exists to answer a question that should remain useful throughout the lifetime of the project:

> **What is yanawa trying to become?**

It is not a language specification.

It does not freeze experimental syntax, implementation architecture, runtime design, memory management, concurrency semantics, or application APIs.

Instead, it defines the principles and technical hypotheses future proposals should be evaluated against.

For broader motivation, see [`vision.md`](./vision.md).

For expanded design philosophy, see [`philosophy.md`](./philosophy.md).

For accepted concrete language decisions, see [`decisions.md`](./decisions.md).

---

## What is yanawa?

yanawa is a strongly typed application language designed to make common software explicit without making it ceremonial.

Its central programming goal is:

> **Strongly typed application development should feel direct.**

yanawa aims to provide strong static guarantees, predictable semantics, and a coherent development experience while reducing mechanical work that exists primarily because the language, libraries, compiler, and application tooling do not share enough information.

The project is particularly interested in the boundary between typed application code and the external systems applications interact with.

---

## Why does yanawa exist?

Modern application development often contains two very different kinds of complexity.

Some complexity is inherent.

Applications must still deal with:

* unreliable networks
* invalid external input
* persistence
* concurrency
* failure
* changing state
* authorization
* distributed systems
* external services

yanawa does not attempt to pretend these problems disappear.

Other complexity is mechanical.

Developers frequently repeat information through:

* DTO definitions
* serialization metadata
* validation schemas
* route metadata
* dependency wiring
* framework annotations
* generated accessors
* mechanical mappings
* reflection-based configuration
* separate framework-specific semantic models

In many cases, the program already contains part of this information through its types and declarations.

yanawa asks:

> **Is the developer providing new information, or repeating information the compiler already knows?**

When information is genuinely new, it should be expressed.

When the information can already be derived safely, yanawa should investigate whether the repetition can be removed.

The goal is not fewer characters.

The goal is less accidental ceremony.

---

## Who is yanawa primarily designed for?

yanawa is initially designed around the realities of application development.

Important early contexts include:

* backend services
* APIs
* workers
* command-line applications
* automation
* configuration-heavy applications
* applications communicating with databases and external services

These contexts expose the design problems yanawa is most interested in:

```text
external systems
       ↓
application boundaries
       ↓
typed values
       ↓
application logic
```

This initial focus does not permanently restrict yanawa to backend development.

The underlying design questions may eventually apply to:

* desktop software
* frontend applications
* distributed systems
* developer tooling
* other application domains

The project should begin with a sufficiently focused environment to evaluate its ideas without claiming to solve every programming domain.

---

## What should yanawa feel like?

yanawa should feel familiar quickly.

Ordinary programming should not require learning unusual replacements for concepts developers already understand.

Code should generally be:

* readable without extensive framework knowledge
* strongly typed without constant annotation
* explicit about meaningful behavior
* predictable by default
* concise where information can be derived safely
* supported by useful diagnostics
* navigable by tooling
* composed from a relatively small set of understandable mechanisms

A developer should ideally spend their learning budget on the areas where yanawa provides meaningful new value rather than relearning loops, variables, imports, or function calls.

The desired experience is not:

> clever code with as little syntax as possible

It is:

> code that communicates what the program does without repeatedly describing what the compiler already knows.

---

# Design invariants

Future language, compiler, tooling, runtime, and ecosystem proposals should be evaluated against the following invariants.

These invariants preserve coherence without preventing evolution.

A proposal that conflicts with one of them is not automatically forbidden, but the conflict should be explicit and justified.

---

## 1. Strong guarantees without repetition

> **Static guarantees should increase confidence without requiring developers to restate information the compiler already has.**

yanawa should provide meaningful compile-time guarantees while avoiding redundant declarations.

For example:

```yanawa
let age = 27
```

should not require:

```yanawa
let age: int = 27
```

unless the explicit annotation communicates useful information.

Inference should remain understandable and predictable.

The goal is not maximum inference.

The goal is strong typing without unnecessary ceremony.

---

## 2. Derived information may be implicit; behavior must remain visible

> **Information that can be safely derived may remain implicit. Behavior with meaningful consequences must remain visible in source code, types, or explicit program structure.**

Inferring a type is different from silently introducing:

* network access
* database access
* transactions
* retries
* concurrency
* background work
* resource acquisition
* state mutation

yanawa should remove redundant information without making important runtime behavior difficult to discover.

A recurring design question is:

> **Is the implicit part information the compiler already knows, or are we hiding behavior?**

---

## 3. Familiar unless there is a reason to improve it

> **Solved concepts should remain familiar. yanawa should introduce novelty only when it can articulate the problem being solved.**

Novelty has a cost.

That cost should purchase a concrete improvement.

Variables, ordinary function calls, conditions, loops, arithmetic, and other established constructs should generally remain recognizable.

yanawa does not need unusual syntax to establish an identity.

---

## 4. Applications are a primary design context

> **Language decisions should be evaluated against the realities of building applications, not only isolated language examples.**

Error handling should eventually be tested through real failure chains.

Nullability should interact well with external data.

Concurrency should be evaluated against requests, workers, cancellation, and timeouts.

Data modeling should survive persistence and serialization boundaries.

A feature that looks elegant in a small example but becomes unpleasant in a real application should be reconsidered.

---

## 5. Safe and predictable by default

> **The easiest way to write ordinary yanawa code should also be the safe and predictable way.**

Current directions compatible with this principle include:

* non-nullable values by default
* expected failures visible through types
* exhaustive handling where useful
* safe conversion of external data
* explicit shared mutation
* predictable concurrent lifetime

Safety should remain proportional.

A mechanism that provides strong guarantees only through excessive ceremony should also be questioned.

---

## 6. Prefer composable primitives over hidden machinery

> **Complex behavior should emerge from a small set of understandable, composable mechanisms rather than overlapping language features or framework-specific machinery.**

Current directions worth exploring include:

```text
struct
enum
trait
functions
composition
```

before introducing larger overlapping abstraction systems.

A typed library should generally be preferred over new language syntax when ordinary language mechanisms can express the concept clearly and safely.

A recurring question should be:

> **Can the existing language express this clearly before we create another mechanism?**

---

## 7. Tooling is part of the language contract

> **A yanawa feature is not fully designed until the toolchain can explain, diagnose, navigate, and support it coherently.**

Tooling is not an afterthought.

Feature design should consider:

* compiler diagnostics
* formatting
* static analysis
* editor understanding
* symbol navigation
* refactoring
* documentation
* testing
* incomplete source code

This does not require complete editor support to exist before every language feature.

It means tooling consequences are part of the feature's design.

---

# Familiarity and innovation

Most of yanawa should deliberately remain familiar.

The project uses four categories when deciding how much design experimentation an area deserves:

```text
familiar
refine
explore
innovate
```

They do not describe implementation status.

They describe how much deviation from established models is currently justified.

---

## Familiar

```text
variables
constants
functions
control flow
modules
imports
primitive types
```

These concepts are already well understood.

yanawa should not spend its innovation budget changing them without concrete evidence that existing models are inadequate.

---

## Refine

```text
type inference
nullability
structured data
polymorphism
error handling
serialization
diagnostics
dependency management
```

These areas already have strong existing models, but there is room to improve safety, coherence, or ceremony.

The goal is recognizable improvement rather than conceptual replacement.

---

## Explore

```text
mutability
concurrency
HTTP/application integration
```

These areas contain meaningful unresolved problems.

yanawa should experiment before deciding whether a distinct model is justified.

A familiar solution remains an acceptable outcome.

---

## Innovate

```text
application boundaries
compiler-aware tooling
```

These are the project's strongest current candidates for deliberate innovation.

They are directly connected to the application-oriented thesis.

Innovation here does not necessarily mean new syntax.

It may happen through:

* semantics
* static types
* compiler analysis
* library design
* first-party packages
* diagnostics
* editor tooling

---

# Application-oriented development

Application-oriented has a specific technical meaning in yanawa.

It does not mean:

* HTTP built into the grammar
* a bundled web framework
* database keywords
* an ORM inside the language
* framework conventions hardcoded into the compiler

The thesis is:

> **yanawa is application-oriented because it treats the boundaries between strongly typed application code and external systems as a first-class language-design problem.**

Applications continuously cross boundaries such as:

```text
HTTP
JSON
databases
configuration
filesystems
command-line input
processes
external services
```

At these boundaries:

```text
external representation
        ↓
parse / decode / validate
        ↓
typed yanawa value
        ↓
application logic
```

The language should investigate whether these transitions can be made more direct by reusing information already represented by yanawa types.

---

## Types as reusable application knowledge

Consider:

```yanawa
struct User:
    id: int
    name: str
    email: str
```

The program already contains structural information about `User`.

An application should not automatically need equivalent declarations such as:

```text
UserDTO
UserSchema
UserSerializer
UserMapper
field metadata repeating the same shape
```

when they contain no additional meaning.

Separate types remain appropriate when concepts are genuinely different.

For example:

```yanawa
struct CreateUser:
    name: str
    email: str

struct User:
    id: int
    name: str
    email: str
```

These represent different application concepts and should remain different types.

yanawa seeks to remove accidental duplication, not useful modeling.

---

## Boundaries remain explicit

Reusing type information should not hide the boundary itself.

Something conceptually similar to:

```yanawa
let user = json.decode<User>(body)?
```

communicates useful information:

* external data is being decoded
* the target type is `User`
* the operation may fail

The compiler may derive mechanical field relationships.

It should not hide the fact that decoding occurred.

A useful summary is:

> **Reduce declaration duplication, not behavioral visibility.**

---

## Compiler-aware libraries

yanawa should explore whether ordinary typed libraries can expose enough static information for the compiler and language server to understand application relationships.

For example:

```yanawa
app.get("/users/{id}", find_user)
```

paired with:

```yanawa
fn find_user(id: int) -> User:
    ...
```

contains a relationship between:

```text
{id}
```

and:

```text
id: int
```

Many current compilers cannot validate this relationship because it exists only inside framework metadata.

yanawa should investigate whether relationships like this can become visible to normal language tooling.

---

## No hardcoded framework semantics

Application awareness must not turn the compiler into a list of special frameworks.

Architectures conceptually equivalent to:

```text
if package == yanawa.http:
    enable special route behavior
```

should be avoided.

The preferred principle is:

> **The compiler should understand general semantic concepts. Libraries should describe domain-specific relationships through those concepts.**

The project should use ordinary language semantics first.

If they are insufficient, stronger approaches may be explored in increasing order of complexity:

```text
ordinary types
      ↓
compile-time-known information
      ↓
constrained declarative semantic information
      ↓
more powerful extension mechanisms only if necessary
```

No final extension mechanism has been selected.

First-party packages should not receive secret semantic capabilities unavailable to third-party packages.

---

# What yanawa should challenge

yanawa should investigate whether ordinary application development can reduce mechanical use of:

* annotation-heavy configuration
* duplicated serialization metadata
* repetitive DTOs
* mechanical mapping between identical structures
* generated getters and setters
* reflection-driven discovery
* duplicated route metadata
* hidden dependency wiring
* framework-specific semantic models disconnected from language tooling

These mechanisms are not categorically forbidden.

The relevant question is whether they provide genuinely new information.

If they do, that information should remain expressible.

If they merely repeat information the language already knows, yanawa should try to avoid the repetition.

---

# Distinctive semantic experiments

The project currently has three semantic experiments considered worthy of future prototypes.

They are experiments, not accepted language semantics.

Their purpose is to produce evidence.

---

## Typed application boundaries with compiler-aware libraries

This is the highest-priority experiment.

The research question is:

> **Can typed application boundaries expose enough structured information for normal yanawa tooling to provide strong guarantees without framework-specific compiler magic?**

Potential prototypes include:

```text
JSON → typed struct
route parameter → handler parameter
configuration → typed fields
CLI input → typed values
```

The experiment must prove that any compiler-aware mechanism is general enough to work beyond one application domain.

It should be rejected or significantly narrowed if:

* framework-specific compiler hardcoding is required
* arbitrary package code must execute inside the compiler
* metadata becomes as verbose as the framework machinery it replaces
* third-party packages cannot participate naturally
* tooling becomes unpredictable or expensive
* hidden behavior is introduced

---

## Immutable-first value semantics

yanawa should prototype whether structured values being immutable by default provides a useful balance between predictable state and practical application development.

A conceptual update operation might resemble:

```yanawa
let updated = user with:
    name = "Morgan"
```

This syntax is not accepted.

The experiment concerns questions such as:

* binding rebinding
* field mutation
* object identity
* nested updates
* shared state
* collections
* concurrency interaction

The model should be rejected if ordinary application state becomes unnecessarily difficult or if mutable escape hatches dominate typical code.

---

## Structured concurrency semantics

yanawa should investigate whether structured task lifetime provides a clearer concurrency model for applications.

Areas worth testing include:

* child-task ownership
* cancellation propagation
* sibling failure
* explicit detached work
* predictable task lifetime

A conceptual source form might resemble:

```yanawa
concurrent:
    let user = users.find(id)
    let posts = posts.find_by_user(id)
```

but no syntax is accepted.

The experiment may conclude that conventional:

```text
async / await
```

combined with structured task scopes is the best solution.

That would be a successful experimental result.

---

# Current directions that are not identity commitments

Some ideas currently look promising but are intentionally not language invariants.

Examples include:

* composition over inheritance
* immutable-by-default values
* structured concurrency
* avoiding reflection
* minimizing annotations
* typed result values
* compiler-aware application libraries

These should continue to be tested.

A current direction should not become permanent merely because it appears in early documentation.

---

# What is not a goal

yanawa is not trying to be novel everywhere.

It is not trying to become the shortest possible programming language.

It is not trying to replace every library abstraction with language syntax.

It is not trying to eliminate all frameworks categorically.

It is not trying to copy another language and change its surface syntax.

It is not trying to combine every popular feature from modern languages.

It is not trying to solve systems programming, frontend development, backend development, scripting, embedded development, and every other domain simultaneously.

It is not trying to hide application complexity behind compiler magic.

It is not trying to make meaningful behavior implicit merely to reduce source-code size.

It is not trying to embed Spring or another application framework into the language under different terminology.

The goal is a coherent language, not a maximal feature set.

---

# What remains unresolved

The language identity intentionally does not decide several major implementation and semantic questions.

These include:

* compiler implementation language
* production compiler backend
* execution target
* runtime architecture
* memory management strategy
* ownership or borrowing semantics
* garbage collection
* reference counting
* final mutability model
* final concurrency model
* final application integration model
* compiler extension mechanism
* HTTP API design
* database abstraction model
* package registry architecture
* self-hosting
* frontend or full-stack strategy

These decisions should be made when the project has sufficient implementation evidence.

They should not be chosen merely to make the project appear more complete.

---

# Evaluating future proposals

Future yanawa proposals should be judged using this document.

A proposal should be able to answer:

1. What concrete problem does this solve?
2. Is the problem inherent or accidental?
3. Is the developer providing new information or repeating information already known?
4. Does meaningful behavior remain visible?
5. Could existing language primitives solve the problem?
6. Why is novelty justified?
7. How does this affect tooling and diagnostics?
8. How does it behave in real application code?
9. What complexity does it add?
10. What evidence would cause the project to reject it?

A proposal that is interesting but cannot answer these questions does not yet belong in yanawa.

---

# Durable identity

yanawa is a strongly typed application language designed to make common software explicit without making it ceremonial.

It deliberately keeps most ordinary programming familiar.

It seeks stronger guarantees without requiring developers to repeat information the compiler already has.

It treats real applications as a primary design context.

It keeps meaningful behavior visible.

It prefers composable primitives over hidden machinery.

It considers tooling part of the language itself.

Its strongest current technical hypothesis is that application boundaries can become more direct when the language, libraries, compiler, and tooling share a coherent understanding of types and static relationships.

That hypothesis must still be proven through real programs.

The project should preserve the principles that survive that evidence and discard the ideas that do not.

> **Strong typing, less ceremony.**
