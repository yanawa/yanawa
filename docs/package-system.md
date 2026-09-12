# yanawa package system

yanawa should provide a simple and reproducible package system integrated into the main `yanawa` toolchain.

Projects should not need an external build tool or package manager for ordinary dependency management.

> One language, one toolchain, one obvious workflow.

This document describes the current direction of projects, manifests, dependencies, lockfiles, packages, and dependency management in yanawa.

Nothing here should be considered stable or final while the language remains experimental.

## core principles

The yanawa package system should be:

* simple
* reproducible
* predictable
* integrated with the main CLI
* easy to understand
* fast enough for normal development
* explicit about dependencies
* compatible with local development
* resistant to unnecessary ecosystem complexity

Developers should not need to learn a separate build system merely to compile and run a yanawa project.

## terminology

yanawa should clearly distinguish between a project, package, and module.

### project

A project is a development workspace managed by yanawa.

Possible structure:

```text
hello/
├── yanawa.toml
├── yanawa.lock
└── src/
    └── main.yna
```

### package

A package is a reusable or executable unit that may be versioned and distributed.

A project may contain one or more packages in the future, but the initial model should prefer one package per project.

### module

A module is part of the source-code organization inside a package.

Example:

```text
src/
├── main.yna
└── user/
    ├── model.yna
    └── service.yna
```

Modules should not require independent package metadata.

## project manifest

Every managed yanawa project should have a single manifest.

Current candidate:

```text
yanawa.toml
```

Example:

```toml
[project]
name = "hello"
version = "0.1.0"
yanawa = "0.1"

[dependencies]
```

The manifest should remain human-readable and easy to edit manually.

## project metadata

Possible project metadata may include:

```toml
[project]
name = "hello"
version = "0.1.0"
description = "A small yanawa application"
license = "MIT"
yanawa = "0.1"
```

Additional metadata should only be added when useful.

The manifest should not become a dumping ground for unrelated configuration.

## source directory

The default project structure may use:

```text
src/
```

Application:

```text
src/main.yna
```

Library:

```text
src/lib.yna
```

Whether `src/` is mandatory remains undecided.

Conventions should reduce configuration without unnecessarily restricting unusual projects.

## creating a project

The main CLI should create projects.

Possible command:

```bash
yanawa new hello
```

Result:

```text
hello/
├── yanawa.toml
└── src/
    └── main.yna
```

A lockfile does not need to exist before dependencies are resolved.

Possible initial `main.yna`:

```yanawa
fn main():
    print("Hello from yanawa")
```

## one CLI

Dependency management should remain part of the main `yanawa` command.

Possible commands:

```bash
yanawa new
yanawa add
yanawa remove
yanawa install
yanawa update
yanawa run
yanawa build
yanawa test
```

yanawa should avoid requiring a separate package-manager executable unless a future technical requirement clearly justifies it.

## adding dependencies

A dependency may be added through the CLI.

```bash
yanawa add postgres
```

This should update the manifest automatically.

Possible result:

```toml
[dependencies]
postgres = "1.4.0"
```

The CLI may resolve the latest compatible stable version when no version is specified.

Explicit version:

```bash
yanawa add postgres@1.4.0
```

The exact command syntax remains under design.

## removing dependencies

Possible command:

```bash
yanawa remove postgres
```

This should:

* remove the dependency from the manifest
* update dependency resolution
* update the lockfile
* remove unused transitive dependencies from the resolved graph

## dependency declarations

Basic dependency syntax should remain concise.

```toml
[dependencies]
postgres = "1.4"
json-schema = "2.1"
```

More complex declarations may eventually use expanded tables.

Possible example:

```toml
[dependencies.postgres]
version = "1.4"
features = ["tls"]
```

Complex syntax should only exist when required.

## development dependencies

Dependencies used only for development may have a separate section.

```toml
[dev-dependencies]
test-data = "1.2"
```

Possible uses include:

* testing helpers
* benchmarks
* development tooling
* local fixtures

Core testing support should preferably remain first-party rather than requiring third-party packages.

## local dependencies

Local packages should be easy to use during development.

Possible example:

```toml
[dependencies]
shared = { path = "../shared" }
```

This is useful for:

* multi-project development
* testing packages before publication
* internal libraries

Local dependencies should not require publication to a registry.

## git dependencies

Git dependencies may eventually be supported.

Possible example:

```toml
[dependencies]
parser = { git = "https://github.com/example/parser", rev = "abc123" }
```

However, registry packages and local paths should remain the preferred forms for normal projects.

Git dependencies can reduce reproducibility when branches or mutable references are used.

## lockfile

yanawa should use a lockfile for reproducible dependency resolution.

Candidate:

```text
yanawa.lock
```

The lockfile should contain exact resolved versions and integrity information.

Example conceptual contents:

```text
postgres 1.4.3
socket 2.1.0
tls 1.8.2
```

The actual format remains undecided.

Developers should not normally edit the lockfile manually.

## reproducible builds

Given the same:

* project source
* manifest
* lockfile
* yanawa version
* supported target environment

dependency resolution should produce the same package graph.

Reproducibility should be a core property of the package system.

## committing the lockfile

The current preferred direction is for application projects to commit:

```text
yanawa.lock
```

Libraries may require a different policy depending on how downstream dependency resolution works.

The final rule should remain simple and well documented.

## semantic versioning

yanawa packages may initially use semantic versioning.

Example:

```text
1.4.2
```

representing:

```text
major.minor.patch
```

Possible interpretation:

* major — incompatible API changes
* minor — backward-compatible features
* patch — backward-compatible fixes

SemVer is familiar and sufficiently expressive for an initial ecosystem.

yanawa should not invent a custom versioning system without a strong reason.

## version requirements

Possible dependency requirements:

```toml
postgres = "1.4"
```

may mean a compatible version within the `1.x` line.

Exact rules must be defined carefully.

Possible explicit forms may eventually include:

```text
=1.4.2
>=1.4
<2.0
```

The system should avoid excessively complex version expressions unless real usage requires them.

## dependency resolution

The resolver should produce one predictable dependency graph.

It should detect:

* incompatible requirements
* dependency cycles where relevant
* unavailable versions
* unsupported yanawa versions
* corrupted package metadata

Errors should explain the conflicting requirements clearly.

Possible diagnostic:

```text
error: incompatible dependency requirements for `http`

package `api` requires http >= 2.0
package `client` requires http < 2.0
```

## transitive dependencies

Applications should generally interact only with dependencies declared directly in their manifest.

If:

```text
application -> postgres -> socket
```

the application should not automatically depend on `socket` as part of its public source namespace.

This prevents accidental reliance on implementation details of another package.

## package visibility

Only explicitly declared direct dependencies should normally be importable.

Example:

```toml
[dependencies]
postgres = "1.4"
```

This allows:

```yanawa
import postgres.Client
```

but not necessarily direct access to PostgreSQL's internal transitive dependencies.

## dependency graph

Tooling should eventually allow developers to inspect the resolved graph.

Possible command:

```bash
yanawa deps
```

Possible output:

```text
hello 0.1.0
└── postgres 1.4.3
    ├── socket 2.1.0
    └── tls 1.8.2
```

This improves debugging and transparency.

## installing dependencies

Possible command:

```bash
yanawa install
```

This should resolve dependencies from:

```text
yanawa.toml
```

while respecting:

```text
yanawa.lock
```

If a valid lockfile exists, installation should not unexpectedly upgrade packages.

## updating dependencies

Explicit updates may use:

```bash
yanawa update
```

Specific package:

```bash
yanawa update postgres
```

Updating should modify the lockfile according to the project's declared version constraints.

Dependency upgrades should never happen silently during unrelated commands.

## build integration

Commands such as:

```bash
yanawa build
yanawa run
yanawa test
```

should automatically ensure required dependencies are available.

Developers should not normally need to run a separate installation command before every build.

The toolchain may resolve missing dependencies automatically while respecting reproducibility rules.

## package cache

Downloaded packages should be cached globally where practical.

Benefits include:

* faster builds
* reduced network usage
* offline development
* storage deduplication

Projects should not need a large dependency directory copied inside every workspace unless the final execution model requires it.

## project artifacts

Generated build artifacts should remain separate from source code and dependency metadata.

Possible directory:

```text
yanawa/
```

or:

```text
build/
```

or another dedicated tool directory.

The exact layout should be decided alongside compiler architecture.

Generated dependency caches should not pollute the source tree unnecessarily.

## registry

yanawa may eventually provide an official package registry.

Possible conceptual identity:

```text
packages.yanawa.org
```

or another official service.

A registry is not required for the first compiler prototype.

The ecosystem should not build registry infrastructure before developers can create useful packages.

## publishing

Future package publication might use:

```bash
yanawa publish
```

Before publishing, tooling should validate:

* package name
* version
* manifest
* build success
* package contents
* dependency metadata

Publishing should be deliberate.

## package names

Package names should be:

* easy to type
* case-consistent
* URL-friendly
* predictable in imports
* resistant to confusing variations

A possible convention is lowercase kebab-case for registry identities:

```text
http-client
json-schema
postgres
```

Import namespaces may map these names into valid yanawa identifiers.

The exact mapping remains undecided.

## name ownership

An official registry will eventually require a policy for package names.

Potential issues include:

* abandoned names
* squatting
* impersonation
* trademark conflicts
* malicious lookalike packages

This does not need to be solved during the initial language implementation, but the system should be designed with these risks in mind.

## namespaces

yanawa may eventually support package namespaces.

Possible examples:

```text
acme/http
yanawa/postgres
```

or:

```text
@acme/http
```

Namespaces could help with:

* organizations
* official packages
* private packages
* name conflicts

However, they also introduce additional complexity.

No namespace model has been chosen.

## first-party packages

Official yanawa packages should be clearly identifiable without pretending they belong to the core standard library.

Possible naming model:

```text
yanawa-postgres
yanawa-sqlite
yanawa-web
```

or an official namespace.

The exact convention remains open.

## language compatibility

Packages may specify compatible yanawa versions.

Possible manifest:

```toml
[project]
yanawa = "0.4"
```

The package manager should detect clearly incompatible packages before compilation.

Possible diagnostic:

```text
error: package `postgres` requires yanawa >= 0.5

current yanawa version: 0.4.2
```

## features

Optional package features may eventually exist.

Possible example:

```toml
[dependencies.postgres]
version = "1.4"
features = ["tls"]
```

Features can reduce unnecessary dependencies and functionality.

However, feature systems can become difficult to reason about.

yanawa should not introduce them until real package requirements justify the complexity.

## build scripts

Arbitrary dependency build scripts should be treated cautiously.

They create security and reproducibility concerns because installing a package may execute code automatically.

The initial package system should avoid arbitrary install-time execution if possible.

Native interoperability may eventually require controlled build hooks.

Any such capability should be explicit and sandboxable where practical.

## security

Package management introduces supply-chain risk.

Future security measures may include:

* package integrity hashes
* signed metadata
* immutable published versions
* provenance information
* dependency auditing
* malicious-package reporting
* registry account protection

The initial system should at minimum support integrity verification through the lockfile or registry metadata.

## immutable releases

Published package versions should ideally be immutable.

Once:

```text
postgres 1.4.0
```

is published, its contents should not silently change.

Corrections should require a new version.

This improves reproducibility and trust.

## dependency integrity

The lockfile should eventually record enough information to detect tampered package contents.

Possible conceptual field:

```text
checksum
```

A downloaded dependency whose checksum does not match should be rejected.

## offline development

Projects should be buildable without network access when all required packages already exist in the local cache.

Possible future option:

```bash
yanawa build --offline
```

This can improve:

* reproducibility
* CI reliability
* travel/offline development
* dependency debugging

## private packages

Private registries or organization packages may eventually be supported.

This is not required for the initial ecosystem.

Any authentication model should avoid storing credentials directly in project manifests.

## workspaces

Large repositories may eventually contain multiple related packages.

Possible structure:

```text
project/
├── yanawa.toml
├── api/
├── core/
└── cli/
```

A workspace model may allow them to:

* share one lockfile
* share build caches
* reference each other locally
* run tests together

Workspaces should not be introduced before single-package projects work well.

## scripts

The manifest should not become a generic shell-script runner by default.

Commands such as:

```bash
yanawa test
yanawa build
yanawa run
yanawa fmt
```

should already cover common project workflows.

Custom task execution may eventually be useful, but it should not recreate the complexity of external build systems inside the manifest.

## package documentation

Published packages should eventually integrate with first-party documentation tooling.

Possible command:

```bash
yanawa docs
```

A registry may display:

* README
* API documentation
* versions
* dependencies
* compatibility
* source repository
* license

Documentation should remain closely connected to package source.

## package licenses

Published package metadata should support license declarations.

Example:

```toml
[project]
license = "MIT"
```

Registry tooling may eventually validate known license identifiers.

## dependency auditing

Future tooling may support:

```bash
yanawa audit
```

Potential checks include:

* known vulnerabilities
* deprecated packages
* compromised versions
* abandoned dependencies

This should be considered future ecosystem infrastructure rather than an initial compiler requirement.

## initialization

Existing directories may be converted into yanawa projects.

Possible command:

```bash
yanawa init
```

This could create:

```text
yanawa.toml
```

without generating a new directory.

## example project

```text
server/
├── yanawa.toml
├── yanawa.lock
└── src/
    ├── main.yna
    └── user/
        ├── model.yna
        └── repository.yna
```

Possible manifest:

```toml
[project]
name = "server"
version = "0.1.0"
yanawa = "0.1"

[dependencies]
postgres = "1.4"
```

Application:

```yanawa
import postgres.Client
import user.UserRepository

async fn main():
    let database = await Client.connect(
        env.require("DATABASE_URL")?
    )?

    let users = UserRepository(
        database = database
    )

    ...
```

The exact APIs remain experimental.

## current direction

The current preferred direction is:

* projects use a single manifest
* `yanawa.toml` is the current manifest candidate
* `yanawa.lock` records exact dependency resolution
* dependency management belongs to the main `yanawa` CLI
* one package per project should be the simplest initial model
* local path dependencies should be supported
* semantic versioning is the initial versioning candidate
* applications should use lockfiles for reproducibility
* direct dependencies should be explicit
* transitive dependencies should not automatically become public imports
* downloaded dependencies should use a shared cache where practical
* an official registry may exist later but is not required for the first compiler
* published versions should eventually be immutable
* supply-chain security should be considered from the beginning
* advanced features such as workspaces, private registries, and arbitrary build scripts should wait for real requirements

## open questions

The following areas remain unresolved:

* final manifest name and format
* final lockfile format
* application versus library lockfile policy
* exact SemVer requirement syntax
* dependency resolver strategy
* package naming rules
* package namespace model
* official package naming
* registry architecture
* registry domain
* package publication workflow
* package authentication
* private packages
* feature flags
* native dependencies
* controlled build hooks
* package signing
* dependency auditing
* workspaces
* multiple packages per project
* source directory conventions
* build artifact layout
* offline behavior
* package documentation hosting

The package system should solve dependency management without becoming a second programming language developers need to learn.

> Dependencies should be explicit. Builds should be reproducible. Tooling should stay boring.
