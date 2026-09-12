# yanawa memory model

yanawa should provide safe and predictable memory behavior without forcing unnecessary complexity into ordinary application code.

Memory management should support the language goals of strong typing, simplicity, low ceremony, and reasonable performance.

> Memory safety should not require constant attention from the developer.

This document describes the current direction and open questions around ownership, references, copying, allocation, resource lifetime, and memory management in yanawa.

Nothing here should be considered stable or final while the language remains experimental.

## core principles

The yanawa memory model should aim for:

* memory safety
* predictable behavior
* simple everyday usage
* minimal manual memory management
* clear value and reference semantics
* safe resource cleanup
* reasonable performance
* useful compiler diagnostics
* interoperability with future compilation targets

The language should avoid exposing low-level memory details unless they provide meaningful control or solve a real problem.

## memory safety

Ordinary yanawa code should not allow unsafe memory behavior such as:

* use-after-free
* dangling references
* double free
* invalid memory access
* accidental access to destroyed objects

The language implementation should prevent these problems through the compiler, runtime, or a combination of both.

Developers should not need to manually call operations such as:

```text
malloc
free
delete
```

for ordinary application code.

## possible memory strategies

Several models remain possible.

### garbage collection

A garbage collector could automatically reclaim unreachable memory.

Advantages may include:

* simple programming model
* familiar behavior for application developers
* easy handling of complex object graphs
* reduced ownership ceremony

Possible disadvantages include:

* runtime overhead
* pause behavior
* less deterministic destruction
* additional runtime complexity

### automatic reference counting

Reference counting could reclaim values when no references remain.

Advantages may include:

* relatively predictable reclamation
* simpler reasoning than manual memory management
* no tracing garbage collector required

Possible disadvantages include:

* reference-counting overhead
* cyclic references
* additional runtime bookkeeping

### ownership

An ownership model could determine memory lifetime at compile time.

Advantages may include:

* strong memory safety
* deterministic cleanup
* low runtime overhead

Possible disadvantages include:

* additional language complexity
* ownership and borrowing rules may increase ceremony
* potentially steeper learning curve

yanawa should not adopt an ownership model only because it is technically powerful.

Any complexity introduced into everyday programming must justify itself.

### hybrid approach

yanawa may eventually combine multiple strategies.

For example:

* simple values may live directly on the stack
* runtime-managed objects may use garbage collection
* external resources may use deterministic cleanup
* specialized types may use explicit ownership

A hybrid model may provide useful ergonomics but could also make the language harder to understand.

The rules must remain predictable.

## value semantics

Simple values should behave predictably when assigned or passed to functions.

Example:

```yanawa
let first = 10
let second = first

second = 20
```

Changing `second` should not change `first`.

This behavior is natural for primitive values.

The same principle may apply to small value-like structs.

```yanawa
struct Point:
    x: int
    y: int
```

Example:

```yanawa
let first = Point(
    x = 10,
    y = 20
)

let second = first
```

Whether this operation copies the struct or shares underlying storage depends on the final memory model.

However, observable behavior should remain easy to understand.

## reference semantics

Some values may need shared identity.

For example:

```yanawa
struct User:
    id: int
    name: str
```

Two references may intentionally represent the same user instance.

Possible behavior:

```yanawa
let first = user
let second = first

second.name = "Taylor"
```

An important question is whether:

```yanawa
first.name
```

also changes.

The answer must be obvious from the type or language rules.

yanawa should avoid situations where developers cannot easily tell whether assignment means:

* copy
* move
* reference
* shared reference

## copy semantics

Some types may be copied automatically.

Possible examples include:

```text
int
float
bool
char
```

Small immutable value types may also support copying naturally.

Example:

```yanawa
let a = 10
let b = a
```

Both values remain independently usable.

The language may eventually define a trait or compiler capability for copyable types.

Possible concept:

```yanawa
trait Copy:
    ...
```

The exact model remains undecided.

## move semantics

If yanawa adopts ownership semantics, some assignments may move values instead of copying them.

Possible example:

```yanawa
let first = resource
let second = first
```

After the move, `first` might no longer be usable.

This model can provide strong guarantees but introduces additional cognitive cost.

Move semantics should only be introduced if the broader memory model clearly benefits from them.

## immutable values

Immutability can simplify memory reasoning.

yanawa already distinguishes variables and constants:

```yanawa
let score = 10
const MAX_SCORE = 100
```

The language may eventually distinguish between:

* rebinding a variable
* mutating an object
* immutable values
* immutable fields

These concepts should remain separate and understandable.

For example:

```yanawa
let user = User(
    name = "Alex"
)
```

The fact that `user` was declared with `let` does not necessarily answer whether:

```yanawa
user.name = "Morgan"
```

is allowed.

The final mutability model must make this clear.

## field mutability

Possible approaches include:

### mutable fields by default

```yanawa
struct User:
    name: str
```

Then:

```yanawa
user.name = "Morgan"
```

would be allowed.

### immutable fields by default

Mutation would require explicit syntax.

Possible direction:

```yanawa
struct User:
    mutable name: str
```

### instance-level mutability

The declaration of the instance may determine whether fields can change.

Possible conceptual syntax:

```yanawa
let user = User(...)
const user = User(...)
```

The exact approach remains unresolved.

yanawa should choose the model that provides the clearest behavior with the least ceremony.

## stack and heap

The language should generally avoid requiring developers to choose between stack and heap allocation manually.

The compiler and runtime should determine appropriate storage whenever possible.

Code such as:

```yanawa
let user = User(
    id = 1,
    name = "Riley"
)
```

should not require an explicit allocation keyword for ordinary usage.

Low-level allocation control may exist later if systems-oriented use cases justify it.

## object lifetime

A value should remain valid for as long as the program can legitimately access it.

The mechanism may involve:

* compiler-managed ownership
* runtime tracing
* reference counting
* scoped lifetimes

Developers should not need to reason about raw memory addresses during normal application development.

## references

yanawa may eventually support explicit reference types.

Possible syntax could resemble:

```yanawa
ref User
```

or:

```yanawa
&User
```

No syntax has been chosen.

Explicit references should only exist if they provide meaningful semantics beyond ordinary variable usage.

The language should avoid introducing pointer-like syntax solely because other languages provide it.

## raw pointers

Raw pointers are not currently considered necessary for ordinary yanawa code.

If future interoperability or systems programming requires them, they should likely exist behind an explicitly unsafe boundary.

Possible conceptual syntax:

```yanawa
unsafe:
    ...
```

or:

```yanawa
unsafe fn ...
```

No unsafe model has been designed.

## unsafe code

yanawa should attempt to keep ordinary code memory-safe.

If unsafe operations eventually exist, they should:

* be explicit
* be isolated
* require deliberate intent
* remain unnecessary for normal application development

The language should make unsafe code easy to identify during review.

## resource management

Memory is not the only resource that requires cleanup.

Programs also manage:

* files
* sockets
* database connections
* locks
* processes
* native handles

These resources often benefit from deterministic cleanup.

Example:

```yanawa
let file = File.open("data.txt")?
```

The language should ensure that `file` is eventually closed safely.

Possible mechanisms include:

* scoped cleanup
* destructors
* defer
* context blocks
* automatic resource traits

No final mechanism has been chosen.

## scoped resources

One possible direction is explicit scoped resource usage.

```yanawa
with File.open("data.txt")? as file:
    let content = file.read()
```

When the block ends, the resource is released.

Another possibility is automatic lifetime-based cleanup.

The syntax should remain minimal and predictable.

## defer

yanawa may consider a `defer` mechanism.

Possible syntax:

```yanawa
let file = File.open("data.txt")?
defer file.close()

let content = file.read()
```

Deferred operations would execute when the current scope exits.

This could work for:

* normal return
* error propagation
* early exits

Whether `defer` is necessary depends on the final resource model.

## destructors

Types may eventually define cleanup behavior.

Possible concept:

```yanawa
impl Drop for Connection:
    fn drop():
        close()
```

Whether user-defined destructors are necessary remains undecided.

Deterministic destruction interacts strongly with the choice between garbage collection, ownership, and reference counting.

## cyclic references

If yanawa uses reference counting or shared references, cycles must be handled intentionally.

Example:

```text
A -> B
B -> A
```

A naive reference-counting implementation may never reclaim these values.

Possible solutions include:

* tracing cycle detection
* weak references
* explicit ownership boundaries

This issue should not be exposed to ordinary developers unless necessary.

## weak references

Weak references may eventually be useful for caches, observers, and cyclic object graphs.

Possible conceptual type:

```yanawa
weak<User>
```

No syntax or behavior has been designed.

This feature should only be added when a real use case exists.

## collections

Collection memory should be managed automatically.

Example:

```yanawa
let users: User[] = []
```

Adding and removing elements should not require manual allocation management.

```yanawa
users.add(user)
users.remove(user)
```

The underlying allocation strategy should remain an implementation detail unless developers explicitly request lower-level control.

## strings

Strings should behave as safe high-level values.

Example:

```yanawa
let name = "Morgan"
```

Developers should not need to manage character buffers manually.

The final string representation may depend on:

* Unicode model
* mutability
* memory efficiency
* interoperability

Whether strings are mutable or immutable should be decided explicitly later.

## function arguments

Passing values to functions should have predictable semantics.

Example:

```yanawa
fn process(user: User):
    ...
```

The language must eventually define whether `user` is:

* copied
* referenced
* moved
* passed using runtime-managed sharing

The common case should not require additional syntax unless the distinction materially affects behavior.

## return values

Returning values should also remain simple.

```yanawa
fn create_user() -> User:
    return User(
        id = 1,
        name = "Casey"
    )
```

The compiler should optimize copies where possible without changing observable language behavior.

Implementation optimizations should not leak unnecessarily into the language syntax.

## concurrency and memory

The memory model must eventually work safely with concurrency.

Potential concerns include:

* shared mutable state
* data races
* synchronization
* sending values across threads
* atomic operations
* thread-safe references

yanawa should not define concurrency independently from memory safety.

Possible future restrictions may prevent unsafe sharing of mutable values.

The exact concurrency model remains undecided.

## async and memory

Asynchronous functions may preserve values across suspension points.

Example:

```yanawa
async fn load_user(id: int) -> User:
    let response = await http.get("/users/{id}")

    return response.json()
```

The runtime or compiler must safely preserve local state while the function is suspended.

This should remain transparent to ordinary application code.

## foreign memory

Interoperability may require working with memory managed by another runtime.

Examples may include:

* JVM objects
* C pointers
* native libraries
* JavaScript objects
* WebAssembly memory

yanawa should define clear ownership boundaries at interoperability points.

Foreign memory should not weaken safety throughout the rest of the language.

## allocation transparency

Ordinary code should generally not need to know whether a value is stored:

* on the stack
* on the heap
* inline
* behind a runtime reference

Example:

```yanawa
let point = Point(
    x = 10,
    y = 20
)
```

The compiler should be free to choose an efficient representation while preserving language semantics.

## performance

Memory safety should not imply ignoring performance.

The implementation should aim for:

* avoiding unnecessary allocations
* eliminating unnecessary copies
* predictable memory usage
* efficient collection behavior
* optimized value representation

However, low-level optimizations should not force complexity into simple source code unless the performance benefit is meaningful.

## diagnostics

Memory-related compiler diagnostics should explain the actual problem clearly.

If yanawa eventually adopts concepts such as moves, ownership, or restricted sharing, diagnostics should avoid requiring the developer to understand compiler internals.

Possible example:

```text
error: value `connection` cannot be used here

`connection` was transferred earlier and is no longer available
```

Diagnostics should suggest practical solutions whenever possible.

## current direction

The current preferred direction is:

* ordinary yanawa code should be memory-safe
* manual allocation and deallocation should not be required
* raw pointers are not part of ordinary application code
* allocation location should usually remain an implementation detail
* primitive values should have intuitive value semantics
* object assignment semantics must be explicit and predictable
* resource cleanup must remain safe during normal returns and error propagation
* unsafe operations, if they exist, should be explicit and isolated
* memory management should not dominate ordinary application development
* the final memory strategy remains intentionally undecided

## open questions

The following areas remain unresolved:

* garbage collection versus ownership versus reference counting
* whether a hybrid model is desirable
* value versus reference semantics for structs
* copy semantics
* move semantics
* instance mutability
* field mutability
* immutable values
* explicit references
* raw pointers
* unsafe blocks
* stack versus heap rules
* deterministic destruction
* resource cleanup syntax
* `defer`
* destructors
* weak references
* cyclic references
* string representation
* collection allocation strategy
* argument passing semantics
* return-value semantics
* concurrency safety
* foreign-memory interoperability

The final model should be chosen based on the kind of programs yanawa is intended to build, not based on language-design fashion.

> Safe by default, simple by default.
