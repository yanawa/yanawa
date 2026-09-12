# yanawa standard library

The yanawa standard library should provide a small, coherent, predictable foundation for building real applications without forcing developers to depend on large frameworks for common tasks.

The standard library is not intended to contain every useful abstraction.

It should contain the functionality that most yanawa programs reasonably expect to have available, with stable APIs and consistent design.

> The standard library should remove friction without becoming a framework.

This document describes the current direction of the yanawa standard library.

Nothing here should be considered stable or final while the language remains experimental.

## core principles

The yanawa standard library should be:

* small enough to understand
* large enough to be useful
* predictable
* internally consistent
* type-safe
* platform-aware
* well documented
* difficult to misuse accidentally
* designed around common application needs
* independent from unnecessary framework conventions

The standard library should avoid becoming a collection of unrelated utilities.

Each module should have a clear responsibility and a strong reason to exist.

## language versus standard library

Not every common feature should become language syntax.

The language itself should primarily define:

* syntax
* types
* control flow
* functions
* structs
* enums
* traits
* modules
* error handling primitives
* memory semantics
* concurrency semantics

Functionality such as:

* files
* networking
* JSON
* HTTP
* clocks
* processes
* environment variables

should generally belong to the standard library or first-party packages rather than the core language.

For example:

```yanawa
import yanawa.fs

let content = fs.read_text("config.txt")?
```

is preferable to introducing dedicated filesystem syntax into the language.

## standard library versus first-party packages

Some features are useful enough to be maintained by the yanawa project but too specialized or too large for the standard library.

These may live as first-party packages.

Possible examples include:

* database drivers
* ORM or query tooling
* web frameworks
* template engines
* cryptographic protocols
* UI frameworks
* cloud integrations
* message brokers
* observability stacks

The distinction should remain intentional.

A feature should not enter the standard library only because it is popular.

## namespace

Standard library modules should use a recognizable namespace.

Current direction:

```yanawa
import yanawa.fs
import yanawa.http
import yanawa.json
import yanawa.time
```

This makes standard library dependencies easy to identify.

Possible benefits include:

* avoiding name collisions
* clear documentation organization
* predictable discovery
* consistent tooling
* obvious ownership by the language project

Whether some foundational modules are automatically available remains undecided.

## prelude

yanawa may provide a small prelude containing fundamental types and operations that are available without imports.

Possible prelude contents:

```text
str
char
bool
int
float
result
option
print
panic
assert
```

Fundamental collection types may also be considered.

The prelude should remain intentionally small.

Developers should not need to memorize a large set of implicitly imported APIs.

## module naming

Standard library modules should use short, clear names.

Possible initial modules:

```text
yanawa.collections
yanawa.env
yanawa.fs
yanawa.http
yanawa.io
yanawa.json
yanawa.math
yanawa.net
yanawa.process
yanawa.random
yanawa.testing
yanawa.time
```

Not all of these modules are guaranteed to exist.

They represent current areas worth exploring.

## collections

Collections are fundamental enough to likely belong in the standard library.

Possible core collection types include:

```text
Array<T>
List<T>
Map<K, V>
Set<T>
Queue<T>
```

However, yanawa should avoid providing multiple collection types with nearly identical behavior unless their semantic or performance differences are meaningful.

### arrays

The current syntax direction uses:

```yanawa
let names: str[] = ["Alex", "Morgan", "Taylor"]
```

This may represent a dynamic array-like collection.

Possible operations:

```yanawa
names.add("Jordan")
names.remove("Morgan")

print(names.length)
print(names[0])
```

The exact API remains under design.

### maps

Possible map syntax:

```yanawa
let scores: map<str, int> = {
    "Alex": 10,
    "Morgan": 15
}
```

Possible inference:

```yanawa
let scores = {
    "Alex": 10,
    "Morgan": 15
}
```

Usage:

```yanawa
scores["Taylor"] = 20

let score = scores["Alex"]
```

The exact map syntax and missing-key behavior remain undecided.

### sets

Possible API:

```yanawa
let tags: set<str> = set()

tags.add("compiler")
tags.add("language")
```

Membership checks should remain simple.

```yanawa
if "compiler" in tags:
    print("Found")
```

Whether `in` becomes a language operator remains undecided.

## iterators

Collections should expose a common iteration model.

Basic iteration:

```yanawa
for user in users:
    print(user.name)
```

Higher-level operations may include:

```yanawa
let active = users.filter(fn user:
    user.active
)
```

or possibly:

```yanawa
let active = users.filter(user -> user.active)
```

Lambda syntax has not yet been defined.

Potential iterator operations include:

* `map`
* `filter`
* `find`
* `any`
* `all`
* `count`
* `reduce`
* `take`
* `skip`
* `enumerate`

These APIs should use consistent naming and avoid unnecessary duplication.

## io

The `yanawa.io` module may provide fundamental input and output functionality.

Possible examples:

```yanawa
import yanawa.io

io.print("Hello")
io.println("World")
```

However, basic `print` may remain available through the prelude.

Console input may look like:

```yanawa
let name = io.read_line()
```

I/O operations that may fail should use typed results.

```yanawa
let line = io.read_line()?
```

The standard library should avoid APIs that silently ignore I/O failures.

## filesystem

Filesystem functionality may live in:

```yanawa
import yanawa.fs
```

Reading text:

```yanawa
let content = fs.read_text("config.txt")?
```

Writing text:

```yanawa
fs.write_text(
    "output.txt",
    content
)?
```

Checking existence:

```yanawa
if fs.exists("config.txt"):
    ...
```

Directories:

```yanawa
fs.create_dir("data")?
```

Possible iteration:

```yanawa
for file in fs.read_dir("data")?:
    print(file.name)
```

Filesystem APIs should use platform-independent path abstractions where practical.

## paths

Paths should not be represented purely as arbitrary strings if a dedicated type improves correctness.

Possible API:

```yanawa
let path = Path("data/users.json")
```

Composition:

```yanawa
let file = path.join("archive")
```

Possible string-like convenience should not compromise platform correctness.

## json

JSON support is common enough in modern applications that it may belong in the standard library.

Possible module:

```yanawa
import yanawa.json
```

Encoding:

```yanawa
let value = json.encode(user)?
```

Decoding:

```yanawa
let user = json.decode<User>(body)?
```

The type system should ideally allow structured decoding without annotation-heavy models.

Example:

```yanawa
struct User:
    id: int
    name: str
```

Then:

```yanawa
let user = json.decode<User>(input)?
```

yanawa should avoid requiring boilerplate serialization metadata for ordinary cases.

Customization may still be necessary for:

* renamed fields
* ignored fields
* custom formats
* version compatibility

The customization model remains undecided.

## serialization

JSON should not necessarily define yanawa's entire serialization model.

A more general abstraction may eventually support:

* JSON
* binary formats
* configuration formats
* network protocols

Possible trait:

```yanawa
trait Encode<T>:
    fn encode(value: T) -> result<bytes, EncodeError>
```

This area should remain small until real requirements emerge.

## bytes

Binary data should have a dedicated representation.

Possible type:

```text
bytes
```

Example:

```yanawa
let data: bytes = fs.read("image.png")?
```

Binary values should not be confused with strings.

Conversions between text and bytes should require an explicit encoding.

Possible API:

```yanawa
let bytes = text.encode("utf-8")?
let text = bytes.decode("utf-8")?
```

Whether UTF-8 is the default string encoding remains a separate language-design decision.

## time

Time APIs should be designed carefully because date and time handling is inherently complex.

Possible module:

```yanawa
import yanawa.time
```

Current time:

```yanawa
let now = time.now()
```

Duration:

```yanawa
let timeout = 5s
```

or:

```yanawa
let timeout = Duration.seconds(5)
```

Possible operations:

```yanawa
let later = now + 5m
```

The standard library should distinguish concepts such as:

* instant
* duration
* local date
* local time
* timezone-aware datetime

It should avoid ambiguous date and time APIs.

## duration literals

Duration literals may improve readability.

Possible syntax:

```text
500ms
5s
10m
2h
```

This syntax should only become part of the language if it provides enough value beyond a library API.

Until then, durations may remain standard library values.

## environment

Environment variable access may live in:

```yanawa
import yanawa.env
```

Example:

```yanawa
let database_url = env.get("DATABASE_URL")
```

Since variables may not exist:

```yanawa
let database_url: str? = env.get("DATABASE_URL")
```

Required variable:

```yanawa
let database_url = env.require("DATABASE_URL")?
```

Environment APIs should distinguish between absence and runtime failure.

## process

Process management may live in:

```yanawa
import yanawa.process
```

Command execution:

```yanawa
let result = process.run(
    "git",
    ["status"]
)?
```

Possible result:

```yanawa
struct ProcessResult:
    exit_code: int
    stdout: str
    stderr: str
```

Long-running child processes may require a separate API.

Security-sensitive shell interpolation should not be the default.

## command arguments

Applications should have a straightforward way to access command-line arguments.

Possible API:

```yanawa
import yanawa.process

let args = process.args()
```

or a dedicated:

```yanawa
import yanawa.cli
```

A full argument parser may belong to a first-party package rather than the core standard library.

## networking

Low-level networking may live in:

```yanawa
import yanawa.net
```

Possible abstractions:

* TCP sockets
* UDP sockets
* addresses
* DNS
* TLS integration boundaries

Example:

```yanawa
let socket = await net.tcp.connect("example.com", 443)?
```

Networking should integrate naturally with yanawa's async model.

## http

HTTP is an important design question for yanawa.

The language aims to support productive application development without requiring a large framework.

However, HTTP should not necessarily become core language syntax.

A standard library or first-party module may provide:

```yanawa
import yanawa.http
```

Client request:

```yanawa
let response = await http.get(
    "https://example.com/users"
)?
```

Parsing JSON:

```yanawa
let users = response.json<User[]>()?
```

Server creation may look like:

```yanawa
let app = http.server()
```

Routes:

```yanawa
app.get("/users", list_users)
app.post("/users", create_user)
```

Another possible direction remains the concise routing syntax currently explored in `syntax.md`.

The final decision should consider whether HTTP routing deserves special language syntax or whether a well-designed library can remain equally concise.

## http responses

A first-party HTTP API should provide typed response helpers.

Possible examples:

```yanawa
return response.ok(user)
```

```yanawa
return response.created(user)
```

```yanawa
return response.not_found()
```

or:

```yanawa
return Response(
    status = 200,
    body = user
)
```

The API should avoid requiring developers to memorize large amounts of framework-specific behavior.

## testing

Testing should be treated as first-party tooling.

A basic testing module may live in:

```yanawa
import yanawa.testing
```

Possible syntax:

```yanawa
test "adds two numbers":
    expect(add(2, 3)).to_equal(5)
```

or:

```yanawa
fn test_addition():
    assert add(2, 3) == 5
```

Whether `test` becomes language syntax or remains a tooling convention is undecided.

The test runner itself should be part of the official toolchain.

Possible command:

```text
yanawa test
```

## assertions

Fundamental assertions may be available globally.

```yanawa
assert user.active
```

Message:

```yanawa
assert user.active, "User must be active"
```

Testing-specific assertions may provide better diagnostics.

```yanawa
expect(value).to_equal(expected)
```

The library should avoid excessive matcher APIs unless they clearly improve failure messages.

## math

Mathematical functionality may live in:

```yanawa
import yanawa.math
```

Possible APIs:

```yanawa
math.sqrt(value)
math.abs(value)
math.min(a, b)
math.max(a, b)
math.floor(value)
math.ceil(value)
```

Constants:

```yanawa
math.PI
math.E
```

Basic arithmetic operators remain part of the language itself.

## random

Random value generation may live in:

```yanawa
import yanawa.random
```

Example:

```yanawa
let value = random.int(1, 100)
```

Security-sensitive randomness must be clearly separated from ordinary pseudo-random generation.

Possible dedicated API:

```yanawa
let token = random.secure_bytes(32)?
```

The library should prevent developers from accidentally using weak randomness for security-sensitive purposes.

## text

String functionality may belong directly to `str` rather than a separate module.

Possible operations:

```yanawa
name.length
name.contains("ana")
name.starts_with("A")
name.ends_with("z")
name.trim()
name.upper()
name.lower()
```

String APIs should be Unicode-aware by default where practical.

The exact string indexing model requires careful design.

## unicode

yanawa should treat Unicode as a first-class concern.

The standard library must clearly define:

* what `char` represents
* whether string indexing operates on bytes, code points, or grapheme clusters
* how lengths are measured
* normalization behavior
* case conversion

The language should avoid pretending that Unicode text behaves like ASCII.

## formatting

Text formatting should integrate with string interpolation.

Basic:

```yanawa
print("Hello, {name}")
```

Formatting options may eventually look like:

```yanawa
print("Total: {price:.2}")
```

The exact formatting grammar remains undecided.

A formatter API should remain consistent across logging, strings, and diagnostics.

## logging

Logging may belong in the standard library or a first-party package.

Possible module:

```yanawa
import yanawa.log
```

Usage:

```yanawa
log.info("Server started")
log.warn("Connection is slow")
log.error("Could not connect: {error}")
```

Structured logging may support fields:

```yanawa
log.info(
    "User created",
    user_id = user.id
)
```

The standard library should avoid locking applications into a rigid logging backend.

## configuration

Configuration management is common but may be too opinionated for the standard library.

Potential first-party support could integrate:

* environment variables
* files
* command-line arguments
* typed configuration structs

Possible example:

```yanawa
struct Config:
    port: int
    database_url: str

let config = config.load<Config>()?
```

This likely belongs in a first-party package rather than the smallest possible standard library.

## database access

Database access is central to the original yanawa vision, but database drivers should probably not live directly in the core standard library.

Possible first-party packages:

```text
yanawa-postgres
yanawa-mysql
yanawa-sqlite
```

or a unified database abstraction.

Usage might eventually look like:

```yanawa
import postgres

let database = await postgres.connect(
    env.require("DATABASE_URL")?
)?
```

The language should avoid requiring a large ORM merely to execute safe database queries.

## sql

A first-party SQL API may eventually provide:

```yanawa
let user = await database.query_one<User>(
    "select id, name, email from users where id = ?",
    id
)?
```

Typed query systems may be explored later.

Compile-time SQL validation should only be considered if it can remain understandable and reliable.

## validation

Validation is common enough to deserve first-party attention but may not belong in the core standard library.

Possible API:

```yanawa
let email = validate.email(input.email)?
```

or validation through ordinary functions and typed errors.

The language should avoid annotation-heavy validation models by default.

## cryptography

Cryptographic primitives are security-sensitive.

Low-level cryptography should not be casually exposed through overly convenient APIs that encourage misuse.

The standard library may provide safe primitives for common needs such as:

* cryptographically secure randomness
* hashing
* password hashing
* TLS through networking APIs

Complex cryptographic APIs may belong in specialized first-party packages.

## security

Standard library APIs should prefer secure defaults.

Examples include:

* safe process execution
* secure random generation
* no implicit shell evaluation
* path handling that avoids accidental traversal where possible
* explicit encoding conversions
* TLS verification enabled by default

Security-sensitive behavior should require explicit intent to weaken.

## operating system integration

Platform-specific functionality may be exposed through a dedicated namespace.

Possible:

```yanawa
import yanawa.os
```

Examples:

```yanawa
os.name
os.arch
os.home_dir()
os.temp_dir()
```

Portable abstractions should be preferred over direct operating-system dependencies.

Platform-specific APIs should be clearly identified.

## platform differences

The standard library should strive for consistent behavior across supported platforms.

When behavior differs, the difference should be documented and visible.

The standard library should not pretend an API is portable when it is not.

Possible conditional capabilities may eventually be necessary.

## errors

All fallible standard library APIs should integrate with the yanawa error model.

Example:

```yanawa
fn read_text(path: Path) -> result<str, FsError>
```

Error types should be structured.

Possible:

```yanawa
enum FsError:
    not_found(Path)
    permission_denied(Path)
    invalid_path(Path)
    io_failure(str)
```

Applications should not need to parse error-message strings to understand failures.

## async consistency

APIs that perform potentially blocking I/O should integrate with the concurrency model.

Examples:

```yanawa
let response = await http.get(url)?
let content = await file.read()?
```

Whether filesystem APIs are synchronous, asynchronous, or offer both forms depends on the runtime model.

The standard library should avoid inconsistent naming such as unrelated `Async*` variants unless necessary.

## resource management

Resources opened through the standard library should integrate with the language resource model.

Example:

```yanawa
let file = fs.open("data.txt")?
```

The language should ensure the resource can be released safely.

Potential mechanisms include:

* deterministic cleanup
* scoped resources
* `defer`
* automatic runtime cleanup

The final API depends on the memory model.

## API naming

Standard library APIs should use consistent naming conventions.

Preferred qualities include:

* verbs for actions
* nouns for values and types
* no unnecessary abbreviations
* predictable pairs of operations

Examples:

```text
read
write
open
close
create
remove
find
contains
parse
encode
decode
```

Avoid arbitrary differences such as:

```text
delete
erase
destroy
remove
```

unless they represent genuinely different operations.

## constructors

Types should prefer simple construction when no validation is required.

```yanawa
let path = Path("users.json")
```

Operations that may fail should make failure explicit.

```yanawa
let address = IpAddress.parse(value)?
```

Construction should not hide expected runtime failure.

## naming consistency

Similar modules should use similar APIs.

For example, if parsing is represented by:

```yanawa
Type.parse(value)
```

then unrelated modules should not arbitrarily use:

```yanawa
Type.from_string(value)
```

without a semantic reason.

Consistency should reduce the amount of API knowledge developers need to memorize.

## discoverability

Standard library APIs should be easy to discover through:

* editor completion
* generated documentation
* consistent naming
* predictable module structure
* clear examples

Developers should not need to search external tutorials for basic operations.

## documentation

Every public standard library API should eventually provide:

* concise description
* parameter documentation
* return type behavior
* error behavior
* examples
* platform limitations where relevant

Documentation should be generated from or closely associated with source code where practical.

## stability

The standard library should have stronger compatibility expectations than ordinary third-party packages.

Once yanawa reaches stable releases, breaking standard library changes should require deliberate versioning and migration planning.

Before stability, experimentation remains expected.

## versioning

The standard library should normally evolve with the language version.

For example:

```text
yanawa 1.4
```

would imply a known standard library version.

Developers should not normally need to manage the core standard library as an independent dependency.

First-party packages may follow separate versioning.

## dependencies

The core standard library should minimize external dependencies.

Reasons include:

* security
* reproducibility
* portability
* long-term maintenance
* stable behavior

Where external implementations are necessary, they should remain hidden behind stable yanawa APIs whenever possible.

## implementation independence

Standard library behavior should be specified independently from a particular internal implementation when possible.

For example:

```yanawa
let users = fs.read_text("users.json")?
```

should retain its meaning even if the internal runtime implementation changes.

Public semantics should not depend unnecessarily on compiler internals.

## standard library size

The standard library should resist uncontrolled growth.

Before adding a new module, ask:

* Is this needed by a large portion of yanawa programs?
* Can a third-party package provide it effectively?
* Does it require language-level integration?
* Can yanawa commit to maintaining this API long-term?
* Does the feature have a clear and stable abstraction?
* Would excluding it force developers into unnecessary framework dependence?

A standard library feature should earn its place.

## possible initial standard library

A minimal useful first standard library may eventually contain:

```text
yanawa.collections
yanawa.env
yanawa.fs
yanawa.io
yanawa.math
yanawa.process
yanawa.random
yanawa.time
```

Depending on the execution model, it may also include:

```text
yanawa.json
yanawa.net
yanawa.http
```

These modules are candidates, not commitments.

## possible first-party ecosystem

Features that may be better maintained outside the core standard library include:

```text
database drivers
database migrations
query builders
web frameworks
validation
configuration loaders
logging backends
CLI frameworks
template engines
cryptography packages
observability
serialization formats beyond core needs
```

These packages can still be maintained officially by the yanawa project.

## application example

A future yanawa application may look like:

```yanawa
import yanawa.env
import yanawa.http
import yanawa.json

struct User:
    id: int
    name: str
    email: str

async fn find_user(id: int) -> result<User?, AppError>:
    ...

get "/users/{id}" async fn user(id: int) -> response<User>:
    let user = await find_user(id)?

    if user == none:
        return response.not_found()

    return response.ok(user)

async fn main():
    let port = env.get("PORT") ?? "8080"

    await http.serve(
        port = int(port)
    )
```

The exact HTTP syntax remains experimental.

The important goal is that common application infrastructure should feel coherent without requiring a large external framework.

## current direction

The current preferred direction is:

* the standard library should remain small and coherent
* functionality should not become language syntax without a strong reason
* common low-level application capabilities should be available without large frameworks
* standard library modules use the `yanawa` namespace
* a small prelude may provide fundamental types and functions
* collections, I/O, filesystem, time, environment, processes, and basic utilities are strong stdlib candidates
* JSON, networking, and HTTP are strong candidates but need further design
* database tooling should likely begin as first-party packages
* testing should be first-party and deeply integrated with tooling
* standard library errors should be typed and structured
* secure defaults should be preferred
* APIs should remain consistent, predictable, and discoverable
* first-party packages may extend the ecosystem without expanding the core standard library indefinitely

## open questions

The following areas remain unresolved:

* exact prelude contents
* final standard library module list
* collection type hierarchy
* string and Unicode model
* path abstraction
* synchronous versus asynchronous filesystem APIs
* JSON serialization customization
* general serialization model
* HTTP client design
* HTTP server design
* whether routing receives special syntax
* testing syntax
* process API
* logging ownership
* configuration ownership
* database abstraction boundaries
* duration literals
* cryptography boundaries
* platform-specific APIs
* standard library dependency policy
* versioning policy
* compatibility guarantees
* documentation generation
* whether some modules should instead become first-party packages

The standard library should grow from real language needs rather than assumptions about what a complete ecosystem is supposed to contain.

> Small enough to stay coherent. Complete enough to build real things.
