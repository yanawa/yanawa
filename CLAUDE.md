# CLAUDE.md

This file defines repository-specific instructions for Claude when working on the yanawa programming language.

Read `AGENTS.md` before making any changes.

`AGENTS.md` contains the general project principles and applies to all agents.

The rules in this file add Claude-specific workflow expectations.

## project context

yanawa is an experimental strongly typed programming language focused on:

* strong typing
* low ceremony
* familiar and readable syntax
* minimal framework dependence
* application-oriented development
* integrated first-party tooling
* predictable behavior

The project is still defining its language semantics and compiler architecture.

Do not confuse exploratory design with finalized specification.

## first action

Before changing the repository:

1. inspect `AGENTS.md`
2. inspect the relevant files under `docs/`
3. inspect the current repository state
4. inspect the current Git branch
5. inspect existing GitHub issues or milestone context when relevant

Do not begin implementation from assumptions when the repository already documents the subject.

## scope

Only perform the requested task.

Do not:

* implement adjacent roadmap items
* redesign unrelated components
* perform broad cleanup
* rename unrelated files
* introduce speculative abstractions
* add future features because they appear convenient

If another change becomes necessary to complete the requested task safely, keep it minimal and explain why it was required.

## design authority

Do not invent permanent yanawa language semantics.

If documentation says an area is unresolved, keep it unresolved unless the task explicitly asks to decide it.

Examples of unresolved areas may include:

* compiler implementation language
* production backend
* memory strategy
* concurrency model
* self-hosting
* runtime architecture
* HTTP language integration
* exact package-registry design

Do not silently convert implementation convenience into language design.

## implementation decisions

When implementation requires a temporary internal choice, distinguish between:

```text
language semantics
```

and:

```text
implementation detail
```

Implementation details should remain replaceable where reasonable.

For example, an interpreter may use host-language reference semantics internally without declaring that yanawa itself uses reference semantics.

## originality

Do not attempt to make yanawa unique by adding unfamiliar syntax without a concrete benefit.

Prefer familiar constructs where they are already clear.

The project should seek originality in:

* application semantics
* type safety
* tooling
* diagnostics
* framework reduction
* concurrency
* developer experience

rather than arbitrary syntax differences.

## repository language

All repository-facing content must be written in English, including:

* source identifiers where practical
* documentation
* comments
* commit messages
* issue titles
* issue descriptions
* milestone titles
* milestone descriptions
* labels
* pull request text

The brand must remain lowercase:

```text
yanawa
```

## public examples

Never use the repository owner's personal information in examples.

Use fictitious neutral names and data.

Examples:

```text
Alex
Morgan
Taylor
Riley
Jordan
Casey
```

## commits

When a requested task changes repository files, create a focused commit after the work is complete unless explicitly told not to commit.

Use English conventional-style commit messages.

Examples:

```text
docs: define compiler architecture direction
feat: tokenize integer literals
test: cover nested indentation
fix: report unresolved imports
refactor: simplify source span handling
chore: initialize compiler workspace
```

Rules:

* lowercase subject
* no trailing period
* concise description
* one meaningful change per commit
* do not bundle unrelated work

Before committing:

1. inspect the diff
2. run relevant validation
3. confirm no unrelated files changed
4. confirm generated or temporary files are not accidentally staged

## GitHub-only tasks

For tasks that modify only GitHub metadata, such as:

* milestones
* labels
* issues
* repository metadata

do not create a Git commit.

Use `gh` when appropriate.

Before creating GitHub resources:

1. inspect existing resources
2. avoid duplicates
3. preserve naming conventions
4. report exactly what was created or changed

Do not invent due dates unless explicitly requested.

## pushing

Do not push commits unless explicitly instructed.

A successful local commit does not imply permission to push.

## destructive actions

Do not perform destructive Git operations unless explicitly requested.

Avoid commands such as:

```text
git reset --hard
git clean -fd
git push --force
```

Do not rewrite history without explicit authorization.

## validation

Run the narrowest relevant validation for the change.

As the compiler evolves, this may include:

```text
formatting
linting
unit tests
integration tests
compiler checks
```

Do not claim validation succeeded unless it actually ran successfully.

If validation cannot run, report that clearly.

## documentation changes

When editing language documentation:

* preserve existing terminology
* keep design direction consistent across files
* distinguish accepted decisions from experimental syntax
* preserve unresolved questions
* do not turn illustrative syntax into specification accidentally

If a documentation change creates a contradiction with another document, resolve the inconsistency within the same task when appropriate.

## compiler work

For compiler changes:

* preserve source spans
* prefer structured diagnostics
* keep stages testable
* avoid target coupling in frontend code
* prioritize correctness
* write regression tests for bugs
* avoid premature optimization

Do not introduce a complicated compiler architecture before the current implementation requires it.

## diagnostics

Treat diagnostics as part of the feature itself.

An implementation is incomplete if it detects an error correctly but reports it poorly when a clear diagnostic is feasible.

Prefer:

```text
error: expected `int`, found `str`
```

over generic messages such as:

```text
type error
```

Preserve source location information wherever practical.

## dependencies

Before adding a dependency:

1. confirm it is necessary
2. inspect whether an existing dependency already solves the problem
3. prefer small and mature dependencies
4. consider security and maintenance impact
5. avoid adding large frameworks for small functionality

Report newly introduced dependencies in the final summary.

## generated files

Do not commit generated artifacts unless the repository intentionally tracks them.

Inspect `.gitignore` and existing conventions before staging.

## comments

Code comments should explain:

* non-obvious reasoning
* invariants
* unusual constraints

Do not comment code merely by restating what the code already says.

## TODO comments

Do not create vague TODOs.

Avoid:

```text
TODO: improve this later
```

If unfinished work must be recorded, prefer a GitHub issue or a precise comment explaining the missing behavior.

## issue discipline

Do not create large speculative issue backlogs unless explicitly requested.

Detailed issues should generally focus on near-term milestones.

Later milestones represent direction and should remain flexible until earlier prototypes provide evidence.

## reporting

After completing repository work, report:

* what changed
* validation performed
* commit hash and message, if a commit was created
* files changed
* anything intentionally left unresolved
* final repository status when relevant

Keep the report concise and factual.

## final repository state

Before finishing a file-changing task, inspect:

```bash
git status --short
```

The expected final state should normally be clean unless the task intentionally leaves files untracked or modified.

If the working tree is not clean, explicitly report why.

## guiding principle

When choosing between moving faster and preserving the coherence of the language, preserve coherence.

yanawa is not being built to maximize feature count.

It is being built to become a language with a clear identity.
