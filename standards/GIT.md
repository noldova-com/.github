# Git

**Scope:** Working-tree safety, commits and what goes into source control, in every repository.
Every change starts from an issue, goes on its own branch and reaches `main` through a reviewed pull request that names that issue.

## Working tree

- Verify the repository, remote, working tree and index before editing.
- Preserve unrelated work: never reset, discard or overwrite it to simplify a task.
- Never silently include, or depend on, changes outside the commit.
  Resolve such dependencies within the authorized scope, or verify the commit in isolation.

## Commits

- A commit message describes the concrete change.
- Commit under a no-reply identity, never a personal address.
- A commit carries no attribution beyond its author and the signature lines its repository asks for, such as co-author trailers.

## Line endings

- Use LF line endings.
- Every text file ends with exactly one newline and no blank line after it.
- Every repository's `.gitattributes` enforces them with this line, and a check fails without it:

```text
* text=auto eol=lf
```

## What goes into source control

- Screenshots never go in; they are attached to the issue or PR instead ([TESTING.md](TESTING.md#evidence)).
- Investigation output, such as probes, measurements, logs and plans, stays in a temporary folder outside the clone, never in a build output folder.
- Source control excludes secrets ([SECURITY.md](SECURITY.md#secrets)), routine logs ([TESTING.md](TESTING.md#evidence)), hand-edited generated output ([CODE.md](CODE.md#files-and-classes)) and unchecked third-party material ([CODE.md](CODE.md#licenses)).
