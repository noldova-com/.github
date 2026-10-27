# Testing

**Scope:** Test authoring, execution, isolation, results, coverage, verification scope and evidence, on every stack.
A flaky test is a bug: file it, fix its cause, and never rerun a check to make it pass.

## The bar

- **100% unit coverage:** all executable product code is covered, as [Coverage](#coverage) says.
- **100% UI coverage:** every user workflow and every view has a UI test of the real application ([UI automation](#ui-automation)).
- **Every target:** the whole suite, unit and UI, passes with 100% coverage on every delivery target.
- **Fast:** a unit or component test file finishes in under 1 second, and the gate (build, checks and unit tests) in under 5 minutes, both measured on a test machine.
  The runner fails a file or a gate that goes over; a slow test is fixed, never given a longer budget.
  Only a maintainer may accept a file over 1 second; the repository's README then names the file, its time and why.
  No test waits longer than the event it needs.
- **Parallel:** any two tests can run at the same time.
  Tests share no ports, files, folders, processes or global state, and nothing runs serially; a test that needs global state gets its own process.
  There are two exceptions.
  An operating-system resource that a platform can't isolate per process, such as the system clipboard or foreground focus on Windows and macOS: tests that need it run last, one at a time, and the repository's README lists each by name in its Checks section.
  A test of a deadline no event can reach, in a public test helper, may fake a timer the spec files share, as [Execution and isolation](#execution-and-isolation) allows.

## What proves what

- A build proves that selected sources produced the expected outputs.
  Tests prove the behavior they exercise.
  Coverage measures execution of the declared inventory.
- Coverage and byte identity prove neither correctness, external-tool compatibility nor native installation.
- Tests exercise observable behavior, branches, boundaries and failures through the owning public API.
- A test written for a fix fails without the fix.
- Regression tests record required behavior.
- A test is named for the behavior it proves.

## Tests and production code

- **No test-only seams.** Never change production code solely to accommodate tests or coverage: no widened APIs or new exports, weakened visibility, test-only parameters or paths, or altered ownership.
  A test uses an accepted product boundary, or is redesigned around one.
  The one exception: a test runner may take how it creates its workers and its clock, with its timers, as constructor parameters, defaulting to the real ones, so its own tests can force an order of worker events or a time without sleeps.
- Test inputs, expected results and fixtures stay independent of production values, so an output change cannot also change the expected answer.
- Every executable production file has a test file that mirrors its path.
  A file is executable when it has a function body; a file that only declares types, values or re-exports is not, though it may still have a test.
  Every test file mirrors a production file that exists; fixtures, setup and configuration files and end-to-end workflows are exempt.
- Doubles of external tools are named fakes with scripted behavior.
- Tests against a live external service are labelled, excluded from the default gate and separately authorized; they record the service, tool version, observed settings and scope, and missing observations stay unknown.

## Determinism

No test is flaky by nature: a test that sometimes fails is a bug in the test or in the code.

- **Wait on events or conditions, never a fixed sleep.**
  A limit only bounds a hang: it is generous and is never what the test checks.
- **A polling wait is bounded** and fails with a message that says what it waited for.
- **Fixed pauses are listed.** Test code pauses for a fixed time only in the files the repository's README lists in its Checks section, each with its reason; review refuses one anywhere else.
- **No retries in test code** and no tight deadline as a pass condition.
- **A flaky test is never skipped or quarantined as the fix.**
- **A file that signals a test is written whole:** to a temporary file first, then renamed into place, so a reader never sees half of it.
  An append-only log that a test polls for a whole line, end of line included, is exempt: a partial read only retries.
- **A race fix** ([CODE.md](CODE.md#asynchronous-work-and-processes)) comes with a test that forces the bad order and fails on the old code.

## Discovery and selection

- Discovery uses explicit identities, compiled locations, markers and naming rules, never incidental working directories or locations outside the selected scope.
- Test identities name the package, file, class or suite, test and data row.
  Reject duplicates and omissions.
- Discovery is deterministic across locales and filesystem order; malformed declarations, invalid rows and unexpectedly empty test files fail it.
- A filtered run reports its selection and its discovered, selected and unselected counts.
  A filter that matches nothing fails; a filtered pass is not the complete gate.
- Only packages without executable production code may have an empty inventory; failed discovery or missing builds are errors.

## Execution and isolation

- Each test gets fresh state; tests establish and release state independently of their order.
- Observe every method and returned promise to completion.
  Failed assertions, throws, rejections and unexpected errors fail the test, or the run when attribution is impossible.
  Observe pending and late failures before completion.
- The execution boundary owns deadlines, cancellation and cleanup.
  A test that exceeds its limit fails, and the run ends after reporting it, so no uncontained work continues; rejecting a wait alone stops nothing.
- Forced interruption needs a separately terminable process or worker, with descendants and resources accounted for.
  Verify that mechanism before claiming it works.
- Crashes, forced termination, cancellation and incomplete cleanup remain explicit unsuccessful or incomplete outcomes.
- Tests own their fixtures: disposable files, repositories, databases and profiles.
  The runner neither bootstraps the application, signs in to services nor installs global state.
- Cleanup runs on success, failure and cancellation, touches only fixture resources and preserves diagnostics.
- Never mutate real data, profiles, projects, the clipboard or the desktop as incidental fixture setup.
- Scoped mock clocks may exercise long deadlines without changing production defaults.
  Wait for the work to start, check before and at the deadline, keep real process and socket cleanup checks, use a real-time guard, and restore timers on success and failure.
- Global mocks need process isolation, never serial execution ([The bar](#the-bar)); advancing a clock proves neither wall time nor native termination.
  A deadline no event can reach, in a public test helper with no injected time source, may fake a timer the spec files share, such as in one browser page, when a maintainer decides it; the spec restores the timers itself, and the repository's README lists each such spec by name in its Checks section.
- A passing assertion alongside an unhandled framework error is not a pass.

## Results and reporting

- Assertions compare values by a documented operation; truthiness or formatted strings never replace the required comparison.
  Failures keep meaningful expected and actual values and their cause, subject to redaction.
- Results record stable identities, outcomes, durations and failure or skip details for every test and data row.
- Distinguish executed, skipped, unselected and unreached tests, and reconcile the totals with discovery and selection.
  A run fails if its counts do not reconcile, a result is unreadable or missing, or an identity repeats.
- A test counts by its first result: one that passed only when run again counts as failed and is recorded as flaky.
- Every report derives from structured results; machine consumers never parse console text.
  Every detail level keeps failures, skip reasons, incomplete coverage and totals.
- The gate fails on any setup, discovery, execution, cleanup, coverage or reporting failure, or any missing or interrupted required result.
- Skips stay visible with declared reasons and never waive required behavior or coverage.
  A skip is declared in the source beside its test, never in a separate list or the runner's configuration, and a skip without a reason fails the run.
  Never convert failures into skips or use hidden exclusions.

## Coverage

- Executable product code has 100% coverage, with behavior and process-boundary checks appropriate to each operation.
  Command-line verification includes arguments, failure paths and process exit.
- Repository automation, such as the build, test, pack and release scripts and the checks, has no coverage percentage and no mirrored test files.
  The build and the test run prove it; it gets a test of its own only for an outcome no run shows, such as what a release publishes.
- Non-executable configuration, such as YAML, gets schema and configuration validation, not a coverage percentage.
  Non-executable definitions need no artificial tests.
- **Exclusions** are explicit, justified and reported, granted only by the repository's README in its Checks section, and declared with the file and the reason.
  The check still measures an excluded file, reports it with its reason, and fails for an exclusion that names no existing file.
  Behavior that only a real external tool can exercise needs separately authorized live verification and an explicit account of the uncovered lines.
- The rethrow in a `catch` that handles one kind of exception ([CODE.md](CODE.md#failures)) is reached with a deliberately failing test double, the only broken double allowed, and that test proves only that the error propagates.
- Unmeasurable bootstrap work is incomplete verification, not an exclusion: record the files, the missing capability and the checks actually run, and close the gap before claiming a complete pass.
- Measure the complete executable inventory, counting unloaded files as uncovered.
  Missing coverage, invalid ranges, malformed reports or unusable source maps fail the analysis; they never reduce its denominator.
- Source changes need fresh artifacts and matching maps; source and package drift invalidates results.
- Report covered and executable file counts and percentages against the tested source identity.
- Inline CI scripts stay small and carry no percentage; the workflow's own runs prove them.

## Verification scope

- During development use targeted checks; before review, run the checks for what the change touches.
- Check a configuration change, such as a workflow, with the tool that reads it.
- CI is the gate: green required checks at the pushed head are the evidence.
  Repeat successful checks locally only after a change, a failure or an unresolved concern.
- Every CI run builds and tests from source and reuses no earlier run's result.
- When CI narrows a run by changed paths, it compares from the merge base, checks both sides of renames, and selects everything when a manifest, lockfile or tool configuration changes or the history or packages cannot be read.
  A narrowed green run proves only what it selected; runs on the main branch are never narrowed.
- Build, pack and install the selected source before testing a package's API; dependencies resolve to those fresh artifacts.
- A full local run is for debugging a failure and for native checks CI cannot cover.

Name the evidence a claim needs:

- Fixtures verify an adapter's inputs, outputs and lifecycle; they do not prove that the real external tool applies supplied instructions or supports a control.
- Protocol and persistence changes need the [data contract](CODE.md#data-contracts) checks plus migration, interruption and recovery checks through their owning boundaries.
- Component tests verify component behavior; claims about the native window, OS integration or real desktop interaction need native evidence.
- Packaging or cross-compilation does not establish execution on a target; installer and update claims need native checks, including existing data and install scope where relevant.
- Ordinary checks use fixtures without sign-in to an external service or paid use.
- A test that compares with an outside tool records that tool and its version, never takes whatever version the machine has as the authority, and pins the answers that depend on the version.

**Performance claims** need representative workloads, recorded conditions and repeated measurements outside coverage.
Check scaling separately; ordinary unit tests assert no machine-dependent speed thresholds, apart from the runner's budgets in [The bar](#the-bar).
Speed alone proves neither allocation nor retention improvements.
An optimized path keeps its regression cases and is tested against a simple reference over systematically varied inputs.
A UI test that checks a performance budget gates only what does not depend on the machine, such as how many rows render or how far content in view moves, and attaches measured timings as evidence, never as a pass condition.

## UI automation

- Every user workflow is an end-to-end test of the real application, run natively on every delivery target.
  It complements component tests; a page with a mocked platform bridge is not enough.
- Use one shared suite with explicit platform-specific launch, path, keyboard and display behavior.
  Record OS, CPU (on GitHub, its thread count only), runner image, application revision and test-tool versions.
  Cross-compilation, emulation and another architecture's pass do not certify a native target.
- Pin test dependencies; verify the exact tool combination on each target before relying on it, and again when the tools or runner images change.
- Keep the application's production security settings intact; never weaken a production binary to let automation attach.
  A test-specific build or launch configuration records its differences, and the distributed binary keeps its own acceptance checks.
- Native OS dialogs, clipboard, installers and updates need their own checks; a fixture substituted for one of them does not verify the OS behavior.
- Assert meaningful application results, not merely that a click succeeded.
  Assert the sizes, spacing, colors and contrast the UI standards specify from computed styles in the running application, on every target.
- Locate elements by stable accessible roles and names, within the view that shows them, so an element of a view being replaced cannot match first.
- Wait for observable state, and collect unexpected failures of every process as part of the result.
- Verify normal closure, interruption and failure cleanup, child processes included.
- Capture established state under the declared animation policy; test animations explicitly where relevant.
- Verify the actual viewport and image dimensions, so host display defaults cannot silently reduce the evidence's resolution.

**Screenshots** are review evidence only: no pixel comparison and no baselines; behavior, computed-style, contrast and accessibility checks prove the UI.
Each workflow that opens the application takes at least one named checkpoint at its main state, each name used once, recording the theme, mode, fonts, zoom, viewport, scale and platform.
On failure keep a trace and capture any usable window.
Report a capture or cleanup failure beside the original failure, never in its place; a failed cleanup step does not stop the later ones.

## Testing the test infrastructure

- Before relying on a runner or a changed reporting path, run controlled negative fixtures through its real entry point: assertion failures, throws, rejections, timeouts, late errors, malformed discovery, declared skips and incomplete results.
  The deliberately failing child never appears as a successful product run.
- Coverage checks demonstrate that an unloaded file, a genuine uncovered branch and malformed coverage data cannot produce a complete result.
- Aggregation checks demonstrate that a failing prerequisite, a cancelled process or a missing report cannot produce a green summary.
- One-shot entry points are verified through the real process boundary: assert both the exit result and that the process actually terminates within its deadline.
  A printed error is not evidence that a process stopped.

## Evidence

- Record scope, revision, toolchain, platform, commands, outcomes and limits in the issue or PR, linking CI and selected redacted artifacts.
- Evidence creates neither a second progress document nor a change to a product contract.
- Keep each attempt's output separately; keep redacted reproduction steps and the relevant failure details in the issue or PR after artifacts expire.
- Commit only reviewed evidence with lasting regression or reference value, such as fixtures; keep routine logs out of source control.
- Attach the screenshots that show a fix or a finding to its issue or PR, never committed.
  Each is checked before posting: nothing private on screen, such as a user name in a path or a test machine's name, and only our own product.
  Screenshots of the reference products stay off GitHub.
- Evidence follows the redaction rules in [SECURITY.md](SECURITY.md#diagnostics-and-redaction).
- If a required check could not run, state the missing claim plainly.
  Old evidence, a copied package, a user report or a fixture-only run is never presented as newly verified implementation.
