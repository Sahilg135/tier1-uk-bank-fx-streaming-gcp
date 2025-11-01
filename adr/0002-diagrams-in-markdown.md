# ADR 0002 — Diagrams as Mermaid blocks inside Markdown

- Status: Accepted
- Date: 2025-11-01

## Context
GitHub renders Mermaid in Markdown, but certain tokens break parsing.

## Decision
Keep diagrams inside `.md` using ```mermaid fences. Avoid HTML `<br/>` and slashes in labels (use `PubSub`, `Topic: ...`). Prefer simple node text.

## Consequences
+ First-class preview on GitHub, easy reviews
− No advanced Mermaid HTML; keep labels concise
