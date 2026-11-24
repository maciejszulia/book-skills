# ADR 0004: Install by symbolic link

## Status

Accepted (commit `e2a5aa9`, 2026-11-16).

## Context

The agent reads skills from its skills directory. A copy of a skill goes stale when the repository changes.

## Decision

The install links each skill directory of the clone into `~/.claude/skills/` with `ln -s`. There is no installer and no package.

## Consequences

- `git pull` in the clone updates every installed skill.
- Moving or deleting the clone breaks every link.
- The agent picks up a new or changed skill only at the start of its next session.

## Evidence

- `README.md`, section "Install".
