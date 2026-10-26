---
title: Keeping catalyst up to date
classification: Public
---

A criterion is installed at one kernel version and one version of each module, and runs the CLI it was installed
with. catalyst moves often. This page explains how a project moves forward, what changes when it does, and what never
changes on its own.

## Versions

- **The kernel** has a version (`0.117.0`), recorded in the criterion's `version.txt` and the project's
  `catalyst.toml`.
- **Each module** has its own version, and declares the kernel it needs (`kernel_version: ">=0.107.0"` in its
  `module.yaml`). The criterion records every installed module in `modules/modules.json`.
- **The format** is the on-disk shape of a deployment, `1.0-rc` today. It changes far less often than the kernel, and
  `catalyst check` verifies the CLI can read it.
- **The CLI** each criterion runs is its own copy, `bin/catalyst.pyz`, with its own runtime. Two projects at
  different versions each run their own; the launcher `~/.catalyst/bin/catalyst` picks the right one from the project
  you are in.

Releases are published by catalyst's repository and modules, and gathered in the distribution repository
(`cantica-tech/cantica-tech`), which keeps each kernel and module release as a zip with its manifest.

## Moving forward: `/sync-framework`

With an agent, `/sync-framework latest` (or a given version) does everything. It needs one confirmation before it
starts. Its mechanical half is two commands:

```sh
catalyst sync plan  --kernel <catalyst checkout>/framework/kernel   # what would change, and the migrations to run
catalyst sync apply --kernel <catalyst checkout>/framework/kernel   # do it, and journal it
```

`--kernel` also takes a release's `kernel-vX.Y.Z.zip`. `plan` writes nothing. It lists what would change:

- the CLI, and the runtime the launcher runs;
- the laws and invariants, and the kernel's reference documents;
- the governing documents, recomposed from the kernel and every installed module (`CODE-OF-CONDUCT.md`,
  `Rules-of-Rules.md`);
- the definitions of new types;
- the migrations between the two versions, in order.

`apply` does it and journals a single `/sync-framework` entry.

A sync is careful with what is yours. A kernel document or a command file edited locally is reported, never
overwritten. A recompose conflict stops the sync before anything is written. A file no release ships is never touched.
An item listed in the criterion's `.frozen` file (`/freeze`) is left alone.

## Migrations: the mechanical and the judgment

Some releases change the shape of what a criterion holds: a new field on a type, a value renamed, a template's next
version. Each such release ships a **migration**, a short document saying what changed and the steps to follow:

- **The mechanical steps** are done by `sync apply`, or by `catalyst migrate --apply`, which adds the template
  versions a migration asks for.
- **The judgment steps** stay with a person and the agent: for example, "for each existing knowledge source, with its
  curator's agreement, record its kind and visibility". `catalyst migrate` lists the ones a deployment has not done.

A migration is never run twice. Each module keeps its own in `migrations/migrations.md`, with the kernel versions it
moves between.

## What never changes on its own: definitions

A type's **definition** (`definitions/<type>.md`, what the type is and is for) is frozen once deployed. An ordinary
sync never touches it. It moves forward only by `/migrate-definition <type> <version>`, a deliberate decision, so the
meaning of a requirement or a rule in your project never shifts silently under you.

## Modules

| Command | What it does |
| --- | --- |
| `catalyst module list` | each installed module: version, source, commit, hash |
| `catalyst module available <id>` | the releases at the module's repository |
| `catalyst module upgrade <id> <source>` | move a module to another release, through `sync plan` and `apply` |
| `catalyst module install <source>` | add a module to a deployment, beside the ones it has |
| `catalyst module verify <id>` | check the installed module against its recorded hash |

A module that needs a kernel newer than the one a shared criterion's CI runs is refused: the CI checks every pull
request with the CLI of its base branch, which could not read it. Move the kernel first, then the module.

## In a shared criterion

A sync is a change like any other: it is journaled, then published with `catalyst share push`, and reviewed through
the criterion's pull request. Teammates get it with `catalyst share pull`, and the launcher then runs the new CLI for
that project.

## Where to go next

- [Modules](09-modules.md): what a module brings.
- Reference: [synchronizing](https://github.com/cantica-tech/catalyst/blob/main/framework/kernel/SYNCHRONIZE.md),
  the [CLI](https://github.com/cantica-tech/catalyst/blob/main/framework/kernel/CLI.md) (`sync`, `migrate`, `module`),
  [21 — installing modules](https://github.com/cantica-tech/catalyst/blob/main/docs/design/21-module-install-design.md),
  and the modules' migrations (for example the
  [knowledge-base module's](https://github.com/cantica-tech/catalyst-knowledge-base/blob/main/migrations/migrations.md)).
