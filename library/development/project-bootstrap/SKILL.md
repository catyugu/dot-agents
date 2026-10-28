---
name: project-bootstrap
description: Bootstrap project from a just-runnable program to a project with sound foundations.
version: "1.0.0"
author: "catyugu"
license: "MIT"
---

# Project bootstrap

Bootstrap a project that may evolve and scale. Start from a prototype, then fix down the core and foundation, make it consistent in style, easy to maintain and reproduce.

## When to use the skill

- When you are starting any project.
- When you are turning a prototype into a real project.

## 1. Build casually for that runs

Start once there is a runnable program — one or few files, commands, with analyzable or visible output. Layout decided before a program runs is guesswork, and the first real program falsifies most of it. If nothing runs yet, write the smallest thing that does.

## 2. Pin the environment/toolchain manager

One per language, declared in the repo, with a committed lockfile. The test: a fresh clone plus one command produces the same interpreter and dependency set.

| Language | Manager | Declared in |
| --- | --- | --- | --- |
| Python | uv | `pyproject.toml`, `.python-version`, `uv.lock` |
| Node | pnpm | `package.json` (`engines.node`), lock file for package manager |
| Rust | cargo | `rust-toolchain.toml`, `Cargo.lock` |
| C, C++ | cmake |  `CMakeLists.txt`  |
| Go | go toolchain | `go.mod` (`go` directive) |
| JVM | gradle wrapper | `gradlew`, `.gradle/` file |

**NEVER a global install to make the project work**: no `pip install`, `npm i -g`, unversioned `npx` / `uvx`. Whatever the build needs gets declared and run through the project environment.

## 2. Freeze the working loop

in `README.md` and `AGENTS.md`, fix down how to do the following:

| Stage | Meaning |
| --- | --- |
| `setup` | configure, install dependencies into the project environment |
| `build` | produce the artifact, or no-op for interpreted projects |
| `test` | every check a change must pass |
| `run` | the program or examples that we produce shall actually work |
| `lint` | formatter check plus static analysis |

## 3. Foundational components

Decide these in order and record the decision somewhere durable (`docs/design.md` or `docs/design/*.md` for larger projects).

### Types

- Usually for languages like C/C++/rust where you may have to decide what data types to use in different cases.
- What types to use for table index or handler? What types to use for flags? What types to use for Real number?
- Use aliased and consistent data types makes the type system more flexible and maintainable, also it reduces unnecessary type conversions.

### Assertions

- Separate programmer error from user input: internal invariants fail loudly, external data returns an error. Never assert on input that comes from outside the program.
- One assertion per real invariant — sizes agree, ordering holds, a resource is owned exactly once — not one per statement.
- C and C++: `assert` vanishes in release builds. If the invariant must hold in production, use a check that aborts with a message and stays enabled. However you may still use macros to bring up the standard assertion functionalities into better semantics.
- A failing assertion means the code is wrong, not the data. It is not a validation layer.

### Logging

- One configuration point at process start. Levels. Diagnostics to `stderr`, data to `stdout`.
- Library code never configures logging and never `print`s: it emits records, the entry point decides the sink.
- Log boundary events with their context (path, id, duration, outcome), not a running commentary of internal lines.
- One format per stream: structured JSON lines when a machine consumes it, plain text when a person does.

### Error handling

- Write the contract down: what raises or returns, what aborts, what each exit code means.
- Expected failures (missing file, bad input, network down) are return values or typed errors caught at the boundary and turned into a message and exit code. Unexpected failures propagate with their stack.
- No swallowing. A caught error is handled, retried deliberately, or converted — and the handler names the failure it expects. Bare `except:` and silent `catch` are bugs.
- One message per error, at the point where it becomes actionable; do not wrap the same error at every layer.
- Exit codes: `0` success, distinct non-zero values per failure class, documented.

### Public surface (only for a library or reusable module)

- Say explicitly what is public: `__all__`, `pub` versus private modules, a header listing exported names, `pub(crate)` where it applies.
- For C/C++: consider using some conventional macros to export things so that it can be built on different platforms and toolchains. 
- Everything else is private by default. Every exported name is a promise that owes a docstring and a test.
- No compat aliases or re-export shims. Rename outright and fix the callers.
- Scattered exports are not an API. One entry module that re-exports the intended surface, or none.

### Integration and distribution(optional)

- For projects with remote and potential colaboration, you MUST add CI.
- Decide if it will be installed or deployed as a service, if YES, you shall add CD.

If you really need CD:

- Distribution reuses the same lockfile; do not grow a second dependency story for packaging.
- Stop there. No installer, no release matrix, no auto-versioning until another person needs the artifact.

## 4. The data-structure conversation

This decides the project's shape and is the one step never done alone. Draft it, put it in front of the user, get a verdict on each item, then write the result down.

Draft the minimum core — typically three to seven entities. For each: name and one sentence of what it is; fields with types and units; invariants (what is always true, what is never allowed); identity and lifetime (what makes two instances equal, who owns it, what may mutate); serialization, if any (field names become API when the format is public).

Present it as a table and ask one concrete question per row, not "what do you think":

| Question | What the answer decides |
| --- | --- |
| Are A and B one entity or two? | split them if they change independently; merge if they always change together |
| Is that field optional, or is there a second variant? | optional fields on one type are the most common design debt |
| Who mutates this after construction? | nobody means immutable; many means one owner and one write path |
| Is the identifier ours or the world's? | ours: opaque, never parsed back into meaning |
| Does this need to be general? | minimum now; extension later is a field with a default or a variant |

Don't write:

- a new field with a default value;
- a closed variant or enum when the alternatives are already known;
- a new function or table row when behaviour differs by configuration.

Refuse for now: base classes with only one subclass, factories, plugin hooks, config files before a second config value exists.

## Done looks like

- one place configures logging and no library prints;
- the public surface is explicit and small, internals are private by default;
- core data structures are plain data types with visible invariants, and the design note says why;
- nothing exists without a current caller: no plugin system, no config layer, no compat shim.

Features go on top from here. The foundations stay frozen until one of them fails in practice — then reopen that single decision, not the whole stack.
