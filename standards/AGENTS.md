# Engineering standards

Noldova's engineering standards for code, tests, interfaces, documents, Git and security.
Humans and AI agents follow them on every project and every stack.
"The project's standards" means these documents plus the repository's own rules.

How the team works, such as who decides, how issues and pull requests move, and what may go public, is the team's own process; these standards hold the rules for the work itself.

## How they are organized

- **General documents** hold for any project and any stack.
- **Stack documents** add what one technology needs: its language rules, framework layout and tooling facts.
  A stack document never relaxes a general rule; it says how the rule applies there.
- **Project rules** live in each repository: its `AGENTS.md`, architecture, and its own coding, testing and UI documents.
  They name the product's components, layout, tokens, tooling and CI.

## The base framework

[CONTEXT](https://github.com/noldova-com/context) is Noldova's base framework, and everything we build is based on it.

- Code uses CONTEXT the way a C# project uses .NET: wherever CONTEXT has the type or helper, such as its collections, text, exceptions and checks, it comes before the host's form, as [CODE.md](CODE.md#ownership) orders it.
- A general capability a project needs, one that isn't the product's own behavior, comes from CONTEXT; when CONTEXT lacks it, it is added to CONTEXT as a package, as [CODE.md](CODE.md#ownership) places it.
- An outside package comes in only as [SECURITY.md](SECURITY.md#dependencies) says.
- A project built on another framework, such as Electron or Angular, keeps it unless a maintainer decides to move it, and uses CONTEXT for what CONTEXT provides.
- CONTEXT is our reference code: it meets these standards, and how it is written and tested, how fast it runs and how fast its tests run are the example every project follows.
  These documents hold the rules; [TESTING.md](TESTING.md#the-bar) sets the speed every suite meets.

## Precedence

1. The general documents.
2. The stack documents for the technologies a project uses.
3. The repository's project rules.

A higher layer wins a conflict.
A lower layer may add rules and make them stricter, never weaker.
Where a standard lets a project grant a named exception, such as a coverage exclusion or a test file allowed a fixed pause, the project lists it in its owning document with its reason, through a reviewed change.
Never use an allowlist that silently weakens a rule.

When a project rule seems to conflict with a standard, the standard holds until the conflict is decided and the owning document changes.

## Enforcement

Review, the compiler, and the build and test runs enforce the standards.
A repository adds a mechanical guard only for a rule review can't reliably catch, and keeps it small.
A guard enforces the rule's intent, not a filename spelling.
A failed guard is fixed at its cause, never by weakening the guard ([CODE.md](CODE.md#failures)).

## Index

Read the documents your work needs:

| When you… | Read |
|---|---|
| design, write or review code: ownership, failures, naming, comments, automation, licenses | [CODE.md](CODE.md) |
| write or review tests: coverage, waits, UI automation, evidence | [TESTING.md](TESTING.md) |
| build or review UI: visuals, interaction, accessibility, shared components | [UI.md](UI.md) |
| write or change a document, or decide which one owns a rule | [DOCUMENTS.md](DOCUMENTS.md) |
| commit, squash or decide what goes into source control | [GIT.md](GIT.md) |
| handle secrets, outside input, dependencies, CI permissions or redaction | [SECURITY.md](SECURITY.md) |
| write TypeScript or Node: types, files, names, packages, pins | [stacks/TYPESCRIPT.md](stacks/TYPESCRIPT.md) |
| write Angular parts, components or specs | [stacks/ANGULAR.md](stacks/ANGULAR.md) |
| write Electron code or desktop tests | [stacks/ELECTRON.md](stacks/ELECTRON.md) |
