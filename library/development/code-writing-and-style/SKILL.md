---
name: code-writing-and-style
description: Karpathy coding principles — think before coding, simplicity, surgical changes, goal-driven execution.
version: "1.4.0"
author: "catyugu"
license: "MIT"
---

## When to Use

Load this skill when:
- Writing code in any language.
- Reviewing your own or others' code.
- You need the mindset principles that apply regardless of language.
- You are about to add feature, do refactoring, fix problems... for an existing project

---

## Karpathy Coding Principles

### 1. Think Before Coding

- Don't assume. Don't hide confusion. Surface tradeoffs.
- If something is unclear, or if multiple interpretations of the requirements exist, ASK.
- If a simpler approach exists, do so.

### 2. Simplicity First

- Minimum code that solves the problem. Nothing speculative.
- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.
- Ask: "Is this overcomplicated?" If yes, simplify.

### 3. Surgical Changes

- Touch only what you must. Clean up only your own mess.
- Don't "improve" adjacent code, comments, or formatting.
- When your changes create orphans: remove imports/variables/functions that your
  changes made unused.
- The test: every changed line should trace directly to the user's request.

### 4. Goal-Driven Execution

- Define success criteria. Loop until verified.
- Transform tasks into verifiable goals:
  - "Add validation" → "Write tests for invalid inputs, then make them pass"
  - "Fix the bug" → "Write or find a test that reproduces it, then make it pass"
  - "Refactor X" → "Ensure tests pass before and after"
- For multi-step tasks, state a brief plan with verification points.

### 5. Abstraction Discipline

- Prefer the simplest implementation that makes the business logic explicit.
- Do not introduce abstractions for hypothetical future requirements.
- Do not introduce an interface unless there are multiple meaningful
  implementations or a real dependency boundary.
- Do not introduce factories, registries, managers, providers, or plugin
  systems without a concrete present need.
- Prefer plain functions and value types over class hierarchies.
- Prefer data tables over polymorphism when differences are primarily
  configuration.
- Avoid pass-through layers.
- After implementation, perform a simplification pass and remove unnecessary
  abstractions.
