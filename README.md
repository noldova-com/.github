# .github

Noldova's shared GitHub files.
GitHub uses them for every repository of the organization that has no copy of its own.

| File | What it is |
|---|---|
| [profile/README.md](profile/README.md) | The organization's front page |
| [CONTRIBUTING.md](CONTRIBUTING.md) | How to report a bug or ask for a change |
| [SECURITY.md](SECURITY.md) | How to report a vulnerability privately |
| [.github/ISSUE_TEMPLATE/](.github/ISSUE_TEMPLATE) | The bug and feature forms |
| [.github/PULL_REQUEST_TEMPLATE.md](.github/PULL_REQUEST_TEMPLATE.md) | The pull request form |
| [standards/](standards/AGENTS.md) | The standards that every repository's code, tests, documents and commits follow |
| [.github/workflows/require-linked-issue.yml](.github/workflows/require-linked-issue.yml) | The shared check that every pull request names its issue, which also runs on this repository's own pull requests |
| [.github/workflows/release-npm.yml](.github/workflows/release-npm.yml) | The shared workflow that releases a repository's npm packages to the registry its caller names |
| [.github/workflows/flake-proof.yml](.github/workflows/flake-proof.yml) | The shared workflow that runs a repository's tests many times over, to prove a flaky test fixed or to hunt one |
| [.github/workflows/versions.yml](.github/workflows/versions.yml) | The Node and npm versions and the runners every shared workflow uses, in one place |

A repository runs the shared check from its own workflow:

```yaml
name: Require linked issue

on:
  pull_request:
    branches: [main]
    types: [opened, edited, synchronize, reopened, ready_for_review]

permissions:
  issues: read
  pull-requests: read

jobs:
  linked-issue:
    uses: noldova-com/.github/.github/workflows/require-linked-issue.yml@<full commit SHA>
```

A pull request's description names its issue on a line of its own that starts, at its first character, with `Issue` or a GitHub closing keyword (`Close`, `Closes`, `Closed`, `Fix`, `Fixes`, `Fixed`, `Resolve`, `Resolves` or `Resolved`), in any letter case, then the number, such as `Closes #123` or `Issue: #123`.
A colon may follow the word directly, and a full stop may follow the number directly, as GitHub accepts it; any other text on the line, or a blank before the word or the full stop, fails the check.
The number names an issue in this repository, not a pull request: GitHub also closes from a reference such as `owner/repo#123`, but the check refuses it.

Keep the calling job named `linked-issue`: GitHub puts the job's name before the check's, so the check reports as "linked-issue / Require linked issue", the name a required-checks rule must list.
The check runs as [Shared settings](#shared-settings) explains.

## Releasing npm packages

A repository releases its npm packages by hand from its own workflow, `.github/workflows/release-npm.yml`.
A public repository publishes to npm's registry by [trusted publishing](https://docs.npmjs.com/trusted-publishers), with no stored token:

```yaml
name: Release npm packages

on:
  workflow_dispatch:
    inputs:
      version:
        description: The version to release, which the commit holds
        required: true
        type: string
      commit:
        description: The full hash of the commit on main to build and release
        required: true
        type: string

permissions: {}

concurrency:
  group: ${{ github.workflow }}
  cancel-in-progress: false

jobs:
  release:
    uses: noldova-com/.github/.github/workflows/release-npm.yml@<full commit SHA>
    permissions:
      contents: write
      packages: write
      id-token: write
    with:
      version: ${{ inputs.version }}
      commit: ${{ inputs.commit }}
      environment: npm
```

- Where a release publishes comes from [Shared settings](#shared-settings): a public repository publishes `@noldova` packages to npm's registry with public access, and a private repository publishes `@noldova-com` packages to GitHub's registry with restricted access; a caller overrides them with `publish-registry-url`, `publish-scope` and `publish-access`.
- A private repository's caller names its own environment, and passes `keep-versions` when it keeps only the newest versions.
- The jobs build and check the archives, publish them in the caller's environment, keep the newest `keep-versions` versions when it is above 0, and tag the commit with the version.
- The build job installs the dependencies from the registry [Shared settings](#shared-settings) names for installs, as the flake proof does; the run's own token goes to GitHub's registry only.
- No package's install scripts run in any job, as [Install scripts](#install-scripts) says, and the publishing token is set only on the publish and prune steps.
- The repository provides the steps as `npm run release -- <step>`: `source <version> <commit>` checks that the commit is on main and holds the version; `check _packages <version> <commit>` checks the built archives; `publish _packages <version> <commit>` publishes them; `prune <version>` deletes the older versions; `tag <version> <commit>` tags the commit.
- Those steps read the registry, the scope, the access and the number to keep from `RELEASE_REGISTRY_URL`, `RELEASE_SCOPE`, `RELEASE_ACCESS` and `RELEASE_KEEP_VERSIONS`; the publish step's token is in `NODE_AUTH_TOKEN`, the prune step's in `RELEASE_REGISTRY_TOKEN`.
- Without a `registry-token` secret, GitHub's registry takes the run's own token, and npm's registry takes trusted publishing: the package's trusted publisher names the calling repository, its `release-npm.yml` and its environment, and npm records the run as the version's provenance.
- Another registry passes its token with `secrets: registry-token: ${{ secrets.<name> }}`, which may name a secret that only the caller's environment holds.
- A new package's first publish to npm's registry can't use trusted publishing, since the package doesn't exist yet: the caller passes a one-time token held only in its environment, the same way, and removes the secret after the first publish, so the line resolves to empty and trusted publishing takes over.
  A public repository's packages on npm's registry get provenance either way.
- The caller grants `contents: write`, `packages: write` and `id-token: write`, whichever registry it publishes to, since a called job gets no permission its caller didn't grant.
- The publish and prune jobs both run in the caller's environment, so an environment that needs approval is approved twice; an environment name that doesn't exist yet is created on first use, with no protection.
- The prune job runs only for GitHub's registry: a version published to npm's registry is never deleted.

### Native addons

A package that ships a native addon declares it in its `package.json`, anywhere under `src/` but outside a `tests` folder, and the release builds it for all six targets, so an app never rebuilds one:

```json
{
  "noldova": {
    "windowsAddons": ["windows-process"],
    "posixAddons": ["posix-process"]
  }
}
```

- Each name is kebab-case, and its source is the package's `src/native/<name>.c`, built with Node-API version 8.
- `windowsAddons` are built for `win32-x64` and `win32-arm64` on a Windows runner; `posixAddons` for `linux-x64` and `linux-arm64` on a Linux runner and for `darwin-x64` and `darwin-arm64` in one job on the macOS runner; the ARM64 targets are cross-compiled.
- Both Linux targets are built with clang against a Debian bullseye sysroot for their architecture, so they need only glibc 2.31.
- A plan job checks that the commit is on main, refuses a declaration that isn't a list of kebab-case names, naming the manifest and the key, and lists the systems that build addons; only the runners of the systems that build addons take part, so a repository without addons waits for no Windows or macOS runner; its `RELEASE_ADDONS` folder is empty.
- The build job hands the builds to `npm run build` in `RELEASE_ADDONS`, the absolute path of a folder in the runner's temporary folder, outside the checkout, that holds `<package path in the repository>/<platform>-<arch>/<name>.node`, such as `src/Example.Package/linux-arm64/posix-process.node`; the folder's root also holds a `built-on-<platform>.txt` from each runner.
- The repository's build puts each build in its package at `native/<platform>-<arch>/<name>.node`, and fails when a declared target's build is missing; its own code loads `native/${process.platform}-${process.arch}/<name>.node`.
- After the build, the release refuses an archive that lacks any of its addons' builds for any target, naming the package, the addon and the target.
- The addon jobs run on the x64 runners of Linux, macOS and Windows that [Shared settings](#shared-settings) names, and need the organization's own runners, which hold the sysroots: a private repository with `RUNNER_TYPE` set to `GITHUB` passes its own runners with `runs-on-<system>-x64`, and a public repository can't release a package with addons yet: the plan job refuses it.

## Proving flakes

A repository proves a flaky test fixed, or hunts one, by hand from its own workflow, `.github/workflows/flake-proof.yml`, which a maintainer starts with `workflow_dispatch` only, never a pull request, issue or comment trigger, since the jobs build and run the tested code:

```yaml
name: Flake proof

# The inputs' defaults, which the inputs below take by their anchors, since an input's default can't read env.
env:
  DEFAULT_RUNS: &default-runs '50'
  DEFAULT_PARALLEL_JOBS_PER_TARGET: &default-parallel-jobs-per-target '2'
  DEFAULT_TARGETS: &default-targets linux-x64 windows-x64

on:
  workflow_dispatch:
    inputs:
      ref:
        description: The branch, tag or commit to test; empty tests the branch the run starts on
        type: string
        default: ''
      filter:
        description: The filters the script takes, separated by spaces; empty runs the whole suite
        type: string
        default: ''
      runs:
        description: Runs in total per target, 1 to 200
        type: string
        default: *default-runs
      jobs:
        description: Parallel jobs per target, 1 to 4
        type: string
        default: *default-parallel-jobs-per-target
      targets:
        description: The targets to run on, separated by spaces, from linux-x64, linux-arm64, windows-x64, windows-arm64, macos-x64 and macos-arm64
        type: string
        default: *default-targets

permissions: {}

jobs:
  versions:
    name: Versions
    uses: noldova-com/.github/.github/workflows/versions.yml@<full commit SHA>
    permissions: {}

  group:
    name: Concurrency group
    needs: versions
    runs-on: ${{ fromJSON(needs.versions.outputs.runs-on-linux-x64) }}
    timeout-minutes: 2
    permissions: {}
    outputs:
      filter: ${{ steps.filter.outputs.hash }}
    steps:
      - name: Hash the filter
        id: filter
        shell: bash
        env:
          FILTER: ${{ inputs.filter }}
        run: |
          hash=$(printf '%s' "$FILTER" | sha256sum | cut -c1-16)
          if [[ ! "$hash" =~ ^[0-9a-f]{16}$ ]]; then
            echo "::error::The filter's hash is \"$hash\", not 16 hex digits."
            exit 1
          fi
          printf 'hash=%s\n' "$hash" >> "$GITHUB_OUTPUT"

  flake-proof:
    needs: group
    concurrency:
      group: ${{ github.workflow }}-${{ inputs.ref || github.ref_name }}-${{ needs.group.outputs.filter }}
      cancel-in-progress: false
    uses: noldova-com/.github/.github/workflows/flake-proof.yml@<full commit SHA>
    permissions:
      contents: read
      packages: read
    with:
      ref: ${{ inputs.ref }}
      filter: ${{ inputs.filter }}
      runs: ${{ inputs.runs }}
      jobs: ${{ inputs.jobs }}
      targets: ${{ inputs.targets }}
```

- The caller's "Concurrency group" job, after versions.yml names its runner, works out the flake-proof job's concurrency group from `ref` and the first 16 hex digits of the filter's SHA-256, since GitHub refuses a run whose group is over 400 characters before it starts, as a filter that names several tests in full makes it. Runs of different filters go side by side; a second run of the same `ref` and `filter` shows its first job done and waits at its flake-proof job until the first run ends.
- A plan job checks the inputs and splits the runs into up to `jobs` parallel jobs per target; each job checks out `ref`, installs the dependencies with `npm ci`, with no package's install scripts, as [Install scripts](#install-scripts) says, builds with `npm run build` and runs the npm script `script`, `test` by default, with the filters, once per run.
- Each job writes its runs, passes and failures to the run's summary and fails when any run failed; a failed run's last 60 log lines go to the job's log, and its whole log to the job's artifact, kept for the days versions.yml names.
- `targets` takes any of `linux-x64`, `linux-arm64`, `windows-x64`, `windows-arm64`, `macos-x64` and `macos-arm64`, in any number and order, separated by spaces; a target named twice runs once, and the run fails before the test jobs start on a name that isn't a target or a target with no runner.
- The caller may also pass `node-version`, `npm-version`, `install-registry-url`, `install-scope`, `retention-days` and `runs-on-<target>`, which [Shared settings](#shared-settings) explains; a caller that passes none takes the defaults.
- A repository proves an end-to-end test with three more inputs, which its caller adds to its own `workflow_dispatch` and passes on:
  - `script`: the npm script each run runs, such as `test:e2e`.
  - `prepare`: npm scripts each job runs once, after `npm run build` and before the runs, separated by spaces, such as an app's install and build. They run with the build's token and with no package's install scripts.
  - `display`: a boolean, off by default. On Linux it runs each run under `xvfb-run`, on a 1920×1080 screen, and `dbus-run-session`, for a test that opens windows; Windows and macOS runners have a display of their own.

  Before it installs anything, each job checks that `script` and every `prepare` script are npm scripts in the package's `package.json`, and fails otherwise.
- A test job may take `timeout-minutes`, 120 by default and up to 360, which a caller with a slow suite raises.
- The jobs install from the registries and run on the runners [Shared settings](#shared-settings) names; a private repository's jobs read the organization's private packages from GitHub's registry with the run's own token, which `packages: read` allows, and a public repository's jobs hand npm no token.
- The caller grants `contents: read` and `packages: read`, since a called job gets no permission its caller didn't grant.

## Install scripts

- The release and flake proof workflows set `npm_config_ignore_scripts`, so every npm their jobs run, `npm ci` and any npm a repository's own scripts run, runs no package's install scripts while the run's token is set.
- `npm ci` then runs none of the repository's own install scripts either, so after it the install step runs the root `package.json`'s `preinstall`, `install`, `postinstall` and `prepare` scripts by name, in that order, each only when the root has it; an npm they run installs with no package's install scripts too.
- Nothing else runs: not `prepublish`, `preprepare` or `postprepare`, not `preinstall` before the dependencies as `npm ci` would run it, and not a workspace's scripts, so an install that needs any other script needs a step of its own in these workflows.
- `npm run` still runs the script it names, such as `npm run build` or `npm test`, but not its `pre` and `post` scripts, so a repository puts what those would run in the script itself.
- A dependency whose install script a repository truly needs is run by name in a step of its own, without a token; none needs one today, since each package with an install script loads a binary it or its platform's optional package ships.

## Shared settings

[versions.yml](.github/workflows/versions.yml) holds the settings every shared workflow uses in one place: the Node and npm versions, the registries to install from and publish to, how long artifacts are kept, and the runners; each shared workflow's first job calls it, and its other jobs take the settings from its outputs, so a caller names none:

- The Node version is `24.21.0`, and the npm version that comes with it, `11.19.0`, is checked after setup.
- A private repository installs the `@noldova-com` scope's packages from GitHub's registry, `https://npm.pkg.github.com`, and every other package from npm's registry; a public repository installs every package from npm's registry, with no token.
- A release from a public repository publishes `@noldova` packages to npm's registry, `https://registry.npmjs.org`, with public access; from a private repository, `@noldova-com` packages to GitHub's registry with restricted access.
- A private repository's artifacts are kept for 1 day, a public repository's for 7.
- A public repository's jobs run on GitHub-hosted runners: `ubuntu-24.04` and `ubuntu-24.04-arm` for Linux, `windows-2025` and `windows-11-arm` for Windows, `macos-15-intel` and `macos-15` for macOS, x64 and arm64.
- A private repository's jobs run where the organization's variables say: with `RUNNER_TYPE` set to `GITHUB`, on the same GitHub-hosted runners; otherwise on the organization's own runners, whose `runs-on` value `RUNS_ON_SELF_HOSTED_<system>` holds as JSON for x64 and `RUNS_ON_SELF_HOSTED_<system>_ARM64` for arm64, for the systems `LINUX`, `WINDOWS` and `MACOS`; a target whose variable is empty has no runner.

A caller overrides any of them for its own runs:

- `node-version` takes an exact version, such as `24.21.0`; with it, npm is checked only when `npm-version` names the version too.
- `install-registry-url` and `install-scope` go together: the install reads that scope's packages from that registry, and the run's token goes only to GitHub's registry.
- `retention-days` takes 1 to 90.
- A release's `publish-registry-url`, `publish-scope` and `publish-access` take a registry's https address, a scope and `public` or `restricted`.
- `runs-on-<target>`, such as `runs-on-linux-arm64`, takes the `runs-on` value as JSON, such as `'"ubuntu-24.04-arm"'` or a private repository's runner group and labels.
- A public repository's runner must be one GitHub hosts, named by one label such as `"ubuntu-24.04"`: versions.yml fails the run before the other jobs start when it isn't.

The versions job itself runs on Linux x64 by the same rule, written out in its `runs-on`, since GitHub doesn't let `runs-on` read a workflow's `env`.
The organization's own runners serve private repositories only, and its variables are visible to them only.
