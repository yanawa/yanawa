# yanawa modules

yanawa should provide a simple and predictable module system.

Projects should be easy to navigate, imports should clearly communicate where code comes from, and developers should not need excessive configuration to organize ordinary applications.

> Project structure should explain itself.

This document describes the current direction of modules, files, imports, and visibility in yanawa.

Nothing here should be considered stable or final while the language remains experimental.

## core principles

The module system should be:

* simple
* predictable
* file-oriented
* easy to navigate
* explicit about dependencies
* compatible with larger projects
* resistant to circular dependency problems
* consistent with yanawa's visibility model
* independent from unnecessary configuration

A developer should usually be able to understand a project's structure by looking at its directories and files.

## files

A yanawa source file represents a compilation unit.

Possible file extension:

```text
.yna
```

Example:

```text
main.yna
user.yna
database.yna
```

The final file extension has not yet been decided.

Possible alternatives may include:

```text
.yw
.yana
.yna
```

The extension should be short, recognizable, and unlikely to conflict with existing formats.

## directories

Directories should naturally organize related code.

Example:

```text
src/
├── main.yna
├── user/
│   ├── user.yna
│   ├── service.yna
│   └── repository.yna
└── auth/
    ├── login.yna
    └── token.yna
```

The module system should avoid requiring developers to duplicate the directory structure inside every source file.

## modules

A directory may represent a module namespace.

For example:

```text
src/user/user.yna
```

could be imported as:

```yanawa
import user.User
```

or:

```yanawa
import user.user.User
```

depending on the final module rules.

yanawa should prefer the simpler form when the mapping remains unambiguous.

## implicit module paths

The current preferred direction is for module paths to follow the project structure automatically.

A file such as:

```text
src/user/service.yna
```

could belong to:

```text
user.service
```

without requiring a declaration such as:

```text
module user.service
```

inside the file.

Explicit module declarations should only exist if they solve a real problem.

## imports

Imports should use readable dotted paths.

```yanawa
import user.User
import user.UserService
```

Standard library imports may follow the same structure.

```yanawa
import yanawa.http
import yanawa.json
```

The exact relationship between imports, packages, and the standard library remains under design.

## grouped imports

Multiple declarations from the same module may be grouped.

Possible syntax:

```yanawa
import user.{User, UserService, UserRepository}
```

This should remain optional.

The following should also be valid:

```yanawa
import user.User
import user.UserService
import user.UserRepository
```

Readability should take priority over reducing import lines.

## module imports

It may be useful to import an entire module.

Possible syntax:

```yanawa
import yanawa.http
```

Usage:

```yanawa
http.get(...)
http.post(...)
```

This can make the origin of functions clear without requiring excessive fully qualified names.

## aliases

Import aliases may be useful when modules have conflicting names.

Possible syntax:

```yanawa
import infrastructure.database as db
```

Usage:

```yanawa
db.connect()
```

Aliases should primarily exist for disambiguation and readability.

## relative imports

The current direction is to avoid complicated relative import syntax when possible.

Instead of:

```text
../../user/service
```

yanawa should prefer stable module paths.

```yanawa
import user.UserService
```

This makes imports less sensitive to moving the current file.

Whether limited relative imports should exist remains an open question.

## visibility

The module system should integrate with declaration visibility.

The current syntax direction considers declarations public by default.

```yanawa
fn greet():
    print("Hello")
```

Private declarations use `private`.

```yanawa
private fn validate_token(token: str) -> bool:
    ...
```

A private declaration should not be accessible from outside its defined visibility scope.

The exact meaning of `private` still needs to be decided.

Possible interpretations include:

* visible only inside the declaration's type
* visible only inside the source file
* visible only inside the current module

These concepts may eventually require separate visibility modifiers.

## module-private visibility

Larger projects may require declarations that are shared inside a module but hidden from the rest of the application.

Possible future modifier:

```yanawa
internal fn normalize_email(email: str) -> str:
    ...
```

or:

```yanawa
module fn normalize_email(email: str) -> str:
    ...
```

No additional visibility modifier should be introduced until a real use case justifies it.

## public api

A module's public API should consist of declarations intentionally accessible from outside the module.

Example:

```yanawa
struct User:
    id: int
    name: str

fn find_user(id: int) -> User?:
    ...
```

Internal implementation details should remain private.

```yanawa
private fn map_database_row(row: Row) -> User:
    ...
```

The language should make it easy to understand which parts of a module form its external contract.

## file names

File names should use lowercase names.

Possible convention:

```text
user.yna
user_service.yna
user_repository.yna
```

Whether yanawa officially prefers:

```text
snake_case
```

or:

```text
kebab-case
```

for file names remains undecided.

The language itself should avoid depending on stylistic naming conventions unless necessary.

## type names

Types may follow conventional PascalCase naming.

```yanawa
struct UserProfile:
    ...
```

Functions and variables may use snake_case.

```yanawa
fn find_user_by_email(email: str) -> User?:
    ...
```

These are currently style conventions rather than compiler requirements.

## entry point

Applications should have a clear entry point.

The current direction uses:

```yanawa
fn main():
    ...
```

Example:

```yanawa
fn main():
    print("Hello from yanawa")
```

A project may use a conventional file such as:

```text
src/main.yna
```

The compiler should not require complex configuration for ordinary executable projects.

## library projects

Libraries may not require a `main` function.

Example:

```text
src/
├── math.yna
├── parser.yna
└── formatter.yna
```

These projects expose reusable declarations instead of an executable entry point.

The distinction between application and library projects may eventually live in a project manifest.

## standard library

The standard library should use a recognizable namespace.

Possible examples:

```yanawa
import yanawa.http
import yanawa.json
import yanawa.fs
import yanawa.time
```

This provides a clear distinction between:

* language syntax
* standard library functionality
* third-party packages
* application modules

The exact namespace is not final.

## third-party packages

External packages should eventually integrate with the same import syntax.

Possible example:

```yanawa
import postgres.Client
```

or:

```yanawa
import packages.postgres.Client
```

The package system has not yet been designed.

Package names should not make ordinary imports unnecessarily verbose.

## project imports

Application code should remain concise.

Example structure:

```text
src/
├── main.yna
├── user/
│   ├── model.yna
│   ├── service.yna
│   └── repository.yna
└── auth/
    └── service.yna
```

Possible imports:

```yanawa
import user.User
import user.UserService
import auth.AuthService
```

The directory structure should already provide most of the information the compiler needs.

## circular dependencies

yanawa should discourage or reject circular module dependencies.

For example:

```text
module A imports B
module B imports A
```

Circular dependencies make systems harder to understand and complicate compilation.

The compiler should ideally detect cycles and provide a useful diagnostic.

Possible diagnostic:

```text
error: circular module dependency

user.service -> auth.service -> user.service
```

Whether some limited cycles are technically allowed should depend on the compiler architecture, but the language should not encourage them.

## unused imports

The compiler or first-party tooling should detect unused imports.

Example:

```text
warning: unused import `user.UserRepository`
```

Whether unused imports are warnings or errors may depend on project settings.

Automatic formatting or editor tooling may eventually remove them safely.

## ambiguous imports

The compiler should reject ambiguous imported names.

Example:

```yanawa
import billing.User
import auth.User

let user: User
```

This is ambiguous.

Aliases may resolve it:

```yanawa
import billing.User as BillingUser
import auth.User as AuthUser
```

Then:

```yanawa
let customer: BillingUser
let session_user: AuthUser
```

## qualified access

Developers should be able to keep module qualification when it improves clarity.

Possible example:

```yanawa
import billing
import auth

let customer: billing.User
let user: auth.User
```

The exact syntax remains under design.

## wildcard imports

Wildcard imports such as:

```text
import user.*
```

should probably be avoided or discouraged.

They can make it difficult to determine where identifiers originate and may introduce accidental naming conflicts.

yanawa should prefer explicit imports.

## automatic imports

The language should avoid large sets of hidden automatic imports.

A small collection of fundamental types or functions may be available globally.

Possible examples:

```text
str
int
bool
print
```

Anything beyond a small prelude should generally require an import.

The exact prelude remains undecided.

## re-exports

Libraries may eventually need to expose declarations originating from internal modules.

Possible syntax:

```yanawa
export user.User
export user.UserService
```

or:

```yanawa
public import user.User
```

This has not yet been designed.

Re-exports should only be introduced if they make library APIs significantly cleaner.

## example project

Possible project:

```text
src/
├── main.yna
├── user/
│   ├── model.yna
│   ├── service.yna
│   └── repository.yna
└── database/
    └── connection.yna
```

`user/model.yna`:

```yanawa
struct User:
    id: int
    name: str
    email: str
```

`user/repository.yna`:

```yanawa
import user.User
import database.Connection

struct UserRepository:
    connection: Connection

    fn find(id: int) -> result<User?, DatabaseError>:
        ...
```

`user/service.yna`:

```yanawa
import user.User
import user.UserRepository

struct UserService:
    repository: UserRepository

    fn find(id: int) -> result<User?, DatabaseError>:
        return repository.find(id)
```

`main.yna`:

```yanawa
import database.Connection
import user.UserRepository
import user.UserService

fn main():
    let connection = Connection.open()
    let repository = UserRepository(connection = connection)
    let users = UserService(repository = repository)

    ...
```

The exact constructor behavior remains experimental.

## current direction

The current preferred direction is:

* files are compilation units
* directories naturally define module structure
* module paths should follow the filesystem where practical
* explicit module declarations should not be required without a reason
* imports use dotted paths
* imports should generally be explicit
* grouped imports may be supported
* aliases should resolve naming conflicts
* complex relative imports should be avoided
* circular dependencies should be detected
* applications use `fn main()` as their entry point
* standard library modules use a recognizable namespace
* project structure should require minimal configuration

## open questions

The following areas remain unresolved:

* final source file extension
* exact mapping between files and module names
* whether directories always create module namespaces
* meaning and scope of `private`
* whether `internal` or module-level visibility is needed
* grouped import syntax
* module aliases
* qualified module imports
* relative imports
* automatic imports and prelude contents
* re-export syntax
* package namespace behavior
* application versus library project configuration
* file naming conventions
* circular dependency rules
* module initialization behavior
* whether source layout requires a conventional `src/` directory

The module system should remain predictable enough that developers rarely need documentation to understand where a declaration comes from.
