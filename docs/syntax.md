# yanawa syntax

This document explores the current syntax direction of yanawa.

The syntax is experimental and may change as the language evolves.

The current goal is to keep yanawa familiar, strongly typed, concise, and predictable.

> Familiar syntax, strong typing, almost no ceremony.

## blocks

yanawa uses indentation to define blocks.

A colon introduces a block.

```yanawa
if active:
    print("Active")
```

Braces are not required.

```yanawa
fn greet():
    print("Hello")
```

Semicolons are not required.

## variables

Mutable variables are declared with `let`.

```yanawa
let name: str = "Alex"
let age: int = 27
let active: bool = true
```

When the type is obvious, it may be inferred.

```yanawa
let name = "Alex"
let age = 27
let active = true
```

Variables declared with `let` may be reassigned.

```yanawa
let score = 0

score = 10
```

## constants

Immutable constant values are declared with `const`.

```yanawa
const MAX_RETRIES: int = 3
const PI = 3.14159
const APP_NAME = "yanawa"
```

`const let` is intentionally not supported because the combination would be redundant.

## primitive types

Possible built-in primitive types include:

```text
str
int
float
bool
char
```

Example:

```yanawa
let username: str = "Taylor"
let age: int = 31
let price: float = 19.90
let active: bool = true
let initial: char = 'T'
```

The final primitive type set has not yet been decided.

## functions

Functions are declared with `fn`.

```yanawa
fn greet(name: str) -> str:
    return "Hello, {name}"
```

Functions without a return value do not need a return type.

```yanawa
fn greet(name: str):
    print("Hello, {name}")
```

Parameters use `name: type` syntax.

```yanawa
fn add(a: int, b: int) -> int:
    return a + b
```

## default parameters

Parameters may define default values.

```yanawa
fn greet(name: str, greeting: str = "Hello") -> str:
    return "{greeting}, {name}"
```

Usage:

```yanawa
greet("Morgan")

greet(
    name = "Morgan",
    greeting = "Welcome"
)
```

## return values

The current syntax uses explicit `return`.

```yanawa
fn square(value: int) -> int:
    return value * value
```

Implicit final-expression returns are still being considered.

Possible alternative:

```yanawa
fn square(value: int) -> int:
    value * value
```

No final decision has been made.

## string interpolation

Expressions may be embedded directly inside strings.

```yanawa
let name = "Jordan"
let age = 24

print("{name} is {age} years old")
```

This should be preferred over unnecessary concatenation.

## conditions

Conditions do not require parentheses.

```yanawa
if age >= 18:
    print("Adult")
elif age >= 13:
    print("Teenager")
else:
    print("Child")
```

String values use normal equality operators.

```yanawa
if name == "Jordan":
    print("Welcome back")
```

There is no need for methods such as `.equals()` for ordinary value comparison.

## logical operators

yanawa uses readable logical operators.

```yanawa
if age >= 18 and active:
    print("Allowed")

if admin or moderator:
    print("Access granted")

if not banned:
    print("Welcome")
```

Possible operators:

```text
and
or
not
```

## switch expressions

`switch` may produce a value.

```yanawa
let status = switch code:
    case 200: "OK"
    case 404: "Not Found"
    case 500: "Server Error"
    default: "Unknown"
```

Multiline cases are also allowed.

```yanawa
let result = switch operation:
    case "+":
        value1 + value2

    case "-":
        value1 - value2

    case "*":
        value1 * value2

    default:
        0
```

Cases do not require `break`.

## arrays

Arrays place the element type before `[]`.

```yanawa
let students: str[] = ["Alex", "Morgan", "Taylor"]
```

Type inference may be used.

```yanawa
let students = ["Alex", "Morgan", "Taylor"]
```

Empty arrays require enough type information for the compiler to determine their element type.

```yanawa
let students: str[] = []
```

Possible fixed-size syntax:

```yanawa
let values: int[3]
```

This syntax is still experimental.

## array operations

Possible collection operations:

```yanawa
students.add("Jordan")
students.remove("Taylor")

print(students.length)
print(students[0])
```

Possible slicing syntax:

```yanawa
let first_three = students[0:3]
```

The exact collection API has not yet been designed.

## loops

Collections may be iterated directly.

```yanawa
for student in students:
    print(student)
```

Ranges:

```yanawa
for i in 0..10:
    print(i)
```

A possible index-and-value form:

```yanawa
for index, student in students:
    print("{index}: {student}")
```

While loops:

```yanawa
let attempts = 0

while attempts < 3:
    attempts += 1
```

## nullability

Values should be non-nullable by default.

```yanawa
let name: str = "Casey"
```

Nullable types use `?`.

```yanawa
let nickname: str? = none
```

`str` and `str?` are different types.

Checking for absence:

```yanawa
if nickname != none:
    print(nickname)
```

Possible fallback operator:

```yanawa
print(nickname ?? "Unknown")
```

The exact behavior of nullable values is still being explored.

## structs

Simple data structures are declared with `struct`.

```yanawa
struct User:
    id: int
    name: str
    email: str
```

Instances may use named fields.

```yanawa
let user = User(
    id = 1,
    name = "Riley",
    email = "riley@example.com"
)
```

Fields may define defaults.

```yanawa
struct User:
    id: int
    name: str
    active: bool = true
```

No explicit constructor should be required for ordinary structures.

## methods

Methods may be declared inside structs.

```yanawa
struct User:
    name: str
    age: int

    fn greet() -> str:
        return "Hello, I'm {name}"

    fn adult() -> bool:
        return age >= 18
```

Usage:

```yanawa
print(user.greet())

if user.adult():
    print("Adult")
```

Whether fields should require explicit `self` access is still undecided.

Possible explicit form:

```yanawa
return self.name
```

Possible concise form:

```yanawa
return name
```

## visibility

Declarations are currently considered public by default.

```yanawa
fn greet():
    print("Hello")
```

Private declarations use `private`.

```yanawa
private fn calculate_hash():
    ...
```

Private fields:

```yanawa
struct User:
    name: str
    private password: str
```

The visibility model may evolve if additional scopes become necessary.

## enums

Possible enum syntax:

```yanawa
enum Status:
    pending
    running
    completed
    failed
```

Usage:

```yanawa
let status = Status.pending
```

Enums with associated values are being considered.

```yanawa
enum Result<T, E>:
    ok(T)
    error(E)
```

## pattern matching

Possible pattern matching syntax:

```yanawa
match result:
    case ok(value):
        print(value)

    case error(message):
        print(message)
```

It is still undecided whether yanawa should provide both `switch` and `match`, or whether one construct should handle both use cases.

## generics

Possible generic syntax:

```yanawa
struct Box<T>:
    value: T
```

Generic functions:

```yanawa
fn first<T>(items: T[]) -> T?:
    if items.length == 0:
        return none

    return items[0]
```

The generic type system has not yet been designed.

## traits

yanawa may prefer traits or interfaces over traditional inheritance-heavy models.

Possible syntax:

```yanawa
trait Printable:
    fn display() -> str
```

Possible implementation:

```yanawa
impl Printable for User:
    fn display() -> str:
        return name
```

This area is still exploratory.

## error handling

yanawa should prefer explicit and type-safe error handling.

A possible result type:

```yanawa
fn divide(a: int, b: int) -> result<int, str>:
    if b == 0:
        return error("Division by zero")

    return ok(a / b)
```

Handling:

```yanawa
let result = divide(10, 2)

match result:
    case ok(value):
        print(value)

    case error(message):
        print(message)
```

A propagation operator may be introduced later.

Possible syntax:

```yanawa
let user = find_user(id)?
```

## async

Possible asynchronous syntax:

```yanawa
async fn fetch_user(id: int) -> User:
    let response = await http.get("/users/{id}")

    return response.json()
```

Usage:

```yanawa
let user = await fetch_user(1)
```

The concurrency model has not yet been defined.

## imports

Possible module import syntax:

```yanawa
import yanawa.http
import app.user.User
```

Possible grouped imports:

```yanawa
import app.user.{User, UserService}
```

The module and package system is not yet defined.

## http

yanawa is exploring first-class or first-party support for common application development tasks without relying on annotation-heavy frameworks.

Possible HTTP routing syntax:

```yanawa
get "/users" fn list_users() -> User[]:
    return users
```

Route parameters:

```yanawa
get "/users/{id}" fn find_user(id: int) -> User?:
    return users.find(id)
```

POST:

```yanawa
post "/users" fn create_user(body: CreateUser) -> User:
    return users.create(body)
```

Possible server startup:

```yanawa
fn main():
    server.start(port = 8080)
```

Whether HTTP belongs in the language syntax, standard library, or a first-party package is not yet decided.

## example

```yanawa
import yanawa.http

struct User:
    id: int
    name: str
    email: str

let users: User[] = []

get "/users" fn list_users() -> User[]:
    return users

get "/users/{id}" fn find_user(id: int) -> User?:
    for user in users:
        if user.id == id:
            return user

    return none

post "/users" fn create_user(body: User) -> User:
    users.add(body)

    return body

fn main():
    server.start(port = 8080)
```

## open questions

The following syntax decisions remain open:

* explicit or implicit returns
* exact primitive type set
* public or private visibility by default
* whether `self` is required inside methods
* classes, structs, traits, or some combination
* inheritance support
* `switch`, `match`, or both
* exact nullable type behavior
* generic syntax and constraints
* module and import system
* error handling model
* async and concurrency model
* HTTP integration
* standard library boundaries

Nothing in this document should be considered stable or final.
