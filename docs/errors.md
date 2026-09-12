# yanawa error handling

yanawa should provide explicit, type-safe, and predictable error handling.

Errors are part of a function's behavior and should be visible whenever they are expected to occur during normal program execution.

The language should avoid forcing developers to use exceptions for ordinary control flow.

> Expected failures should be represented in the type system.

This document describes the current direction of error handling in yanawa.

Nothing here should be considered stable or final while the language remains experimental.

## core principles

Error handling in yanawa should be:

* explicit
* type-safe
* concise
* difficult to ignore accidentally
* readable at the call site
* easy to propagate
* separate from unrecoverable program failures
* integrated with pattern matching and the type system

The language should distinguish between:

* expected failures
* unexpected failures
* programmer errors

These categories should not necessarily use the same mechanism.

## expected failures

Operations that may reasonably fail during normal execution should expose that possibility in their return type.

Examples include:

* parsing user input
* reading a file
* querying a database
* performing network requests
* validating data
* looking up a missing value
* authentication failures

A possible result type is:

```text
result<T, E>
```

where:

* `T` represents a successful value
* `E` represents an error value

Example:

```yanawa
fn divide(a: int, b: int) -> result<int, str>:
    if b == 0:
        return error("Division by zero")

    return ok(a / b)
```

The function signature makes failure visible.

## success and error values

A result may contain either:

```text
ok(value)
```

or:

```text
error(value)
```

Example:

```yanawa
fn parse_age(value: str) -> result<int, str>:
    if not value.is_numeric():
        return error("Invalid age")

    return ok(int(value))
```

The exact naming and casing of `ok`, `error`, and `result` are still under discussion.

## handling results

Results should be handled explicitly.

Possible syntax:

```yanawa
let result = divide(10, 2)

match result:
    case ok(value):
        print(value)

    case error(message):
        print(message)
```

This makes both possible outcomes visible.

## pattern matching

Pattern matching should work naturally with typed results.

```yanawa
match fetch_user(id):
    case ok(user):
        print(user.name)

    case error(message):
        print("Could not fetch user: {message}")
```

The compiler should ideally verify that all relevant result cases are handled.

## error propagation

Explicitly matching every intermediate result can become repetitive.

yanawa may provide a propagation operator for forwarding an error to the caller.

Possible syntax:

```yanawa
let user = find_user(id)?
```

If `find_user(id)` succeeds:

```text
ok(user)
```

the value is assigned to `user`.

If it fails:

```text
error(reason)
```

the current function returns the error immediately.

Example:

```yanawa
fn load_profile(id: int) -> result<Profile, UserError>:
    let user = find_user(id)?
    let settings = load_settings(user.id)?

    return ok(Profile(
        user = user,
        settings = settings
    ))
```

This keeps propagation concise while preserving typed errors.

## propagation rules

The `?` operator should only be valid when the current function can return a compatible error type.

Example:

```yanawa
fn load_user(id: int) -> result<User, UserError>:
    let user = repository.find(id)?

    return ok(user)
```

The compiler should reject propagation when the error type cannot be returned safely.

This behavior should remain explicit in the function signature.

## custom error types

Applications should be able to define domain-specific errors.

Possible enum syntax:

```yanawa
enum UserError:
    not_found
    invalid_email
    database_failure(str)
```

A function may return:

```yanawa
fn find_user(id: int) -> result<User, UserError>:
    ...
```

Handling:

```yanawa
match find_user(id):
    case ok(user):
        print(user.name)

    case error(UserError.not_found):
        print("User not found")

    case error(UserError.invalid_email):
        print("Invalid email")

    case error(UserError.database_failure(message)):
        print("Database error: {message}")
```

The exact enum pattern syntax remains under design.

## structured errors

Errors should preferably carry structured information instead of relying only on strings.

Example:

```yanawa
struct ValidationError:
    field: str
    message: str
```

Then:

```yanawa
fn validate_email(email: str) -> result<str, ValidationError>:
    if not email.contains("@"):
        return error(ValidationError(
            field = "email",
            message = "Invalid email address"
        ))

    return ok(email)
```

Structured errors make it easier for applications, APIs, and tools to handle failures correctly.

## nullable values versus errors

Absence and failure should represent different concepts.

Use a nullable type when the absence of a value is normal.

```yanawa
fn find_cached_user(id: int) -> User?:
    ...
```

This means:

> A user may or may not exist in the cache.

Use a result when an operation itself may fail.

```yanawa
fn load_user(id: int) -> result<User, DatabaseError>:
    ...
```

This means:

> Loading the user may succeed or fail.

A function may potentially combine both concepts if required.

```yanawa
fn search_user(email: str) -> result<User?, DatabaseError>:
    ...
```

The database query may fail, while a successful query may still find no user.

## assertions

Assertions represent programmer assumptions rather than expected runtime failures.

Possible syntax:

```yanawa
assert age >= 0
```

Optional message:

```yanawa
assert age >= 0, "Age cannot be negative"
```

An assertion failure should normally indicate a programming error.

Assertions should not be used to validate ordinary user input or expected external failures.

## panic

yanawa may provide a mechanism for unrecoverable failures.

Possible syntax:

```yanawa
panic("Unexpected application state")
```

`panic` should be reserved for situations where continuing execution is not reasonable.

Examples may include:

* violated internal invariants
* impossible states
* corrupted runtime state
* unrecoverable initialization failures

The language should discourage using panic for ordinary application errors.

## exceptions

Traditional exception-based error handling is currently not the preferred direction for yanawa.

For example, the language should avoid requiring ordinary code to rely on:

```text
try
catch
throw
```

for expected failures.

This does not necessarily mean exceptions can never exist internally or for interoperability.

However, typed results should be the default model exposed to yanawa developers.

## interoperability

Future interoperability with other runtimes may require translating exceptions into yanawa errors.

For example, a Java exception or native error could potentially be converted into:

```text
result<T, ExternalError>
```

The boundary should prevent foreign exception behavior from leaking unpredictably into normal yanawa code.

The exact interoperability model remains undecided.

## application errors

Application layers should be able to translate domain errors into external representations.

For example:

```yanawa
fn get_user(id: int) -> response<User>:
    match users.find(id):
        case ok(user):
            return response.ok(user)

        case error(UserError.not_found):
            return response.not_found()

        case error(error):
            return response.internal_error(error)
```

The exact HTTP API is experimental.

The important principle is that domain errors should remain independent from transport concerns where possible.

## compiler diagnostics

Compiler errors are different from runtime application errors.

yanawa compiler diagnostics should explain:

* what went wrong
* where it happened
* what the compiler expected
* what it received
* possible relevant context

Example:

```text
error: expected `int`, found `str`

  --> example.yna:4:15
   |
 4 | let age: int = "twenty"
   |                ^^^^^^^^
```

Diagnostics should prefer useful explanations over cryptic error codes.

## error messages

Runtime errors should also be clear and structured.

A useful error should ideally answer:

* what failed
* why it failed
* where it failed
* whether it can be recovered from

Applications should not need to parse human-readable error strings to understand error categories.

## error composition

Larger applications may need to combine different error types.

Possible approaches include:

* enum-based application errors
* automatic compatible conversions
* explicit mapping
* generic error wrappers

Possible example:

```yanawa
enum AppError:
    database(DatabaseError)
    validation(ValidationError)
    network(NetworkError)
```

Then:

```yanawa
fn create_user(input: CreateUser) -> result<User, AppError>:
    let email = validate_email(input.email)?
    let user = repository.save(email)?

    return ok(user)
```

How error conversions interact with `?` remains an open design question.

## cleanup

Error propagation should not make resource cleanup unsafe.

Resources such as:

* files
* sockets
* database connections
* locks

must remain safely releasable when a function exits early.

The final resource-management model depends on yanawa's runtime and memory model and has not yet been designed.

## async errors

Asynchronous functions should use the same error model whenever possible.

Possible example:

```yanawa
async fn fetch_user(id: int) -> result<User, NetworkError>:
    let response = await http.get("/users/{id}")?

    return ok(response.json())
```

Async code should not require a completely separate error-handling system.

## example

```yanawa
enum LoginError:
    user_not_found
    invalid_password
    database_failure(str)

fn login(email: str, password: str) -> result<User, LoginError>:
    let user = users.find_by_email(email)?

    if user == none:
        return error(LoginError.user_not_found)

    if not user.password.matches(password):
        return error(LoginError.invalid_password)

    return ok(user)
```

Usage:

```yanawa
match login(email, password):
    case ok(user):
        print("Welcome, {user.name}")

    case error(LoginError.user_not_found):
        print("User not found")

    case error(LoginError.invalid_password):
        print("Invalid password")

    case error(LoginError.database_failure(message)):
        print("Database failure: {message}")
```

## current direction

The current preferred direction is:

* expected failures use typed results
* `result<T, E>` represents success or failure
* `ok(value)` represents success
* `error(value)` represents failure
* pattern matching handles results explicitly
* `?` may propagate compatible errors
* custom structured error types are preferred over strings
* nullable values represent absence, not failure
* assertions represent programmer assumptions
* panic is reserved for unrecoverable states
* exceptions are not the default application error model

## open questions

The following areas remain unresolved:

* exact spelling and casing of `result`, `ok`, and `error`
* whether result is a built-in type or standard library type
* exact `?` propagation semantics
* automatic error conversions
* stack traces
* panic behavior
* assertion behavior in production builds
* resource cleanup during propagation
* interaction with async code
* interoperability with exception-based runtimes
* exhaustive pattern matching requirements
* standard error interfaces or traits
* compiler diagnostic format

The error system should remain small, explicit, and consistent with the rest of the language.

> Failure should be visible without becoming ceremony.
