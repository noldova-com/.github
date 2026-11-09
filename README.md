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
| [.github/workflows/linked-issue.yml](.github/workflows/linked-issue.yml) | The shared check that every pull request names its issue, which every repository, this one included, runs from its own `require-linked-issue.yml` |
| [.github/workflows/build-and-test.yml](.github/workflows/build-and-test.yml) | The shared workflow that builds and tests a repository on each of its targets, with the checks every repository runs |
| [.github/workflows/release-npm.yml](.github/workflows/release-npm.yml) | The shared workflow that releases a repository's npm packages to the registry its caller names |
| [.github/workflows/update-dependents.yml](.github/workflows/update-dependents.yml) | The shared workflow that proposes a released version to the repositories that pin its packages |
| [.github/workflows/flake-proof.yml](.github/workflows/flake-proof.yml) | The shared workflow that runs a repository's tests many times over, to prove a flaky test fixed or to hunt one |
| [.github/workflows/docs-image.yml](.github/workflows/docs-image.yml) | The shared workflow that builds a repository's docs site and pushes it as a container image, with the nginx settings in [.github/docs-image/](.github/docs-image) |
| [.github/workflows/propose-pins.yml](.github/workflows/propose-pins.yml) | This repository's own workflow that proposes each new tag to the repositories that call these workflows |
| [.github/workflows/versions.yml](.github/workflows/versions.yml) | The Node and npm versions and the runners every shared workflow uses, in one place |

A repository runs the shared check from its own workflow, `.github/workflows/require-linked-issue.yml`:

```yaml
name: PR

on:
  pull_request:
    branches: [main]
    types: [opened, edited, synchronize, reopened, ready_for_review]

permissions:
  issues: read
  pull-requests: read

jobs:
  check:
    name: Check
    uses: noldova-com/.github/.github/workflows/linked-issue.yml@<full commit SHA> # <tag>
```

A pull request's description names its issue on a line of its own that starts, at its first character, with `Issue` or a GitHub closing keyword (`Close`, `Closes`, `Closed`, `Fix`, `Fixes`, `Fixed`, `Resolve`, `Resolves` or `Resolved`), in any letter case, then the number, such as `Closes #123` or `Issue: #123`.
A colon may follow the word directly, and a full stop may follow the number directly, as GitHub accepts it; any other text on the line, or a blank before the word or the full stop, fails the check.
The number names an issue in this repository, not a pull request: GitHub also closes from a reference such as `owner/repo#123`, but the check refuses it.

Keep the workflow named `PR` and the calling job named `Check`: GitHub puts the calling job's name before the shared job's, so the check reports as "Check / Linked issue", the name a required-checks rule must list, and a pull request lists it as "PR / Check / Linked issue".
The check runs as [Shared settings](#shared-settings) explains.

## Building and testing

A repository builds and tests every pull request and every push to main from its own workflow, `.github/workflows/build-and-test.yml`, which names its settings only:

```yaml
name: Build and test

on:
  pull_request:
    branches: [main]
    types: [opened, synchronize, reopened, ready_for_review]
  push:
    branches: [main]

permissions:
  contents: read

concurrency:
  group: ${{ github.workflow }}-${{ github.event.pull_request.number || github.ref }}
  cancel-in-progress: ${{ github.event_name == 'pull_request' }}

jobs:
  build-and-test:
    name: Build and test
    uses: noldova-com/.github/.github/workflows/build-and-test.yml@<full commit SHA> # <tag>
    permissions:
      contents: read
      packages: read
    with:
      targets: linux-x64 windows-x64
      publishes: true
```

- `targets` names the targets, from `linux-x64`, `linux-arm64`, `windows-x64`, `windows-arm64`, `macos-x64` and `macos-arm64`; each gets a job of its own, named as its runner's system and architecture, such as "Linux x64".
- Each target's job builds with `npm run build`, then checks with the `pipelines` steps of [`@noldova/tools.pipelines.github`](https://www.npmjs.com/package/@noldova/tools.pipelines.github), installed at the exact version `PIPELINES_TOOL` names from npm's registry: `pipelines toolchain` checks the Node and npm versions [Shared settings](#shared-settings) names; `pipelines line-endings` checks that `.gitattributes` holds `* text=auto eol=lf` and that every tracked text file uses LF line endings and ends with exactly one newline; then `npm run test -- --skip-test-details --skip-coverage-details` runs the tests, and `pipelines unchanged` checks that the build and the tests changed no tracked file and left no untracked file outside the ignored outputs.
- With `publishes: true`, for a repository that releases packages as [Releasing npm packages](#releasing-npm-packages) says, `pipelines reproducible _packages` builds the commit again in another folder and checks that the archives are byte-identical, with Git's `core.longpaths` set for that step, since the folder's longer path can take a deep file past Windows' path limit, and `pipelines archive` checks them as the release does, at the version `version.json` holds.
- With `tool-from-source: true`, for the repository that holds the tool, the jobs install the tool from the archives the build just packed in `_packages/` instead, so a change to a step is checked by the run that tests it.
- `timeout-minutes` bounds each target's job, 30 by default; the Node, npm, install registry and runner overrides are those of [Shared settings](#shared-settings).
- The build steps get the run's own token for GitHub's registry only, as the release's build does; the tests and the checks get none.
- An "All targets" job passes only when every target passed. Keep the calling job named `Build and test`: GitHub puts its name before the shared jobs', so a required-checks rule lists "Build and test / All targets", and a pull request lists each target as "Build and test / Build and test / Linux x64".

## Releasing npm packages

A repository releases its npm packages by hand from its own workflow, `.github/workflows/release-npm.yml`.
A public repository publishes to npm's registry by [trusted publishing](https://docs.npmjs.com/trusted-publishers), with no stored token:

```yaml
name: Release npm packages

on:
  workflow_dispatch:
    inputs:
      version:
        description: The version to release, which the commit's version.json holds
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
- A private repository's caller names its own environment, and passes `keep-versions` and `secrets: inherit` when it keeps only the newest versions.
- The jobs first tag the commit with the version, after checking that it is on main and holds the version, then build and check the archives, publish them in the caller's environment, and keep the newest `keep-versions` versions when it is above 0.
  The tag comes first, before the publish waits for its approval, because GitHub lets the run's own token tag only a commit whose workflow files match main's, so a workflow change merged during the wait would leave the version untagged; when the tag is refused, nothing is built or published.
  A version whose build or publish then fails keeps its tag: a release of the same version at the same commit can still publish it, and releasing other changes needs a new version.
- The build job installs the dependencies from the registry [Shared settings](#shared-settings) names for installs, as the flake proof does; the run's own token goes to GitHub's registry only.
- No package's install scripts run in any job, as [Install scripts](#install-scripts) says, and the publishing token is set only on the publish and prune steps.
- The tag, build, publish and prune jobs run the steps of [`@noldova/tools.pipelines.github`](https://www.npmjs.com/package/@noldova/tools.pipelines.github), installed at the exact version `RELEASE_TOOL` names from npm's registry: `pipelines source` checks that the commit is on main and that its `version.json` holds the version; `pipelines archive` checks the built archives; `pipelines publish` publishes them and waits until the registry serves them; `pipelines prune` deletes the older versions no repository locks; `pipelines tag` tags the commit.
- So the repository provides `version.json` at its root, such as `{ "version": "1.4.0" }`; a `package.json` under `src/`, outside a `tests` folder, for every package it releases, and `"private": true` on any other; and a build that packs each of those packages, and nothing else, into `_packages/` at the version, recording the commit it was built from as `gitHead`, with its other packages as dependencies at that same version.
- The build answers `npm run build --silent -- --list-copies`, which builds nothing and prints one line for each file it copies unchanged into a package, with tabs between the source's path from the repository's root, the package's name and the path inside the package, then `end of copies` as the last line, also when it copies nothing; `pipelines archive` checks that each archive holds each copy, byte for byte equal to its source, in the release and in the build with `publishes: true`.
- Each package's archive holds `package/LICENSE`, its manifest has a `license` text, and it holds no file of a kind that holds credentials: `.npmrc`, `.netrc`, `.env` or `.env.<name>`, `id_rsa`, `id_dsa`, `id_ecdsa`, `id_ed25519`, or a name ending in `.pem`, `.key`, `.p12` or `.pfx`; `pipelines archive` and `pipelines publish` fail otherwise.
- A repository that passes `keep-versions` needs no release script of its own: `pipelines prune` deletes each package's older release versions on GitHub's registry, and only after checking that the version just published is the newest of each package.
  It keeps every version a `package-lock.json` holds on the default branch of a repository the version-updates App can reach, and deletes nothing when it can't read them all.
  The job runs only for GitHub's registry, so a caller on another registry that passes `keep-versions` gets no pruning and no failure; the step itself refuses any other registry, deleting nothing.
  It takes its token from `GITHUB_TOKEN`, which holds the `registry-token` secret when the caller passes one and the run's own token otherwise; the publish step's token is in `NODE_AUTH_TOKEN`.
  It reads the lock files with a token of the App, made as [the dependents job](#updating-dependents) makes its own, with contents and metadata read only, from the `VERSION_UPDATES_APP_KEY` secret of the caller's `updates-environment`, `version-updates` unless the caller names another; so the caller passes `secrets: inherit`, and the job fails first when the key is empty.
  `secrets: inherit` can't be combined with `secrets: registry-token: …`, and `registry-token` isn't a name a caller's secret can have, so a caller that passes `registry-token` can't prune.
- Without a `registry-token` secret, GitHub's registry takes the run's own token, and npm's registry takes trusted publishing: the package's trusted publisher names the calling repository, its `release-npm.yml` and its environment, and npm records the run as the version's provenance.
- Another registry passes its token with `secrets: registry-token: ${{ secrets.<name> }}`, which may name a secret that only the caller's environment holds.
- A new package's first publish to npm's registry can't use trusted publishing, since the package doesn't exist yet: the caller passes a one-time token held only in its environment, the same way, and removes the secret after the first publish, so the line resolves to empty and trusted publishing takes over.
  A public repository's packages on npm's registry get provenance either way.
- The caller grants `contents: write`, `packages: write` and `id-token: write`, whichever registry it publishes to, since a called job gets no permission its caller didn't grant.
- The publish job runs in the caller's environment, which an environment that needs approval approves; an environment name that doesn't exist yet is created on first use, with no protection.
  The prune job runs in the caller's `updates-environment`, with no approval of its own: the publish approval covers its deletes, and it deletes only versions no repository locks.
- The prune job runs only for GitHub's registry: a version published to npm's registry is never deleted.
- GitHub's registry serves a version's metadata with only some of its manifest's fields, although the archive it serves holds the whole manifest: `pipelines publish` sends the archive as built, and the registry's metadata drops `peerDependenciesMeta`, `exports`, `type` and `publishConfig`.
  npm reads `exports` and `type` from the installed package, but it resolves peers from the registry's metadata, so it takes a module's optional peers as required and installs them at the root that installs the module.
  So a root that installs a module with optional peers from GitHub's registry installs with `--legacy-peer-deps`, set as `legacy-peer-deps=true` in that root's `.npmrc` or passed by the script that installs it, and provides the peers it needs itself.

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
- The build job hands the builds to `npm run build` in `RELEASE_ADDONS`, the absolute path of a folder in the runner's temporary folder, outside the checkout, that holds `<package path in the repository>/<platform>-<arch>/<name>.node`, such as `src/noldova.example.package/linux-arm64/posix-process.node`; the folder's root also holds a `built-on-<platform>.txt` from each runner.
- The repository's build puts each build in its package at `native/<platform>-<arch>/<name>.node`, and fails when a declared target's build is missing; its own code loads `native/${process.platform}-${process.arch}/<name>.node`.
- After the build, the release refuses an archive that lacks any of its addons' builds for any target, naming the package, the addon and the target.
- The addon jobs run on the x64 runners of Linux, macOS and Windows that [Shared settings](#shared-settings) names, and need the organization's own runners, which hold the sysroots: a private repository with `RUNNER_TYPE` set to `GITHUB` passes its own runners with `runs-on-<system>-x64`, and a public repository can't release a package with addons yet: the plan job refuses it.

## Updating dependents

After a release, a repository proposes the version to every repository that pins its packages exactly, with a second job in its release workflow:

```yaml
  dependents:
    needs: release
    uses: noldova-com/.github/.github/workflows/update-dependents.yml@<full commit SHA>
    permissions:
      contents: read
      packages: read
    with:
      version: ${{ inputs.version }}
      commit: ${{ inputs.commit }}
    secrets: inherit
```

- The job runs `pipelines dependents` from [`@noldova/tools.pipelines.github`](https://www.npmjs.com/package/@noldova/tools.pipelines.github), at the exact version `PIPELINES_TOOL` names, in a checkout of the commit; its README says what the step changes and proposes.
- Its token is a GitHub App's, made in the job from the organization's `VERSION_UPDATES_CLIENT_ID` variable, the App's client ID, and the `VERSION_UPDATES_APP_KEY` secret of the caller's `environment`, `version-updates` unless the caller names another; that environment deploys from `main` only.
  The caller passes `secrets: inherit`, since a called workflow reads no secret the caller doesn't pass, an environment's included; the job fails first when the key is empty.
- The App needs contents, issues and pull requests write and metadata read; the repositories its installation is given are the ones the step checks, so a repository joins or leaves by the installation's list.
  The installation must include the releasing repository too, since the step reads the release notes there; it is never updated itself.
- A public repository's run proposes only to public repositories, so a private repository's name never shows in its public log; a private repository's run proposes to every repository the installation is given.
  The step reads whether the releasing repository is private with the App's token, and counts it as public when it can't read it.
- The commits are the App's bot account's, as author and committer.
- One releasing repository's runs take turns, so two quick releases never race on the same branches; another repository's run goes on at the same time.
- npm refreshes the lock files from the registry [Shared settings](#shared-settings) names for installs; the run's own token goes to GitHub's registry only, so the caller grants `packages: read`, and the packages a lock file names must let the calling repository read them.

## Reaching the callers

A repository calls these workflows pinned at a full commit, with the tag as a comment, such as `@<full commit SHA> # v2.5.0`, so a change here reaches it only through a new tag and a pull request there:

- A maintainer tags `main` after each change that merges here, `v<major>.<minor>.<patch>`: a patch for a fix, a minor for a new input or behavior, and a major for a change a caller must make changes for.
- The tag starts this repository's `propose-pins.yml`, which runs `pipelines pins` from [`@noldova/tools.pipelines.github`](https://www.npmjs.com/package/@noldova/tools.pipelines.github), at the exact version `PIPELINES_TOOL` names; its README says which pins move and which stay.
  In each repository that calls these workflows at an older version of the same major, it proposes the new pins in one pull request with its own issue, from the branch `pipelines/pins-.github`, and a later tag updates that same pull request.
- The proposals carry the release notes of the pull request whose merge commit is tagged: its section headed `## Release notes`, or `## Release notes for <version>`, up to the next heading of that level or higher or a line that starts with `Agent:`, a pull request's signature; so a pull request that will be tagged writes its notes there.
- Since this repository is public, the step proposes only to public repositories; a private repository moves its pins itself.
- In a repository with a Dependabot file, Dependabot moves the outside actions, such as `actions/checkout`, once a week, and the step moves only this repository's pins; elsewhere the same pull request also moves the outside actions to the versions these workflows use.
- The job's token is the version-updates App's, as in [Updating dependents](#updating-dependents), with workflows write as well; the job runs in this repository's `version-updates` environment, which deploys from the version tags only and holds `VERSION_UPDATES_APP_KEY`.

## Proving flakes

A repository proves a flaky test fixed, or hunts one, by hand from its own workflow, `.github/workflows/flake-proof.yml`, which a maintainer starts with `workflow_dispatch` only, never a pull request, issue or comment trigger, since the jobs build and run the tested code:

```yaml
name: Flake proof

# The inputs' defaults, which the inputs below take by their anchors, since an input's default can't read env.
env:
  DEFAULT_RUNS: &default-runs '10'
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
  - `display`: a boolean, off by default. On Linux it runs each run under `xvfb-run`, on a 1920×1080 screen, and `dbus-run-session`, for a test that opens windows, with an unlocked `gnome-keyring` secret service on a new empty keyring, for a test whose app keeps a secret; it installs `gnome-keyring` on a GitHub-hosted runner and fails on a self-hosted one without it. Windows and macOS runners have a display and a secret store of their own.

  Before it installs anything, each job checks that `script` and every `prepare` script are npm scripts in the package's `package.json`, and fails otherwise.
- A test job may take `timeout-minutes`, 120 by default and up to 360, which a caller with a slow suite raises.
- The jobs install from the registries and run on the runners [Shared settings](#shared-settings) names; a private repository's jobs read the organization's private packages from GitHub's registry with the run's own token, which `packages: read` allows, and a public repository's jobs hand npm no token.
- The caller grants `contents: read` and `packages: read`, since a called job gets no permission its caller didn't grant.

## Building a docs image

A private repository with a docs site builds it into a container image on each push to main from its own workflow, `.github/workflows/docs.yml`:

```yaml
name: Docs

on:
  push:
    branches: [main]

permissions: {}

jobs:
  docs:
    name: Docs
    uses: noldova-com/.github/.github/workflows/docs-image.yml@<full commit SHA> # <tag>
    permissions:
      contents: read
      packages: write
    with:
      build: npm ci && npm run build && npm run docs:build
      output: _build/docs/browser
      image: <repository>-docs
```

- `build` is the bash command that installs the dependencies and builds the site, run from the repository's root with `-euo pipefail`; `output` is the folder it writes the site to, which must hold an `index.html`; `image` is the image's name under the organization on `ghcr.io`, lowercase.
  `tag` is the tag it is pushed under, the commit's full hash by default; a branch's trial run passes a tag of its own, and nothing deploys by tag.
- The site job checks out the commit, sets up Node and npm as [Shared settings](#shared-settings) says, runs `build` with no package's install scripts, as [Install scripts](#install-scripts) says, and with the run's token for GitHub's registry only, then keeps the folder as the run's artifact.
- The image job builds the image without Docker, with [crane](https://github.com/google/go-containerregistry/blob/main/cmd/crane/README.md) `v0.22.1`, which it downloads from its release and checks against the SHA-256 the release's `checksums.txt` gives for `go-containerregistry_Linux_x86_64.tar.gz`; nothing is installed on the runner.
  It appends one layer onto `nginxinc/nginx-unprivileged` `1.30.6-alpine`, pinned by the digest of its multi-platform index, for `linux/amd64`: the site under `/usr/share/nginx/html/`, and [nginx.conf](.github/docs-image/nginx.conf) and [headers.conf](.github/docs-image/headers.conf) from this repository at the workflow's own commit, under `/etc/nginx/`, so no repository keeps its own copy.
  The layer's files are root's, readable by all, and dated at the commit's time, so a new commit's files never share an older one's validators.
- It pushes `ghcr.io/<organization>/<image>:<tag>` with the run's own token, which `packages: write` allows and which goes to `ghcr.io` only, then writes the image, the tag and the digest to the run's summary and to the workflow's `image` and `digest` outputs; whatever deploys the site takes it by that digest.
- The image runs nginx as user 101 on port 8080, writing only under `/tmp`, so it runs on a read-only root with a writable `/tmp`, and with one worker process, since `auto` would start one per host CPU, whatever the container's CPU and memory limits.
  It serves each prerendered page at its path, `/a/b` from `/a/b/index.html`, answers 404 for a path with no file, never another page, and answers `GET /healthz` with 200 and `ok`.
  Every answer carries the headers in `headers.conf`: `Cache-Control`, a year and immutable for a file whose name carries an 8-character content hash, such as `main-ABCD1234.js`, and `no-cache` for the rest; HSTS; a content security policy that limits plugins, the base address, form targets and framing, and leaves scripts and styles alone; `nosniff`; and the referrer policy.
- Both jobs run on the Linux x64 runner [Shared settings](#shared-settings) names, and the image job checks that it is Linux x64.
- The caller grants `contents: read` and `packages: write`, since a called job gets no permission its caller didn't grant; the site job keeps only `contents: read` and `packages: read`.
- A new image package is private; the organization's owners choose whether it is public.

## Install scripts

- The build and test, release and flake proof workflows set `npm_config_ignore_scripts`, so every npm their jobs run, `npm ci` and any npm a repository's own scripts run, runs no package's install scripts while the run's token is set.
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
One more variable, `RUNS_ON_INTERNAL_FEED`, holds as JSON the `runs-on` value of the one runner that writes an app's internal update feed, in a runner group open only to that app's release workflow; no other job runs there.
