# shared-go

The shared golang workflow repository for the organization.

## Motivation

Releasing go libraries and programs is quite formulaic and hopefully boring.  Trying
to maintain lots of duplicate workflows that *should* be the same is tedious and
error prone.

The solution is to create a set of workflows that are designed to be tested and
reused throughout the organization and company.  This repository is designed to
test the workflows before they are put into production as well as provide a
way for dependabot to do much of the boring stuff for us (like keeping
dependencies up to date).  And to test everything.

## Maintaining a Repository Using `shared-go`

### Dependabot & Automation

To be efficient, dependabot really should be setup for each repo and allowed to
commit updates.  Protect robot's rights; never let a human do a robot's work.

Do ensure that branch protection rules requiring a `pull request` are in place
and require the `All checks passed check.` status to succeed to protect against
broken code from automatically being merged.

This does not prevent maintainers from pushing code as they deem reasonable, but
do think about using the workflow & a PR to verify code changes.  It is reasonable
to over-ride the requirement for an approval to merge your PR.  "You break it,
you fix it" rules are in effect.

### Clean Code

Strive to run with all the checking on in the workflow.  Despite being annoying
or tedious at times, the value is more consistent code across projects.  "Running
clean" means there are fewer exceptions to trip us up.

### Multiple Architectures

Expect to generate output for multiple architectures (x86 and arm64 today, others
in the future), so make code and containers work everywhere when possible.

### Changelog

No need to update CHANGELOG.md files anymore, as that information is automatically
gathered by goreleaser for us based on commit messages.  The CHANGELOG.md files
will be removed so the source of truth is more clear soon.

### Commit Messages

The following comment "prefixes" will automatically categorize your work in the
release notes, so please use them.

- `feat(deps):` or `fix(deps):` will group the commit into **Dependency Updates**
- `feat:` or `feat([:word:]):` (where `[:word:]` is any word except `deps`) will group the commit into **New Features**
- `fix:` or `fix([:word:]):` (where `[:word:]` is any word except `deps`) will group the commit into **Bug Fixes**
- `doc:` or `doc([:word:]):` (where `[:word:]` is any word) will group the commit into **Documentation Updates**
- The following prefixes cause the commit to be omitted from the release notes.
   - `chore:`
   - `test:`
   - `merge conflict` (can happen anywhere in the commit message)
   - `Merge pull request` (can happen anywhere in the commit message)
   - `Merge remote-tracking branch` (can happen anywhere in the commit message)
   - `Merge branch` (can happen anywhere in the commit message)
   - `go mod tidy` (can happen anywhere in the commit message)
- Everything else will show up as **Other Work**

### Releasing

Releases are as simple as creating and pushing a semver tag in the following format:

```
v[0-9]+.[0-9]+.[0-9]+
```

The ci workflow will take care of the rest for you.

Example:
```bash
git pull --tags
git tag v1.0.1
git push --tags
```

Tags should be **forever** but releases can be removed if needed.

## Workflow Usage

For your go project you are going to want at least 2 workflows:

- A ci workflow.
- A dependabot approving workflow.

Project tracking is handled by the GitHub project's built-in "Auto-add to
project" workflow, so no workflow is needed here.

This workflow does the following things automatically:

- generates, then builds the program (may be disabled)
- runs all the unit tests (may be disabled)
- runs gofmt style checking (may be disabled)
- runs golangci-lint checking (may be disabled)
- checks the copyright headers (may be disabled)
- dependency license checking (may be disabled)
- reports test coverage and optionally enforces a minimum ([go-test-coverage](https://github.com/vladopajic/go-test-coverage))
- releasing via [goreleaser](https://goreleaser.com) (may be disabled)

## Reusable Workflows

Reference these from your repo by pinned commit SHA (dependabot keeps them
current):

- [`ci.yml`](.github/workflows/ci.yml): build, test, lint, and release.
- [`auto-releaser.yml`](.github/workflows/auto-releaser.yml): automatically tag or release patch/minor versions.
- [`approve-dependabot.yml`](.github/workflows/approve-dependabot.yml): approve dependabot PRs and enable auto-merge.

### CI Workflow (`ci.yml`)

```yaml
# SPDX-FileCopyrightText: 2023 Comcast Cable Communications Management, LLC
# SPDX-License-Identifier: Apache-2.0
---
name: 'CI'

on:
  push:
    branches:
      - main
    paths-ignore:
      - README.md
    tags:
      - 'v[0-9]+.[0-9]+.[0-9]+'
  pull_request:
  workflow_dispatch:

permissions:
  contents:      write
  packages:      write
  pull-requests: write   # used by the optional coverage PR comment

jobs:
  ci:
    uses: xmidt-org/shared-go/.github/workflows/ci.yml@<commit-sha> # vX.Y.Z
    with:
      release-type: program    # or: library
    secrets: inherit
```

The workflow file must grant at least the permissions shown, because a reusable
workflow can't be given more than its caller allows.

#### Inputs

| Name | Required | Type | Default | Description |
| :--- | :---: | :---: | :--- | :--- |
| `go-version` |  | string | `^1.20.x` | This input is deprecated. The value present in the go.mod file is used instead. |
| `go-version-latest` |  | boolean | `True` | Will always use the latest version of go available. |
| `go-generate-skip` |  | boolean | `True` | Skip running go generate if needed. |
| `go-generate-deps` |  | string |  | The line sparated list of go generate dependencies to install via `go install` prior to running `go generate`. |
| `working-directory` |  | string | `.` | The working directory for this project. |
| `description` |  | string | `${{ github.repository }} does something important.` | What this project/package does. |
| `license` |  | string |  | The license this project should use if it doesn't support REUSE/SPDX. |
| `build-skip` |  | boolean | `False` | Skip building the program. |
| `goreportcard-skip` |  | boolean | `False` | Skip running the goreportcard update. |
| `lint-skip` |  | boolean | `False` | Skip running the lint check. |
| `lint-timeout` |  | string | `5m` | The timeout to pass on to the linter. |
| `lint-version` |  | string | `latest` | The golangci-lint version to use. |
| `license-skip` |  | boolean | `False` | Skip building the license check. |
| `release-arch-amd64` |  | boolean | `True` | Set to enable amd64 binary and dockers to be built. |
| `release-arch-arm64` |  | boolean | `True` | Set to enable arm64 binary and dockers to be built. |
| `release-binary-name` |  | string |  | If the project needs a custom name, use set it here. |
| `release-custom-file` |  | boolean | `False` | If the project needs a custom release file, use that instead. |
| `release-debs` |  | boolean | `True` | If set to true, release generic debs as well. |
| `release-docker` |  | boolean | `False` | If set to true, release a container to gocr as well. |
| `release-docker-extras` |  | string |  | Provides a way to set the `extra_files` field with the list of files/dirs to make available. |
| `release-docker-file` |  | string | `Dockerfile` | Set to the docker file and path if you don't want the default of `Dockerfile` in the project root. |
| `release-docker-latest` |  | boolean | `False` | If set to true, release this container as the latest. |
| `release-docker-major` |  | boolean | `False` | If set to true, release this container as the latest for the major version. |
| `release-docker-minor` |  | boolean | `False` | If set to true, release this container as the latest for the minor version. |
| `release-main-package` |  | string | `.` | Path to main.go file or main package. |
| `release-notify-skip` |  | boolean | `False` | Deprecated: has no effect. Release notification of platform repos was removed. |
| `release-project-name` |  | string |  | The project name / binary name to use if not the repo name. |
| `release-rpms` |  | boolean | `True` | If set to true, release generic rpms as well. |
| `release-skip` |  | boolean | `False` | Skip releasing the program. |
| `release-skip-publish` |  | string |  | Set to --skip-publish to skip publishing. |
| `release-type` | ✅ | string |  | The type of artifact to expect and release. [ `library`, `program` ]. |
| `release-with-extra-contents` |  | string |  | The list of any extra files to include in the packaged releases. See goreleaser nfpm contents for examples. |
| `release-with-unique-user` |  | string | `true` | If set to true will add a user and group with the same name as the program. Otherwise root is assumed. |
| `copyright-skip` |  | boolean | `False` | Skip validating that all files have copyright and licensing information. |
| `style-skip` |  | boolean | `False` | Skip building the gofmt check. |
| `tests-race` |  | boolean | `True` | If set to "true" (default), race condition checking will be performed during unit tests. Otherwise no race condition checking will be done. |
| `tests-skip` |  | boolean | `False` | Skip running the unit tests. |
| `tests-min-threshold` |  | number | `80` | The minimum total test coverage percentage (0-100) required. The unit tests fail if coverage is lower. Set to 0 to report coverage without enforcing a minimum. |
| `tests-post-comment` |  | boolean | `true` | If set to true (default), post (and keep updated) a coverage report comment on the pull request. Comments are never posted for pull requests from forks or from dependabot, because their token is read-only. |
| `upload-skip` |  | boolean | `False` | Skip uploading the artifacts. |
| `yaml-lint-skip` |  | boolean | `True` | Skip linting yaml files. |

#### Test coverage

Coverage is checked with [go-test-coverage](https://github.com/vladopajic/go-test-coverage),
which needs no token.  Results show up in these places:

- **The check on the PR.**  The unit tests fail when total coverage is below
  `tests-min-threshold` (default 80).  Set it to `0` to only report.
- **The job summary and an annotation.**  Always produced: total coverage, the
  pass/fail result, and a per-package table (lowest first).
- **A PR comment (on by default).**  Set `tests-post-comment: false` to
  turn it off.  The caller must grant `pull-requests: write` (as in the
  example above) whether or not the comment is enabled.  If a repo has several
  modules, each `working-directory` gets its own comment; set the input to
  `false` on jobs that repeat a directory so they don't compete.

  One comment is posted and then updated on each push, passing or failing.
  It is skipped for dependabot and fork PRs, and if posting fails the
  job and the rest of the workflow still pass.

Per-file and per-package minimums can be set in a `.testcoverage.yml` file; see
the go-test-coverage documentation.

### Auto Releaser (`auto-releaser.yml`)

```yaml
# SPDX-FileCopyrightText: 2024 Comcast Cable Communications Management, LLC
# SPDX-License-Identifier: Apache-2.0
---
name: 'Automatically release patch versions.'

on:
  schedule: # Run every day at 12:00 UTC
    - cron: '0 12 * * *'
  workflow_dispatch:

permissions:
  contents: write
  actions:  write   # lets the release job dispatch your CI workflow

jobs:
  release:
    uses: xmidt-org/shared-go/.github/workflows/auto-releaser.yml@<commit-sha> # vX.Y.Z
    with:
      release-workflow: ci.yml   # your CI workflow; it needs a workflow_dispatch trigger
    secrets: inherit
```

Tags created with `github.token` don't trigger other workflows, so after
tagging the auto-releaser dispatches `release-workflow` on the new tag.  The
dispatched run executes your CI workflow as it exists at that tag, so its
`with:` settings are not repeated.  If `release-workflow` is empty the job
fails before creating a tag, rather than leaving an unreleased one.

#### Inputs

| Name | Required | Type | Default | Description |
| :--- | :---: | :---: | :--- | :--- |
| `branch` |  | string | `main` | Branch to release from. |
| `patch-list` |  | string | `bugfix, Bugfix, BugFix, BUGFIX, chore, Chore, CHORE, fix, Fix, FIX, perf, refactor, Refactor, REFACTOR, test, Test, TEST, tests, Tests, TESTS` | Comma separated list of commit types that should trigger a patch release. |
| `minor-list` |  | string | `feat, Feat, FEAT, feature, Feature, FEATURE` | The specific minor prefix names to use for minors. |
| `tag-filter` |  | string | `^.+$` | A string to filter tags by. By default allows all tags. This is useful if you want to only consider tags that match a certain pattern for the next version calculation. Primarily this is used to ignore older tags in the shared-go repo that are used for testing the release process. This is mapped directly into the ietf-tools/semver-action 'tagFilter' input. |
| `which` |  | string | `tag` | Create a 'release' or 'tag'. |
| `release-workflow` |  | string |  | File name of the caller's CI workflow (e.g. `ci.yml`). It must have a `workflow_dispatch` trigger. A tag created with the default GITHUB_TOKEN does not trigger other workflows, so after tagging this workflow is dispatched on the new tag to run the release. Required when `which` is `tag`. |
| `release-ignore-paths` |  | string | `.github/** **/*.md LICENSE LICENSES/** .reuse/** .gitignore .gitattributes` | Newline-separated list of glob patterns. If every file changed since the last matched tag falls within these patterns, the auto-release is skipped — preventing patch bumps from chore(deps) or docs/CI commits that didn't touch shippable code. Pass an empty string to disable the check. |

### Dependabot Approver (`approve-dependabot.yml`)

```yaml
# SPDX-FileCopyrightText: 2023 Comcast Cable Communications Management, LLC
# SPDX-License-Identifier: Apache-2.0
---
name: 'Approve Dependabot'

on:
  pull_request_target:

permissions:
  pull-requests: write
  contents: write

jobs:
  dependabot:
    uses: xmidt-org/shared-go/.github/workflows/approve-dependabot.yml@<commit-sha> # vX.Y.Z
```

The repo must have *Allow auto-merge* enabled.  No secrets are needed.

## Secrets and Variables

Everything uses the built-in `github.token` where it can. The secrets below
are the only ones that remain, because `github.token` cannot do the job.
Callers pass them with `secrets: inherit`.

### Secrets

| Secret | Used by | Required | Why `github.token` isn't enough |
| :--- | :--- | :---: | :--- |
| `RPM_GPG_PRIVATE_KEY` | `ci.yml` (`release`) | if `release-rpms` or `release-debs` | Private key used to sign RPM/DEB packages. |
| `RPM_GPG_PRIVATE_PASSWORD` | `ci.yml` (`release`) | if `release-rpms` or `release-debs` | Passphrase for the signing key. |

Tagging, the dependabot approver, tag cleanup and `tests.yml` all use
`github.token`.

### Variables

Variables are not secret and are set under *Settings → Secrets and variables →
Actions → Variables*.

| Variable | Used by | Required | Description |
| :--- | :--- | :---: | :--- |
| `RPM_GPG_PUBLIC_KEY` | `ci.yml` (`release`) | if `release-rpms` or `release-debs` | Public key matching `RPM_GPG_PRIVATE_KEY`, shipped in the RPM/DEB packages. |

### Dependabot runs

Workflows triggered by dependabot (`pull_request` and `pull_request_target`
events on dependabot's PRs) do **not** see regular Actions secrets.  They only
get secrets stored under *Settings → Secrets and variables → Dependabot*, and
the `GITHUB_TOKEN` is read-only on `pull_request`.  Variables are not affected.

| Secret | Needed for dependabot PRs? | Notes |
| :--- | :---: | :--- |
| `RPM_GPG_PRIVATE_KEY`, `RPM_GPG_PRIVATE_PASSWORD` | no | `release` only runs on version tags. |

The auto-merge itself needs no secrets: `approve-dependabot.yml` runs on
`pull_request_target` with `github.token` and its write permissions, and the
repo must have *Allow auto-merge* enabled.

## Workflow Development

There are three special directories:
- command
- library
- program

The `command` directory represents a go program in a cmd structured directory for
release testing purposes.

The `library` directory represents a go library for release purposes.

The `program` directory represents a go program for release purposes.

Releases are tested automatically now.  The repo is tagged, but the release is
not actually uploaded.  To verify the release is what you expect, comment out
the `--skip-publish` options and trigger a PR.

Test tags (`v1.0.x`) are created by `tests.yml` and removed weekly by
`cleanup-tags.yml`.  Dependabot PRs skip the test tag because their token is
read-only.

Generally create a PR to test a change.

As it makes sense, add more test workflows (see `tests.yml`) to implement more tests.
The idea is to generally cover the use cases as well as bugs that we find.

### Releasing

After code has been tested to prove it works, release it with a release version
starting with `v3.0.0` and follows [semantic versioning](http://semver.org/)
conventions.  Dependabot should take care of the rest for us.
