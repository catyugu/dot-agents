---
name: software-engineering
description: Guide software engineering decisions independently of programming language, from runnable prototypes and reproducible environments to data contracts, module boundaries, public APIs, tests, refactoring, CI, and distribution. Use when starting or formalizing a project, reviewing architecture or code organization, choosing abstractions and error policies, or planning build and release workflows. Leave language-specific conventions and tooling to the project and language-style-guide.
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
- Keep this guidance technology-neutral. Follow the repository and `language-style-guide` for language idioms, syntax, formatting, and specific tool commands rather than prescribing them here.
- **Start with something that runs.** For a new project, produce the smallest runnable program with observable or analyzable output before deciding the long-term layout. Once the prototype works, establish the foundations below and keep them stable until an actual problem justifies reopening a specific decision.

## 2. Establish the Project Foundations

### Reproducible Environment and Working Loop

- Choose a standard, maintainable toolchain and dependency-resolution approach for each part of the project. Declare dependencies and supported versions in the repository, commit lockfiles where supported, and make a fresh clone reproducible with one documented setup command. Record any prerequisites the project cannot pin.
- Keep the repository as the source of truth for environment and dependency resolution; do not invent a parallel build or package-management mechanism.
- Do not require undocumented global installations or ad-hoc environment state. Invoke tools through the declared project environment and use established build and packaging facilities.
- Document the working loop in the repository's user and contributor instructions, using project-native commands or a small common entry point. Each operation must actually work:
  - `setup`: configure and install the declared dependencies into the project environment.
  - `build`: produce the artifact, or be an explicitly documented no-op if no build is needed.
  - `test`: run the checks required for a change.
  - `run`: execute the program or a real example successfully.
  - `lint`: check formatting and static-analysis rules.
- Record consequential foundational decisions and their rationale in a concise, durable design note when warranted; do not create speculative process documentation.

### Types, Data Structures, and Invariants

- Choose consistent representations for recurring concepts such as identifiers, units, states, and numerical precision. Make conversions and semantic distinctions explicit where they affect correctness.
- For each consequential data structure or domain model, decide its meaning, minimal constraints, invariants, identity, ownership, lifetime, mutability, access rules, and whether generalization is actually needed. Put major choices before the user rather than silently locking in an architecture.
- Keep simple data structures simple. Couple data and operations only where that relationship expresses essential domain semantics.
- Avoid speculative classifications, hierarchy layers, factories, plugin hooks, and configuration branches without a current need. Prefer direct operations and composition when they reduce complexity.
- **Use assertions for programmer errors, not for external input validation.** Check actual internal invariants; reject invalid external data with explicit, actionable errors at system boundaries. Do not scatter defensive checks across trusted internal code.
- If an invariant is a production-safety requirement, enforce it with a check that remains enabled in production. An invariant failure indicates a defect; it is not a substitute for input validation.

### Logging and Error Handling

- Configure logging once at the application boundary; choose levels, formats, and destinations for the project's actual operators and consumers. Keep diagnostics separate from program output where applicable.
- Reusable components must not configure global logging or emit unsolicited output; let the application select sinks. Log significant boundary events with context and avoid commentary from hot loops. Prefer structured records for machine consumers and readable output for people.
- Define a consistent error contract: distinguish expected operational failures from programmer defects; specify how failures propagate, when they terminate work, and how callers observe them. Turn expected failures into actionable results at an appropriate boundary, and preserve diagnostic context for unexpected failures.
- Handle only failures that can be recovered, deliberately retried, or translated under a defined policy. Never suppress errors silently or report the same failure repeatedly; produce one actionable message where it can be used.
- Document externally observable failure statuses and exit codes where applicable. Centralize defaults and fallbacks; use them only when required by the contract, not to mask invalid internal state.

### Public API (Libraries and Reusable Modules)

- Explicitly designate the smallest supported public interface. Treat everything else as internal by default; avoid accidental exports, compatibility aliases, and forwarding layers.
- Keep public contracts independent of incidental implementation and deployment details unless those details are intentionally part of the contract.
- Every public operation and data contract is a maintenance promise: document its semantics and test observable behavior. Avoid leaking third-party implementation details into the interface.

### Tests and Verification

- Choose a testing utility, conventions, and granularity appropriate to the project. Favor a few high-value end-to-end or module-level tests covering real usage, core invariants, important interface contracts, and meaningful error paths.
- Add a regression test for a defect when feasible. Do not unit-test every function, branch, or hypothetical boundary case merely to increase test counts; coverage percentages are not a requirement or success criterion.
- Keep tests simpler than the software they exercise. Verify actual behavior rather than internal implementation details. Run the documented build, test, run, and lint operations before claiming success.

## 3. Writing and Organizing Code

### Style, Naming, and Modules

- Follow the project's established naming and style conventions. Prefer clear names, straightforward control flow, and readable expressions over clever compression and deeply nested conditions; leave syntax and idioms to the relevant language guidance.
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
- For installable libraries and reusable packages, verify installation, packaging, and consumption from an independent project using the installed artifact instead of the source tree.
- Decide whether the project is shipped as source, a package, an executable/library, or a deployed service, and which targets it supports. If the project will be installed or deployed as a service, add the corresponding CD workflow; otherwise do not invent release machinery.
- Distribution must reuse the declared toolchain and dependency versions instead of maintaining a second, conflicting dependency story. Use established ecosystem packaging tools, and verify the artifact from an external consumer's perspective.

## Done Looks Like

The project runs from a clean clone through a documented loop; core types and invariants are explicit; input validation, assertions, logging, and expected errors have distinct roles; the public API is small; meaningful tests pass; and CI, installation, and distribution exist only at the level the project's actual use requires. No unused extension point or compatibility layer survives without a real consumer.
