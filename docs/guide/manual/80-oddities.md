---
title: "80 — Oddities, gotchas and sharp edges"
classification: Public
---

This chapter lists what surprises a newcomer to the catalyst fleet: known bugs and their fixes, CI quirks, environmental requirements, historical traps, and design choices that look wrong but are deliberate (with their rationale). It reflects the state on **2026-10-31**, refreshed on **2026-11-01**: kernel **0.121.0**, SE module 3.6.0, KBM 0.6.1, catalyst-ui 0.73.0, catalyst-users 0.11.0, catalyst-mcp 0.4.1, catalyst-kb-engine 0.16.1. An entry fixed since is struck through (~~like this~~) and says what fixed it; entries new on 2026-11-01 say **New**. Section 11 gathers the new facts.

Each entry gives:

- **Symptom** — what you see.
- **Cause** — why it happens.
- **Fix / workaround** — what to do about it.
- **Source** — where it is recorded.

Every source is a public repository. `cantica-tech/<repo>:<path>` is a file in one; a bare path is a file in the repository the section is about; "CHANGELOG x.y.z" is that release's entry in the repository's `CHANGELOG.md`.

## Contents

- [0. Fleet-wide conventions that bite](#0-fleet-wide-conventions-that-bite)
- [1. Kernel — cantica-tech/catalyst](#1-kernel--cantica-techcatalyst)
  - [1.1 Release and CI](#11-release-and-ci)
  - [1.2 Tests](#12-tests)
  - [1.3 Identity, signing, journal](#13-identity-signing-journal)
  - [1.4 Migrations, sync, modules](#14-migrations-sync-modules)
  - [1.5 Documentation drift](#15-documentation-drift-in-the-kernel-repo)
  - [1.6 Historical traps](#16-historical-traps-kernel)
  - [1.7 Deliberate design choices](#17-deliberate-design-choices-that-look-odd)
- [2. SE module — cantica-tech/catalyst-software-engineering](#2-se-module--cantica-techcatalyst-software-engineering)
- [3. KBM — cantica-tech/catalyst-knowledge-base](#3-kbm--cantica-techcatalyst-knowledge-base)
- [4. catalyst-ui (Studio, VS Code, desktop)](#4-catalyst-ui--cantica-techcatalyst-ui)
- [5. catalyst-users (IAM facade, local Keycloak)](#5-catalyst-users--cantica-techcatalyst-users)
- [6. catalyst-mcp](#6-catalyst-mcp--cantica-techcatalyst-mcp)
- [7. catalyst-kb-engine](#7-catalyst-kb-engine--cantica-techcatalyst-kb-engine)
- [8. catalyst-docs (public site)](#8-catalyst-docs--cantica-techcatalyst-docs)
- [9. Distribution repo — cantica-tech/cantica-tech](#9-distribution-repo--cantica-techcantica-tech)
- [10. Cross-cutting summary tables](#10-cross-cutting-summary-tables) (with [10.3 Triage](#103-triage-of-the-most-common-failures))
- [11. What changed on 2026-10-31 and 2026-11-01](#11-what-changed-on-2026-10-31-and-2026-11-01)

---

## 0. Fleet-wide conventions that bite

| # | Oddity | Symptom | Cause / rationale | Fix / workaround | Source |
|---|---|---|---|---|---|
| 0.1 | **Branch models differ per repository** | You push to the wrong branch, or look for a `release` branch that does not exist. | Kernel, UI and catalyst-users use **development → main → release**, each step a PR, then a tag. The SE module, KBM (since 2026-11-01; it had no development branch before), catalyst-mcp and catalyst-kb-engine work on development and tag **on main**, with no release branch; catalyst-docs has development and main but no tags. The distribution repo is main-only. | Check `git branch -a` before releasing. | each repository's branches |
| 0.2 | **Two tag-naming eras** | <code>git tag \&#124; sort -V \&#124; tail</code> shows `v0.34.0` (kernel) or `v0.30.0` (UI) as "latest". | Early tags carry a `v` prefix; later ones do not (kernel `0.121.0`, UI `0.73.0`). | Use <code>git tag \&#124; grep -v '^v' \&#124; sort -V</code>. | tags of cantica-tech/catalyst and cantica-tech/catalyst-ui |
| 0.3 | **The criterion lives outside the project** (law L5) | A project holds only `catalyst.toml`; there is no `.criterion/` folder, no Taskfile and no `.claude/commands`. | Decision A1 (revised 2026-10-08, design 13). The criterion lives at `$HOME/.catalyst/projects/<name>/criterion`, with its own `uv` runtime. | Find it with `catalyst where`; ready it with `catalyst open`. Never add catalyst files to a host project. | cantica-tech/catalyst:docs/design/13-criterion-home-requirement.md; docs/decisions.md A1 |
| 0.4 | **IDs are permanent, and citing one early burns it** (law L7) | Gaps in a deployment's numbering. | Citing an ID (in a journal intent, a commit, prose) before `catalyst new` allocates it can reserve that number, and IDs are never reused. Each ID is suffixed with the creator's userid hash. | Name an ID only after `catalyst new` prints it. Never "fix" gaps. | kernel `INVARIANTS.md` L7 |
| 0.5 | **`Fixed` is not closed** | Backlogs list as "open" bugs that were fixed and released long ago. | In the SE module's `module.yaml`, BUG's closed states are only `Closed` and `WontFix`. `Fixed` means "fix landed, not yet verified". | Verify each fix, then `catalyst status set <BUG> Closed` (Catalyst Studio 0.73.0 adds Close Bug in the editor), then regenerate the backlog. Never hand-edit `BACKLOG.md` (L6). | cantica-tech/catalyst-software-engineering:module.yaml |
| 0.6 | **Every signed command needs the identity provider** | Commands fail with `IamNotLoggedInError`, or sign as the **system account**. | Since kernel 0.106.1 deployments sign via catalyst-users → a Keycloak (OIDC). With nobody logged in, the system account signs; since 0.120.0 `catalyst hook start` says so when a session starts. | Run `docker compose up -d` in catalyst-users, then `catalyst identity login` (or `--password`). ~~catalyst keeps its own login, separate from `catalyst-users login`~~ — since 0.120.0 one login serves both when they name the same provider and login client. The dev realm keeps a login 12 h idle / 7 days at most since 2026-11-01 (it was 30 min / 10 h). | cantica-tech/catalyst:CHANGELOG 0.120.0; cantica-tech/catalyst-users:keycloak/realm-catalyst.json |
| 0.7 | **Law L3: nothing leaves the machine without a yes** | A catalyst command exits `3` and prints a preview. | Publishing verbs (`share push`, PRs, sync) preview their effect and refuse without `--yes` (INV-4); since 0.120.0 so do `criterion protect` and `hook install`. Exit `3` is distinct from `4` (no deployment, since 0.121.0) and `2` (usage). | Show the preview to the person; re-run with `--yes` only after explicit assent. | kernel `INVARIANTS.md` L3; docs/roadmap.md R3.6 |
| 0.8 | **Never hand-edit generated files** (L6) | `check` fails after editing `BACKLOG.md`, indexes, composed `CODE-OF-CONDUCT.md`, back-references or the criterion's `Taskfile.common.yml`. | They are regenerated by their commands; deployed definitions change only by migration. | Use the generating verb (`/show-backlog`, `catalyst sync apply`, `catalyst link/unlink`). | kernel L6 |
| 0.9 | **Hand-made git changes are detected** | `check` and the commit-msg hook warn "unrecorded change". At format 1.0 this becomes an error. | Built in 0.42.0. `unrecorded.level` warns while `catalyst.toml` says `format = "1.0-rc"` and errors at `1.0` or with `strict_journal`. `check` reads the declared format since 0.95.0. | Adopt the change (`catalyst journal adopt` / `/adopt`) or revert it. Declaring format 1.0 happens at the 1.0 release. | cantica-tech/catalyst:docs/roadmap.md R7 |
| 0.10 | ~~**Ruff version skew**~~ (fixed 2026-10-31) | CI's lint fails on something local ruff accepted, or format differs. | The cantica-tech/standards reusable `lint.yml@v0.1.2` runs `uvx ruff@0.16.10`. ~~catalyst-users and catalyst-kb-engine resolved `ruff>=0.7` to 0.17.2.~~ Since 2026-10-31 catalyst-mcp, catalyst-users and catalyst-kb-engine all pin `ruff==0.16.10`. | Before pushing a Python change, run `uvx ruff@0.16.10 check --no-cache .` **and** `uvx ruff@0.16.10 format --check --no-cache .` repo-wide (or `task standards:lint`). | cantica-tech/standards:Taskfile.standards.yml; each repo's pyproject.toml and uv.lock |
| 0.11 | **`task lint` needs a sibling standards checkout** | ~~Local lint passes while CI's Standards job fails.~~ Since 2026-10-31 `task lint` (kernel and UI) stops and says the standards checkout is missing. | The Taskfile include of `../standards/Taskfile.standards.yml` is `optional: true` (`STANDARDS_DIR` defaults to `../standards`). | Clone cantica-tech/standards beside the repo. | cantica-tech/catalyst:Taskfile.yml; cantica-tech/catalyst-ui:Taskfile.yml |
| 0.12 | **Synced `.standards/` copy older than CI's** | ~~catalyst's `.standards/VERSION` says `v0.1.1`~~ (v0.1.2 since kernel 0.120.0). Still `v0.1.1` in catalyst-ui and the SE module, while CI calls `lint.yml@v0.1.2`. | The standards sync was not re-run there after 0.1.2. The Standards job checks that the copy is "in sync", so expect a failure or a no-op depending on what changed. | Re-sync `.standards/` from cantica-tech/standards v0.1.2. | each repo's .standards/VERSION; .github/workflows/standards.yml |
| 0.13 | **Uneven CI across the fleet** | ~~Some repos have no CI at all.~~ Since 2026-10-31 every public repository has CI. | Reusable standards workflow plus tests: kernel, UI. Own `ci.yml` with ruff, pyright and pytest: catalyst-mcp, catalyst-kb-engine. catalyst-users: `ci.yml` (ruff, pytest, integration tests) plus the standards workflow with Pyright (0.11.0). SE module: standards lint only, no tests. KBM: standards plus an install against the latest kernel. catalyst-docs: Pages plus the standards lint. | See next steps ([90](90-next-steps.md)). | each repo's `.github/workflows` |
| 0.14 | **The kernel names no module** (law L9, INV-30) | The Deployment Rules CI job fails on a word in a changelog, a comment or a script. | `scripts/check_kernel_purity.py` derives a denylist from every catalogued module's manifest (ids, entity prefixes such as `BUG`/`REQ`, folders, command names) and scans `framework/kernel`, `scripts` and **root `*.md`/`*.txt` — including `CHANGELOG.md`**. | Run `python3 scripts/check_kernel_purity.py` after editing CHANGELOG or anything kernel-side, **before** pushing a release bump. | cantica-tech/catalyst:scripts/check_kernel_purity.py; commits `c594f9d` (0.55.0), `293cfc6`/`0aa71d1` (0.52.x) |

The branch models of 0.1, side by side:

```mermaid
flowchart LR
  subgraph three["development, main, release: kernel, catalyst-ui, catalyst-users"]
    d1["development"] -->|"PR"| m1["main"] -->|"PR"| r1["release"] --> t1(["tag"])
  end
  subgraph tagmain["tags on main: SE module, KBM, catalyst-mcp, catalyst-kb-engine"]
    d2["development"] -->|"PR"| m2["main"] --> t2(["tag"])
  end
  subgraph mainonly["main only: distribution repo"]
    m3["main"]
  end
```

The bug lifecycle behind 0.5. The SE module declares no transitions, so these are the usual walks, not enforced ones; only `Closed` and `WontFix` count as closed:

```mermaid
stateDiagram-v2
    state "Under Review" as UnderReview
    [*] --> Open
    Open --> UnderReview
    UnderReview --> Fixed
    Open --> Fixed
    Fixed --> Closed : verified
    Open --> WontFix
    UnderReview --> WontFix
    Closed --> [*]
    WontFix --> [*]
```

Signing (0.6): every signed verb asks catalyst-users who is logged in, and catalyst-users asks the Keycloak.

```mermaid
sequenceDiagram
    participant A as Agent or person
    participant C as catalyst CLI
    participant U as catalyst-users
    participant K as Keycloak
    A->>C: a signed verb (new, status set, journal adopt)
    C->>U: who is logged in on this machine?
    U->>K: OIDC token and live membership check
    alt someone is logged in
        K-->>U: verified identity
        U-->>C: issuer and subject
        C-->>A: change journaled, signed by that person
    else nobody logged in
        U-->>C: no session
        C-->>A: IamNotLoggedInError, or the system account signs
        Note over C: since 0.120.0 hook start says so at session start
    end
```

The assent rule of 0.7 (law L3) as a decision:

```mermaid
flowchart TD
    v["Publishing verb: share push, PR, sync"] --> p["Prints a preview, exits 3"]
    p --> q{"Did the person say yes?"}
    q -->|"yes"| y["Re-run with --yes"] --> out["The change leaves the machine"]
    q -->|"no"| n["Nothing leaves the machine"]
```

How a hand-made change is treated (0.9):

```mermaid
flowchart TD
    h["Commit made outside catalyst"] --> d["check or the commit-msg hook finds an unrecorded change"]
    d --> f{"Declared format"}
    f -->|"1.0-rc"| w["Warning"]
    f -->|"1.0, or strict_journal"| e["Error"]
    w --> r["Adopt it with catalyst journal adopt, or revert it"]
    e --> r
```

What the purity scan of 0.14 reads and where it looks:

```mermaid
flowchart LR
    cat["framework/modules/catalog.md"] --> man["Each catalogued module's manifest"]
    man --> deny["Denylist: module ids, entity prefixes, folders, command names"]
    deny --> scan{"check_kernel_purity.py"}
    k["framework/kernel"] --> scan
    s["scripts"] --> scan
    r["Root .md and .txt, CHANGELOG.md included"] --> scan
    scan -->|"a hit"| fail["Deployment Rules job fails"]
    scan -->|"clean"| ok["Job passes"]
```

---

## 1. Kernel — cantica-tech/catalyst

### 1.1 Release and CI

| # | Oddity | Symptom | Cause | Fix / workaround | Source |
|---|---|---|---|---|---|
| 1.1.1 | **Purity scan broke two release PRs** | The 0.52.x and 0.55.0 release PRs failed Deployment Rules. | Their CHANGELOG entries named the SE module or its entity types. The check now denies a plain-word module command only as `/<name>`. | See 0.14. Release notes in the module repo may name the pairing; the kernel CHANGELOG may not. | commits `c594f9d`, `293cfc6`, `17a0e98` |
| 1.1.2 | **Deployment Rules CI builds scratch deployments from the module catalog** | CI fails when a catalogued module's `main` is broken. | `framework/modules/catalog.md` lists `software-engineering` and `knowledge-base` on branch `main`. CI clones each at `--depth=1`, runs `catalyst init` with `CATALYST_ALLOW_STATIC_IDENTITY=1` (the static identity backend verifies nobody; tests only), then runs `check_deployment.py --require`, `check_command_parity.py --require` and `catalyst check`. | A module's `main` must always pass against the kernel's development branch. Release the kernel first when a module needs a newer kernel. | .github/workflows/deployment-rules.yml |
| 1.1.3 | **Trace gate and force pushes** | After a force push, the previous tip is gone and the commit range cannot be computed. | Handled: when `before` is unknown, CI checks `origin/main..sha`. | Avoid force pushes on development anyway. | deployment-rules.yml; commit `35e4900` |
| 1.1.4 | **Every commit must cite an artifact or rule ID, or be `chore:`** | The trace job fails on a commit message such as `fix: typo`. | Law L2. `catalyst trace <range>` runs in CI; a commit-msg hook does the same locally. | Use `fix(<bug ID>):`, `feat(<requirement ID>):` or `chore:`. `chore:` must change no rule's behaviour. | deployment-rules.yml |
| 1.1.5 | **Old commit-msg hooks don't update themselves** | A machine keeps an old hook script. | A hook installed before 0.117.1 keeps its old script; no migration rewrites it. | Re-run `catalyst hook install`. | docs/results/2026-10-26-quickstart.md |
| 1.1.6 | **The commit-msg hook could fail open** | Commits passed untraced on a fresh machine. | Fixed in 0.117.1: the hook now runs the criterion's own CLI, found through `catalyst.toml`, when no `catalyst` launcher is installed. | Confirm `which catalyst` works; run `catalyst open`. | CHANGELOG 0.117.1 |
| 1.1.7 | **`package_release --publish-dir` could publish an unreleased kernel** | The distribution repo received a kernel that was never tagged. | Fixed in 0.116.1: it publishes only a tagged release. | Publish only from a tag; check the distribution repo's newest kernel after `task release:publish`. | CHANGELOG 0.116.1; docs/results/2026-10-26-catalyst-docs-phase-3.md |
| 1.1.8 | ~~**Second-module CI fixture pinned old**~~ (fixed in 0.120.0) | ~~`tests.yml` checks out cantica-tech/catalyst-knowledge-base at ref **0.4.1**~~; it is at 0.6.1 since 0.120.0. | Pinned when the fixture was added; bumped in 0.120.0. `CATALYST_REQUIRE_FIXTURES=1` makes the test fail instead of skipping when the fixture is absent. | Bump the ref when a kernel change needs newer KBM behaviour. Locally, set `CATALYST_KB_MODULE` or keep a sibling checkout, else the test skips. | .github/workflows/tests.yml; tests/test_second_module.py |
| 1.1.9 | **Test matrix is 3 OS × Python 3.11/3.13** | Windows-only failures. | Windows has been a 1.0 target since decision A4 (no symlinks, colons or absolute paths). Many fixes followed: git objects are read-only on Windows, delete-pending state, CRLF on git stdin, backslashes in JSON, the `serve` refused-request body. | Expect Windows-specific bugs. Use Python helpers, not `cat`, in tests (commit `35ef25a`). | commits `ea14027`, `8623403`, `3b38141`, `1894be6`, `f78012c`, `680a9f3`, `acb92b6` |
| 1.1.10 | **The criterion's Taskfile is composed, not the project's** | `task <cmd>` in the project does nothing catalyst-related. | `catalyst init` composes `Taskfile.common.yml` **inside the criterion** from the kernel template, one task per `CODE-OF-CONDUCT.md` §4 command (a module ships no task: `Taskfile.module.yml` has not been read since 0.70.0, SE removed it in 3.5.0, and `contributions.taskfile` is deprecated). `scripts/check_command_parity.py` keeps it 1:1 with §4. Since 0.120.0 each task names the command the way the agent serves it (`/catalyst:<name>` for Claude Code, `CMD_PREFIX`). | `catalyst task <name> -- <args>`; `catalyst task` lists them. | CLAUDE.md; scripts/check_command_parity.py; CHANGELOG 0.120.0 |
| 1.1.12 | **New: the release steps miss `SYNCHRONIZE.md`** | The 0.120.0 and 0.121.0 releases left a `SYNCHRONIZE.md` change unjournaled; a kernel commit (`0c761ce`) was adopted afterwards. | The generated `SYNCHRONIZE.md` (`gen_docs.py`, migrations section) changes during a release, outside a journaling verb. | Journal it as part of the release steps (next kernel round); until then `catalyst unrecorded` shows it and `journal adopt` records it. | the fix round of 2026-11-01 |
| 1.1.13 | **New: an unknown `agent run` command exited 1 on CI** | The first 0.121.0 CI failed on every runner. | `agent run` looked for the agent's CLI before checking the command, so with no `claude` on PATH an unknown command exited 1, not 2. | Fixed in `40a59f3`: the command is checked first. | commit `40a59f3` |
| 1.1.11 | **The criterion repository's name is not derived from the project's** | Guessing `<project>-criterion` as the remote can be wrong; the kernel's own uses a generic name. | Historical. | Read `catalyst_repo_url` in `catalyst.toml` rather than guessing. | catalyst.toml |

The Deployment Rules job of 1.1.2 and 1.1.4, step by step:

```mermaid
sequenceDiagram
    participant CI as Deployment Rules job
    participant Cat as Module catalog
    participant Mod as Module repo, main
    participant Cli as catalyst from the commit
    CI->>Cli: catalyst trace over the pushed range
    Cli-->>CI: every commit cites an ID or is chore
    CI->>Cat: read the catalogued modules
    loop each module
        CI->>Mod: git clone --depth=1
        CI->>Cli: init a scratch deployment, static identity
        CI->>Cli: check_deployment.py --require
        CI->>Cli: check_command_parity.py --require
        CI->>Cli: catalyst check
    end
```

### 1.2 Tests

| # | Oddity | Symptom | Cause | Fix / workaround | Source |
|---|---|---|---|---|---|
| 1.2.1 | **Concurrency tests are timing-sensitive** | `test_concurrent_appends_are_serialised` launches 8 processes that journal at once and asserts a whole, chained journal. It failed again on Windows with Python 3.11 during the 0.120.0 round (the second time): still open. | It exercises the journal lock, which on Windows must wait through the delete-pending state (commit `8623403`). Two other CI races were fixed by relaxing tests: helpers saw git repack files mid-walk, and a local event-stream test expected exactly the next batch. | Fixed: helpers don't walk `.git`, and the test expects "a later batch" (commits `51b8047`, `a258f8c`). If a concurrency test fails once, rerun it and look at the lock's retry window on Windows; the flake is on the next kernel round's list. | tests/test_catalyst_journal_shards.py; git log |
| 1.2.2 | **Conditional skips** | Tests skip silently on a dev machine. | Skip conditions: Python < 3.11 (`tomllib`); no sibling module checkout (token budgets, kernel schemas); no `uv` (runtime builds); `pycryptodome` missing (membership, via `importorskip`); Windows (module-install permission test); **golden corpus not captured** (always skipped in CI, because corpora are gitignored). | Install `requirements-dev.txt`; keep sibling checkouts `../catalyst-<module>`; install `uv`. | tests/test_*.py |
| 1.2.4 | **New: a POSIX split ate a Windows interpreter path** | The password-command test failed on Windows. | An unquoted interpreter path was split POSIX-style, losing its backslashes. | The kernel's test quotes the path (`5377ac2`). The same gap remains for Windows users of catalyst-users' password command (open in catalyst-users). | commit `5377ac2` |
| 1.2.3 | **The golden-corpus checks never run in CI** | Regressions against real deployments are only caught locally. | The corpora are copies of real criteria, so they are gitignored. | Run `scripts/capture_golden_corpus.py` locally before risky format changes. It refuses `.git` and `.ledger`. | tests/test_golden_corpus.py; scripts/capture_golden_corpus.py |

### 1.3 Identity, signing, journal

| # | Oddity | Symptom | Cause | Fix / workaround | Source |
|---|---|---|---|---|---|
| 1.3.1 | **`journal adopt` used to sign as the git author** | Adopted changes were recorded under whoever git said committed them. | Fixed in 0.119.1. Adopt now signs as the verified identity; the git author is kept only as `commit_author`. Earlier adopt entries stay as written (append-only). | Upgrade to ≥ 0.119.1; never use `--as` to impersonate. | CHANGELOG 0.119.1 |
| 1.3.2 | **`share push` could say "nothing to publish" while the criterion held changes** | Work was left unshared while the command exited 0. | Fixed in 0.119.2. Git read errors and uncommitted leftovers now fail loudly. | Upgrade to ≥ 0.119.2; on older kernels, check `git status` in the criterion after a push. | CHANGELOG 0.119.2 |
| 1.3.3 | **Adopt could chain an already-consumed commit** | Duplicate history in the journal. | Fixed in 0.118.1: such a commit is now recorded as `superseded` history only. | — | CHANGELOG 0.118.1 |
| 1.3.4 | **`init` refused when git `user.name` differed from the provider's name** | `catalyst init` failed on a machine with a nickname in git config. | Fixed in 0.117.1. The kernel's `init` takes `--git-username`. | — | CHANGELOG 0.117.1 |
| 1.3.5 | **Unknown `--command` names are kept forever** | Journal entries name commands the deployment does not define. | `journal append` only **warns** on an unknown command (0.88.0), and the journal is append-only. | Use the deployment's §4 names (`fix`, not `/fix-bug`). | docs/roadmap.md R4.8 |
| 1.3.6 | **Roles are enforced** | A writing verb refuses: "role does not grant command". | Decision D5 (0.93.0). A role grants commands named with a leading `/` in `actions`, or everything with `*`; tier names (`chore`/`fix`/`feature`) are not checked. A deactivated user signs nothing. | `role_check = "warn"` in `catalyst.toml` downgrades this to a warning. Fix the role in catalyst-users. | docs/decisions.md D5 |
| 1.3.7 | **Redaction weakens immutability on purpose** | A journal entry's content is a tombstone. | Decision A5: `catalyst journal redact` replaces a mistakenly journaled secret with an auditable tombstone, keeping the entry and its hash. Purging git history stays a separate, manual job. | — | docs/decisions.md A5 |
| 1.3.9 | **New: `identity status` can disagree with the signer** | `catalyst identity status` says logged in, yet entries are signed by the system account. | A lapsed login: `status` reads local files only (a tokens file exists), while signing verifies with the provider and falls back. | Log in again; the longer dev-realm sessions (12 h / 7 days) make it rarer. To be fixed in the next kernel round. | the fix rounds of 2026-10-31 and 2026-11-01 |
| 1.3.10 | **New, 0.120.0: one `identity.json` shape** | `check` warns that the pointer names its provider by `url` and `realm`; an old login is not found. | BUG-000057: the pointer names its provider by issuer; tokens are kept per canonical issuer (a loopback host is `localhost`). A token file kept under the old shape is not read. | `catalyst migrate --apply`, then log in once more. | CHANGELOG 0.120.0 |
| 1.3.11 | ~~**`share create` (git driver) signed as actor `catalyst`**~~ | Entries signed by a fixed actor. | Fixed in 0.120.0 (BUG-000049): it signs as the verified identity, after its `share` grant. Every writing verb also checks its own command's grant (BUG-000050). | Upgrade. | CHANGELOG 0.120.0 |
| 1.3.8 | **The Stop hook is session-scoped** | The Stop hook does not block on failures that existed before the session. | ADR-002: `hook start --quiet` records what the check already fails on; Stop blocks only on what the session broke. Every failure exits 2 (exit 1 is non-blocking in Claude Code — the "fail open" bug of R0.16). | Don't add catalyst hooks to a project's `.claude/settings.json`; `catalyst agent install claude-code` wires them at user level. | docs/adr/ADR-002-session-scoped-stop-hook.md; docs/roadmap.md R0.16 |

The session-scoped Stop hook of 1.3.8:

```mermaid
sequenceDiagram
    participant CC as Claude Code
    participant H as catalyst hook
    participant K as catalyst check
    CC->>H: session start, hook start --quiet
    H->>K: what fails already?
    K-->>H: baseline findings
    Note over CC: the session works
    CC->>H: Stop, hook stop
    H->>K: what fails now?
    K-->>H: current findings
    alt a finding the session added
        H-->>CC: exit 2, the stop is blocked
    else only baseline findings
        H-->>CC: exit 0
    end
```

### 1.4 Migrations, sync, modules

| # | Oddity | Symptom | Cause | Fix / workaround | Source |
|---|---|---|---|---|---|
| 1.4.1 | **Module migrations are keyed by kernel version** | A module release's migration never fires on sync. | A module migration runs only when the kernel version moves across it. Fixed in 0.117.2: the sync plan includes module migrations outside the kernel range, and `migrate --since` finds those keyed to older kernels. | Every module release pins a **minimum kernel** (SE 3.4.x needs ≥ 0.83.0; KBM needs ≥ 0.107.0). When in doubt, `catalyst migrate` lists what a deployment is behind on. | CHANGELOG 0.117.2 |
| 1.4.2 | **Format changes come as many small releases** | Nearly every minor kernel version may change the layout. | Decision B6: each format change ships with its own migration in `framework/kernel/migrations/`. | `/sync-framework` (`catalyst sync plan` → `apply`) per deployment. Read CHANGELOG "No layout change" lines. | docs/decisions.md B6; CHANGELOG preamble |
| 1.4.3 | **Migration 0.106.0 kept roles under slugged IDs** | Roles did not match the provider's role IDs. | Fixed in 0.106.1. | — | CHANGELOG 0.106.1 |
| 1.4.4 | **A module's own `catalyst.toml` must never ship** | A governed module repository could pass its project file to installing criteria. | Fixed in the kernel; KBM relies on "a module's own catalyst.toml never ships". | — | cantica-tech/catalyst-knowledge-base:README.md |
| 1.4.5 | **`module upgrade` from a deployment's own CLI crashed building from a checkout** | — | Fixed in 0.110.1. There is still no dry run of `module upgrade` (R6, open). | Run `catalyst try` around it. | CHANGELOG 0.110.1; docs/roadmap.md R6 |
| 1.4.6 | **Enum values `none`, `n/a`, `-` read as empty** | A required enum field failed `check`. | Fixed in 0.110.0 (YAML read `None` as a null). KBM had the same problem with a `None` classification value and renamed it `Local only`. | Avoid those literals as enum values. | CHANGELOG 0.110.0; cantica-tech/catalyst-knowledge-base:module.yaml |
| 1.4.7 | **`new` could create a file INV-7 rejects** | A title ending in a number gave an invalid file name. | Fixed in 0.68.0. | — | CHANGELOG 0.68.0 |
| 1.4.8 | **Table-row entities are second-class** | A row's `Linked` column is not read as links. `edit` and `history` don't handle rows. | Rows (roadmap items, features, reconciliation cases, analyses) live inside index or roadmap tables, not in their own files. Earlier gaps (`view` and `graph` leaving rows out, `new` and `link` refusing them) are fixed. | Edit rows through their module verbs (`/roadmap`). | docs/results/2026-10-26-view-table-rows.md; CHANGELOG 0.91.0 |
| 1.4.9 | **Legacy pointer / in-project criterion** | `check` warns about a `<name>.catalyst` pointer or a `.criterion/` folder. | Deprecation window (0.94.0): read-only legacy support, writes refused at format 1.0. | `catalyst migrate --apply`, or `move --to-home`. | docs/roadmap.md R3.4, R7 |
| 1.4.10 | ~~**`init` needs a catalyst checkout or release `.pyz`**~~ (fixed in 0.120.0) | ~~`catalyst init` cannot run from the `~/.catalyst/bin` launcher in a new repo.~~ | REQ-000106: every `catalyst.pyz` carries its `framework/kernel`, so `init` runs from the launcher with no `--kernel`. catalyst-mcp still needs `CATALYST_MCP_CATALYST_ROOT` (6.4). | Upgrade to ≥ 0.120.0. | CHANGELOG 0.120.0; cantica-tech/catalyst-mcp:README.md |
| 1.4.11 | **New, 0.121.0: module releases live only in the distribution repository** | A module repository no longer holds `catalyst/modules/…`; publishing refuses a module not at its tag. | REQ-000112: `package_release.py` stages modules in the kernel checkout's ignored `catalyst/` and pushes the distribution repository alone; a manifest `entry` only with a UI bundle. | `task release:publish PUBLISH_DIR=… -- --module <id> --no-kernel` from a module checkout at its tag. | CHANGELOG 0.121.0 |
| 1.4.12 | **New, 0.121.0: "no deployment" exits 4** | A script that tested exit `2` for "not in a project" now sees `4`. | REQ-000110, `fw-STRUCTURE-000025`: `2` is a usage error only. `check` and the hooks still exit `0` outside a project. | Test `4`. No caller in the fleet tested `2` for it. | CHANGELOG 0.121.0 |

Why a module migration could be skipped (1.4.1): sync walks kernel versions, and a module migration fires only when that walk crosses the kernel version it is keyed to.

```mermaid
flowchart LR
    from["Deployment at kernel A"] --> walk["sync plan walks kernel A to B"]
    walk --> q{"Module migration keyed inside A to B?"}
    q -->|"yes"| run["It runs"]
    q -->|"no, before 0.117.2"| skip["It was skipped"]
    q -->|"no, since 0.117.2"| run2["Included in the plan"]
```

### 1.5 Documentation drift in the kernel repo

| # | Oddity | Symptom | Cause | Fix / workaround | Source |
|---|---|---|---|---|---|
| 1.5.1 | ~~**`docs/adr/` statuses are not authoritative**~~ (fixed in 0.120.0) | ~~ADR-003…007, 009, 011…016 said "Pending".~~ | Since 0.120.0 the `docs/adr` statuses follow the criterion's decisions. | The deployment's record stays authoritative (`catalyst view <ADR>`). | CHANGELOG 0.120.0 |
| 1.5.2 | ~~**`docs/adr/PARKING.md` reuses taken numbers**~~ | It "anticipates" ADR-015…032 with unrelated topics. | Marked historical in 0.120.0. | Read it as history. | CHANGELOG 0.120.0 |
| 1.5.3 | ~~**The R6 table contradicts itself**~~ | — | The R6 row was re-derived in 0.120.0. | — | CHANGELOG 0.120.0 |
| 1.5.4 | ~~**The feature register was last refreshed 2026-10-16**~~ | — | Re-derived in 0.120.0. | — | CHANGELOG 0.120.0 |
| 1.5.5 | ~~**`beta/TRIAL.md` may use old verbs**~~ | — | Since 0.120.0 it uses the current verbs. | — | CHANGELOG 0.120.0 |
| 1.5.6 | **The quickstart is heavy** | A newcomer needs Python 3.11+, uv, git, **Docker + a throwaway Keycloak 26.4.0**, and three checkouts on `development`, about 25 min. | Identity is mandatory since 0.106. One step stays manual: approving `catalyst-users login` in a browser. | A sandboxed `CATALYST_HOME` keeps the machine clean. | beta/QUICKSTART.md; docs/results/2026-10-26-quickstart.md |

### 1.6 Historical traps (kernel)

| # | Trap | What happened | Lesson / guard | Source |
|---|---|---|---|---|
| 1.6.1 | **The Stop hook failed open** | Any exception exited 1, which Claude Code treats as non-blocking; one corrupt journal byte disabled enforcement. | All failures now map to exit 2 (R0.16). | docs/roadmap.md R0.16 |
| 1.6.2 | **Self-governance suspended, then restored** | Journalling of catalyst's own refactor was off from 2026-10-07 to 2026-10-10 (decision D1). | Journal entries from that window are absent; every catalyst change is journaled since. | docs/decisions.md D1 |
| 1.6.3 | **The plugins era** | The `plugins/` submodule tree and the `catalyst-git` submodule are gone (0.72.0, ADR-022). catalyst-git is an optional extension. | Old notes mentioning catalyzers or plugins are history. | docs/adr/ADR-022-remove-the-plugin-system.md; docs/decisions.md B3 |
| 1.6.4 | **Risk-management module dropped** | It was the second module from 2026-10-17 to 10-22, then replaced by KBM. Its repository and 0.1.0 release are kept but not catalogued or tested. | Don't add it back to the catalog without a new decision. | docs/decisions.md B1 |
| 1.6.5 | **`~/.catalyst-users` legacy home** | Kernels ≤ 0.110.1 read `~/.catalyst-users`. catalyst-users 0.6.0 moved its home to `~/.catalyst/users` and left a symlink. | Remove the link only once no machine runs a kernel ≤ 0.110.1. | docs/results/2026-10-26-deployments-0117-ui-0651.md, 2026-10-27-kb-azure-devops-local-checkouts.md |

### 1.7 Deliberate design choices that look odd

| Choice | Rationale | Source |
|---|---|---|
| The criterion is in `$HOME`, not in the repo | Keeps host repositories clean (only `catalyst.toml`); one runtime per criterion; shared via git (`share`), or via `catalyst serve`. | design 13; decisions A1 |
| Agents never read `module.yaml` whole; prose-size targets retired | The kernel reads it and agents see generated `catalyst spec` output. Cutting it to 15 KB would have forced migrations for 8 types in every deployment. | decisions D6 |
| Reference docs generated from code, between markers | Arguments, schemas, keys, migrations and enforcement come from code and are checked in CI; the "why" stays prose. | decisions D7; roadmap R4.4 |
| Rules may be headings **or** their own files | Both shapes are allowed; ADR-001 is superseded. | decisions B7 |
| Four-eyes analysis is configurable | Two blind passes plus a person's decision by default; `analysis_passes = 1` allowed, and recorded as single-pass. | decisions B5 |
| Approval gates are opt-in, in `IAM/gates.json` | No gate unless declared; the author's approval doesn't count; `--force` doesn't pass a gate. | decisions G; design 25 |
| Proposals are reconciliation cases of kind `proposal` | They reuse the reconciliation verbs; `accept` is refused when the base moved, and for the proposer. | decisions P; design 26 |
| `catalyst try` never applies | It runs the real verb on a scratch worktree and reports the findings it adds and clears; there is no `--apply`. | decisions T; design 27 |
| Rules have a strength (`must` by default) | No existing rule loosens; a breach is reported as a violation, warning or note. | decisions S |
| People live outside catalyst (catalyst-users) | catalyst is a facade over any IAM; `IAM/identity.json` points at it; an append-only signers ledger is keyed by issuer and subject. | decisions I; designs 28/29 |
| No `serve` route writes or starts an agent run | Writes and runs go through a UI's host after the person confirms; `serve` only reads, from origins a deployment allows. | docs/roadmap.md R6; decisions B8 |
| The plugin system was removed | It was unused complexity; extensions act on deployments, never on catalyst. | ADR-022; decisions B3 |
| Meta-tags were removed for a `tags:` field (not built yet) | No uses. | decisions B4 |
| Windows is a 1.0 target | Decision A4. | decisions A4 |

---

## 2. SE module — cantica-tech/catalyst-software-engineering

| # | Oddity | Symptom | Cause | Fix / workaround | Source |
|---|---|---|---|---|---|
| 2.1 | **No criterion of its own** | You can't find SE's requirements anywhere in its own repo. | Its changes are tracked as work items in catalyst's own record. | Commit messages cite them. | git log |
| 2.2 | ~~**Release zips committed in the module repo**~~ (removed in 3.5.0) | ~~34 committed zips under `catalyst/modules/…`.~~ | An old release flow wrote distribution-layout output into the repo; since kernel 0.121.0 the packager writes nothing there. | The distribution repo is the only home. | commit `85beef4` |
| 2.3 | ~~**`ROADMAP.md` is a stale "Draft/Proposed" plan**~~ | — | Marked historical on 2026-10-31. | Read it as history. | ROADMAP.md |
| 2.4 | **Aliases are load-bearing** | The 3.1.0 regression broke `/create-*` and <code>/roadmap-add\&#124;update\&#124;merge\&#124;remove</code>; fixed in 3.1.1. 3.1.0 is not in the distribution repo. | ADR-020/021 keep old names as aliases. | Test aliases on every release. | commit `23cea3c`; cantica-tech/catalyst:docs/adr/ADR-020, ADR-021 |
| 2.5 | **Minimum-kernel pins** | 3.0.0 needs ≥ 0.57.0; 3.3.0 to 3.6.0 need ≥ 0.83.0. | See 1.4.1. | — | module.yaml |
| 2.6 | **CI is lint only** | No tests run on the module itself. | It is exercised by the kernel's Deployment Rules job instead (1.1.2). | — | .github/workflows/standards.yml |
| 2.8 | ~~**`REQ.Feature`'s back-reference was named `Requirements`**~~ (fixed in 3.5.0) | The feature's field is `Requirement(s)`. | Fixed in 3.5.0. | — | commit `cc6872a` |
| 2.9 | **New, 3.6.0: a roadmap row is Done once every linked item is closed** | A row linking only features turns `Done` when they close; before, only linked requirements could make it `Done`. | `/show-backlog`'s rule. | Run `/show-backlog` after closing features. | commit `e43c912` |
| 2.10 | ~~**`Taskfile.module.yml` shipped though never read**~~ (removed in 3.5.0) | — | See 1.1.10. | — | commit `5865ab2` |
| 2.7 | **Closed states are narrow** | BUG: `Closed`/`WontFix`; REQ: `Completed`/`Abandoned`; STEP: `done`/`abandoned`; TEST: `Passing`. A BUG reaches Fixed or Closed only with its steps closed; a REQ completes only with ≥ 1 step, all closed (law SE-L1). | — | — | module.yaml |

## 3. KBM — cantica-tech/catalyst-knowledge-base

| # | Oddity | Symptom | Cause | Fix / workaround | Source |
|---|---|---|---|---|---|
| 3.1 | ~~**main only**~~ (since 2026-11-01: development → main) | `development` is the default branch; CI runs on it; `main` takes it by pull request. | — | Tag on main. | branches |
| 3.2 | **Needs kernel ≥ 0.107.0** | Older kernels refuse the module. | It relies on guards with field conditions, sources falling due, gated builds, and a module's own `catalyst.toml` never shipping. It is the CI proof that the kernel names no module type. | — | README.md |
| 3.3 | **Old module id `cantica-knowledge-base`** | 0.1.0 was published under another id (renamed in 0.2.0). ~~Legacy zips are under `catalyst/modules/`.~~ Removed on 2026-11-01 with the module's other zips; `catalyst/` is git-ignored. | Rename in B1's revision. | — | cantica-tech/catalyst:docs/decisions.md B1; commits `4013032`, `b16c780` |
| 3.4 | **`None` as an enum value** | A `None` hosted-embedding classification failed check. | See 1.4.6; the value is now `Local only`. | — | module.yaml |
| 3.5 | ~~**Stale design text**~~ | `docs/engine-and-onboarding.md` §6 listed "open questions" already answered. | Since 2026-10-31 the engine design records what was decided, and `GUIDE.md` says the engine is released. | — | commit `f37f581` |
| 3.6 | ~~**No CI, no `.standards/`**~~ | — | Since 2026-10-31: `standards.yml` and `install.yml` (installs the module against the latest kernel), `.standards/` at 0.1.2. | — | .github/workflows |
| 3.7 | **Designed to run alone or beside any module** | A KB can be built off any project, including an SE-governed one. | Law L9 plus the second-module decision. | — | cantica-tech/catalyst:docs/decisions.md B1 |

## 4. catalyst-ui — cantica-tech/catalyst-ui

| # | Oddity | Symptom | Cause | Fix / workaround | Source |
|---|---|---|---|---|---|
| 4.1 | **Studio must import catalyst-core only via browser-safe subpaths** | The browser bundle pulls in Node modules. | catalyst-core's root export `"."` is not browser-safe; `./graph-artifact` is the only safe runtime subpath today. Type-only imports from the root are fine. | — | packages/catalyst-core/package.json `exports` |
| 4.2 | **CI quirk: "No test files found" in catalyst-host-vscode** | `mocha "dist/test/**/*.test.js"` once found nothing on GitHub Actions, though it passed locally. | Suspected incremental `tsc --build` cache or mtime differences between macOS and Linux; never confirmed. | The `build` script is `tsc --build --force` (host-vscode and host-electron), and `pretest` runs `build`. Investigate if it recurs. | packages/catalyst-host-vscode/package.json |
| 4.3 | **`npm run lint` builds first** | Lint is slow and writes `dist/`. | Type-aware ESLint needs each package's declarations; every package is composite, so `typecheck` equals `build`. | — | package.json; commit `a016f17` |
| 4.4 | ~~**ESLint bulk suppressions**~~ (gone in 0.71.0) | ~~39 suppressions across 16 files.~~ | Fixed in code, behaviour unchanged; `eslint-suppressions.json` removed. | — | commits `0dd215b`, `c24d8ed`, `2122036` |
| 4.5 | **The Taskfile says it mirrors `ci.yml` "exactly", but `task lint` also runs `standards:lint`** | — | The standards checks are a separate `standards.yml` (reusable `lint.yml@v0.1.2`, `typecheck: false`). | See 0.11. | .github/workflows/standards.yml; Taskfile.yml |
| 4.6 | ~~**Node version mismatch**~~ (fixed in 0.70.0) | — | Node 24 everywhere: `.nvmrc`, `engines`, CI, Desktop. | — | commit `51bff70` |
| 4.7 | **`catalyst-trace` job needs a deploy key** | Every PR fails if the `CRITERION_DEPLOY_KEY` secret is missing or revoked. | The job clones the private criterion (URL and branch from `catalyst.toml`) and runs its `bin/catalyst.pyz trace`. | Keep the deploy key valid. | .github/workflows/ci.yml |
| 4.8 | ~~**CI on push only to development**~~ (fixed in 0.70.0) | — | CI and Standards also run on a push to `main` and `release`. | — | commit `52754ab` |
| 4.9 | **Marketplace publishing is separate from the release cycle** | — | `task publish:vscode-extension` (→ `vsce publish`) is its own task. `…:release` refuses unless HEAD is exactly the release tag, then copies the `.vsix` into the distribution repo's `catalyst/vsix/v<ver>/` with a `manifest.json` (`kernelVersion >= 0.54.0`). | Run `task release:vsix:publish` for every UI release tag (step 5 of `task release:checklist`, 0.71.0); it builds in a temporary worktree, then commits and pushes the distribution repo. The Marketplace stopped at 0.66.0 (a 0.69.0 publish was refused as "already exists"); a publish is the owner's call. | Taskfile.yml |
| 4.10 | ~~**The unsigned install task persists a global setting**~~ (fixed in 0.71.0) | — | It sets `extensions.verifySignature: false` for the install only, then restores the previous value. | — | commit `67d36a2` |
| 4.11 | **Desktop app gotchas** | Unsigned on every OS (needs an Apple Developer ID plus notarization, and Azure Artifact Signing). No auto-update (`publish: null`; Azure Blob vs GitHub Releases undecided). The bundled `catalyst.pyz` comes from `catalyst-version.txt` and needs Python ≥ 3.11 on PATH (macOS ships 3.9). The `.deb` maintainer has no address. | Signing keys and the update feed are not chosen yet. | Bump `catalyst-version.txt` only **after** that kernel is in the distribution repo. In a VS Code terminal, `unset ELECTRON_RUN_AS_NODE`, or the app runs as plain Node. | docs/desktop.md; .github/workflows/desktop.yml |
| 4.12 | ~~**Stale docs**~~ (refreshed 2026-10-31) | — | The READMEs, `desktop.md` and `catalyst-modules.md` were refreshed against the code. | — | commit `000e005` |
| 4.17 | **New: `VERIFIED_KERNEL_VERSION` is the paired kernel** | A test fails while it is older than the desktop app's `catalyst-version.txt`. | Since 0.70.0 (BUG-000047) the kernel-sync offer depends on it, and (BUG-000048) a UI module with no deployment open is checked against it. | Raise it with every paired kernel (`task release:checklist` step 1). | packages/catalyst-core/src/kernel-version.ts |
| 4.18 | **New: Close Bug / Reopen Bug greyed out** | The context-menu entries are disabled. | 0.73.0: they need the `/status` grant in that criterion, and a trusted workspace. | Grant `/status` in catalyst-users. | packages/catalyst-host-vscode/package.json |
| 4.19 | **New: `configureCriterion` is titled "Set Criterion Repo"** | The command ID and its title differ. | Renamed in 0.72.0, with an upload-repo icon. Keeping or renaming the ID is an open owner question. | — | packages/catalyst-host-vscode/package.json |
| 4.13 | **One IAM test can fail on a long-lived dev realm** | `test_provisioned_grants_reach_the_token` fails locally. | The realm's `developer` role holds old extra grants; CI uses a fresh realm. | In catalyst-users: `rm -rf .keycloack/data` (sic), then `catalyst-users setup --force`. | docs/results/2026-10-23-iam-settings.md |
| 4.14 | **Test fixtures are excluded from catalyst** | — | `packages/catalyst-core/src/test/fixtures` is a sample criterion; `.catalystignore` excludes it so neither catalyst nor a KB built off the repo reads it. | — | .catalystignore |
| 4.15 | **Deliberate UI design choices** | — | **W1:** no HTTP write route; only the host runs verbs, after confirmation. **C2:** agent runs are spawned by the VS Code host. **C6/D3:** a module is data only; R6.29 (sandboxed module UI) is disabled, not dropped. **S7:** payload types are generated from catalyst `spec/`, vendored at one kernel release (`npm run spec:sync -w catalyst-core`); a field an older kernel lacks is optional. **S6:** axe-clean on every Studio page. **S4:** `elkjs` does layout only. **C4:** Apache-2.0 from 0.47.0 (proprietary before). **R6.1:** the proposal loop, composer and run monitor were removed because they wrote files outside the journal. | — | docs/decisions.md; docs/design/09-ui-brand-portability.md |
| 4.16 | **Naming risk** | "Catalyst UI" is Tailwind's; npm `catalyst-ui` is taken; another "Catalyst Studio" is on the Marketplace. | No USPTO or EUIPO search done yet. | — | docs/decisions.md C3; docs/design/25-brand-spike.md |

## 5. catalyst-users — cantica-tech/catalyst-users

| # | Oddity | Symptom | Cause | Fix / workaround | Source |
|---|---|---|---|---|---|
| 5.1 | **`.keycloack` is misspelled, and load-bearing** | — | The data dir is `.keycloack/data` in `compose.yaml` and CI. | Never "fix" one side only. | compose.yaml; .github/workflows/ci.yml |
| 5.2 | **Keycloak's H2 database could not be written in CI** | Fixed. | Docker created the mounted dir owned by root. | CI runs `mkdir -p .keycloack/data && chmod 777` before `docker compose up`. | .github/workflows/ci.yml |
| 5.3 | **Integration tests skip by default** | — | They need `CATALYST_USERS_IT=1` plus `docker compose up -d`. Onboarding also needs Docker plus a catalyst checkout (`CATALYST_CHECKOUT`, else `../catalyst`). After the bootstrap admin is replaced: `CATALYST_USERS_IT_ADMIN` and `CATALYST_USERS_IT_ADMIN_PASSWORD_COMMAND`. | — | tests/integration/conftest.py |
| 5.4 | **Trace skipped on fork PRs** | — | The deploy key is absent on forks, so only a notice is printed. | — | .github/workflows/ci.yml |
| 5.5 | **The realm is always imported** | The compose Keycloak is never greenfield (`--import-realm`). The dev realm `keycloak/realm-catalyst.json` holds no person and no secret. | — | Greenfield trials use a separate Keycloak on **port 8081**. | README.md; docs/onboarding.md |
| 5.6 | **System account uses the `catalyst-admin-cli` client** | — | Keycloak's own `admin-cli` checks the password but carries no groups, roles or grants. The `static` provider is test-only and refused unless `CATALYST_ALLOW_STATIC_IDENTITY=1`. | — | README.md |
| 5.7 | **Home moved to `~/.catalyst/users` (0.6.0)** | — | A legacy `~/.catalyst-users` is moved once, with a symlink left behind. `CATALYST_USERS_HOME` or `CATALYST_HOME` override it. | See 1.6.5. | CHANGELOG |
| 5.8 | **Identity rules** | — | Users are disabled, never deleted. Usernames are immutable (0.8.0). A changed email becomes unverified. `is_member` asks the provider live, never token claims. A role reached only through a composite role appears in neither `users roles` nor `roles holders` (0.10.0). | — | CHANGELOG |
| 5.9 | **Private criteria cannot be branch-protected on the free plan** | A direct push to a `criterion` branch is caught only after the fact (`catalyst check`, `criterion integrity`). | GitHub's free plan refuses branch protection and rulesets on private repositories (HTTP 403). | Revisit on a paid plan or when a second writer joins. | cantica-tech/catalyst:docs/results/2026-10-23-catalyst-users.md, 2026-10-26-landing-phase-2.md |
| 5.10 | **Only repo besides kernel and UI with a `release` branch** | — | — | — | branches |
| 5.11 | ~~**Stale design doc**~~ | — | `docs/development/python-identity-libraries.md` §10 and §11 marked decided in 0.11.0. | — | CHANGELOG 0.11.0 |
| 5.13 | **New, 0.11.0: loopback spellings are one provider** | A token or system account at `127.0.0.1` matched settings at `localhost` only from 0.11.0. | `config.canonical_issuer`/`same_issuer` (`localhost`, `127.0.0.1`, `::1`); calls keep the URL they were given, since Keycloak refuses a token at another host. | — | CHANGELOG 0.11.0 |
| 5.14 | **New, 0.11.0: `setup` exits 1 when the provider is down** | Scripts that relied on exit 0 now stop. | The settings are still written, unchecked. | `setup --force` once the provider is up. | CHANGELOG 0.11.0 |
| 5.15 | **New, 0.11.0: global options anywhere; overrides keep the system account** | `catalyst-users whoami --json` works; `CATALYST_USERS_SERVER=http://127.0.0.1:8080 catalyst-users token --system` keeps the system account. | BUG-000049, BUG-000050. | — | CHANGELOG 0.11.0 |
| 5.16 | **New, 0.11.0: no test client in the dev realm** | `catalyst-tests` is gone from `realm-catalyst.json`. | The integration tests add it from `keycloak/realm-catalyst-tests.json` to the realm they run against; the live client was deleted. `set_admin.py` keeps other admins unless `--sole-admin`. | — | CHANGELOG 0.11.0 |
| 5.17 | **New: the password command on Windows** | A Windows interpreter path in `password_command` loses its backslashes. | A POSIX-style split (see 1.2.4). | Quote the path; a fix is pending. | the fix round of 2026-10-31 |
| 5.12 | **The stdlib core is vendored into the kernel** | Kernel identity code mirrors catalyst-users' standard-library core. | Decision I and `scripts/vendor_identity.py`. | Change it in catalyst-users first, then re-vendor. | cantica-tech/catalyst:scripts/vendor_identity.py; docs/decisions.md I |

## 6. catalyst-mcp — cantica-tech/catalyst-mcp

| # | Oddity | Symptom | Cause | Fix / workaround | Source |
|---|---|---|---|---|---|
| 6.1 | **Read-only over HTTP** | Writes and people-reads are refused over streamable HTTP. | catalyst would sign as the server, not the caller. | Use stdio. | docs/design.md |
| 6.2 | **Deny by default** | — | `catalyst-admin` creates criteria and manages people. A criterion's admins, asked of the IdP at decision time, update and watch it. A watch acts as its starter; if that identity is unresolvable at trigger time, the action is denied. The git author never authorizes anything. | — | docs/design.md |
| 6.3 | **Watches** | — | They adopt only unpushed local commits; saved files write nothing. Watches die with the server and never push. | — | docs/design.md |
| 6.4 | **Needs a catalyst checkout** | `catalyst init` fails through the launcher. | See 1.4.10. `CATALYST_MCP_CATALYST_ROOT` is required, with the module beside it as `catalyst-<module>`. | — | README.md |
| 6.5 | **Repeated create said "group made"** | Fixed in 0.2.0. | catalyst-users never deletes a group; the group step is last and idempotent. | — | CHANGELOG |
| 6.6 | **Members refused for criteria not served by `catalyst serve`** | — | catalyst records members only there (design 32). | — | docs/design.md §Implementation 4 |
| 6.7 | **Tests** | — | The Keycloak testcontainer needs Docker (marker `integration`); live catalyst tests need `CATALYST_MCP_IT_CATALYST`; one watch test is skipped on Windows. `ruff==0.16.10` pinned (see 0.10). Since 0.4.1 the watch's integration test adopts as the identity resolved over stdio. | — | pyproject.toml; tests |
| 6.8 | ~~**Stale lines in `docs/design.md`**~~ (fixed in 0.4.1) | — | Steps 2 and 5 and D5 corrected. | — | CHANGELOG 0.4.1 |
| 6.9 | **New: pins catalyst-users 0.10.0** | 0.11.0's fixes (loopback issuers, overrides keeping the system account) are not in catalyst-mcp yet. | 0.4.1 moved from 0.8.0 to 0.10.0. | Move to 0.11.0 (pending). | pyproject.toml |

## 7. catalyst-kb-engine — cantica-tech/catalyst-kb-engine

| # | Oddity | Symptom | Cause | Fix / workaround | Source |
|---|---|---|---|---|---|
| 7.1 | **Ollama had no default address** | Search, MCP, `serve` and `build-vectors` failed with an `ollama:` model. | Pydantic AI's Ollama provider has no default base URL. | Since 0.14.1 `OLLAMA_BASE_URL` defaults to `http://localhost:11434/v1`. | README.md; src/catalyst_kb_engine/build/vectors.py |
| 7.2 | **Token host must match** (eased in 0.16.1) | Token rejected. | Keycloak builds its issuer from the host. ~~`serve` and `token --system` must both use `localhost` (or both `127.0.0.1`).~~ Since 0.16.1 (the engine's BUG-000047) a loopback provider accepts a token got at either loopback name. Give `CATALYST_USERS_SERVER` to `serve` only: with the engine's catalyst-users 0.10.0, set for `token --system` it overrides the settings and drops the system account (0.11.0 keeps it). `serve` needs the API client's secret. | — | docs/catalyst-docs-progress.md; commit `0347553` |
| 7.3 | **Slow reads** | A full criterion read takes about 2.5 min; a full vector build about 46 min; incremental (`--previous`) seconds to about 18 s. | Each `catalyst view` is a new Python process (about 0.8 s). | Use `--previous`. The kernel's batch `view` (several IDs in one process, 0.120.0, REQ-000105) is the fix; whether the engine uses it yet was not checked. | docs/catalyst-docs-progress.md; cantica-tech/catalyst:CHANGELOG 0.120.0 |
| 7.4 | **Read requirements** | — | `--history N` needs full clones; GitHub visibility via `gh` (signed in); Azure DevOps anonymously or `AZURE_DEVOPS_EXT_PAT`; other hosts are public only if anonymously readable; only committed markdown is read; `.catalystignore` is honoured. Never publish from a private source: public builds read GitHub's default-branch commit. | — | README.md |
| 7.5 | **mcp 2.x renamed `FastMCP` to `MCPServer`** | — | Upstream API change. | — | docs/catalyst-docs-progress.md |
| 7.6 | **Integration tests need `KB_ENGINE_IT=1`** | CI runs offline tests only. | Since kernel 0.120.0 the integration test also needs `CATALYST_ALLOW_STATIC_IDENTITY=1` and a kernel release (`CATALYST_RELEASE`) or checkout. | — | tests |
| 7.9 | **New: tests on `httpx2`; catalyst-users 0.10.0** | — | 0.16.1. The move to catalyst-users 0.11.0 is pending. | — | pyproject.toml |
| 7.7 | **Search quality ceiling** | Hybrid recall 0.865@10, MRR 0.569, 7 of 52 questions missed. | Mostly vocabulary mismatch between questions and rule text. Query expansion with `llama3.2:3b` gave nothing reliable at about 10 s per question, and was abandoned. | Better rule wording or a synonym glossary. | cantica-tech/catalyst:docs/results/2026-10-30-search-query-expansion.md |
| 7.8 | **Placeholder source** | `onboarding.py` writes "TODO: where the documents live" when no source is known. | Intended. | — | src/catalyst_kb_engine/onboarding.py |

The token-host rule of 7.2 as a flow:

```mermaid
sequenceDiagram
    participant Caller as Caller
    participant U as catalyst-users token --system
    participant K as Keycloak
    participant S as catalyst-kb-engine serve
    Caller->>U: ask for a system token
    U->>K: log in at host H
    K-->>U: token, issuer built from host H
    Caller->>S: request with the token
    S->>K: verify against the issuer at host H2
    alt H equals H2
        S-->>Caller: 200
    else localhost versus 127.0.0.1, engine 0.16.1 or later
        S->>K: introspect at the token's own loopback name
        S-->>Caller: 200
    else another host name
        S-->>Caller: token rejected
    end
```

## 8. catalyst-docs — cantica-tech/catalyst-docs

| # | Oddity | Symptom | Cause | Fix / workaround | Source |
|---|---|---|---|---|---|
| 8.1 | **Public builds are recorded by hand** | A Pages build has no build record in the knowledge base. | Pages builds daily (cron 05:17) and on push to main, but Actions cannot reach the private criterion. | The curator runs `catalyst-kb-engine record-site <site url>` after each deployment. | README.md; .github/workflows/pages.yml; cantica-tech/catalyst:docs/results/2026-10-30-catalyst-docs-public-build-recorded.md |
| 8.2 | ~~**`pages.yml` pins engine 0.12.0**~~ (0.16.1 since 2026-10-31) | — | The site built with 0.16.1 was checked identical; actions, uv, MkDocs and Material are pinned by SHA or version. It still installs Ollama via its script on every run. | — | .github/workflows/pages.yml |
| 8.3 | **Vector reuse depends on the live site** | Builds sometimes take about 46 min. | Each run downloads the previously served `catalyst-vectors.tar.gz`; if that fails, it quietly falls back to a full embed. | — | .github/workflows/pages.yml |
| 8.4 | **`docs/user-generated/` excluded** | — | Opted out via `.catalystignore`. | — | .catalystignore |
| 8.5 | ~~**Guide pages 06–07 may be stale**~~ | — | Since 2026-10-31 they say membership is built, per-entity visibility and tag workflows are not. | — | docs/guide/ |
| 8.6 | **New: a Standards (lint) workflow** | A push fails on Markdown lint. | Since 2026-10-31 `standards.yml` runs the cantica-tech standards v0.1.2 lint (Markdown, YAML, workflows). A page needs its title in front matter and no extra H1. | `task --taskfile ../standards/Taskfile.standards.yml --dir . standards:lint` before pushing. | .github/workflows/standards.yml |

## 9. Distribution repo — cantica-tech/cantica-tech

| # | Oddity | Symptom | Cause | Fix / workaround | Source |
|---|---|---|---|---|---|
| 9.1 | **Binaries committed per release** | `.git` is about 106 MB (2026-11-01) and grows with every kernel. | It holds 107 kernel versions (`catalyst.pyz`, zip, manifest each), SE v1.0.0–v3.6.0, KBM v0.2.0–v0.6.1 and `.vsix` up to v0.73.0, all in commits "Deploy release: …". Since kernel 0.121.0 it is the **only** home of module releases. | Consider GitHub release assets. | repo |
| 9.2 | **Holes** | No SE v3.1.0 (only v3.1.1); no KBM v0.4.0. | Releases skipped or withdrawn. | — | repo |
| 9.3 | **Not everything is distributed here** | catalyst-users, catalyst-mcp and catalyst-kb-engine are absent. | Kernel, modules and vsix only. | — | repo |
| 9.4 | **The example project is pre-L5** | `catalyst/example/` (tempconv) builds an in-project `.criterion`, and `build.py` calls `init --user …`. | Not updated since R3. Inferred from code, not run. | — | catalyst/example/build.py |

---

## 10. Cross-cutting summary tables

### 10.1 Environment a new maintainer needs

| Need | Why |
|---|---|
| Python ≥ 3.11 (CI also 3.13), `uv` | Kernel floor (`tomllib`); a runtime per criterion |
| Docker plus catalyst-users' Keycloak (`docker compose up -d`), logged in via `catalyst identity login` or `catalyst-users login` (one login since kernel 0.120.0, for the same provider) | All signing |
| Node 24 (CI), npm workspaces | catalyst-ui |
| `gh` signed in | kb-engine visibility, PR flow |
| Ollama with `nomic-embed-text` | kb-engine vectors |
| Sibling checkouts: `../standards`, `../catalyst-software-engineering`, `../catalyst-knowledge-base` | `task lint`, purity scan, module tests |
| `uvx ruff@0.16.10` | CI parity |
| A `CRITERION_DEPLOY_KEY` secret in the UI's and catalyst-users' CI | Trace jobs |
| For releases: a Marketplace publisher login, Apple and Azure signing (not set up yet), the API client's secret | Publishing, desktop signing, live checks |

### 10.2 Version-pin lag (as of 2026-11-01)

The 2026-10-31 lags are closed: the kernel's KBM fixture is 0.6.1, catalyst-docs' Pages build uses engine 0.16.1,
catalyst's `.standards/` is v0.1.2, catalyst-ui runs Node 24 everywhere, and every Python repository pins ruff 0.16.10.
What lags now:

| Consumer | Pins | Current |
|---|---|---|
| catalyst-mcp, catalyst-kb-engine → catalyst-users | 0.10.0 | 0.11.0 |
| catalyst-ui, SE module `.standards/` | v0.1.1 | v0.1.2 |
| VS Code Marketplace listing | 0.66.0 | 0.73.0 (distribution repository) |

The same lag as a graph: each consumer, the version it pins, and the version that exists now.

```mermaid
flowchart LR
    m["catalyst-mcp and catalyst-kb-engine"] -->|"pin 0.10.0"| cu["catalyst-users 0.11.0"]
    s["catalyst-ui and SE .standards/"] -->|"v0.1.1"| std["standards v0.1.2"]
    mk["VS Code Marketplace"] -->|"0.66.0"| ui["catalyst UI 0.73.0"]
```

### 10.3 Triage of the most common failures

A first look at what fails most often, pointing to the entry that explains it:

```mermaid
flowchart TD
    start{"What failed?"}
    start -->|"trace job"| tr["A commit cites no artifact or rule ID, 1.1.4"]
    tr --> trf["Reword as fix(ID), feat(ID) or chore:"]
    start -->|"Deployment Rules"| dr{"Which step?"}
    dr -->|"purity"| pu["A module word in kernel files or CHANGELOG, 0.14"]
    dr -->|"scratch deployment"| sd["A catalogued module's main is broken, 1.1.2"]
    start -->|"Standards or lint"| li{"Locally green?"}
    li -->|"yes"| sk["No ../standards checkout, 0.11, or ruff skew, 0.10"]
    li -->|"no"| fx["Run uvx ruff@0.16.10 check and format"]
    start -->|"IamNotLoggedInError"| kc["Keycloak down or login expired, 0.6"]
    kc --> kcf["docker compose up -d, then catalyst identity login"]
    start -->|"exit 3"| as["A preview waiting for assent, 0.7"]
    start -->|"exit 4"| nd["No deployment here, 1.4.12"]
    start -->|"signed by the system account"| sa["Lapsed login, 1.3.9"]
    start -->|"UI catalyst-trace job"| ui["Deploy key missing or revoked, 4.7"]
    start -->|"Windows only"| win["Windows-specific path or lock behaviour, 1.1.9"]
```

---

## 11. What changed on 2026-10-31 and 2026-11-01

Fixed since the first edition (struck above): 0.10, 0.12 (catalyst), 0.13, 1.1.8, 1.4.10, 1.5.1–1.5.5, 2.2, 2.3, 2.8,
2.10, 3.1, 3.3 (zips), 3.5, 3.6, 4.4, 4.6, 4.8, 4.10, 4.12, 5.11, 6.8, 8.2, 8.5; eased: 0.6 (one shared login), 0.11
(`task lint` says the checkout is missing), 7.2 (loopback), 7.3 (a batch `view` exists). Chapter 30 §11 strikes the
CLI's own list the same way.

New facts, with the entry that explains each:

| Fact | Since | Entry |
|---|---|---|
| No deployment exits `4`; `2` is a usage error only | kernel 0.121.0 | 1.4.12 |
| `id next --peek` shows the next ID without reserving it | kernel 0.121.0 | [30 §6.1](30-cli-catalyst.md) |
| Module releases live only in the distribution repository; a manifest `entry` only with a UI bundle | kernel 0.121.0 | 1.4.11, 9.1 |
| `identity login --password`; one login shared with catalyst-users; one `identity.json` shape | kernel 0.120.0 | 0.6, 1.3.10 |
| `identity status` can say logged in while the system account signs | open | 1.3.9 |
| `criterion protect` and `hook install` ask (exit `3`) | kernel 0.120.0 | 0.7 |
| The release steps leave `SYNCHRONIZE.md` unjournaled | open | 1.1.12 |
| The journal-shards concurrency test flaked again on Windows | open | 1.2.1 |
| The password command on Windows | open in catalyst-users | 1.2.4, 5.17 |
| catalyst-users: loopback issuers, `setup` exits 1, flags anywhere, no test client in the realm | catalyst-users 0.11.0 | 5.13–5.16 |
| A dev-realm login lasts 12 h idle and 7 days at most | 2026-11-01 | 0.6 |
| A roadmap row is Done when every linked feature and requirement is closed | SE 3.6.0 | 2.9 |
| `VERIFIED_KERNEL_VERSION` is the paired kernel; Close/Reopen Bug need `/status` | catalyst UI 0.70.0–0.73.0 | 4.17–4.19 |
| catalyst-mcp and catalyst-kb-engine pin catalyst-users 0.10.0 | pending move to 0.11.0 | 6.9, 7.9, 10.2 |
| The Marketplace listing stopped at 0.66.0 | owner's call | 4.9, 10.2 |
