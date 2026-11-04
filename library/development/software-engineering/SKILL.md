---
name: software-engineering
description: Software engineering habits that decide whether a change holds up — system-level thinking, naming and code style, module boundaries, data-oriented design, logging and error handling, comments and docs, tests, refactoring, and build/dependency/distribution discipline. Load it when writing or reviewing code in any language, planning or carrying out a refactor, deciding whether to add an abstraction or a dependency, setting logging and error-handling policy, or shaping build, CI and packaging for a project.
version: "2.0.0"
author: "catyugu"
license: "MIT"
---

# Software Engineering

## 1. Basic Principles

- Software engineering is systems engineering; code is only one part of it.
- Aim for simple, clear, efficient, and reliable systems. Once functionality and correctness requirements are met, reduce the amount of code, documentation, build, integration, and distribution system as much as possible, along with abstraction levels, runtime overhead, and maintenance burden.
- Design only for confirmed requirements. Do not build mechanisms in advance for imagined use cases, possible future features, or boundary cases that do not yet exist.
- Avoid meaningless abstraction. Abstraction usually means coupling. Introduce an abstraction only when it genuinely expresses clear semantics or significantly reduces complexity.

## 2. Writing Code

### Style and Naming

- Keep naming style consistent and follow the conventions of the language and its ecosystem used by the project.
- Names, interfaces, and control flow should be as self-explanatory as possible rather than relying on comments and documentation.
- Avoid anything that seriously harms readability, such as overly compressed expressions, complex nested conditions, and multi-level nested function calls.
- Prefer verbose but easy to understand and test code over clever, terse, hard-to-read code.

### Modules and Overall Design

- Each module should have a clear, stable responsibility; avoid several modules overlapping on the same work.
- Module dependencies should be as unidirectional, clear, and traceable as possible; avoid circular dependencies. A reasonable amount of code duplication is acceptable when it avoids unnecessary cross-module coupling and indirection.
- A module should have a clear, distinct function; avoid intermediate modules that only forward or aggregate.
- Modules should always depend on data structures and structural interoperation between each other, never on each other's concrete implementation.
- Inside a module, trust the input data (usually only the necessary assertions at the start of a function; avoid check branches). At interfaces and on the user side, stay wary of input data.

### Data and Operations

- Think in terms of data-oriented programming: pure data containers generally should not provide any member functions; only containers that need specific semantics (sparse matrices, quaternions, and the like) should provide limited, essential member functions.
- Always avoid multiple inheritance; what multiple inheritance can accomplish can usually be accomplished with composition.
- A function should avoid too many parameters. With more than four parameters, check whether it has too many responsibilities, whether related parameters can be grouped, and whether dependencies can be bound in advance. Do not treat parameter count as a hard limit, though. Several independent, clearly meaningful parameters are usually better than a configuration object that lacks real semantics.

### Logging, Exceptions, and Unexpected Behavior

- Always manage default and fallback values in one place; never scatter them. Unless you are at the IO layer, do not fall back without a clear reason or business requirement; invalid data should go straight down the exception path.
- A program should generally have logs. Logs should always be consistent and leveled; the default level is usually INFO, and DEBUG-level logs are usually only needed while debugging. Avoid overusing logs: apart from DEBUG-level entries, no log should be emitted inside a hot loop.
- Establish one error-handling style inside the program. Avoid try-catch outside the IO layer unless there is a compelling reason for it. A try-catch deep in the system can hide real errors and lead to performance fluctuations and testing difficulties.

### Comments and Documentation

- Do not overuse comments. Naming, interface design, and code style should be as self-explanatory as possible without them. Always, and only, comment where it is necessary:
    - There are implicit data contracts, preconditions, or side effects.
    - The function and parameter names do not convey the meaning on their own. (For example, which unit do quantities carry?)
    - The data layout has a specific contract. (For example, what each dimension of a multidimensional array means, what each offset level of a flattened one-dimensional array means, or what each bit in a network or hardware control protocol means.)
    - The business logic is not obvious from the code itself (for example, an uncommon formula or a bit-twiddling trick is used).
- Avoid long comments that explain implementation details. Such comments go stale as soon as the implementation changes.
- A README should help users understand the project quickly, install it, and get started; it should not become a collection of every internal detail.
- Separate user-facing usage instructions from maintainer-facing design notes.
- Where possible, generate detailed documentation from code and comments with automated tools instead of maintaining large amounts of interface documentation by hand.
- Keep documentation concise, accurate, and up to date. Stale content can be deleted outright; no historical migration record is needed for every internal refactor. Avoid maintaining process documents, checklists, and auxiliary tools that serve no real purpose.

### Testing and Verification

- Tests should verify actual behavior, invariants, interface contracts, and critical boundary and error paths; avoid over-testing internal implementation details.
- Prefer tests that cover real usage over large numbers of low-value boilerplate tests written for coverage.
- When fixing a defect, add a regression test that reliably reproduces the problem wherever possible.
- Test code should follow the same principles of simplicity and maintainability. Avoid building test frameworks whose complexity exceeds that of the application or library itself.

## 3. How to Refactor

### Refactoring Principles

- Refactoring should have a clear goal, such as reducing complexity, eliminating duplication, clarifying responsibilities, improving interfaces, improving performance, or fixing a real defect.
- Before changing anything, understand the existing code's call relationships, data contracts, and actual uses, so that a local change does not create duplicated logic or confused responsibilities in other modules.
- Prefer subtraction. Before introducing a new abstraction, check whether deleting, merging, inlining, moving, or unifying existing implementations solves the problem.
- Refactoring should not expand the public interface, dependency scope, or user-visible configuration surface without reason.
- While preserving the required functionality and behavior, change the internal structure boldly; do not patch locally and leave an unreasonable design in place.
- Do not sacrifice performance, readability, or testability to shorten the code.
- A refactor may rewrite or delete tests that depend on the old internal structure in the same change; do not keep an unreasonable production interface just so old tests keep passing.

### Removing Redundancy

- Remove duplicate checks for boundary cases that do not exist.
- Remove extension points, configuration parameters, and generalization mechanisms built for requirements that do not yet exist.
- Remove functions, classes, modules, caches, and helper scripts that have no actual consumers or have lost their purpose.
- Remove old implementations that a new implementation replaced.
- Eliminate duplicate data structures, type conversions, wrapper functions, and multiple implementations of the same logic.
- Delete intermediate layers, aggregation layers, and pure forwarding interfaces with no clear purpose.
- Clean up special branches, duplicated defaults, and scattered compatibility logic left behind by historical evolution.
- Remove dead comments, outdated documentation, old implementations, and their now-invalid tests.

## 4. Engineering and Distribution

### Build and Dependencies

- Prefer the mature, standard build and dependency management mechanisms of the language ecosystem; do not maintain by hand what a mature tool already does.
- Dependencies should stay explicit, traceable, and version (range) determined. When introducing a dependency, weigh its build cost, transitive dependencies, and distribution cost.
- Public interfaces should expose as little third-party implementation detail as possible, so an internal dependency does not accidentally become one users must carry.

### Cross-Platform Applications

- Unify build logic across platforms as far as possible. Introduce platform-specific handling only where real platform differences exist, and localize it.
- Do not require users to install a pile of tools or libraries before they can build the project.
- Do not significantly increase the build and installation burden on downstream users for the convenience of internal development.

### Automated Integration and Deployment

- Clearly separate the responsibilities of local development, automated testing, and formal release. Do not hand every verification to CI.
- State which platforms, toolchains, and feature combinations CI/CD must cover.
- Prefer existing build, test, packaging, and platform tools; do not reimplement what they already provide.
- Manage dedicated scripts and configuration centrally; avoid scattering several overlapping automation directories around the repository.
- Add build configuration, release artifacts, or automation stages only when the project genuinely needs them. Avoid building elaborate distribution infrastructure ahead of time.
- For projects that involve building, installing, and distributing, verify that real external consumers can use the installed artifact.
