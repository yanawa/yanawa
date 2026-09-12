# yanawa tooling

yanawa should provide a cohesive first-party toolchain.

Developers should not need to assemble unrelated tools just to create, build, test, format, document, and maintain an ordinary yanawa project.

> One language, one toolchain, one obvious workflow.

Tooling is part of the language experience, not an afterthought.

This document describes the current direction of the yanawa CLI, compiler interface, formatter, linter, testing tools, documentation tooling, editor integration, diagnostics, and developer workflow.

Nothing here should be considered stable or final while the language remains experimental.

## core principles

yanawa tooling should be:

* integrated
* predictable
* fast
* consistent
* easy to discover
* useful without extensive configuration
* scriptable when necessary
* friendly to editors and CI systems
* designed around clear diagnostics
* built around sensible defaults

A developer should be able to install yanawa and immediately understand how to work with a project.

## one primary command

The primary interface to the yanawa toolchain should be:

```text
yanawa
```

Common workflows should live under this command.

Possible commands include:

```bash
yanawa new
yanawa init
yanawa run
yanawa build
yanawa check
yanawa test
yanawa fmt
yanawa lint
yanawa add
yanawa remove
yanawa install
yanawa update
yanawa docs
yanawa clean
```

The language should avoid requiring separate executables such as:

```text
yanawac
yanawafmt
yanawatest
yanawadoc
```

unless internal architecture requires them.

Even if separate binaries exist internally, ordinary developers should primarily interact with `yanawa`.

## command discoverability

Running:

```bash
yanawa
```

or:

```bash
yanawa help
```

should show a concise overview of available commands.

Possible output:

```text
yanawa — the yanawa programming language toolchain

Usage:
  yanawa <command> [options]

Commands:
  new       Create a new project
  init      Initialize a project
  run       Build and run a project
  build     Build a project
  check     Analyze a project without producing output
  test      Run tests
  fmt       Format source code
  lint      Analyze code quality
  add       Add a dependency
  remove    Remove a dependency
  install   Resolve project dependencies
  update    Update dependencies
  docs      Generate documentation
  clean     Remove generated build artifacts
```

Help output should be short enough to scan quickly.

Detailed help should remain available through:

```bash
yanawa help build
```

or:

```bash
yanawa build --help
```

## project creation

A new project may be created with:

```bash
yanawa new hello
```

Possible result:

```text
hello/
├── yanawa.toml
└── src/
    └── main.yna
```

The generated project should remain minimal.

The tool should avoid creating large amounts of boilerplate.

Possible `main.yna`:

```yanawa
fn main():
    print("Hello from yanawa")
```

## project initialization

An existing directory may become a yanawa project using:

```bash
yanawa init
```

This should create only the files necessary to define the project.

The command should not overwrite existing source files without explicit permission.

## running projects

The normal development command should be:

```bash
yanawa run
```

This should:

1. discover the project
2. resolve required dependencies
3. validate the project
4. compile what is necessary
5. execute the application

Passing arguments to the program should remain straightforward.

Possible syntax:

```bash
yanawa run -- user.json
```

Everything after `--` would belong to the application rather than the yanawa CLI.

The exact argument model remains undecided.

## building projects

Production or distributable output should use:

```bash
yanawa build
```

The command should compile the current project according to the selected target and build mode.

Possible options may eventually include:

```bash
yanawa build --release
yanawa build --target <target>
```

The exact meaning of targets depends on future compiler architecture.

## check

`yanawa check` should analyze source code without producing final executable output.

```bash
yanawa check
```

This command should perform work such as:

* parsing
* name resolution
* type checking
* module validation
* dependency validation
* selected static analysis

It should be optimized for fast feedback.

Editor integrations may use the same compiler infrastructure internally.

## incremental work

Repeated development commands should avoid performing unnecessary work.

For example:

```bash
yanawa check
```

after modifying one file should ideally avoid reprocessing an entire large project when previous results remain valid.

Incremental compilation or analysis should be explored when compiler architecture supports it.

Performance should not require developers to understand or manually manage compiler caches.

## build modes

yanawa may eventually support development and release builds.

Possible commands:

```bash
yanawa build
yanawa build --release
```

Development builds may prioritize:

* compilation speed
* diagnostics
* debugging information

Release builds may prioritize:

* runtime performance
* binary size
* optimization

The default development workflow should remain fast.

## formatter

yanawa should provide an official formatter.

Command:

```bash
yanawa fmt
```

The formatter should have one primary canonical style.

Developers should not spend significant time debating formatting conventions that tooling can decide automatically.

The formatter may control:

* indentation
* spacing
* line breaks
* import formatting
* collection formatting
* function signatures
* struct declarations

Example:

```yanawa
fn greet(name:str)->str:
 return "Hello, {name}"
```

could become:

```yanawa
fn greet(name: str) -> str:
    return "Hello, {name}"
```

## formatter philosophy

Formatting should be intentionally opinionated.

Configuration should remain limited.

A formatter with dozens of stylistic options undermines the value of having an official formatter.

Reasonable configuration may exist for concerns such as:

* maximum line width
* newline behavior

Most syntax formatting should remain standardized.

## format checking

CI environments should be able to verify formatting without modifying files.

Possible command:

```bash
yanawa fmt --check
```

A non-formatted project should return a non-zero exit status.

## linter

yanawa should provide first-party linting.

Possible command:

```bash
yanawa lint
```

Linting should focus on code that is valid but potentially:

* suspicious
* unnecessarily complex
* inefficient
* misleading
* error-prone

Possible diagnostics include:

```text
warning: variable `result` is never used
```

```text
warning: condition is always true
```

```text
warning: unnecessary nullable type
```

```text
warning: unreachable branch
```

Some checks may belong directly in the compiler rather than the linter.

The boundary should remain intentional.

## compiler errors versus lints

Compiler errors should represent invalid code.

Example:

```text
error: expected `int`, found `str`
```

Lints should generally represent valid but questionable code.

Example:

```text
warning: variable `count` is never read
```

The toolchain should avoid turning style preferences into compiler errors.

## lint configuration

Lint rules may eventually support levels such as:

```text
allow
warn
deny
```

However, configuration should remain simple.

Projects should have useful linting without adding configuration.

## automatic fixes

Some diagnostics may provide safe automatic fixes.

Possible command:

```bash
yanawa lint --fix
```

Examples may include:

* removing unused imports
* replacing redundant syntax
* applying safe simplifications

Automatic fixes should never silently change program behavior.

## testing

Testing should be part of the official toolchain.

Command:

```bash
yanawa test
```

A project should not require a separate external test runner for ordinary testing.

Possible test syntax remains under discussion.

One direction:

```yanawa
test "adds two numbers":
    assert add(2, 3) == 5
```

Another direction:

```yanawa
fn test_addition():
    assert add(2, 3) == 5
```

The tooling should not force a final language syntax prematurely.

## test discovery

Tests should be discoverable through predictable conventions.

Possible approaches include:

* dedicated test syntax
* conventional test directories
* conventional function names
* compiler metadata

Possible project structure:

```text
src/
    calculator.yna

tests/
    calculator_test.yna
```

The exact model remains undecided.

## test filtering

Developers should be able to run specific tests.

Possible commands:

```bash
yanawa test calculator
```

or:

```bash
yanawa test --filter calculator
```

The exact CLI syntax remains open.

## test output

Passing tests should produce concise output.

Failures should include enough context to debug quickly.

Possible output:

```text
running 12 tests

✓ addition
✓ subtraction
✗ division by zero

expected:
  error(DivisionError.zero)

received:
  ok(0)

11 passed
1 failed
```

Useful failure messages are more important than decorative output.

## assertions

Built-in assertions should provide useful diagnostics.

```yanawa
assert result == expected
```

When an assertion fails, the test runner should ideally display both values.

Testing should not require large matcher libraries merely to get useful errors.

## benchmarks

Benchmarking may eventually be integrated.

Possible command:

```bash
yanawa bench
```

This is not required for the initial toolchain.

Benchmarks should only become first-party when compiler and runtime performance are stable enough for results to be meaningful.

## documentation generator

yanawa should provide first-party API documentation generation.

Possible command:

```bash
yanawa docs
```

Public declarations may use documentation comments.

Possible syntax:

```yanawa
/// Finds a user by identifier.
///
/// Returns `none` when no user exists.
fn find_user(id: int) -> User?:
    ...
```

The exact documentation-comment syntax remains open.

## documentation output

Generated documentation should include:

* modules
* structs
* enums
* traits
* functions
* methods
* parameters
* return types
* examples
* linked type references

Documentation should be easy to host as static files.

A future official package registry may generate documentation automatically.

## documentation examples

Code examples inside documentation may eventually be compiled as part of tests.

This can prevent documentation from silently becoming outdated.

Possible behavior:

```text
yanawa test --docs
```

or automatic inclusion in:

```bash
yanawa test
```

The exact model remains undecided.

## language server

Editor support should use the Language Server Protocol where practical.

The official language server should support features such as:

* diagnostics
* completion
* hover information
* go to definition
* find references
* rename symbol
* signature help
* semantic highlighting
* document symbols
* workspace symbols
* formatting
* code actions

The language server should reuse compiler infrastructure rather than implement a second parser or type system.

## language server distribution

The language server should preferably ship with the yanawa toolchain.

Developers should not need to install a separate compiler-compatible LSP version manually.

Possible internal command:

```bash
yanawa lsp
```

This may exist primarily for editor integrations rather than direct human use.

## editor integrations

Official or first-party integrations may eventually exist for:

* Visual Studio Code
* Neovim
* Zed
* IntelliJ-based editors
* other LSP-compatible editors

The core language support should remain editor-independent through LSP.

Editor plugins should remain thin where possible.

## syntax highlighting

A lightweight grammar may provide highlighting before the full language server is available.

Eventually, semantic highlighting from the language server should distinguish symbols more accurately.

Possible categories include:

* types
* variables
* functions
* parameters
* fields
* traits
* enums
* keywords

## code completion

Completion should be type-aware.

Example:

```yanawa
let user: User = ...

user.
```

The editor may suggest:

```text
name
email
greet()
```

Completion should use the same semantic information as the compiler.

## go to definition

Developers should be able to navigate from:

```yanawa
users.find(id)
```

to the relevant declaration.

Navigation should work across:

* project modules
* dependencies
* standard library code

## rename

Symbol rename should understand scope and references.

Renaming:

```text
UserRepository
```

should update only references to that symbol, not arbitrary matching strings.

This requires semantic compiler information rather than textual replacement.

## diagnostics

Compiler diagnostics are one of the most important parts of the yanawa developer experience.

Errors should answer:

* what happened
* where it happened
* what was expected
* what was received
* what may fix the problem

Example:

```text
error: expected `int`, found `str`

  --> src/main.yna:4:20
   |
 4 | let age: int = "twenty"
   |                ^^^^^^^^ expected `int`
```

Diagnostics should prefer plain language over compiler jargon.

## diagnostic suggestions

Where confidence is high, diagnostics may include suggestions.

Possible example:

```text
error: `name` may be `none`

  --> src/user.yna:18:11
   |
18 | print(name.upper())
   |       ^^^^

help: check the value before using it

    if name != none:
        print(name.upper())
```

Suggestions should not overwhelm the primary error.

## error recovery

The compiler should attempt to continue analysis after an error when doing so is safe.

This allows one command to report multiple useful problems.

However, cascading errors caused entirely by one earlier failure should be suppressed when practical.

## machine-readable diagnostics

Editors and CI systems need structured diagnostics.

yanawa should eventually support a machine-readable output mode.

Possible command:

```bash
yanawa check --format json
```

Human-readable output should remain the default.

The internal diagnostic model should not be tied exclusively to terminal rendering.

## terminal output

CLI output should remain readable and restrained.

Color may improve diagnostics but should never be required to understand them.

The CLI should detect non-interactive environments and behave appropriately.

Possible global options:

```text
--color auto
--color always
--color never
```

Exact options remain undecided.

## exit codes

CLI commands should provide reliable exit codes.

General principle:

```text
0   success
non-zero   failure
```

Automation should not need to parse human-readable text to determine whether a command succeeded.

## quiet and verbose modes

Some workflows may benefit from reduced or increased output.

Possible flags:

```bash
yanawa build --quiet
yanawa build --verbose
```

The normal output should already be useful without requiring either mode.

Verbose output should primarily support debugging the toolchain itself.

## logging

Internal compiler/toolchain logs should remain separate from normal command output.

A debug option may eventually expose more information.

Possible:

```bash
YANAWA_LOG=debug yanawa build
```

or another consistent mechanism.

This should not pollute normal development workflows.

## clean

Generated artifacts may be removed with:

```bash
yanawa clean
```

This should not remove:

* source files
* manifest
* lockfile
* user-created project data

Cleanup should only target known generated artifacts.

## dependency commands

Package-system operations should integrate naturally with the CLI.

Examples:

```bash
yanawa add postgres
yanawa remove postgres
yanawa install
yanawa update
```

Developers should not need a separate package manager.

## dependency inspection

Possible future command:

```bash
yanawa deps
```

This may display the dependency graph.

Example:

```text
server 0.1.0
└── postgres 1.4.3
    ├── socket 2.1.0
    └── tls 1.8.2
```

## toolchain information

Developers should be able to inspect their installation.

Possible:

```bash
yanawa version
```

Output:

```text
yanawa 0.1.0
```

More detailed information may use:

```bash
yanawa version --verbose
```

Possible output:

```text
yanawa 0.1.0
compiler 0.1.0
target arm64-apple-darwin
```

Exact versioning depends on future architecture.

## environment diagnostics

A future command such as:

```bash
yanawa doctor
```

may help diagnose installation and environment problems.

Possible checks:

* installation path
* toolchain version
* target availability
* package cache
* permissions
* editor integration

This is useful but not required for the initial toolchain.

## toolchain installation

Installing yanawa should provide the complete basic toolchain.

Developers should not need to manually combine:

* compiler
* formatter
* package manager
* test runner
* language server

Distribution details remain undecided.

Possible future installation methods include:

* official installer
* system package managers
* downloadable archives
* source builds

## toolchain versions

The compiler and official tools should remain version-compatible.

Installing:

```text
yanawa 1.2
```

should provide a known compatible set of tools.

The project should avoid situations where developers independently manage incompatible formatter, LSP, and compiler versions.

## project-specific versions

Projects may eventually declare required yanawa versions.

Possible manifest:

```toml
[project]
yanawa = "0.4"
```

Tooling should clearly report mismatches.

Possible diagnostic:

```text
error: this project requires yanawa >= 0.4

installed version: 0.3.2
```

Whether automatic toolchain switching exists later is undecided.

## formatter stability

Formatting should be deterministic.

Given the same source and formatter version, `yanawa fmt` should produce the same output.

Formatter changes should be treated carefully because they can create large unrelated diffs across projects.

## lint stability

Lint rules may evolve more frequently than syntax.

New lint rules should avoid suddenly breaking existing projects unless explicitly configured as errors.

The toolchain should distinguish between:

* correctness diagnostics
* suspicious code
* style suggestions

## CI integration

The toolchain should work naturally in continuous integration environments.

A basic pipeline might run:

```bash
yanawa fmt --check
yanawa check
yanawa test
yanawa build --release
```

These commands should be deterministic and non-interactive in CI.

## official CI helpers

Official GitHub Actions or similar integrations may eventually simplify setup.

For example:

```text
setup-yanawa
```

However, core commands should remain usable without provider-specific tooling.

## caching

Compiler and dependency caches should improve performance automatically.

CI systems may cache these directories, but correctness must never depend on caches being present.

A corrupted cache should be safely recoverable.

## reproducibility

Tooling should support reproducible workflows.

Given the same:

* source
* manifest
* lockfile
* yanawa version
* build target

the build system should aim to produce equivalent behavior.

Exact binary reproducibility may require additional constraints and is a separate goal.

## build scripts

The core tooling should avoid arbitrary hidden build-script execution where possible.

Package installation should not casually execute untrusted code.

If build hooks become necessary for native interoperability, they should be:

* explicit
* constrained
* visible
* designed with security in mind

## code generation

Code generation may eventually be useful, but generated code should not become a requirement for ordinary language features.

yanawa should avoid recreating annotation processors or macro-heavy workflows simply to compensate for missing language functionality.

If code generation exists, its boundaries should remain clear.

## macros

A macro system is not currently required.

Macros can be powerful but also introduce:

* hidden code
* difficult diagnostics
* tooling complexity
* syntax fragmentation
* longer compilation

yanawa should first determine whether normal language features, generics, traits, and functions can solve the relevant problems.

## repl

An interactive REPL may eventually be useful.

Possible command:

```bash
yanawa repl
```

Example:

```text
yanawa> let name = "Alex"
yanawa> print("Hello, {name}")
Hello, Alex
```

Whether a REPL is practical depends on the final compilation and runtime model.

It should not be treated as an initial requirement.

## scratch execution

A lightweight alternative to a REPL may allow running a single source file.

Possible:

```bash
yanawa run script.yna
```

This could be useful for:

* experimentation
* scripts
* examples
* learning

The project model should not make simple programs unnecessarily difficult.

## documentation lookup

The CLI may eventually provide local documentation search.

Possible:

```bash
yanawa docs str
```

or:

```bash
yanawa help type str
```

This is a future ergonomics feature rather than a core requirement.

## compiler architecture reuse

The toolchain should avoid duplicating language logic.

Ideally, the following should share core compiler components:

* compiler
* `yanawa check`
* language server
* formatter where appropriate
* documentation generator
* test discovery
* static analysis

There should be one authoritative parser and semantic model.

Multiple independent implementations would create inconsistent behavior.

## stable internal APIs

Compiler internals do not need to become public APIs immediately.

Tooling should be allowed to evolve rapidly while the language is experimental.

Public plugin APIs should only be introduced when stable use cases emerge.

## plugin system

A general tooling plugin system is not currently required.

Before introducing one, yanawa should determine whether official tooling and external tools using documented interfaces are sufficient.

Plugins can increase flexibility but also fragment the development experience.

## extensibility

Third-party tools should eventually be able to work with yanawa through stable formats and protocols where useful.

Possible interfaces include:

* LSP
* machine-readable compiler diagnostics
* generated documentation metadata
* package metadata
* formatter integration

Extensibility should not require embedding arbitrary code inside the compiler.

## security

Tooling should treat project inputs as potentially untrusted.

Important areas include:

* dependency installation
* build hooks
* documentation generation
* editor integration
* compiler input
* package metadata

Opening an unknown yanawa project in an editor should not automatically execute arbitrary project code.

## performance

Tooling should feel responsive during normal development.

Important goals include:

* fast project checks
* fast incremental rebuilds
* low editor diagnostic latency
* efficient package caching
* parallel work where safe

Performance should be measured rather than assumed.

## startup time

Simple commands should start quickly.

For example:

```bash
yanawa fmt
yanawa check
yanawa version
```

should not require noticeable runtime initialization unrelated to their task.

A heavy runtime architecture should not make every tool command slow.

## telemetry

yanawa tooling should not silently collect user data.

If telemetry is ever introduced, it should be:

* clearly documented
* privacy-conscious
* optional
* transparent about what is collected

No telemetry is required for the initial project.

## error reporting

Compiler crashes and internal tool failures should be clearly distinguished from user-code errors.

Possible output:

```text
internal compiler error

the yanawa compiler encountered an unexpected failure

please report this issue with:
- yanawa version
- target
- minimal reproduction
```

Internal errors should never masquerade as ordinary source diagnostics.

## bug reports

A future:

```bash
yanawa bug-report
```

command may collect non-sensitive environment information useful for issue reports.

Any collected information should be shown before sharing.

This is a future convenience feature.

## standard workflow

A typical development workflow should eventually look like:

```bash
yanawa new server
cd server

yanawa add postgres

yanawa run
yanawa test
yanawa fmt
yanawa check
```

Preparing a release:

```bash
yanawa fmt --check
yanawa test
yanawa build --release
```

No external build system should be required for ordinary projects.

## minimal toolchain

The first useful toolchain does not need every feature described in this document.

A reasonable early toolchain may contain:

```text
yanawa run
yanawa build
yanawa check
yanawa fmt
yanawa test
```

Dependency commands may follow when the package system becomes functional.

The language server and advanced linting can evolve after the compiler has a stable enough semantic model.

## current direction

The current preferred direction is:

* `yanawa` is the primary toolchain command
* normal development should not require separate build or package tools
* `yanawa run` builds and executes projects
* `yanawa build` produces build output
* `yanawa check` provides fast semantic validation
* `yanawa fmt` provides canonical formatting
* `yanawa lint` provides first-party static analysis
* `yanawa test` provides first-party testing
* `yanawa docs` generates documentation
* dependency management integrates into the same CLI
* editor support should use LSP
* the language server should reuse compiler infrastructure
* diagnostics are a first-class part of the language experience
* tools should share compatible versions
* defaults should minimize configuration
* CI workflows should use the same commands developers use locally
* advanced extensibility should wait for real requirements

## open questions

The following areas remain unresolved:

* exact CLI command set
* CLI argument conventions
* build output layout
* development and release build semantics
* incremental compilation strategy
* formatter configuration boundaries
* lint architecture
* lint severity configuration
* testing syntax
* test discovery
* benchmark support
* documentation comment syntax
* documentation output format
* LSP architecture
* editor plugin distribution
* toolchain installation
* toolchain version management
* project-specific compiler versions
* REPL support
* script execution
* machine-readable compiler output
* cache locations
* plugin system
* build hooks
* code generation
* macro support
* telemetry policy
* internal compiler error reporting
* official CI integrations

The toolchain should make the correct workflow obvious.

> Good tooling should disappear into the development process.
