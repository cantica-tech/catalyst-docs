---
title: What catalyst is
classification: Public
---

catalyst is a persistent, checkable memory for software built with AI coding agents. An agent forgets between
sessions why the code is the way it is, and nothing it leaves behind says which requirement a line serves. catalyst
keeps that answer: documented rules, work that traces to those rules, and a journal of every change, kept beside the
project and checked by a command line rather than trusted.

It is for developers and teams who build with an agent and still need to answer "why does this code do X?" months
later, and "what rule does this change serve?" before it lands.

## The problem it solves

Without a record, three things go wrong as soon as an agent does real work:

- **Why is lost.** The reasoning behind a change lives in a chat that is gone the next day.
- **Rules drift.** What the system must do is scattered across tickets, comments and memory, and nothing checks that
  the code still does it.
- **Who did what is unclear.** An agent's edit and a person's look alike in git, and neither says what it was for.

catalyst answers each with a structure the agent works inside, and a tool that checks the structure holds.

## The chain

Everything in catalyst hangs on one chain:

```text
a change ──▶ an artifact ──▶ a rule ──▶ a domain
```

- A **rule** is a documented statement of expected behaviour, with a permanent ID, for example
  `br-AUTH-000003-Ab3xR9pQ`: "a password-reset link expires after 30 minutes". Rules live in rule documents, grouped
  by **domain** (`AUTH`).
- An **artifact** is a piece of work: a requirement, a bug, a step. Each one names the rule it serves.
- A **change** to the project is journaled with the artifact and the rules it served.

The chain's one invariant: no work happens without a traceable link down to a documented rule. Following IDs in either
direction answers both questions, from the code to its reason, and from a rule to every change made for it. Rules are
never deleted: a retired rule keeps its ID and says why it was retired, so every reference to it still resolves.

## What it looks like

A fictional project with one rule, one piece of work fixing it, and one journal entry:

```text
rules/business-rules.md
  br-AUTH-000003-Ab3xR9pQ  ✅ A password-reset link expires after 30 minutes.

BUG-000012-Ab3xR9pQ  reset link never expires
  Targets: br-AUTH-000003-Ab3xR9pQ   Status: Fixed   Signed-off-by: Ada Lovelace

the journal
  {"command": "/status", "action": "status-change", "tier": "fix",
   "artifact": "BUG-000012-Ab3xR9pQ", "targets": ["br-AUTH-000003-Ab3xR9pQ"],
   "intent": ["Reset links honour the 30-minute expiry again"],
   "files": [{"path": "src/auth/reset.py", "before": "9f2c…", "after": "41ab…"}]}
```

Months later, the journal lists every change made for that rule, and can rebuild the project's files as they stood at
any moment.

## The pieces

- **The kernel** is the part every project shares: rules and domains, the journal, people and signing, sharing, and
  the `catalyst` command line. It knows no particular way of working.
- **A process module** adds the kinds of work a project tracks. The software-engineering module brings requirements,
  steps, bugs and features; the knowledge-base module brings knowledge bases and their builds. A project runs one
  module, or several side by side.
- **The criterion** is a project's record: its rules, its artifacts, its journal, its definitions. It lives outside
  the project, in catalyst's home (`~/.catalyst/projects/<name>/criterion`). The project itself holds one small file,
  `catalyst.toml`, which names its criterion and holds no path, so it is the same on every clone.
- **The CLI**, `catalyst`, does everything mechanical: it allocates IDs, writes the journal with real file hashes,
  regenerates indexes and checks the chain. The agent keeps only the judgment.
- **The agent** works through the CLI. Claude Code is the supported and tested agent today; it is wired once per
  machine, never into a project.

## What it is not

- **Not a project-management tool.** It records why work exists and what it changed. Boards and sprints, if a team
  wants them, come from a module.
- **Not a replacement for git.** Git keeps the code's history. The journal keeps the reasons, and points into git by
  each file's exact content hash.
- **Not a trust-me agent.** The agent's word is never the record. `catalyst check` verifies the chain, and the agent's
  turn ends on a passing check.

catalyst is in beta. Its on-disk format is `1.0-rc`: it becomes `1.0` once the multi-user trial needs no change to it.

## Where to go next

- [Getting started](02-getting-started.md): a governed project in one sitting.
- [The ten laws](03-the-ten-laws.md): what an agent working under catalyst must judge for itself.
- Reference: the [glossary](https://github.com/cantica-tech/catalyst/blob/main/framework/kernel/GLOSSARY.md), the
  [laws and invariants](https://github.com/cantica-tech/catalyst/blob/main/framework/kernel/INVARIANTS.md), and
  [catalyst's README](https://github.com/cantica-tech/catalyst/blob/main/README.md).
