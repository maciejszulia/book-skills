# ADR 0003: Paraphrase, not copies

## Status

Accepted (commit `e2a5aa9`, 2026-11-16).

## Context

The books are copyrighted. The skills need their ideas, not their text.

## Decision

The skills paraphrase ideas and do not reproduce text from the books. Each skill lists its public sources (publisher page or public table of contents, authors' public example code, C++ Core Guidelines) with a URL and a check date. Rule IDs of the C++ Core Guidelines come from the 2026-06-14 version and carry a short Polish paraphrase instead of the quoted title.

## Consequences

- The text of the skills can be published under the MIT License.
- The skills depend on the model's general knowledge of each book; the README asks readers to buy the books for the full argument.

## Evidence

- `README.md`, section "Paraphrase, not copies".
- "Źródła" sections of each `*/SKILL.md`; `cpp-core-guidelines/SKILL.md` line 103.
