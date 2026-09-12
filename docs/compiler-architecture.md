# yanawa compiler architecture

The yanawa compiler should be designed as a collection of clear, independent stages rather than a single monolithic transformation.

Each stage should have a well-defined responsibility, produce useful intermediate information, and avoid depending unnecessarily on later compilation decisions.

> The compiler should understand yanawa before deciding how yanawa executes.

This document describes the current architectural direction of the yanawa compiler.

The implementation language, execution target, runtime architecture, intermediate representations, and code generation strategy are still undecided.

Nothing here should be considered stable or final while yanawa remains experimental.

## core principles

The compiler architecture should aim for:

* clear separation of responsibilities
* predictable compilation stages
* useful diagnostics
* reusable semantic infrastructure
* testable components
* target-independent language analysis
* support for future tooling
* incremental evolution
* minimal coupling between frontend and backend
* correctness before optimization

The compiler should prioritize producing a correct and understandable implementation before pursuing advanced optimizations.

## compilation pipeline

A possible high-level compilation pipeline is:

```text
source code
    ↓
lexer
    ↓
tokens
    ↓
parser
    ↓
syntax tree
    ↓
name resolution
    ↓
type checking
    ↓
semantic representation
    ↓
lowering
    ↓
intermediate representation
    ↓
code generation
    ↓
target output
```

Not every stage must exist as a separate subsystem internally.

The important goal is to preserve clear conceptual boundaries.

## frontend and backend

The compiler should broadly separate into two areas.

### frontend

The frontend understands the yanawa language itself.

Responsibilities may include:

* source loading
* lexical analysis
* parsing
* syntax validation
* name resolution
* type checking
* semantic analysis
* language diagnostics

The frontend should remain mostly independent from the final execution target.

### backend

The backend transforms validated yanawa programs into executable or lower-level output.

Responsibilities may include:

* lowering
* intermediate representation
* optimization
* target-specific code generation
* runtime integration
* artifact production

This separation should allow yanawa to experiment with multiple execution targets without redesigning the language frontend.

## source files

The compiler receives yanawa source files as input.

Possible extension:

```text
.yna
```

Example:

```text
src/main.yna
```

A source manager should track information such as:

* file identity
* file path
* source contents
* line boundaries
* byte offsets
* character positions

This information should remain available throughout compilation for diagnostics and tooling.

## source locations

Compiler structures should preserve source locations whenever practical.

For example, the compiler should know that:

```yanawa
let age: int = "twenty"
```

contains a type mismatch at the string literal.

A diagnostic may then report:

```text
error: expected `int`, found `str`

  --> src/main.yna:1:16
   |
 1 | let age: int = "twenty"
   |                ^^^^^^^^
```

Source information should not be discarded early in compilation.

## lexer

The lexer transforms raw source text into tokens.

Example source:

```yanawa
let age: int = 27
```

Possible token stream:

```text
LET
IDENTIFIER("age")
COLON
IDENTIFIER("int")
EQUAL
INTEGER(27)
NEWLINE
```

The lexer should recognize language elements such as:

* identifiers
* keywords
* numeric literals
* string literals
* character literals
* operators
* punctuation
* indentation
* newlines
* comments

## indentation

Because yanawa uses indentation-based blocks, the lexer or parser must represent indentation structurally.

Example:

```yanawa
if active:
    print("Active")
```

may conceptually produce:

```text
IF
IDENTIFIER(active)
COLON
NEWLINE
INDENT
IDENTIFIER(print)
LEFT_PAREN
STRING("Active")
RIGHT_PAREN
NEWLINE
DEDENT
```

Using explicit `INDENT` and `DEDENT` tokens may simplify parsing.

The exact implementation remains open.

## whitespace

Ordinary spaces should generally not be significant except where indentation defines blocks.

The compiler should clearly distinguish:

* indentation
* spacing inside expressions
* blank lines
* newlines

Formatting differences should not unexpectedly alter semantics except where indentation is structurally meaningful.

## tabs and spaces

yanawa should eventually define a consistent indentation policy.

Possible directions include:

* spaces only
* tabs allowed but normalized
* formatter-enforced indentation

The lexer must never allow visually similar indentation to produce surprising block structures.

The final rule should prioritize predictability.

## keywords

The lexer should distinguish reserved language keywords.

Current candidates include:

```text
let
const
fn
struct
enum
trait
impl
if
elif
else
for
while
switch
case
match
return
import
private
async
await
none
true
false
and
or
not
```

This list is not final.

Keywords should only be reserved when the language actually requires them.

## identifiers

Identifiers should support predictable naming rules.

Examples:

```yanawa
user
user_name
User
UserRepository
```

Unicode identifiers may eventually be supported, but this decision should consider:

* readability
* tooling
* normalization
* security
* confusable characters

The initial compiler may use a simpler identifier model.

## literals

The lexer should recognize literals such as:

```yanawa
42
3.14
"hello"
'A'
true
false
none
```

Literal syntax should map cleanly into the type system.

Numeric literal typing requires further design.

## comments

yanawa should support normal source comments.

Possible syntax:

```yanawa
// single-line comment
```

Documentation comments may eventually use:

```yanawa
/// Documentation comment
```

Whether block comments exist remains undecided.

Comments should not complicate parsing unnecessarily.

## parser

The parser transforms tokens into structured syntax.

For example:

```yanawa
let age: int = 27
```

may become conceptually:

```text
VariableDeclaration
├── name: age
├── type: int
└── value:
    IntegerLiteral(27)
```

The parser should validate grammatical structure but should avoid performing type analysis.

## parser strategy

Possible parser strategies include:

* recursive descent
* Pratt parsing for expressions
* parser combinators
* generated parsers
* hybrid approaches

A hand-written recursive-descent parser combined with Pratt parsing for expressions may provide good control over:

* diagnostics
* language evolution
* operator precedence
* recovery behavior

No strategy has been selected yet.

The implementation should favor maintainability and diagnostics over theoretical elegance.

## concrete and abstract syntax

The compiler may distinguish between:

* concrete syntax representation
* abstract syntax tree

A concrete syntax tree preserves most source structure.

An abstract syntax tree keeps primarily semantic structure.

For example, formatting tools may benefit from preserving:

* comments
* whitespace
* exact token positions

while type checking may not need them.

The exact representation strategy remains undecided.

## syntax tree

A syntax tree should represent constructs such as:

```text
Program
Module
Import
VariableDeclaration
ConstantDeclaration
FunctionDeclaration
StructDeclaration
EnumDeclaration
TraitDeclaration
ImplDeclaration
IfExpression
SwitchExpression
MatchExpression
Loop
Return
Call
BinaryExpression
UnaryExpression
Literal
Identifier
FieldAccess
```

The final node set should emerge from the grammar rather than being designed independently.

## expression parsing

Expressions require clear precedence rules.

For example:

```yanawa
a + b * c
```

should parse as:

```text
a + (b * c)
```

Possible precedence categories include:

```text
assignment
or
and
equality
comparison
addition
multiplication
unary
call
field access
```

The final precedence table should remain small and unsurprising.

## parser diagnostics

Syntax errors should identify the actual unexpected input whenever possible.

Example:

```yanawa
fn greet(name str):
```

Possible diagnostic:

```text
error: expected `:` after parameter name

  --> src/main.yna:1:15
   |
 1 | fn greet(name str):
   |               ^^^
```

The parser should avoid generic messages such as:

```text
unexpected token
```

when a more specific explanation is available.

## error recovery

The parser should attempt to recover after syntax errors where safe.

This allows the compiler and editor tooling to report multiple issues in one analysis.

Useful synchronization points may include:

* newlines
* declarations
* dedentation
* closing delimiters

Recovery should avoid producing large numbers of misleading cascading errors.

## name resolution

After parsing, the compiler must determine what every identifier refers to.

Example:

```yanawa
let user = find_user(id)
```

The compiler must resolve:

```text
user
find_user
id
```

according to their scopes.

Name resolution should understand:

* local variables
* parameters
* functions
* struct fields
* types
* modules
* imports
* traits
* enum variants

## symbol tables

The compiler may maintain symbol tables representing declarations visible within scopes.

Possible information includes:

```text
name
kind
type
visibility
source location
module
```

Symbol representation should be reusable by:

* type checking
* diagnostics
* language server features
* documentation generation

## scopes

Possible scopes include:

* project
* module
* type
* function
* block
* pattern

Example:

```yanawa
let value = 10

if active:
    let value = 20
    print(value)

print(value)
```

Whether shadowing like this is allowed remains a language-design decision.

The compiler architecture should support detecting and describing scope relationships clearly.

## imports

Name resolution should integrate with the module system.

Example:

```yanawa
import user.User
```

The compiler should resolve `User` to the declaration exposed by the relevant module.

Import resolution should detect:

* missing modules
* missing declarations
* ambiguous names
* visibility violations
* dependency cycles where relevant

## semantic analysis

Semantic analysis verifies rules that are not purely grammatical.

Examples include:

* duplicate declarations
* invalid visibility
* invalid assignments
* unreachable declarations
* invalid control-flow usage
* incorrect generic usage
* invalid trait implementations
* invalid return behavior

Some semantic checks may overlap with type checking.

The architecture should prioritize clear responsibilities without forcing artificial boundaries.

## type checker

The type checker is one of the central components of the yanawa compiler.

It should determine and validate types across the program.

Example:

```yanawa
let age: int = 27
```

The compiler validates:

```text
declared type: int
value type: int
```

Invalid example:

```yanawa
let age: int = "twenty"
```

The compiler should detect:

```text
declared type: int
value type: str
```

and report the mismatch before execution.

## type inference

The type checker should infer types where yanawa permits it.

Example:

```yanawa
let name = "Alex"
```

becomes conceptually:

```text
name: str
```

Likewise:

```yanawa
let values = [1, 2, 3]
```

may become:

```text
values: int[]
```

Inference should reduce repetition without making types unpredictable.

## function checking

Functions should be validated against their declared signatures.

```yanawa
fn add(a: int, b: int) -> int:
    return a + b
```

The type checker validates:

* parameter types
* expression types
* return type
* all reachable return paths

Invalid:

```yanawa
fn add(a: int, b: int) -> int:
    return "invalid"
```

should fail during compilation.

## nullable analysis

The compiler should track nullable types.

Example:

```yanawa
let name: str? = find_name()
```

Unsafe usage should fail:

```yanawa
print(name.upper())
```

After validation:

```yanawa
if name != none:
    print(name.upper())
```

the compiler may narrow `name` to `str` inside the block.

This requires control-flow-aware type analysis.

## control-flow analysis

The compiler may eventually build a control-flow representation to understand behavior such as:

* definite assignment
* unreachable code
* nullable narrowing
* exhaustive matching
* return paths
* variable lifetime
* data-flow analysis

Example:

```yanawa
fn classify(age: int) -> str:
    if age >= 18:
        return "adult"

    return "minor"
```

The compiler can prove all paths return a `str`.

## exhaustiveness

Enums and pattern matching may require exhaustiveness checking.

Example:

```yanawa
enum Status:
    pending
    running
    complete
```

Then:

```yanawa
match status:
    case pending:
        ...
    case running:
        ...
```

could produce:

```text
error: non-exhaustive match

missing case:
  complete
```

Exhaustiveness checking should be implemented as semantic analysis rather than runtime behavior.

## semantic representation

The compiler may lower parsed syntax into a more semantic representation after name resolution and type checking.

Possible names include:

```text
HIR
Typed AST
Semantic IR
```

This representation may contain:

* resolved symbols
* explicit types
* normalized constructs
* lowered syntax sugar
* source references

For example:

```yanawa
let name = "Alex"
```

may become:

```text
VariableDeclaration
name: name
type: str
value: StringLiteral("Alex")
```

The type is now explicit internally even though it was inferred in source.

## lowering

Lowering transforms high-level language constructs into simpler compiler representations.

Example source:

```yanawa
if name != none:
    print(name)
```

may eventually lower into explicit control flow.

Likewise:

```yanawa
for user in users:
    print(user.name)
```

may lower into iterator operations.

Lowering should remove syntax sugar before target-specific code generation.

## why lowering matters

Keeping high-level syntax separate from lower-level representations allows yanawa to:

* evolve syntax without rewriting every backend
* share optimization passes
* simplify code generation
* support multiple targets
* test language semantics independently

This is especially important if yanawa eventually targets more than one execution environment.

## intermediate representation

yanawa will likely benefit from an intermediate representation between semantic analysis and final code generation.

Possible architecture:

```text
source
↓
AST
↓
typed semantic representation
↓
yanawa IR
↓
target backend
```

A custom IR may represent concepts such as:

* functions
* typed values
* branches
* calls
* allocations
* loads
* stores
* returns

The exact IR design should wait until code generation requirements are better understood.

## high-level and low-level IR

The compiler may eventually benefit from multiple IR levels.

Possible structure:

```text
AST
↓
HIR
↓
MIR
↓
target IR
```

Where:

* HIR preserves more language semantics
* MIR represents simpler control flow
* target IR maps closely to the execution target

This architecture should only be introduced when its benefits justify the complexity.

The first compiler may use fewer stages.

## control-flow representation

A lower-level IR may represent functions as basic blocks.

Example:

```text
entry:
    compare age >= 18
    branch adult, minor

adult:
    return "adult"

minor:
    return "minor"
```

This representation can simplify:

* code generation
* optimization
* data-flow analysis
* reachability analysis

## code generation

After semantic validation and lowering, the compiler must produce executable output.

The final target remains undecided.

Possible directions include:

* native machine code
* LLVM IR
* JVM bytecode
* WebAssembly
* JavaScript
* another virtual machine
* yanawa-specific bytecode

The frontend architecture should avoid assuming one target prematurely.

## backend interface

A conceptual backend interface may receive a validated lower-level representation.

For example:

```text
compile(module_ir) -> target_artifact
```

Different backends could theoretically implement:

```text
JVM backend
native backend
JavaScript backend
WebAssembly backend
```

Multiple backends are a future possibility, not an initial requirement.

The first implementation should likely focus on one target.

## target independence

Language semantics should not accidentally depend on quirks of the first backend.

For example, if the first backend uses the JVM, yanawa should not automatically inherit every Java semantic unless intentionally chosen.

Likewise, a JavaScript backend should not redefine yanawa numbers or nullability merely because JavaScript behaves differently.

Backend-specific limitations must be explicit.

## runtime

yanawa may require a runtime depending on its execution model.

Possible runtime responsibilities include:

* memory management
* garbage collection
* async scheduling
* panic handling
* stack traces
* standard library support
* reflection if it exists
* process initialization

The runtime should remain as small as practical.

The runtime architecture cannot be finalized until the execution target and memory model are selected.

## standard library integration

The compiler must understand how projects access the standard library.

Possible model:

```yanawa
import yanawa.fs
```

The standard library may be:

* precompiled
* compiled with the project
* linked into output
* provided by a runtime

This depends on the backend.

The source-level API should remain consistent regardless of internal packaging.

## compiler driver

The compiler driver coordinates the compilation pipeline.

Conceptually:

```text
load project
resolve dependencies
load source files
lex
parse
resolve names
type check
lower
generate target output
```

The user should normally access this through:

```bash
yanawa build
```

rather than invoking compiler stages manually.

## project compilation

Compilation operates on projects, not merely individual files.

The compiler should understand:

```text
yanawa.toml
src/
dependencies
modules
target
```

This allows semantic analysis across module boundaries.

Single-file execution may still exist as a convenience.

## dependency compilation

Dependencies should be validated and compiled according to the package system.

The compiler should avoid rebuilding unchanged dependencies unnecessarily.

Dependency artifacts may eventually be cached.

The exact artifact model depends on the chosen backend.

## incremental compilation

Incremental compilation is desirable but should not complicate the first implementation unnecessarily.

A future compiler may cache information such as:

* parsed syntax
* module signatures
* type information
* generated IR
* compiled artifacts

When one file changes, only affected components should be rebuilt.

The dependency graph should make invalidation predictable.

## compiler database

A future architecture may represent compiler computations as queries.

Examples:

```text
parse(file)
resolve(module)
type_of(expression)
lower(function)
```

Cached query results could support:

* incremental compilation
* language server responsiveness
* dependency tracking

This architecture is powerful but should not be introduced before the compiler needs it.

## diagnostics pipeline

Diagnostics should be a shared compiler subsystem.

Every stage may produce structured diagnostics.

Examples:

```text
lexer diagnostic
parser diagnostic
resolution diagnostic
type diagnostic
backend diagnostic
```

Each diagnostic may contain:

```text
severity
message
source span
labels
notes
suggestions
```

Rendering to the terminal should happen separately from creating the diagnostic.

## diagnostic severity

Possible severity levels:

```text
error
warning
note
help
```

Errors prevent successful compilation.

Warnings indicate valid but suspicious code.

Notes and help provide context.

The compiler should avoid introducing excessive severity categories.

## diagnostic codes

yanawa may eventually assign stable identifiers to diagnostics.

Possible:

```text
Y0012
```

However, error codes should complement readable messages rather than replace them.

The initial compiler may not need stable diagnostic codes.

## compiler errors

Invalid user programs should never crash the compiler.

Input such as malformed syntax, invalid Unicode, invalid types, or incomplete editor buffers should produce diagnostics rather than internal failures.

Compiler robustness is especially important for language server integration.

## internal compiler errors

Unexpected compiler failures should be clearly identified.

Possible output:

```text
internal compiler error

the compiler encountered an unexpected condition
```

This should be distinguishable from user-code errors.

Debug builds may include additional diagnostic context.

## tooling reuse

Compiler components should be reusable by first-party tooling.

The language server should reuse:

* parser
* symbol resolution
* type checker
* diagnostics

The documentation generator may reuse:

* module discovery
* semantic model
* type information

`yanawa check` should essentially expose semantic compilation without final code generation.

There should not be separate competing implementations of yanawa semantics.

## formatter architecture

The formatter may need access to a syntax representation that preserves enough original structure to format code reliably.

This is one reason a lossless syntax tree may eventually be useful.

The formatter should not depend on successful type checking.

It should ideally format syntactically incomplete files where practical.

## language server architecture

The language server will need fast access to compiler information while files are being edited.

Requirements may include:

* parsing incomplete source
* partial semantic analysis
* incremental updates
* cancellation of outdated analysis
* shared project state

This should influence compiler architecture without dominating the first compiler prototype.

## testing compiler stages

Each compiler stage should be independently testable.

### lexer tests

Input:

```yanawa
let age = 27
```

Expected tokens:

```text
LET
IDENTIFIER
EQUAL
INTEGER
```

### parser tests

Input:

```yanawa
fn add(a: int, b: int) -> int:
    return a + b
```

Expected syntax structure can be validated.

### type checker tests

Input:

```yanawa
let age: int = "invalid"
```

Expected:

```text
type error
```

### diagnostic tests

Compiler output may use snapshot or structured diagnostic tests.

### integration tests

Full source programs should eventually compile and execute.

The compiler should not rely only on end-to-end tests.

## golden tests

Compiler behavior may benefit from golden or snapshot tests.

Example source:

```text
tests/ui/type-mismatch.yna
```

Expected diagnostics:

```text
tests/ui/type-mismatch.stderr
```

This approach can make diagnostic regressions visible.

The exact testing strategy remains undecided.

## parser fuzzing

Once parsing is mature enough, fuzz testing may help discover crashes or pathological inputs.

Potential fuzz targets include:

* lexer
* parser
* formatter
* module loader

Robust handling of arbitrary source input is valuable for compiler security and editor integration.

## performance

Correctness should come before compiler performance during early development.

Later, performance work may focus on:

* source loading
* parser speed
* type checking
* incremental recompilation
* dependency caching
* parallel compilation
* code generation

Optimization should be measurement-driven.

## parallel compilation

Independent modules may eventually be analyzed or compiled concurrently.

The compiler architecture should avoid unnecessary global mutable state that makes parallelization difficult.

Parallel compilation is not required for the first implementation.

## deterministic compilation

The compiler should aim for deterministic behavior.

Given equivalent:

* source
* dependencies
* toolchain
* target
* configuration

the compiler should produce equivalent semantic results.

Nondeterministic declaration ordering or diagnostics should be avoided.

## compiler configuration

Compiler behavior should require minimal configuration.

Most project configuration should come from:

```text
yanawa.toml
```

Command-line flags may override specific behavior.

Compiler configuration should not become a second language.

## optimization

Optimization should not define the initial compiler architecture.

The first meaningful goal is:

```text
correct yanawa program
        ↓
correct executable behavior
```

Possible future optimizations include:

* constant folding
* dead-code elimination
* inlining
* escape analysis
* allocation elimination
* loop optimization

Many optimizations may eventually be delegated to a backend such as LLVM, JVM, or another target runtime.

## debug builds

Development builds should preserve useful information for debugging.

Possible information includes:

* source locations
* function names
* variable information
* stack traces

The final debugging model depends on the target.

## release builds

Release builds may enable stronger optimization.

Possible command:

```bash
yanawa build --release
```

Optimization must never change defined program semantics.

## debugging support

The compiler should eventually support debugging through standard tools where possible.

Potential options depend heavily on the target:

* native debug information
* JVM debugging
* source maps
* WebAssembly debugging

A custom debugger should not be created unless existing protocols cannot provide an adequate experience.

## target configuration

Future target selection may look like:

```bash
yanawa build --target <target>
```

Target names and support remain completely undecided.

The source language should not require target-specific syntax for normal code.

## target-specific code

Some applications may eventually require platform-specific behavior.

Possible mechanisms include:

* conditional compilation
* platform modules
* target capabilities

No model has been designed yet.

Conditional compilation should not become an uncontrolled preprocessor system.

## macros and compiler plugins

The initial compiler should not require macros or compiler plugins.

These systems significantly increase:

* compiler complexity
* tooling complexity
* diagnostics complexity
* build reproducibility concerns

yanawa should first determine what can be expressed through:

* functions
* generics
* traits
* enums
* first-party tooling

Compile-time metaprogramming should only be introduced after concrete needs emerge.

## reflection

Runtime reflection is not currently required.

Reflection can simplify some frameworks but also introduces hidden behavior and runtime complexity.

One of yanawa's goals is to avoid requiring reflection-heavy frameworks for ordinary application development.

If metadata inspection becomes necessary, the language should explore more explicit alternatives first.

## compiler repository structure

The final source layout depends on the implementation language.

Conceptually, compiler responsibilities may eventually map to components such as:

```text
compiler/
├── source
├── lexer
├── syntax
├── parser
├── resolve
├── types
├── semantic
├── ir
├── backend
└── diagnostics
```

This is a conceptual architecture, not a required repository structure.

The actual implementation should avoid unnecessary fragmentation.

## possible first compiler

The first compiler does not need to implement the entire architecture described here.

A minimal first implementation may support only:

```text
source loading
↓
lexer
↓
parser
↓
basic AST
↓
basic type checking
↓
simple execution target
```

Initial language features may include:

```text
primitive values
variables
functions
arithmetic
conditions
print
```

More advanced systems should be added incrementally.

## bootstrap strategy

yanawa does not need to be self-hosted initially.

The first compiler should be implemented in an existing language suitable for compiler development.

Possible future self-hosting should only be considered when yanawa itself is mature enough to express the compiler clearly and reliably.

Self-hosting is a milestone, not a prerequisite.

## implementation language

The compiler implementation language remains undecided.

Possible candidates may include:

```text
Rust
C++
Java
Zig
```

The decision should consider:

* memory safety
* compiler-development ecosystem
* performance
* maintainability
* target integration
* tooling
* portability
* developer familiarity

The implementation language should be chosen for engineering reasons rather than branding.

## first execution target

The first execution target remains undecided.

Possible options include:

### interpreter

Advantages:

* rapid language experimentation
* simple initial execution
* easier debugging

Disadvantages:

* may require later backend redesign
* lower runtime performance

### custom bytecode

Advantages:

* controlled semantics
* portable execution model
* useful intermediate target

Disadvantages:

* requires building and maintaining a VM

### JVM bytecode

Advantages:

* mature runtime
* garbage collection
* cross-platform execution
* large ecosystem

Disadvantages:

* JVM semantics may influence language design
* frontend use remains separate

### LLVM

Advantages:

* mature native code generation
* optimization infrastructure
* multiple native targets

Disadvantages:

* additional complexity
* native runtime responsibilities

### JavaScript

Advantages:

* direct browser execution
* easy frontend experimentation

Disadvantages:

* JavaScript semantics differ substantially from yanawa goals
* backend behavior may be constrained by target semantics

### WebAssembly

Advantages:

* portable execution
* browser and server potential

Disadvantages:

* runtime and ecosystem complexity
* not necessarily the easiest initial target

The first target should optimize for learning, implementation clarity, and language validation rather than attempting to solve every future use case.

## architecture before optimization

The first goal should not be:

> Build the fastest compiler possible.

The first goal should be:

> Build a compiler whose behavior we understand.

Clear architecture will make later optimization much easier.

## current direction

The current preferred direction is:

* compiler stages should have clear responsibilities
* language frontend should remain mostly target-independent
* source locations should survive throughout compilation
* indentation may become explicit `INDENT` and `DEDENT` tokens
* parsing and semantic analysis should remain separate concepts
* name resolution should produce reusable symbol information
* type checking should operate on resolved semantic structures
* type inference should become explicit internally after analysis
* high-level syntax should be lowered before target-specific generation
* some form of intermediate representation will likely be useful
* diagnostics should be structured and shared across compiler stages
* compiler infrastructure should be reusable by tooling
* the first compiler should remain intentionally small
* correctness and diagnostics come before optimization
* implementation language remains undecided
* execution target remains undecided
* self-hosting is not an initial requirement

## open questions

The following areas remain unresolved:

* compiler implementation language
* first execution target
* interpreter versus compiler-first approach
* parser implementation strategy
* lossless syntax tree versus AST-only architecture
* exact AST structure
* expression precedence
* indentation tokenization
* identifier Unicode support
* comment syntax
* semantic representation
* number of IR levels
* custom IR design
* runtime requirements
* module compilation model
* dependency artifact model
* incremental compilation architecture
* compiler query system
* optimization strategy
* debugging information
* target abstraction
* conditional compilation
* reflection
* macros
* compile-time execution
* compiler plugin system
* eventual self-hosting

The architecture should remain flexible until implementation provides enough evidence to make stronger decisions.

> Understand the language first. Lower it second. Optimize it last.
