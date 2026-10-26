---
title: Working day to day
classification: Public
---

Most days, catalyst asks three things of whoever changes the project, person or agent: say what kind of change it is,
say which rule it serves, and record it as it happens. This page walks through a change from start to commit, then
shows what to do when something was changed outside catalyst.

## Before starting: the tier

Every change has a **tier**, stated before the work begins:

| Tier | What it is | What it needs (software-engineering module) |
| --- | --- | --- |
| **chore** | changes no rule's behaviour: a typo, a dependency bump, a refactor that keeps every rule | no artifact, only its journal entry |
| **fix** | restores a documented rule's behaviour | a bug (`BUG-`) against the rule it restores |
| **feature** | adds or changes behaviour | a requirement (`REQ-`) with the rules it targets, and steps as the work goes |

When unsure, take the higher tier. If the work turns out bigger than announced, escalate the tier as soon as that is
clear, opening what the new tier needs then. Never lower it afterwards to save ceremony.

A feature from the roadmap (`FEAT-`) becomes a requirement when work on it starts. A bug is never used to smuggle in a
feature.

## During the work

**Open the artifact first.** `catalyst new` allocates its ID, fills its template, signs it and journals its creation:

```sh
catalyst new BUG --title "Reset links never expire" --field Targets=br-AUTH-000003-Ab3xR9pQ \
    --field Domain=AUTH --field Severity=High --command /create-bug --tier fix
```

With an agent, the same is the `/create bug` command, which reaches Claude Code as `/catalyst:create`. A feature's requirement gets **steps**
(`STEP-`), each opened when its piece of work starts, at least one before the requirement closes.

**Journal each change as it is made.** After editing files, the change is recorded with the rules it served and its
intent, the goal and not a label:

```sh
catalyst journal append --command /create-step --action update --artifact STEP-000041-Ab3xR9pQ \
    --intent "reset links honour the 30-minute expiry again" \
    --file src/auth/reset.py --file tests/test_reset.py \
    --target br-AUTH-000003-Ab3xR9pQ --target BUG-000012-Ab3xR9pQ --tier fix
```

The CLI writes the time, the signer, and each file's exact git content hash before and after. Statuses, step records
and the journal follow the work in the smallest practical units. They are never reconstructed in one batch at the end,
unless the batching was announced before the work started.

**Move statuses with the CLI**: `catalyst status set BUG-000012-Ab3xR9pQ Fixed --intent "…"`. A type's workflow says
which moves are allowed and what each needs; a requirement cannot close without a step, for example. `catalyst view`
shows what blocks each next state.

## Committing

Every commit cites the artifact or rule it serves, or starts with `chore:`:

```text
fix(BUG-000012-Ab3xR9pQ): reset links expire after 30 minutes again
```

`catalyst hook install` adds a commit-message hook that refuses a commit citing nothing, and `catalyst trace` checks a
range of commits the same way in CI. Nothing is pushed without the user's yes.

## Checking

```sh
catalyst check
```

The check verifies the chain and the record: IDs well formed and unique, every artifact grounded in a rule, every
rule indexed and in a registered domain, indexes current, and every governed file's current state recorded in the
journal. With an agent, the end-of-turn hook runs it, and blocks the agent until what this session broke passes
again. Failures that were already there when the session started do not block it.

## Changes made outside catalyst

Sometimes a file is changed straight in git: a hotfix, a teammate without the agent, a merge. `catalyst unrecorded`
lists every commit, after the project's baseline, whose changes no journal entry records. Each one is resolved with
`/adopt`, and the user decides:

- **adopt it**: `catalyst journal adopt <commit>` records it after the fact, marked as a manual change, with its
  commit and author;
- **reject it**: the commit is reverted, with the user's assent;
- **dispute it**: it becomes a reconciliation case for a person to decide.

During the beta, an unrecorded change is a warning. From format `1.0` on, it is an error.

## Looking back

- `catalyst journal show --rule br-AUTH-000003-Ab3xR9pQ`: every change made for a rule.
- `catalyst journal show --artifact BUG-000012-Ab3xR9pQ --content`: one artifact's history, with each file's content.
- `catalyst journal restore <timestamp> <dir>`: the project's governed files as they stood at that moment, rebuilt
  into a side directory. The live tree is never touched.
- `catalyst why <ID>`: what an ID is and what cites it.

## What waits for whom

- `catalyst backlog`: open work by type and status, missing links, and rules no work targets.
- `catalyst inbox`: what waits for a person, each item with its evidence and what would settle it, such as a
  reconciliation case, a gate awaiting approval, or an overdue source.

## Where to go next

- [People, roles and signing](06-people-roles-and-signing.md): who may run which command.
- [Working together](07-working-together.md): sharing the criterion, proposals and approvals.
- Reference: the [CLI](https://github.com/cantica-tech/catalyst/blob/main/framework/kernel/CLI.md), the
  [meta-rules](https://github.com/cantica-tech/catalyst/blob/main/framework/kernel/rules-of-rules.template.md) (§6
  tiers, §12 the journal, §23 updates as work happens), and the software-engineering module's
  [commands](https://github.com/cantica-tech/catalyst-software-engineering/blob/main/code-of-conduct.module.md).
