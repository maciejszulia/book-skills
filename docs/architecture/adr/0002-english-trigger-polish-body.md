# ADR 0002: English trigger, Polish body in Controlled Polish

## Status

Accepted (commit `e2a5aa9`, 2026-11-16).

## Context

The agent decides from the `description` field whether a skill fits a task. The steps of a skill must have one reading, because the agent acts on them.

## Decision

The `description` in the front matter is English. The body is Polish, written in Controlled Polish: one meaning per sentence, digits for counts, a glossary with forbidden synonyms.

## Consequences

- English trigger words ("concurrency", "deadlock", "profile") match English task descriptions.
- A reader without Polish can see when a skill triggers, but cannot read its steps.
- Some descriptions list Polish trigger phrases as well (`math-thinking`: "policz to matematycznie").

## Evidence

- `README.md`: "an English description that triggers the skill, and a body in Polish written in Controlled Polish".
- Front matter and "Słownik" sections of each `*/SKILL.md`.
