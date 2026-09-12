# yanawa concurrency

yanawa should provide a concurrency model that is safe, understandable, and practical for modern application development.

Concurrency should make it easier to perform independent work without forcing developers to manage low-level synchronization in ordinary code.

> Concurrency should scale work, not complexity.

This document describes the current direction of asynchronous programming, tasks, threads, shared state, cancellation, and synchronization in yanawa.

Nothing here should be considered stable or final while the language remains experimental.

## core principles

The yanawa concurrency model should aim for:

* safe concurrency by default
* simple asynchronous code
* structured lifetime of concurrent tasks
* explicit shared mutable state
* minimal synchronization boilerplate
* predictable cancellation
* useful compiler diagnostics
* integration with typed error handling
* consistency between synchronous and asynchronous code

Concurrency primitives should solve common application problems without exposing unnecessary runtime details.

## async functions

Functions that may suspend execution can be declared with `async`.

```yanawa
async fn fetch_user(id: int) -> User:
    let response = await http.get("/users/{id}")

    return response.json()
```

Calling an async function produces work that must be awaited or otherwise scheduled.

```yanawa
let user = await fetch_user(1)
```

The exact representation of async return values remains undecided.

## await

`await` pauses the current asynchronous task until the requested operation completes.

```yanawa
let response = await http.get("/users")
```

The runtime should be free to perform other work while the current task is suspended.

`await` should be explicit so that potentially suspending operations remain visible in source code.

## async return types

The language should avoid forcing developers to manually write implementation-specific future or promise types for ordinary async functions.

This:

```yanawa
async fn load_user(id: int) -> User:
    ...
```

should be preferred over something like:

```text
fn load_user(id: int) -> Future<User>
```

unless exposing the task value itself is useful.

The compiler and runtime may represent asynchronous results internally without leaking unnecessary complexity into common code.

## async errors

Async functions should use the same typed error model as synchronous functions.

```yanawa
async fn fetch_user(id: int) -> result<User, NetworkError>:
    let response = await http.get("/users/{id}")?

    return ok(response.json())
```

Usage:

```yanawa
let user = await fetch_user(1)?
```

Concurrency should not require a separate error-handling philosophy.

## tasks

yanawa may represent independently executing asynchronous work as a task.

Possible syntax:

```yanawa
let task = async fetch_user(1)
```

or:

```yanawa
let task = spawn fetch_user(1)
```

Then:

```yanawa
let user = await task
```

The exact syntax remains undecided.

A task should have a clear lifecycle and should not silently outlive the scope that created it unless explicitly detached.

## structured concurrency

The preferred direction is structured concurrency.

Concurrent tasks should normally exist inside a scope that owns them.

Possible example:

```yanawa
async fn load_dashboard(user_id: int) -> Dashboard:
    async:
        let user_task = spawn fetch_user(user_id)
        let posts_task = spawn fetch_posts(user_id)
        let stats_task = spawn fetch_stats(user_id)

        let user = await user_task
        let posts = await posts_task
        let stats = await stats_task

        return Dashboard(
            user = user,
            posts = posts,
            stats = stats
        )
```

When the scope ends, its child tasks should normally:

* complete
* fail
* or be cancelled

The language should avoid leaving forgotten background tasks running accidentally.

## concurrent expressions

yanawa may eventually provide a concise way to run independent operations concurrently.

Possible syntax:

```yanawa
let user, posts = await all(
    fetch_user(id),
    fetch_posts(id)
)
```

or:

```yanawa
let results = await all [
    fetch_user(id),
    fetch_posts(id),
    fetch_settings(id)
]
```

The final syntax should favor clarity over cleverness.

## task groups

A task group may provide explicit structured concurrency for dynamic collections of work.

Possible example:

```yanawa
let users = await task_group:
    for id in ids:
        spawn fetch_user(id)
```

The group would complete only when all child tasks have completed or failed according to the group's policy.

This concept requires further exploration.

## cancellation

Long-running work should be cancellable.

Example scenarios include:

* HTTP clients disconnecting
* application shutdown
* timeouts
* user navigation
* superseded requests

Cancellation should propagate naturally through structured task hierarchies.

Possible conceptual behavior:

```yanawa
async fn process_request():
    let data = await load_data()
    let result = await process(data)

    return result
```

If the parent request is cancelled, child operations should normally receive cancellation as well.

The exact API remains undecided.

## explicit cancellation

Tasks may eventually expose explicit cancellation.

Possible syntax:

```yanawa
let task = spawn expensive_operation()

task.cancel()
```

Awaiting a cancelled task should produce a predictable result or typed cancellation error.

Cancellation should not leave shared state partially mutated without clear rules.

## timeouts

Timeouts are common enough that yanawa should provide a simple first-party mechanism.

Possible syntax:

```yanawa
let user = await timeout(
    5s,
    fetch_user(id)
)?
```

Alternative:

```yanawa
let user = await fetch_user(id).timeout(5s)?
```

The exact time literal and API syntax remain undecided.

Timeouts should integrate with cancellation rather than merely ignoring work that continues in the background.

## sleeping

A standard async timing primitive may look like:

```yanawa
await sleep(500ms)
```

or:

```yanawa
await time.sleep(500ms)
```

Duration literals such as:

```text
500ms
5s
2m
```

may be useful but have not been decided.

## threads

yanawa may provide access to operating-system threads when needed.

Possible syntax:

```yanawa
let thread = thread.spawn:
    run_worker()
```

However, threads should not necessarily be the primary concurrency abstraction for ordinary application development.

Async tasks may be more appropriate for:

* network servers
* database operations
* file I/O
* high-concurrency applications

Threads may remain useful for:

* CPU-bound work
* native libraries
* dedicated workers
* lower-level concurrency

## CPU-bound work

CPU-intensive operations should not block an async runtime worker indefinitely.

The language or standard library may provide an explicit way to schedule CPU-bound work.

Possible concept:

```yanawa
let result = await cpu.run:
    calculate_report()
```

The exact API depends on the runtime design.

## shared state

Shared mutable state is one of the primary sources of concurrency bugs.

yanawa should make it clear when values are shared across concurrent execution.

Ordinary local state should remain isolated whenever possible.

```yanawa
async fn handle_request():
    let request_state = RequestState()
```

This value belongs to the current task unless explicitly shared.

## immutable sharing

Immutable values should be safe to share across tasks whenever their underlying representation permits it.

```yanawa
const config = load_config()
```

Multiple tasks may read shared immutable configuration without synchronization.

The language should favor immutable shared state where practical.

## mutable sharing

Mutable values shared between concurrent tasks should require an explicit synchronization mechanism or another safe abstraction.

The language should avoid silently allowing unsafe data races.

Possible example:

```yanawa
let counter = shared Counter()
```

or:

```yanawa
let counter: shared<Counter>
```

No syntax has been chosen.

The important principle is that shared mutation should be visible.

## data races

yanawa should aim to prevent data races through some combination of:

* type-system restrictions
* runtime synchronization
* ownership rules
* isolation
* actor-style communication
* immutable sharing

The final approach depends heavily on the memory model.

The language should not claim compile-time race freedom unless its type system can genuinely provide it.

## locks

Low-level locks may eventually exist for cases where shared state is unavoidable.

Possible types include:

```text
Mutex<T>
RwLock<T>
```

Example:

```yanawa
let users = Mutex<User[]>([])
```

Possible usage:

```yanawa
with users.lock() as locked:
    locked.add(user)
```

The exact syntax is undecided.

Lock handling should minimize the risk of forgetting to release a lock.

## scoped locks

Locks should ideally release automatically when their scope ends.

Possible example:

```yanawa
with counter.lock() as value:
    value += 1
```

This should remain safe even when:

* the function returns early
* an error is propagated
* a cancellation occurs

This behavior must align with the resource-management model.

## channels

Message passing may provide a safer alternative to shared mutable state.

Possible API:

```yanawa
let sender, receiver = channel<Message>()
```

Sending:

```yanawa
await sender.send(message)
```

Receiving:

```yanawa
let message = await receiver.receive()
```

Channels may be useful for:

* worker pools
* pipelines
* event processing
* communication between isolated tasks

The exact channel model remains undecided.

## bounded channels

Channels may support limited capacity.

Possible example:

```yanawa
let sender, receiver = channel<Message>(capacity = 100)
```

When full, sending may suspend until capacity becomes available.

This can provide natural backpressure.

## backpressure

Concurrency systems should avoid producing work faster than consumers can handle it.

Bounded queues, streams, and task limits may provide backpressure.

This is particularly important for:

* HTTP servers
* database workloads
* message processing
* file pipelines

Backpressure should be supported by first-party abstractions rather than reinvented by every application.

## actors

An actor model may eventually be explored.

An actor owns its mutable state and receives messages sequentially.

Possible conceptual syntax:

```yanawa
actor Counter:
    value: int = 0

    fn increment():
        value += 1

    fn current() -> int:
        return value
```

Usage might resemble:

```yanawa
await counter.increment()
```

Actors can reduce shared-state complexity but introduce a significant programming model.

yanawa should not adopt actors as a core feature without a clear need.

## async streams

Some asynchronous operations produce multiple values over time.

Possible concept:

```yanawa
async fn events() -> stream<Event>:
    ...
```

Consumption:

```yanawa
for await event in events():
    handle(event)
```

The exact stream model remains undecided.

Streams may be useful for:

* network data
* database cursors
* file processing
* event systems
* server-sent events
* WebSockets

## parallel iteration

yanawa may eventually support parallel processing of collections.

Possible syntax:

```yanawa
let results = parallel map users:
    process(user)
```

or a library-style API:

```yanawa
let results = users.parallel_map(process)
```

Parallel collection APIs should not be introduced before the execution and memory models are sufficiently defined.

## synchronization primitives

Possible future synchronization tools may include:

* mutexes
* read/write locks
* semaphores
* barriers
* atomics
* channels
* task groups

The standard library should expose only the primitives necessary for real applications.

High-level abstractions should be preferred when they can express common patterns safely.

## atomics

Atomic values may eventually be needed for performance-sensitive concurrent code.

Possible types:

```text
AtomicInt
AtomicBool
```

These should be considered advanced tools rather than ordinary application primitives.

Their memory-ordering model must remain understandable if exposed.

## global state

Mutable global state should be discouraged.

Example:

```yanawa
let requests = 0
```

If this value can be accessed concurrently, its semantics become difficult to reason about.

The language may eventually:

* restrict mutable globals
* require explicit shared state
* initialize global values before concurrency begins
* encourage dependency passing instead

No final rule has been chosen.

## async main

Applications may eventually support asynchronous entry points.

Possible syntax:

```yanawa
async fn main():
    let server = http.server(port = 8080)

    await server.run()
```

Whether `main` may be automatically executed within an async runtime remains undecided.

## servers

Server workloads are an important concurrency use case for yanawa.

A server may internally run many requests concurrently.

Possible application code:

```yanawa
get "/users/{id}" async fn find_user(id: int) -> User:
    return await users.find(id)?
```

The developer should not need to manually create a thread for each request.

The runtime or HTTP implementation should manage scheduling safely.

## database operations

Database APIs should integrate naturally with asynchronous execution.

```yanawa
async fn find_user(id: int) -> result<User?, DatabaseError>:
    return await database.query(
        "select * from users where id = ?",
        id
    )
```

Whether database APIs are async by default depends on the standard library and runtime architecture.

## blocking operations

Blocking operations should be identifiable.

An async function should avoid unexpectedly blocking the entire scheduler.

Tooling or the compiler may eventually warn when known blocking operations are used incorrectly inside async contexts.

Possible diagnostic:

```text
warning: blocking operation inside async function

consider using the asynchronous file API
```

## task failure

If a child task fails, structured concurrency should define what happens to sibling tasks.

Possible policies include:

* cancel siblings immediately
* wait for all tasks and collect errors
* allow explicit policy selection

The default should be simple and predictable.

## multiple errors

Concurrent work may produce multiple failures.

Possible aggregate type:

```text
TaskErrors<E>
```

or a collection of typed errors.

The exact behavior remains undecided.

## detached tasks

Sometimes applications genuinely need background work.

yanawa may support explicit detached tasks.

Possible syntax:

```yanawa
detach send_analytics()
```

or:

```yanawa
spawn detached:
    send_analytics()
```

Detached work should be visually explicit because it escapes structured task ownership.

The language should discourage accidental fire-and-forget behavior.

## shutdown

Applications should have a predictable way to stop concurrent work.

A future runtime may support graceful shutdown:

```yanawa
async fn main():
    let server = http.server(port = 8080)

    await server.run()
```

When shutdown is requested, the runtime may:

* stop accepting new work
* cancel or complete outstanding tasks
* release resources
* close connections

Exact semantics remain runtime-dependent.

## compiler diagnostics

Concurrency-related diagnostics should focus on understandable behavior.

Possible example:

```text
error: mutable value `users` cannot be shared safely

`users` is accessed by multiple concurrent tasks

consider using a synchronized container or message passing
```

If the language introduces concepts such as task ownership or sendability, compiler messages should explain them in practical language.

## runtime independence

The concurrency syntax should avoid being unnecessarily coupled to a single runtime implementation.

For example:

```yanawa
async fn load():
    let data = await fetch()
```

should remain meaningful whether the runtime eventually uses:

* an event loop
* a work-stealing scheduler
* virtual threads
* operating-system threads
* another execution strategy

Implementation details should remain invisible unless developers explicitly require control.

## interoperability

Foreign runtimes may have their own concurrency models.

Possible examples include:

* JVM threads
* JavaScript event loops
* native threads
* WebAssembly hosts

yanawa should provide clear interoperability boundaries without forcing foreign concurrency semantics throughout the language.

## example

```yanawa
async fn load_profile(id: int) -> result<Profile, ProfileError>:
    let user_task = spawn users.find(id)
    let preferences_task = spawn preferences.find(id)

    let user = await user_task?
    let preferences = await preferences_task?

    return ok(Profile(
        user = user,
        preferences = preferences
    ))
```

Possible HTTP usage:

```yanawa
get "/profiles/{id}" async fn profile(id: int) -> response<Profile>:
    let profile = await load_profile(id)?

    return response.ok(profile)
```

The exact HTTP and task syntax remain experimental.

## current direction

The current preferred direction is:

* asynchronous functions use `async`
* suspension points use `await`
* async errors use the same typed result model as synchronous errors
* structured concurrency is preferred
* child task lifetimes should normally be tied to their parent scope
* cancellation should propagate through task hierarchies
* detached background work should require explicit intent
* shared mutable state should be visible and protecte
