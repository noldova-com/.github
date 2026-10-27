# Code

**Scope:** Source code, tests, scripts and generated-code ownership, on every stack.
Testing is owned by [TESTING.md](TESTING.md), security by [SECURITY.md](SECURITY.md); stack details by the [stack documents](AGENTS.md#index).

Never overcode: use the simplest implementation that preserves correctness, ownership and clarity.
Add code, files or abstractions only for a present requirement.
Code reads like the code around it.

## Ownership

Every behavior has one named owner responsible for its state and invariants.
Apply SOLID through cohesive responsibilities, valid objects, explicit dependencies, small APIs and genuine polymorphism.
Apply DRY to duplicated knowledge and behavior, not merely similar code; it does not justify premature base classes or ownerless abstractions.

Place behavior with its owner:

1. The host (language, framework or platform) provides it: use that form.
2. One package needs it: use a method of its owning concept.
3. Several packages need it: place it in the lowest package that owns the concept, preserving dependency direction.

When work shows that behavior belongs to another owner, move it there before going on; a local copy "for now" is how buckets and duplicate types begin.

- **No buckets.** No `helper`, `helpers`, `util`, `utils`, `utility`, `common`, `shared` or `misc` folders, files, types or exports, no `Helper`/`Utility` classes, static grab bags or unrelated free functions.
  Renaming a bucket does not establish ownership; incidental words in precise domain terms are allowed.
- An independent responsibility becomes a named concept, such as a reader, parser, tracker or policy.
- Free functions are allowed only where a language, framework or platform requires them; their substantive behavior stays class-owned.

## Object-oriented design

Production behavior belongs to classes representing named concepts.

- Constructors establish valid objects, validating before assignment.
- Prefer instance methods for owned state.
  Static methods or classes represent a cohesive concept that needs no instance state.
- Language modules and namespaces are not surrogate objects for unrelated functions.
  Prefer composition when delegating a distinct responsibility.
- Add behavior to the concrete type instead of `if` or `switch` ladders over kind tags.
  A dispatcher is permitted only at a genuine runtime-dynamic boundary (parsing an external tool's events, decoding wire messages), and is narrow, named and tested as such.
- Additions to existing or global types live only in foundation packages, each adding the members of its own subject, reviewed and tested there.
- Methods return concrete classes, primitives, enums or concrete runtime collections.
  Structured results use named classes; polymorphic results use meaningful base or abstract classes.
  Interfaces describe consumed or implemented contracts, not results.
  Do not return interfaces, shape aliases, anonymous objects, tuples or property bags, except at the wire boundary.
- Keep public APIs small and intentional; internal mechanics stay private.
- Prefer immutable data when mutation is not part of the concept.

## Files and classes

- Each implementation file declares one named concept and is named for it.
  A file that declares none, such as an entry point, keeps its own name.
  Related, subordinate and private concepts use separate files, located by domain and responsibility.
- Prefer small, cohesive classes and files; no god objects or generic context or state containers.
  Split a file only when a subordinate concept can be named independently.
  Cohesion outranks a line limit: file length is a review signal, not permission to extract ownerless behavior.
- Do not declare an empty constructor when the implicit one is the complete contract.
- Generated outputs use the verified generator's formatting and are never edited by hand.

## Lifetime and performance

Review speed and memory from the first line.

- Keep builders, buffers and worklists local.
  Results do not retain producer machinery; shared data keeps its contracted lifetime.
- Own retained objects, callbacks, subscriptions, timers, processes, files, sockets, databases and caches, including their release on normal and exceptional exits.
  Garbage collection does not replace cleanup, and one caller finishing does not end a shared resource's lifetime.
  Review references escaping through returns, fields, closures and package boundaries.
- Copy caller-owned data only when an independent snapshot is needed.
  Shallow copies share elements; views and stored callbacks need explicit retention, mutation and lifetime contracts.
- Read-only declarations, concrete types and a variable's last use prove neither exclusive ownership nor deep immutability.
  Never invalidate data on those assumptions.
- Bound collections that grow with history, such as transcripts, events and logs.
- Avoid hot-path allocation, repeated scans, reparsing and unnecessary copies.
  Pool or batch only for a demonstrated need.
- Stream potentially unlimited text, or enforce a documented size limit before retaining it.
  Arrays of chunks also consume memory; they do not make unbounded input safe.

## Canonical values

- Each package keeps its canonical values in one owner: messages, labels, protocol tokens, file extensions, environment and command names, regular expressions, formatting values and policy limits.
- Self-evident local algorithm literals, such as incrementing by `1`, stay inline.
  Timeouts and size limits always have names.
- Semantic alternatives use enums or concrete value objects; enums own their values.
- Exceptions take their text from the owner or a diagnostic template; never duplicate owned text.

## Methods and control flow

- A method performs one coherent operation at its owner's level of abstraction, with direct, readable control flow.
  Inputs, outputs, mutation and failure are visible in its API.
- Do not rely on ambient mutable state, hidden fallbacks or distant side effects.
  Avoid boolean flags that make one method perform unrelated operations.
- Never shorten names, add wrappers or extract code to satisfy a line width.

## Data contracts

- Each protocol message or model owns its wire form: its constructor validates invariants, it serializes to explicitly typed plain data, and it parses untrusted input with a validating reader that names invalid fields in a protocol error.
- Required fields and known values are validated; optional fields define their absence and default.
- Plain data is confined to serialization; domain operations use validated instances.
- Each table's owner provides one row mapping that produces the same domain classes.
- Declare supported versions and any capability negotiation.
  Reject incompatible versions and unsupported operations before execution.
  Ignore additional informational fields only where explicitly allowed.
- The protocol and each data format declare their own versions; none copies the application version.
  A version field exists only where a consumer checks it at a persistent or process boundary, and never twice in one value.
- Breaking changes need an explicit version and compatibility decision.
- Test that an unsupported version is refused, and test required fields, optional additions and incompatible changes.
  Data that outlives a build, such as persisted records, keeps compatibility tests with its older and newer forms; round trips alone do not prove compatibility.

## Failures

- Fail explicitly with a domain-specific exception class that names what went wrong.
- An exception is an internal failure; an expected failure the user sees is a modeled result, and neither is turned into the other to keep running.
- Each package has one base exception that every exception it owns extends, the boundary its callers catch; a narrower type is added only for a condition that occurs.
- An exception carries no numeric code: its type is its identity, callers catch by type, and the facts of its condition are its readonly fields.
- Never replace missing knowledge with zero, an empty value, a guessed match or a silent fallback.
  A requested setting is never reported as the observed one.
- Narrow untrusted values at their boundary; inside, trust the declared types and do not repeat their checks.
  Semantic invariants the type system cannot express (ranges, ordering, non-empty values, cross-field consistency) are validated once by their owner, expressed in the type where possible, and never replaced by assertions or silent fallbacks.
- Preserve the original cause when translating errors between layers; redact secrets before an error is stored or shown.
- A `catch` that handles one kind of exception rethrows every other kind ([TESTING.md](TESTING.md#coverage) says how that rethrow is covered).

**Repair the cause a failure reveals.**
Do not add a guard, retry, fallback, skipped step or widened type whose only purpose is to hide an unexplained defect.
Do not weaken an assertion, silently remove a failing case or narrow a verification scope to make a gate pass.
When an accepted requirement changes, update its owning contract and tests deliberately; a failed check alone does not establish that its expectation is wrong.

**Recovery** is valid only for a modeled condition: a classified transient network failure, a supported recovery path of an external tool, or a documented user cancellation.
State when it applies, preserve the failure evidence and verify the resulting behavior.
It obeys the retry rules below and never guesses that an uncertain side effect did not occur.
Do not invent impossible internal states to justify defensive code or coverage.
Code that no input can reach is removed, not covered: needing a deliberately broken test double to reach a line is the sign to delete it, apart from the rethrow above.

## Asynchronous work and processes

- Use the asynchronous API for file, network and process work wherever one exists; a blocking call is allowed only in a short-lived command-line process.
- Every asynchronous operation has an owner that observes its completion and errors.
  Background work has an explicit lifetime and failure path; no unobserved rejections or callback failures.
- Operations that can stall have documented deadlines and cancellation.
  Long-lived streams and services instead define their shutdown and, where appropriate, an inactivity policy; no arbitrary whole-session timeout.
- Use a timer, delay or polling wait only where the product's behaviour needs one, such as a deadline that stops stalled work or a display that updates over time; never a grace period for work that can be awaited.
- Cancellation and timeout stop the owned work, not just the caller's wait.
  Release timers, listeners, streams and child processes on success, failure and cancellation, and prevent late callbacks from changing disposed state.
- Retries are bounded by attempts and elapsed time, use an appropriate delay, stop on cancellation, and apply only to failures classified as transient.
  For operations with side effects, use idempotency or verify the existing outcome first; an uncertain response is not proof that nothing happened.
- Code never relies on the order of independent events, such as a process's exit against its last message.
  Make the order explicit, for example with a reply or an acknowledgement ([TESTING.md](TESTING.md#determinism)).
- Own child-process shutdown, including descendants where applicable.
  Observe process errors, exit status and output-stream completion.
  Terminating a parent is not proof that its descendants stopped.
- One-shot automation terminates promptly with a nonzero status after a fatal failure.
  Printing an error or setting an exit code is not enough if resources keep the process alive.
  Cleanup has a bounded deadline and its failures are surfaced; forced termination avoids interrupting protected writes and flushes necessary diagnostics where possible.

## Naming

- Name a type for its responsibility: `SourceReader`, `SnapshotTracker`, `DiagnosticCollector`.
  Use `Service` only when no more specific role describes it.
- A method is a verb phrase naming its action (`readString`, `formatChildPath`); a property is a noun; a boolean is a predicate (`isEmpty`, `hasValue`, `canRead`).
  `is` states a fact or a fixed classification, `can` and `may` a capability, `should` a decision, and the name never claims more than its check proves: a token check is not `isValidDeclaration`.
- A method or function that returns a promise ends in `Async` (`readFileAsync`); a name a framework or an interface fixes, such as `ngOnInit`, keeps its own name.
  This and the asynchronous-API rule hold for new and changed code; renaming existing methods is each repository's own cleanup, and an exported rename waits for a major version.
- "Component" names only UI building blocks; other composable units use another word.
- Use explicit names, not abbreviations, and do not adopt an external tool's abbreviations.
- Product-owned enum string values match their member names exactly (`InvalidParams = "InvalidParams"`).
  A spelling fixed outside the product, such as a command-line word or a database column, stays out of the enum; the owner of canonical values maps each member to it.
  Use explicit values; use numbers only where numeric meaning or representation is required (bit flags, binary formats, native interop); never mix strings and numbers or rely on implicit ordering.
- Before changing enum values, inspect comparisons, reverse lookups and serialization; update declarations, consumers and tests together.
  A casing change also changes serialized values.

## Formatting

- Keep formatting mechanical and consistent; automate it when tooling exists.
- Line endings and final newlines are owned by [GIT.md](GIT.md#line-endings).

## Comments and API documentation

- No comments unless the code stays confusing after a rename or restructure; prefer renaming.
  Rationale goes in the issue, the PR or the owning document.
- Doc comments appear only in the handwritten public API declarations below.
- Every public type, member and overload has original documentation sufficient for use without reading the implementation: each parameter, the result, the contractual failures, and examples of normal and boundary use.
  No invented contracts and no empty tags.
- Examples compile or run during verification.
- Reference pages and editor help derive from the declarations with compiler-aware tooling, never text extraction or a second copy; private details are excluded.
- Automated checks compare the declarations with the implementation; humans review meaning.

## Automation

- Scripts follow these standards.
  Standalone automation uses the project's language for builds, tests, packaging, generation, fixtures and developer tools; no other-language scripts or wrappers.
- Keep runnable entry points and bootstrap configuration directly in `scripts/`; group supporting implementations by responsibility.
- A repository holds no PowerShell, `cmd` or batch files: its scripts are in the project's language, and every shell step, such as a workflow's `run:`, is Bash.
  A project may grant a named exception for a test fixture that is itself the program under test, listing it with its reason in its own documents.
- Workflows never encode machine-specific paths.
- Every entry point works from a clean clone on every supported platform; no globally installed tool is an unstated prerequisite.
- Builds and tests leave tracked files unchanged and write only to their declared output folders.
- A build first removes its stale output and the own packages it installed before, and a failed step stops every step that depends on it; loose compiler output is not a finished build.
- CI workflow files may use small inline scripts, toolchain commands and existing entry points under the same security and failure rules ([TESTING.md](TESTING.md#coverage) owns how inline scripts are proven).

## Licenses

- Each repository's `LICENSE` file is the license authority.
- Every repository-owned source, test, script, style, template, workflow and generated file whose format supports comments begins with the license header in its comment form; generators emit it themselves.
  Formats without comments, such as JSON, are excluded.

```ts
/**
 * @license
 * Copyright (c) Noldova.
 *
 * This source code is licensed under the license found in the
 * LICENSE file in the root directory of this source tree.
 */
```

Markup, such as HTML templates, puts `<!--` and `-->` on lines of their own around the same text, without the ` * ` prefixes:

```html
<!--
@license
Copyright (c) Noldova.

This source code is licensed under the license found in the
LICENSE file in the root directory of this source tree.
-->
```

YAML uses `#` lines with the same text.

- Third-party material is never copied in without checking its license and recording any obligation; AGPL-licensed references are read, not copied.
- Before importing code from another repository, review its documentation; import only the requested files, check ownership, correctness and security, and explain material changes.
  Record the source revision in the report, never in public text when the source is private.
  Never copy whole trees or make a reference repository a source or build dependency.
