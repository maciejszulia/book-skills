# ADR 0001: One `SKILL.md` per skill, no code

## Status

Accepted (commit `e2a5aa9`, 2026-11-16).

## Context

Each book gives a set of practices. The agent needs the practices as steps and a checklist, not as a program.

## Decision

Each skill is one directory with exactly one `SKILL.md`. The repository holds no scripts, no tests and no build files. The five C++ skills share one section order: goal, when to use, glossary, steps for writing code, review checklist, pitfalls, sources.

## Consequences

- A skill installs and updates as plain text.
- Nothing in the repository runs, so nothing can be tested automatically.
- A reader of one C++ skill finds the checklist at the same place in the other four.

## Evidence

- `git ls-files`: `LICENSE`, `README.md` and six `*/SKILL.md` files.
- `README.md`: "Each skill is a single `SKILL.md`".
- `## ` headings in `cpp-*/SKILL.md` and `efficient-programs/SKILL.md`.
