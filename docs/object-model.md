# yanawa object model

yanawa should provide a simple object model focused on composition, clear data structures, and predictable behavior.

The language should avoid requiring traditional object-oriented patterns when simpler abstractions are enough.

> Data should be easy to model, and behavior should stay close to the data it belongs to.

This document describes the current direction of structs, methods, traits, composition, visibility, and object behavior in yanawa.

Nothing here should be considered stable or final while the language remains experimental.

## core principles

The yanawa object model should prefer:

* simple data structures
* composition over inheritance
* explicit behavior
* minimal boilerplate
* predictable construction
* clear ownership of methods
* reusable behavior through traits or similar abstractions
* immutable design where practical
* inheritance only if a strong use case justifies it

The object model should not require developers to create classes merely to group fields together.

## structs

`struct` is the primary construct for defining structured data.

```yanawa
struct User:
    id: int
    name: str
    email: str
```

A struct defines a named type with a fixed set of fields.

Instances may be created using named arguments.

```yanawa
let user = User(
    id = 1,
    name = "Morgan",
    email = "morgan@example.com"
)
```

Named construction improves readability and reduces dependence on parameter order.

## default values

Struct fields may provide default values.

```yanawa
struct User:
    id: int
    name: str
    active: bool = true
```

Construction may omit fields with defaults.

```yanawa
let user = User(
    id = 1,
    name = "Taylor"
)
```

The compiler should initialize:

```text
active = true
```

automatically.

## required fields

Fields without default values are required during construction.

```yanawa
struct Server:
    host: str
    port: int
```

This should be invalid:

```yanawa
let server = Server(
    host = "localhost"
)
```

The compiler should report that `port` is missing.

## methods

Structs may define methods directly.

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
let user = User(
    name = "Alex",
    age = 28
)

print(user.greet())
```

Methods should behave like functions associated with a specific type.

## field access inside methods

yanawa may allow fields to be referenced directly inside methods.

```yanawa
struct User:
    name: str

    fn greet() -> str:
        return "Hello, {name}"
```

An explicit form may still exist:

```yanawa
return "Hello, {self.name}"
```

The current preferred direction is to allow direct access when there is no ambiguity.

Explicit `self` may be required when:

* a parameter shadows a field
* a local variable has the same name
* explicitness improves clarity

Example:

```yanawa
struct User:
    name: str

    fn rename(name: str):
        self.name = name
```

The exact rule remains under design.

## mutation

Methods may mutate struct fields when the instance itself is mutable.

Possible example:

```yanawa
struct Counter:
    value: int = 0

    fn increment():
        value += 1
```

Usage:

```yanawa
let counter = Counter()

counter.increment()
```

The interaction between `let`, mutability, and struct fields still needs a formal model.

Possible questions include:

* are all `let` instances mutable?
* can individual fields be immutable?
* should methods declare when they mutate state?

## immutable fields

yanawa may eventually support immutable fields.

Possible syntax:

```yanawa
struct User:
    const id: int
    name: str
```

or:

```yanawa
struct User:
    readonly id: int
    name: str
```

No final syntax has been chosen.

A simpler model may instead make entire values immutable depending on how they are declared.

## construction

Ordinary structs should not require explicit constructors.

```yanawa
struct Point:
    x: int
    y: int
```

The language should automatically provide a natural construction syntax.

```yanawa
let point = Point(
    x = 10,
    y = 20
)
```

This removes the need for boilerplate constructors for simple types.

## custom initialization

Some types may require validation or initialization logic.

yanawa may support explicit factory functions.

```yanawa
struct Email:
    value: str

fn Email.parse(value: str) -> result<Email, EmailError>:
    if not value.contains("@"):
        return error(EmailError.invalid)

    return ok(Email(value = value))
```

Usage:

```yanawa
let email = Email.parse("user@example.com")?
```

Whether constructors themselves can contain validation remains undecided.

## associated functions

Types may define functions that do not require an instance.

Possible syntax:

```yanawa
struct User:
    id: int
    name: str

    fn create(name: str) -> User:
        return User(
            id = 0,
            name = name
        )
```

Usage could be:

```yanawa
let user = User.create("Morgan")
```

The language may distinguish instance methods from type-level functions automatically based on whether instance fields are accessed.

Alternatively, an explicit modifier such as `static` may exist.

This remains undecided.

## composition

yanawa should prefer composition over inheritance.

Example:

```yanawa
struct Address:
    city: str
    country: str

struct User:
    name: str
    address: Address
```

Usage:

```yanawa
let user = User(
    name = "Riley",
    address = Address(
        city = "Tokyo",
        country = "Japan"
    )
)

print(user.address.city)
```

This keeps relationships explicit and avoids unnecessary type hierarchies.

## inheritance

Traditional class inheritance is not currently part of the preferred direction.

yanawa should not introduce inheritance only because object-oriented languages commonly provide it.

Before inheritance is added, the language should determine whether the same problems can be solved clearly through:

* composition
* traits
* generics
* enums
* delegation

If inheritance is eventually introduced, it should remain limited and predictable.

## traits

Traits may define reusable behavior contracts.

Possible syntax:

```yanawa
trait Printable:
    fn display() -> str
```

A type may implement a trait.

```yanawa
struct User:
    name: str

impl Printable for User:
    fn display() -> str:
        return name
```

Usage:

```yanawa
fn print_value<T: Printable>(value: T):
    print(value.display())
```

Traits should focus on behavior rather than storing state.

## trait requirements

Traits may require multiple methods.

```yanawa
trait Repository<T>:
    fn find(id: int) -> result<T?, RepositoryError>
    fn save(value: T) -> result<T, RepositoryError>
```

Implementation:

```yanawa
impl Repository<User> for UserRepository:
    fn find(id: int) -> result<User?, RepositoryError>:
        ...

    fn save(value: User) -> result<User, RepositoryError>:
        ...
```

The exact generic trait syntax remains under design.

## default trait methods

Traits may eventually provide default implementations.

Possible syntax:

```yanawa
trait Printable:
    fn display() -> str

    fn print():
        print(display())
```

Whether this should be allowed depends on how much complexity it introduces.

## multiple traits

A type should likely be able to implement multiple traits.

```yanawa
impl Printable for User:
    ...

impl Serializable for User:
    ...
```

This can provide reusable capabilities without multiple inheritance.

## trait composition

Generic constraints may combine traits.

Possible syntax:

```yanawa
fn process<T: Printable + Serializable>(value: T):
    ...
```

The exact syntax remains undecided.

## polymorphism

yanawa should support polymorphism without requiring deep inheritance hierarchies.

A function accepting a trait could work with any compatible implementation.

```yanawa
fn render(value: Printable):
    print(value.display())
```

or through generics:

```yanawa
fn render<T: Printable>(value: T):
    print(value.display())
```

Whether trait values use dynamic dispatch, static dispatch, or both depends on the runtime and compiler architecture.

## enums as data models

Some problems traditionally modeled through inheritance may be better represented with enums.

Example:

```yanawa
enum Shape:
    circle(radius: float)
    rectangle(width: float, height: float)
```

Then:

```yanawa
fn area(shape: Shape) -> float:
    return match shape:
        case circle(radius):
            PI * radius * radius

        case rectangle(width, height):
            width * height
```

This can model closed sets of variants more clearly than subclass hierarchies.

## visibility

Struct fields and methods follow the language visibility model.

Current direction:

```yanawa
struct User:
    name: str
    private password: str
```

Private methods:

```yanawa
struct User:
    password: str

    private fn hash_password() -> str:
        ...
```

The exact scope of `private` is still being defined.

## encapsulation

yanawa should support encapsulation without forcing boilerplate getters and setters.

A private field should remain inaccessible directly.

```yanawa
struct Account:
    private balance: float
```

A method may expose meaningful behavior instead.

```yanawa
struct Account:
    private balance: float

    fn deposit(amount: float):
        balance += amount

    fn current_balance() -> float:
        return balance
```

The language should prefer behavior-oriented APIs over mechanical getter/setter generation.

## properties

Dedicated property syntax is not currently considered necessary.

Instead of:

```text
get balance
set balance
```

ordinary fields and methods should handle most use cases.

Property syntax should only be introduced if real usage demonstrates a meaningful benefit.

## equality

yanawa may generate equality behavior automatically for simple structs.

Possible example:

```yanawa
let first = Point(x = 1, y = 2)
let second = Point(x = 1, y = 2)

if first == second:
    print("Equal")
```

An open question is whether structural equality should be automatic for all structs or require explicit support.

## string representation

Types may eventually define how they are represented as strings.

Possible trait:

```yanawa
trait Display:
    fn display() -> str
```

Example:

```yanawa
impl Display for User:
    fn display() -> str:
        return "User(name = {name})"
```

Tooling may provide a default debug representation independently.

## copying and references

The semantics of assigning or passing structs depend on yanawa's memory model.

Example:

```yanawa
let first = user
let second = first
```

It is not yet decided whether this means:

* copying the value
* sharing a reference
* moving ownership
* runtime-managed references

The object model should not decide this independently from the memory model.

## identity versus value

Some types may represent values while others represent entities with identity.

Example value-like type:

```yanawa
struct Point:
    x: int
    y: int
```

Example entity-like type:

```yanawa
struct User:
    id: int
    name: str
```

Whether the language needs separate constructs for reference and value semantics remains an open question.

The preferred direction is to keep the model small unless real requirements justify additional concepts.

## destructuring

Structs may support destructuring.

Possible syntax:

```yanawa
let user = User(
    id = 1,
    name = "Alex"
)

let User(id, name) = user
```

or:

```yanawa
let { id, name } = user
```

No syntax has been chosen.

Destructuring should integrate naturally with pattern matching if supported.

## extension methods

yanawa may eventually allow adding behavior to existing types without modifying their original declaration.

Possible syntax:

```yanawa
impl str:
    fn blank() -> bool:
        return length == 0
```

Usage:

```yanawa
if name.blank():
    ...
```

Extension methods can improve ergonomics but may also make method origins harder to understand.

This feature should only be introduced if its benefits justify that complexity.

## object creation example

```yanawa
struct Address:
    city: str
    country: str

struct User:
    id: int
    name: str
    address: Address
    active: bool = true

    fn greet() -> str:
        return "Hello, I'm {name}"

    fn deactivate():
        active = false
```

Usage:

```yanawa
let user = User(
    id = 1,
    name = "Jordan",
    address = Address(
        city = "Osaka",
        country = "Japan"
    )
)

print(user.greet())

user.deactivate()
```

## trait example

```yanawa
trait Display:
    fn display() -> str

struct User:
    name: str
    email: str

impl Display for User:
    fn display() -> str:
        return "{name} <{email}>"
```

Usage:

```yanawa
fn show<T: Display>(value: T):
    print(value.display())
```

## current direction

The current preferred direction is:

* `struct` is the primary data-modeling construct
* ordinary structs require no explicit constructors
* named field construction is preferred
* methods may live inside structs
* direct field access inside methods is preferred when unambiguous
* composition is preferred over inheritance
* traditional inheritance is not currently required
* traits may provide shared behavior contracts
* implementations may live outside struct declarations
* enums may model closed polymorphic hierarchies
* private fields should not require mechanical getters and setters
* the object model should remain small and predictable

## open questions

The following areas remain unresolved:

* exact `self` rules
* struct mutability
* immutable fields
* value versus reference semantics
* copying versus moving
* associated functions
* whether `static` exists
* custom constructors
* trait syntax
* trait objects and dynamic dispatch
* generic trait constraints
* default trait methods
* automatic equality
* string representation
* destructuring syntax
* extension methods
* inheritance
* delegation
* object identity
* interaction with the future memory model

The object model should grow only when simpler tools such as structs, composition, traits, and enums are no longer sufficient.
