---
name: project-bootstrap
description: Turn a just-runnable program into a project with frozen foundations — pin the environment manager and build system, settle assertions, logging, error handling, public exports and distribution, then agree on the core data structures with the user before writing features. Use when starting a repository from scratch, when a prototype or single file starts to grow, when asked to "set up the project" or decide the layout and tooling, or when a new project needs its first honest design pass. Not for adding features to a project that already has these foundations.
category: development
version: "1.0"
license: "MIT"
---

# Project bootstrap

Bootstrap is a freeze, not a build-out. Each item gets decided once, written down, and left alone until a real need moves it. The order matters: every step constrains the next, and the last one constrains everything that follows.

## Gate: something already runs

Start once there is a runnable program — one file, one command, visible output. Layout decided before a program runs is guesswork, and the first real program falsifies most of it. If nothing runs yet, write the smallest thing that does, then come back.

## 1. Pin the environment manager

One per language, declared in the repo, with a committed lockfile. The test: a fresh clone plus one command produces the same interpreter and dependency set.

| Language | Manager | Declared in | One-command install |
| --- | --- | --- | --- |
| Python | uv | `pyproject.toml`, `.python-version`, `uv.lock` | `uv sync --locked` |
| Node | pnpm | `package.json` (`engines.node`), `pnpm-lock.yaml` | `pnpm install --frozen-lockfile` |
| Rust | cargo | `rust-toolchain.toml`, `Cargo.lock` | `cargo fetch` |
| C, C++ | CMake presets + pinned compiler | `CMakePresets.json`, `CMakeLists.txt` | `cmake --preset <name>` |
| Go | go toolchain | `go.mod` (`go` directive) | `go mod download` |
| JVM | gradle wrapper | `gradlew`, `.gradle/` files committed | `./gradlew build` |

Rules that keep it pinned:

- Never a global install to make the project work: no `pip install`, `npm i -g`, unversioned `npx` / `uvx`. Whatever the build needs gets declared and run through the project environment.
- The runtime version lives in the repo file, not in a README sentence.
- Build tooling (formatter, linter, test runner) is a declared dependency too, invoked through the same manager.

## 2. Freeze the build loop

Five names, one command each, implemented in one place (Makefile, justfile, package scripts, a `tasks.py`) and stated in the README:

| Command | Meaning |
| --- | --- |
| `setup` | install dependencies into the project environment |
| `build` | produce the artifact, or say "nothing to build" for interpreted projects |
| `test` | every check a change must pass |
| `run` | the program, on the sample input |
| `lint` | formatter check plus static analysis |

Every later instruction can then say "run `test`" and mean it; CI copies the same command. If no test exists yet, `test` runs the smoke check that proves the program runs.

Do not add CI, coverage gates, docs sites or release automation now. They encode a workflow that does not exist yet.

## 3. Foundational components

Decide these in order and record the decision somewhere durable (a README section or `docs/design.md`). Each is cheap now and expensive once code depends on it.

### Assertions

- Separate programmer error from user input: internal invariants fail loudly, external data returns an error. Never assert on input that comes from outside the program.
- One assertion per real invariant — sizes agree, ordering holds, a resource is owned exactly once — not one per statement.
- C and C++: `assert` vanishes in release builds. If the invariant must hold in production, use a check that aborts with a message and stays enabled.
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
- Everything else is private by default. Every exported name is a promise that owes a docstring and a test.
- No compat aliases or re-export shims. Rename outright and fix the callers.
- Scattered exports are not an API. One entry module that re-exports the intended surface, or none.

### Integration and distribution (only when something consumes it)

- Decide how it is installed and run *before* writing packaging: console entry point, container image, single binary, or "run it from the checkout".
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

Extensibility, concretely — and nothing more:

- a new field with a default value;
- a closed variant or enum when the alternatives are already known;
- a new function or table row when behaviour differs by configuration.

Refuse now: base classes with one subclass, registries, factories, plugin hooks, generic parameters for a single concrete use, config files before a second config value exists.

When a decision is hard to reverse — on-disk format, public API, identity — say so and ask for confirmation rather than inferring it. Record the outcome and its reason in the design note; a later reader needs the constraint, not the deliberation.

## Done looks like

- fresh clone, `setup`, `test`, `run` works with no undocumented global state;
- README states the five commands, the runtime version, and the error and exit-code contract;
- one place configures logging and no library prints;
- the public surface is explicit and small, internals are private by default;
- core data structures are plain data types with visible invariants, and the design note says why;
- nothing exists without a current caller: no plugin system, no config layer, no compat shim.

Features go on top from here. The foundations stay frozen until one of them fails in practice — then reopen that single decision, not the whole stack.
