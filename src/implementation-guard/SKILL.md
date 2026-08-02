---
name: implementation-guard
description: >
  Reviews code implementation quality: correctness, maintainability,
  error handling, performance, and structural design.
  Use this skill after writing or modifying code, and before
  considering an implementation task done, to check it against
  sound development practices.
---

# Goal

Enforce clean, correct, and maintainable code.

Prefer consistency with the 
existing repository coding practices and design patterns
unless it is clearly harmful.

You may use any available tools, linters, static analyzers,
type checkers, security scanners, test suites, and repository context
to improve development accuracy.

## Correctness

Pay special attention to

* logic that doesn't match the stated intent of the change
* unhandled edge cases (empty input, null/undefined, boundary values)
* off-by-one errors and incorrect conditionals

## Error handling

Pay special attention to

* failure paths that are silently swallowed or ignored
* errors surfaced too generically to act on (e.g. losing the original cause)
* missing handling around operations that can fail (I/O, network, parsing)

Do not ask for error handling around scenarios that cannot actually occur;
validation belongs at system boundaries, not everywhere internally.

## Performance

Pay special attention to

* unnecessary work inside loops (repeated I/O, allocations, re-computation)
* algorithmic complexity that won't scale with realistic input sizes
* obvious redundant or duplicate operations

## Code inspection

Pay special attention to

* poor separation of concerns
* tightly coupled components
* excessive cyclomatic complexity

## Architecture

Pay special attention to the use of inheritance. 
Prefer composition over inheritance
unless inheritance provides a simpler, more maintainable model.