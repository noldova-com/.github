# Documents 

**Scope:** Which document owns a rule, what documents hold, and the form of Markdown in every repository.
Code comments and API documentation are owned by [CODE.md](CODE.md#comments-and-api-documentation).

## One owner per rule

- Each rule has one owning document; other documents link to it rather than copy it.
- Coding, testing, UI and release rules live only in these standards; a repository never copies or restates them, and keeps only what is its own.
- A repository's `AGENTS.md` is short: its agent instructions and a "When you… | Read" table that names the owning document for each kind of work.
  It may summarize critical rules, naming their owners.
- A repository's requirements live in `documents/`, one small document per subject, never `docs/`.
  Each opens with its title and a `**Scope:**` line that says what it owns.
  A document that grows a second subject is split, so each stays readable on its own.
- When a requirement changes, update its owning document.
  Remove contradictions; superseded rationale stays in Git history.
- Record reusable decisions in their owning documents, and apply accepted corrections to similar work in scope.
- Copied proposals and historical references do not become requirements.

A typical repository's owners:

| Subject | Owner |
|---|---|
| Product introduction and user-facing explanations | `README.md` |
| Agent instructions and when to read each owner | `AGENTS.md` |
| The product's purpose and direction | `documents/VISION.md` |
| Components and their boundaries, in outline | `documents/ARCHITECTURE.md` |
| One subject, such as packages, build, storage, a protocol or a feature | its own `documents/<SUBJECT>.md` |
| Its checks: what they run, coverage exclusions, fixed pauses and tests that run alone | `README.md`, its Checks section |
| Code rules only this repository has, such as its file names or exports | `documents/ARCHITECTURE.md` |
| A specification of what it ships, such as a component kit's themes and controls | its own `documents/<SUBJECT>.md` |
| Contribution, issue and PR requirements | the organization's `.github` repository, `CONTRIBUTING.md`, unless the repository needs its own |
| Private vulnerability reporting and disclosure | the organization's `.github` repository, `SECURITY.md`, unless the repository needs its own |
| License terms | `LICENSE` |

## What documents hold

- Documents define current requirements, without status labels or lifecycle sections.
- Keep Proposed, Implemented and Verified apart.
  An accepted rule is not proof that it is built, and a built feature is not proof that it works.
  A verified claim names reproducible evidence and its scope; a passing build, correct behavior and coverage are different claims.
- Keep useful invariants and compact diagrams, not duplicate API inventories or speculative designs.
- State an unresolved boundary briefly, and resolve it before implementation that depends on it.
- Issues and PRs hold scope, acceptance criteria, progress, findings and evidence.
  Contracts change with approved requirements, not with work sessions; they hold no run histories or verification totals, and no dates other than decision dates.
- Verified commands and prerequisites belong in the README or the owning tooling guide; never invent commands for missing tooling.
- External references may establish behavior; their prose is not copied.

## Markdown form

- Put each sentence on its own line, so changes to different sentences of one paragraph do not conflict.
  A list item's or block quote's later sentences continue at the item's indentation or behind the quote's markers.
  Headings, table rows, code blocks, comments, HTML and link definitions keep their lines.
  Abbreviations such as e.g. and numbers such as 1.5 do not end a sentence.

## Checks

Documentation changes need a review of their links, form, content and consistency with the owning documents.
