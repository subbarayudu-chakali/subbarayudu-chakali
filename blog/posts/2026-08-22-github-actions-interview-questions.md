# GitHub Actions Interview Questions & Answers

A practical, interview-focused reference for **GitHub Actions** — from the core
vocabulary to real-world CI/CD, security, and troubleshooting questions. I've
grouped the questions by theme so you can skim to what you need. Answers are kept
concise but complete enough to say out loud in an interview.

---

## Fundamentals

**1. What is GitHub Actions?**

GitHub Actions is a CI/CD and automation platform built into GitHub. It lets you
run automated workflows in response to events in your repository (pushes, pull
requests, issues, releases, schedules, and more). Workflows are defined as YAML
files stored in `.github/workflows/`.

**2. What are the core components of GitHub Actions?**

- **Workflow** — an automated process defined in a YAML file; contains one or more jobs.
- **Event** — the trigger that starts a workflow (e.g. `push`, `pull_request`).
- **Job** — a set of steps that run on the same runner. Jobs run in parallel by default.
- **Step** — an individual task; either a shell command (`run`) or an action (`uses`).
- **Action** — a reusable unit of code (the smallest portable building block).
- **Runner** — the server that executes a job (GitHub-hosted or self-hosted).

**3. Where are workflow files stored and in what format?**

In the `.github/workflows/` directory at the root of the repository, written in
YAML (`.yml` or `.yaml`). You can have multiple workflow files; each runs independently.

**4. What is the difference between a job and a step?**

A **job** is a collection of steps that run on a single runner and share the same
filesystem. A **step** is a single command or action inside a job. Steps in a job
run sequentially and share state; jobs are isolated from each other unless you
explicitly pass data or add dependencies.

**5. What is an action versus a workflow?**

An **action** is a reusable, packaged piece of automation (e.g. `actions/checkout`).
A **workflow** is the orchestration file that wires events, jobs, and steps together.
You *use* actions inside a workflow.

**6. What are the types of actions?**

- **JavaScript actions** — run directly on the runner via Node.js; fast, cross-platform.
- **Docker container actions** — run inside a container; full control of the environment (Linux runners only).
- **Composite actions** — bundle multiple steps (shell commands and other actions) into one reusable action.

---

## Events & triggers

**7. What is the `on` keyword?**

`on` defines the events that trigger a workflow. It can be a single event, a list,
or a map with filters:

```yaml
on:
  push:
    branches: [main]
    paths: ['src/**']
  pull_request:
    branches: [main]
  workflow_dispatch:
  schedule:
    - cron: '0 2 * * *'
```

**8. What is `workflow_dispatch`?**

It enables manually triggering a workflow from the GitHub UI, API, or CLI. You can
define `inputs` that the user provides at run time.

**9. What is `repository_dispatch`?**

An event that lets you trigger a workflow via an external API call (a webhook-style
POST to the GitHub API), useful for integrating third-party systems.

**10. How do you schedule a workflow?**

Use the `schedule` event with POSIX cron syntax (always in UTC):

```yaml
on:
  schedule:
    - cron: '0 0 * * MON'   # every Monday at 00:00 UTC
```

**11. What is the difference between `pull_request` and `pull_request_target`?**

`pull_request` runs in the context of the PR's merge commit and, for forked PRs,
has a **read-only** token and no secrets — this is the safe default. `pull_request_target`
runs in the context of the **base** repository with full write token and secrets,
which is powerful but dangerous: never check out and execute untrusted PR code under it.

**12. Can one workflow trigger another?**

Yes, several ways: `workflow_run` (run a workflow after another completes),
`workflow_call` (reusable workflows), `repository_dispatch`, or having a workflow
push a commit/tag. Note: a token's default `GITHUB_TOKEN` does **not** trigger new
workflow runs to prevent infinite loops — use a PAT or the `workflow_run` event.

**13. What are activity types?**

Many events have sub-types. For example `pull_request` supports `opened`,
`synchronize`, `closed`, `reopened`, etc. Filter with `types`:

```yaml
on:
  pull_request:
    types: [opened, reopened]
```

---

## Runners

**14. What is a runner?**

A runner is the machine that executes your jobs. GitHub-hosted runners are fresh
VMs provisioned per job; self-hosted runners are machines you manage.

**15. GitHub-hosted vs. self-hosted runners — trade-offs?**

- **GitHub-hosted**: zero maintenance, clean environment every run, pay per minute
  (free tier for public repos), limited customization, no persistent state.
- **Self-hosted**: full control of hardware/software, can access private networks,
  cheaper for heavy usage, but you own patching, security, and scaling. State can
  leak between jobs, so they're riskier for public repositories.

**16. How do you specify the runner?**

With `runs-on`:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest        # or windows-latest, macos-latest
  deploy:
    runs-on: [self-hosted, linux, x64]   # label matching for self-hosted
```

**17. What are runner labels?**

Labels are tags that route jobs to specific runners. Self-hosted runners can carry
custom labels (e.g. `gpu`, `prod`) so `runs-on` can target exactly the right machine.

**18. What is a runner group?**

Runner groups organize self-hosted runners for access control at the org/enterprise
level, letting you restrict which repositories can use which runners.

**19. What are matrix builds?**

A matrix runs a job multiple times across combinations of variables (OS, language
versions, etc.):

```yaml
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest]
    node: [18, 20, 22]
runs-on: ${{ matrix.os }}
```

Use `include`/`exclude` to add or remove specific combinations, `fail-fast: false`
to keep running other combos when one fails, and `max-parallel` to cap concurrency.

---

## Secrets, variables & security

**20. How do you handle secrets in GitHub Actions?**

Store them as encrypted secrets at the repository, environment, or organization
level, then reference with `${{ secrets.NAME }}`. They're masked in logs and never
exposed to workflows triggered by forks (for `pull_request`).

**21. What is `GITHUB_TOKEN`?**

An automatically generated, short-lived token available in every workflow run as
`${{ secrets.GITHUB_TOKEN }}`. It authenticates against the repo with permissions
you can scope, and it's revoked when the job finishes.

**22. How do you scope permissions for `GITHUB_TOKEN`?**

With the `permissions` key, following least privilege:

```yaml
permissions:
  contents: read
  pull-requests: write
```

Set it at the workflow or job level. Default can be read/write or restricted,
depending on org/repo settings.

**23. Difference between secrets and variables?**

**Secrets** are encrypted and masked in logs (for sensitive data like API keys).
**Variables** (`vars` context) are plain configuration values, visible in logs,
for non-sensitive settings.

**24. What are environments and environment protection rules?**

Environments (e.g. `staging`, `production`) group deployment configuration and
secrets. Protection rules add required reviewers, wait timers, and branch
restrictions before a job targeting that environment can run.

**25. How do you prevent script injection attacks?**

Never interpolate untrusted input (like `github.event.pull_request.title`) directly
into a `run` script — an attacker could inject shell commands. Instead pass it
through an environment variable and reference the env var:

```yaml
env:
  TITLE: ${{ github.event.pull_request.title }}
run: echo "$TITLE"
```

**26. What is OIDC and why use it for cloud deployments?**

OpenID Connect lets a workflow request a short-lived token from a cloud provider
(AWS, GCP, Azure) instead of storing long-lived cloud credentials as secrets. The
cloud trusts GitHub's identity provider and issues temporary credentials — far more
secure and no secret rotation.

**27. How do you pin actions securely?**

Pin third-party actions to a **full commit SHA** rather than a tag, because tags can
be moved to point at malicious code:

```yaml
uses: actions/checkout@8f4b7f84864484a7bf31766abe9204da3cbe65b3   # v4.0.0
```

---

## Data flow, context & expressions

**28. What are contexts?**

Contexts are objects holding run-time information, accessed via `${{ }}`. Common
ones: `github`, `env`, `job`, `steps`, `runner`, `secrets`, `vars`, `matrix`,
`needs`, `inputs`.

**29. How do steps share data within a job?**

Via **step outputs** written to `$GITHUB_OUTPUT`:

```yaml
- id: gen
  run: echo "version=1.2.3" >> "$GITHUB_OUTPUT"
- run: echo "Version is ${{ steps.gen.outputs.version }}"
```

Files on disk are also shared because steps in a job share the runner filesystem.

**30. How do jobs share data?**

Jobs are isolated. Pass small values with **job outputs** + `needs`; pass files with
**artifacts** (`upload-artifact` / `download-artifact`):

```yaml
jobs:
  build:
    outputs:
      ver: ${{ steps.gen.outputs.version }}
    steps: [...]
  deploy:
    needs: build
    steps:
      - run: echo ${{ needs.build.outputs.ver }}
```

**31. What is `needs`?**

`needs` declares a dependency so one job waits for another to finish. It creates the
job execution order (a DAG) and exposes the upstream job's outputs.

**32. What are environment files like `$GITHUB_ENV`, `$GITHUB_OUTPUT`, `$GITHUB_PATH`?**

Special files whose paths are in env vars. Writing to them affects later steps:
`$GITHUB_ENV` sets environment variables, `$GITHUB_OUTPUT` sets step outputs, and
`$GITHUB_PATH` prepends directories to `PATH`. (These replaced the deprecated
`::set-output` and `::set-env` workflow commands.)

**33. What are workflow commands?**

Special strings printed to stdout that the runner interprets, e.g. `::error::`,
`::warning::`, `::group::`/`::endgroup::`, `::add-mask::`, `::notice::`. They control
logging, masking, and annotations.

**34. Give an example of an expression with a function.**

```yaml
if: ${{ contains(github.event.head_commit.message, '[skip ci]') == false }}
```

Built-in functions include `contains`, `startsWith`, `endsWith`, `format`, `join`,
`toJSON`, `fromJSON`, `hashFiles`, plus status checks `success()`, `failure()`,
`always()`, `cancelled()`.

---

## Control flow

**35. What does the `if` conditional do?**

`if` controls whether a step or job runs, based on an expression:

```yaml
- name: Deploy
  if: github.ref == 'refs/heads/main' && success()
  run: ./deploy.sh
```

**36. What is the difference between `success()`, `always()`, `failure()`, and `cancelled()`?**

Status check functions used in `if`. By default steps run only if previous steps
succeeded. `always()` runs regardless of status (great for cleanup/reporting),
`failure()` runs only when something failed, `cancelled()` runs when the run was
cancelled.

**37. What is `continue-on-error`?**

When `true`, a failing step/job doesn't fail the overall workflow — the run continues
and the failure is recorded but treated as non-blocking.

**38. What is `timeout-minutes`?**

Caps how long a job (or step) may run before it's forcibly cancelled. Default job
timeout is 360 minutes (6 hours). Setting a lower value avoids stuck runs eating minutes.

**39. What is `concurrency`?**

Limits concurrent runs in a group and can cancel in-progress ones:

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

Useful to cancel outdated CI runs when new commits land on a PR.

---

## Reuse & efficiency

**40. What are reusable workflows?**

Workflows callable from other workflows with `workflow_call`. They centralize common
logic (build, deploy) and accept `inputs`, `secrets`, and return `outputs`:

```yaml
# caller
jobs:
  call:
    uses: org/repo/.github/workflows/deploy.yml@main
    with:
      environment: production
    secrets: inherit
```

**41. Reusable workflows vs. composite actions — when to use which?**

Use a **composite action** to package a sequence of steps used *within* a job. Use a
**reusable workflow** to share entire jobs/orchestration across repos. Composite
actions can't define jobs, runners, or `if` at job level; reusable workflows can.

**42. How does caching work?**

`actions/cache` stores and restores directories (dependencies, build outputs) keyed
by a hash of lock files:

```yaml
- uses: actions/cache@v4
  with:
    path: ~/.npm
    key: npm-${{ hashFiles('**/package-lock.json') }}
    restore-keys: npm-
```

Many setup actions (`setup-node`, `setup-python`) have built-in caching too.

**43. Caching vs. artifacts — what's the difference?**

**Caching** speeds up runs by reusing files between runs (dependencies), keyed and
best-effort. **Artifacts** persist build outputs (binaries, reports, test results)
to download later or pass between jobs; they're intentional, retained, and versioned.

**44. How do you upload and download artifacts?**

```yaml
- uses: actions/upload-artifact@v4
  with: { name: build, path: dist/ }
# in another job
- uses: actions/download-artifact@v4
  with: { name: build }
```

**45. What is a composite action and how do you write one?**

An action defined in an `action.yml` with `runs.using: "composite"` that bundles
steps:

```yaml
runs:
  using: "composite"
  steps:
    - run: npm ci
      shell: bash
    - run: npm test
      shell: bash
```

Note that `run` steps in composite actions **require** the `shell` key.

---

## CI/CD, deployment & real-world

**46. Describe a typical CI pipeline in GitHub Actions.**

On `pull_request`: checkout → set up language → install deps (cached) → lint → run
tests (matrix) → build → upload coverage/artifacts. On merge to `main`: build →
push image → deploy to staging → (approval) → deploy to production, often via
environments and OIDC.

**47. How do you deploy to AWS/GCP/Azure securely from Actions?**

Prefer OIDC with the cloud's official login action (`aws-actions/configure-aws-credentials`,
`google-github-actions/auth`, `azure/login`) to assume a role with short-lived
credentials — no static keys in secrets.

**48. How do you build and push a Docker image?**

Use `docker/login-action`, `docker/setup-buildx-action`, and `docker/build-push-action`,
authenticating to the registry (GHCR, Docker Hub, ECR) and pushing tags built from
`github.sha` or release tags.

**49. What is GitHub Container Registry (GHCR) and how do you push to it?**

GHCR (`ghcr.io`) is GitHub's container registry. Authenticate with `GITHUB_TOKEN`
(needs `packages: write` permission) and push `ghcr.io/OWNER/IMAGE:TAG`.

**50. How would you implement a manual approval gate before production?**

Target a `production` environment that has **required reviewers** configured. The
deploy job pauses until an authorized reviewer approves in the UI.

**51. How do you run only when specific files change?**

Use `paths`/`paths-ignore` filters on the event, or the `dorny/paths-filter` action
for per-job conditional logic in monorepos.

---

## Troubleshooting & operations

**52. How do you debug a failing workflow?**

Read the step logs and annotations, re-run with **debug logging** by setting the
repo secrets/variables `ACTIONS_STEP_DEBUG: true` and `ACTIONS_RUNNER_DEBUG: true`,
add temporary `run: env` / `set -x` steps, or use a tmate/SSH debugging action to
interactively inspect the runner.

**53. Why might secrets be empty in a workflow?**

Common causes: the workflow was triggered by a fork via `pull_request` (secrets are
withheld), the secret is defined at the wrong scope, environment protection blocks
it, or a typo in the name. Secrets are also unavailable in composite actions unless passed in.

**54. A scheduled workflow didn't run on time — why?**

Scheduled runs can be delayed during high load and run in UTC. Also, GitHub disables
`schedule` triggers on repositories with no activity for 60 days, and only runs
schedules from the default branch's workflow file.

**55. How do you cancel or limit redundant runs?**

Use `concurrency` groups with `cancel-in-progress: true` to auto-cancel superseded runs.

**56. What are annotations?**

Inline messages (errors, warnings, notices) attached to specific files/lines shown
in the PR and run summary, produced via workflow commands or actions like linters.

**57. What is the job summary?**

Markdown you append to `$GITHUB_STEP_SUMMARY` that renders on the run's summary page
— handy for test reports, links, and tables without digging through logs.

**58. How do you handle flaky tests / retries?**

There's no native step retry; use an action like `nick-fields/retry`, add retry
logic in your test runner, or split flaky tests. For whole jobs, re-run failed jobs
from the UI/API.

---

## Governance & advanced

**59. How do you restrict which actions can be used in an org?**

In org/repo settings you can allow only GitHub-authored actions, actions from
verified creators, or an explicit allowlist of `owner/repo` patterns — a key
supply-chain control.

**60. What is the difference between org, repo, and environment secrets?**

**Org secrets** are shared across selected repositories, **repo secrets** are local
to one repo, and **environment secrets** are scoped to a specific deployment
environment (and gated by its protection rules).

**61. How do you pass data from a matrix job's parallel runs to a single downstream job?**

Each matrix run writes distinct artifacts (or job outputs keyed by matrix value); a
downstream job with `needs` downloads all artifacts and aggregates them. Job outputs
from a matrix collapse, so artifacts are the reliable path for per-combination data.

**62. What are the main limits to be aware of?**

Concurrent job limits per plan, 6-hour max job time, 35-day max workflow time
(including waits), API rate limits, artifact/log retention (default 90 days,
configurable), and cache eviction (10 GB per repo, LRU). Know these when designing pipelines.

**63. How would you migrate from Jenkins to GitHub Actions?**

Map Jenkins stages → jobs, build agents → runners (self-hosted for parity),
credentials → secrets/OIDC, shared libraries → reusable workflows/composite actions,
and cron triggers → `schedule`. Start with CI, then move deployments behind environments.

---

## Quick-fire round

- **Default working directory?** The repo checkout root (`GITHUB_WORKSPACE`).
- **How to check out code?** `actions/checkout@v4` (it's not automatic).
- **Marketplace?** Where published, reusable actions are discovered.
- **`uses` vs `run`?** `uses` invokes an action; `run` executes a shell command.
- **Default shell?** `bash` on Linux/macOS, `pwsh` on Windows (configurable via `shell:`).
- **Can jobs run sequentially?** Yes — chain them with `needs`.
- **Fail a step deliberately?** `exit 1` in a `run` step.
- **Skip CI?** Include `[skip ci]` in the commit message (for supported events).

---

These cover the ground most GitHub Actions interviews walk through — from "what is a
runner" to script-injection defenses and OIDC. The best follow-up prep is to build a
small pipeline yourself: a matrix test job, a cached dependency install, an artifact
handoff, and an environment-gated deploy. Once you've wired those by hand, the answers
above stop being memorized and start being obvious.
