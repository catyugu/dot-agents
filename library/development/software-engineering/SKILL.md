---
name: software-engineering
description: Build and maintain software with sound engineering foundations, from a first runnable prototype through reproducible toolchains, types and invariants, public APIs, tests, refactoring, CI, and distribution. Use when starting or formalizing a project, writing or reviewing code in any language, choosing data structures or abstractions, defining logging and error handling, refactoring, or shaping build and release workflows.
license: MIT
metadata:
  version: "2.1.0"
  author: "catyugu"
---

# Software Engineering

## 1. Operating Principles

- Treat software engineering as systems engineering; code is only one part of the system.
- Meet the actual functionality and correctness requirements, then minimize code, documentation, abstraction layers, dependencies, build and distribution machinery, runtime overhead, and maintenance cost.
- Design only for confirmed requirements. Do not add mechanisms for hypothetical features, extension points, or boundary cases that do not exist.
- Introduce an abstraction only when it expresses a real semantic distinction or significantly reduces complexity. Abstraction also creates coupling.
- Ask the user before committing to a consequential or uncertain architectural decision; present concrete alternatives and trade-offs instead of guessing.
- **Start with something that runs.** For a new project, produce the smallest runnable program with observable or analyzable output before deciding the long-term layout. Once the prototype works, establish the foundations below and keep them stable until an actual problem justifies reopening a specific decision.

## 2. Establish the Project Foundations

### Reproducible Environment and Working Loop

- Select one standard toolchain/dependency manager per language. Declare versions and dependencies in the repository, commit the relevant lockfile, and make a fresh clone reproducible with one documented setup command. Document compiler and platform prerequisites explicitly where a package manager cannot pin them.
- Follow the language ecosystem rather than inventing a new manager: Python (`uv`, `pyproject.toml`, `.python-version`, `uv.lock`); Node (`pnpm`, `package.json`, `engines.node`, lockfile); Rust (`cargo`, `rust-toolchain.toml`, `Cargo.lock`); C/C++ (`CMakeLists.txt` and declared compiler/toolchain requirements); Go (`go.mod` and its toolchain directive); JVM (Gradle wrapper).
- Never require ad-hoc global package installs (`pip install`, `npm install -g`, unversioned `npx` or `uvx`) to make a project work. Run project dependencies through its declared environment. Use standard build and packaging tools rather than reimplementing them.
- Document the working loop in `README.md` and `AGENTS.md`, using native tool commands or a small common entry point. Each operation must actually work:
  - `setup`: configure and install the declared dependencies into the project environment.
  - `build`: produce the artifact, or explicitly do nothing for an interpreted project.
  - `test`: run the checks required for a change.
  - `run`: execute the program or a real example successfully.
  - `lint`: check formatting and static-analysis rules.
- Record durable foundational decisions, not meeting notes or speculation, in `docs/design.md` or focused files under `docs/design/` when the project warrants them.

### Types, Data Structures, and Invariants

- Choose consistent representations for indices, handles, flags, real numbers, and other recurring concepts. Use meaningful aliases when they express a real contract and prevent needless conversions.
- For each consequential data structure, decide its purpose, minimal constraints, fields, generic versus fixed design, invariants, identity, ownership, lifetime, mutability, and read/write permissions. Put major choices before the user rather than silently locking in an architecture.
- Prefer plain data for data containers. Give member functions only to types with genuine semantic operations, such as sparse matrices or quaternions.
- Do not invent closed variants/enums to restate already-known alternatives without useful type semantics, or add configuration-driven behavior branches where a direct operation suffices. Avoid base classes with one subclass, speculative factories, plugin hooks, and multiple inheritance; favor simple data and composition.
- **Use assertions for programmer errors, not for external input validation.** Assert actual internal invariants (shape agreement, ordering, ownership, etc.); turn bad external inputs into explicit errors. Do not scatter defensive checks across trusted internal code.
- In C/C++, remember that `assert` may disappear in release builds. Use an always-enabled failing check for an invariant that must be enforced in production. A failed assertion means the implementation is wrong, not that validation succeeded.

### Logging and Error Handling

- Configure logging once at the application entry point; use consistent levels and formats. INFO is a reasonable default, with DEBUG for detailed diagnosis. Send diagnostics to `stderr` and program data to `stdout`.
- Libraries must not configure global logging or print diagnostics; they emit log records and let the application choose the sink. Log boundary events with useful context (path, identifier, duration, outcome), not every internal step. Avoid non-DEBUG logging in hot loops. Use JSON lines for machine consumers and readable text for people.
- Define a consistent error contract: what returns an error, what throws or propagates, which invariants abort, and what exit codes mean. Expected failures (bad input, missing file, unavailable network) return explicit errors or typed exceptions handled at an appropriate boundary. Unexpected failures propagate with diagnostic context.
- Catch only failures that can be handled, deliberately retried, or converted at the boundary, normally an I/O or application boundary. Never silently swallow an error or catch everything without a defined policy. Produce one actionable message where the error becomes actionable; do not repeatedly wrap or log the same failure.
- Use exit code `0` for success and documented, distinct nonzero codes for meaningful failure classes where the program is a CLI. Centralize defaults and fallbacks; use them only when required by the contract, not to mask invalid internal state.

### Public API (Libraries and Reusable Modules)

- Explicitly designate the small supported public surface (`__all__`, `pub`, exported headers, or a single entry module as appropriate). Everything else is private by default. Avoid scattered re-exports, compatibility aliases, and forwarding shims.
- For C/C++ libraries, make cross-platform export/visibility declarations explicit where shared-library builds need them.
- Every public name is a maintained contract: document its semantics and test its observable behavior. Keep third-party implementation details out of public interfaces unless they are intentionally part of the contract.

### Tests and Verification

- Choose a testing utility, conventions, and granularity appropriate to the project. Favor a few high-value end-to-end or module-level tests covering real usage, core invariants, important interface contracts, and meaningful error paths.
- Add a regression test for a defect when feasible. Do not unit-test every function, branch, or hypothetical boundary case merely to increase test counts; coverage percentages are not a requirement or success criterion.
- Keep tests simpler than the software they exercise. Verify actual behavior rather than internal implementation details. Run the documented build, test, run, and lint operations before claiming success.

## 3. Writing and Organizing Code

### Style, Naming, and Modules

- Follow the project's established conventions and the language ecosystem. Prefer clear names, straightforward control flow, and readable expressions over clever compression and deeply nested conditions.
- Give each module one distinct responsibility. Keep dependencies unidirectional, clear, and traceable; avoid cycles, duplicate responsibility, and layers that only forward or aggregate.
- Couple modules through documented data contracts and interfaces rather than another module's implementation details. Accept a little local duplication when it avoids unnecessary cross-module coupling.
- Avoid adding defensive checks inside trusted modules beyond the assertions needed for real invariants; validate external data at boundaries.
- If a function needs many parameters, check whether it has too many responsibilities or whether some inputs form a meaningful group. Do not impose an arbitrary argument count or introduce an empty configuration object just to shorten a signature.

### Comments and Documentation

- Make names, interfaces, and structure explain the ordinary code. Comment only the non-obvious contracts, units, layouts, invariants, side effects, unusual formulas, and decisions that matter to maintainers.
- Avoid narrating implementation line by line. Comments that duplicate code become stale; update or delete outdated documentation with the code it describes.
- Keep `README.md` focused on purpose, setup, use, and the working loop. Separate user-facing usage from maintainer design notes; generate detailed API reference from code when practical.
- Do not maintain checklists, process documents, or auxiliary tools without a demonstrated purpose. Public API documentation is an exception to the general preference for fewer comments: every exported contract needs explanation.

## 4. Refactoring

- Refactor with a concrete goal: simplify logic, remove duplication, clarify responsibilities, improve an interface or performance, or fix a defect. First understand the call graph, data contracts, and actual consumers.
- Prefer subtraction: delete, merge, inline, move, or unify before introducing an abstraction. Remove dead helpers, duplicated defaults and checks, unnecessary wrappers and conversions, unused caches, speculative generalizations, old implementations, and obsolete configuration paths.
- Preserve the required external behavior, but change internal structure decisively when a local patch would perpetuate a poor design. Do not expand the public surface, dependencies, or configuration without a requirement.
- Preserve performance, readability, and testability rather than optimizing solely for line count. Change or delete tests that depended on discarded internals; do not preserve unreasonable production interfaces just to keep those tests passing.
- Remove stale comments, documentation, and supporting code along with the obsolete implementation. Do not leave compatibility shims for callers you can update directly.

## 5. Integration and Distribution

- Keep platform-independent build logic unified; localize unavoidable platform differences. State which platforms, toolchains, and feature combinations the project supports. Do not make downstream consumers install unnecessary tools or carry incidental dependencies.
- **Add CI when a project is remote and may involve collaboration.** Keep it minimal and maintainable: automate the essential builds, tests, and lint checks, while retaining a useful local verification loop. Do not delegate every check to CI.
- For installable C/C++ libraries, implement and verify installation, packaging, exported targets, and consumption from a separate project using the installed artifact rather than the source tree.
- Decide whether the project is shipped as source, a package, an executable/library, or a deployed service, and which targets it supports. If the project will be installed or deployed as a service, add the corresponding CD workflow; otherwise do not invent release machinery.
- Distribution must reuse the declared toolchain and dependency versions instead of maintaining a second, conflicting dependency story. Use established ecosystem packaging tools, and verify the artifact from an external consumer's perspective.

## Done Looks Like

The project runs from a clean clone through a documented loop; core types and invariants are explicit; input validation, assertions, logging, and expected errors have distinct roles; the public API is small; meaningful tests pass; and CI, installation, and distribution exist only at the level the project's actual use requires. No unused extension point or compatibility layer survives without a real consumer.
