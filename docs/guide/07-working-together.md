---
title: Working together
classification: Public
---

A criterion starts on one machine. This page explains how a team shares it, how one person's change reaches the
others, how a change is proposed and approved before it lands, how disagreements are settled, and who belongs to a
project.

## Sharing a criterion

Sharing is opt-in. `catalyst share create <url>` publishes the criterion to its own repository, private like every
criterion, and records it in the project's `catalyst.toml`. A teammate then runs `catalyst open` (or
`catalyst share join <url>`) in their clone of the project and gets the same criterion in their own catalyst home.

Two drivers carry a shared criterion:

- **git** (the default): the criterion is a repository, and changes land through pull requests.
- **a catalyst server** (`catalyst serve`, self-hosted): each push applies directly once the structural checks pass,
  each user signs with their own token (`catalyst share login`), and the server can hold the project's membership
  (below).

The commands are the same either way:

| Command | What it does |
| --- | --- |
| `catalyst share status` | this criterion against its shared copy |
| `catalyst share pull` | bring in what the shared copy has (refused while you have unpublished work) |
| `catalyst share push -m "…"` | publish your changes: with git, a topic branch and a pull request |

`share push` rebases, runs `catalyst check` (which also checks that a merge lost nothing), shows what it would publish,
and asks. Nothing leaves the machine without your yes. Each person's journal is a file of its own, and the indexes are
regenerated, so they merge cleanly. IDs never collide, because each carries its creator's userid.

With git, the criterion repository's CI runs the same checks on every pull request, and `catalyst criterion protect`
(or `share create --protect`) makes them required. Those are a shared criterion's real controls: who may merge, review,
and the required check.

## When two versions diverge: reconciliation

The agent never resolves a merge. When two people changed the same entity, or a change is disputed, the push stops
and a **reconciliation case** (`RECON-`) records both versions for a person to decide:

```sh
catalyst reconcile RECON-000004-Ab3xR9pQ accept --intent "the newer acceptance criteria"
```

A case is decided by `accept` (the proposed version), `accept-with-edits --from <file>` (a version the decider writes),
or revised with `propose --from <file>`. Who may decide depends on their reconciliation level, a grant from
catalyst-users: `full` may decide, `propose` may only propose a resolution, `none` neither. A case's status changes only through `catalyst reconcile`.

## Proposing a change

A change can be offered for review instead of made: `catalyst propose <ID> --from <file>` opens a proposal carrying
the new content. Reviewers see it in their inbox with its difference from the current version, and someone else
decides it. A contributor's change to a linked entity is always a proposal (see
[People, roles and signing](06-people-roles-and-signing.md)).

Before any write, `catalyst try <change>` makes it on a scratch copy of the criterion and reports what it would do,
including every finding `catalyst check` would add. Nothing is kept.

## Approvals by several people

A project may declare **approval gates** in `IAM/gates.json`: which states, or which rules' text, need approvals, and
from which role. Until the current version has them (`catalyst approve <ID> --gate <gate>`), the move or the edit is
refused. An author never approves their own work, and `--force` does not pass a gate. Without the file, there is no
gate.

## Who belongs to a project

**Membership** is kept by the catalyst server, never in the criterion, which is cloned onto many machines where a list
of people could leak.

- The provider holds two groups per project, `/criteria/<name>/admins` and `/criteria/<name>/members`.
- The server keeps the project's **membership ledger**: one encrypted, append-only entry per change (who was added or
  removed, by whom, when and why), chained so an entry cannot be rewritten once fingerprinted. The criterion keeps
  only the ledger's fingerprints, in its journal.
- Access fails closed: the provider **and** the ledger must both name a person.
- The Admin manages membership in every project, and a criterion admin in their own, with `catalyst members
  add|remove|list`. Only people already known to the provider can be added.
- A project shared by git alone has no members but its admins.

Membership lets a person reach the project. It grants no command: grants stay the provider's.

## Who sees what: visibility (designed, not yet built)

Today everyone who has the criterion can read all of it. Design 30 describes what comes next:

- **The realm**: no one outside the provider's realm has access to anything, with no guest and no shared link.
- **Classification and clearance**: every entity gets a `Classification`, and every person a clearance (a grant
  `clear:<Level>`). An entity is visible when it belongs to the project (or is linked to something visible there) and
  its classification is at or below the person's clearance.
- **Elevation**: an Admin can raise one person's clearance for one entity assigned to them.
- **User groups**: groups of people inside a project, kept on the server like membership.
- **Absent, not hidden**: an entity a person may not see is never listed, returned or counted.
- **The server is the boundary**: reading through `catalyst serve` is where visibility is enforced.

## Where to go next

- [Keeping catalyst up to date](08-keeping-catalyst-up-to-date.md).
- Reference: the [CLI](https://github.com/cantica-tech/catalyst/blob/main/framework/kernel/CLI.md) (sharing,
  reconciliation, proposals, approvals, members), and the designs
  [14 — sharing backends](https://github.com/cantica-tech/catalyst/blob/main/docs/design/14-sharing-backends.md),
  [19 — the server](https://github.com/cantica-tech/catalyst/blob/main/docs/design/19-server-design.md),
  [25 — approval gates](https://github.com/cantica-tech/catalyst/blob/main/docs/design/25-approval-gates.md),
  [26 — proposals](https://github.com/cantica-tech/catalyst/blob/main/docs/design/26-proposals.md),
  [30 — visibility](https://github.com/cantica-tech/catalyst/blob/main/docs/design/30-visibility.md) and
  [32 — project membership](https://github.com/cantica-tech/catalyst/blob/main/docs/design/32-project-membership.md);
  catalyst-users' [membership](https://github.com/cantica-tech/catalyst-users/blob/main/docs/membership.md).
