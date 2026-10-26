---
title: Modules
classification: Public
---

The kernel knows rules, domains, people, the journal and sharing, but not what kind of work a project tracks. That is
a **process module**'s: the types of work, their fields and workflows, the commands that create them, and the
module's own laws. This page explains what a module is, and the two that exist today.

## What a module brings

A module is a separately versioned repository. Its manifest, `module.yaml`, declares:

- **its entity types**: each one's ID prefix, folder, fields, states and the moves between them, and how it grounds
  in a rule: directly (`required`), through a parent (`inherited`), or not at all (`none`);
- **its tiers**: which artifact a chore, a fix or a feature needs;
- **its commands**, with their procedures in `code-of-conduct.module.md`;
- **its meta-rules** (`rules-of-rules.module.md`, in its own namespace such as `rr-SE-`) and **invariants**
  (`INVARIANTS.module.md`), including up to three laws of its own;
- **its definitions** (what each type is for) and **migrations** (how a deployment moves between its versions);
- **the kernel it needs** (`kernel_version`).

At install and at every sync, catalyst composes the kernel and every installed module into one criterion: the
governing documents are recomposed, the types' folders and templates created, the commands made available to the agent.
The CLI reads each type's definition from the module, so it handles every type without naming any.

## Alone or side by side

A project runs one module, or several. `catalyst init --module` installs the first, and `catalyst module install`
adds another later. Two laws keep that safe:

- **The kernel knows no module.** Nothing in it names a module's types, folders or commands (law L9). catalyst's own
  checks fail if it ever does.
- **No module knows another.** A module works alone, and beside any other, in either order: it never requires, names
  or assumes another module's types, commands or tiers. The knowledge-base module, for instance, reads a criterion
  through catalyst's CLI whichever modules wrote it.

## The software-engineering module

The module for building software, `catalyst-software-engineering`. Its namespace is `SE`, and its artifacts ground in
the kernel's rules.

| Type | ID | What it is | Grounds in a rule |
| --- | --- | --- | --- |
| Requirement | `REQ-` | a concrete requirement derived from the rules, or opened to introduce new ones | directly |
| Step | `STEP-` | one unit of implementation work toward one requirement or bug | through its parent |
| Bug | `BUG-` | a documented rule that does not hold in the running application | directly |
| Test | `TEST-` | a check that a targeted rule actually holds | directly |
| House-keeping | `HK-` | tools, scripts or process supporting the development itself | directly |
| Feature | `FEAT-` | an idea or roadmap item, before work on it starts | no |
| Decision | `ADR-` | one decision: its context, what was decided, the alternatives, its consequences | no |
| Roadmap row | `RM-` | a row of a named roadmap, ingested from an outside source | no |

Its tiers: a **chore** needs no artifact, a **fix** a bug, a **feature** a requirement. Its law, SE-L1: the tier
decides the artifact, a requirement's steps are opened as the work happens, and a roadmap feature becomes a
requirement when work on it starts, never a bug.

Its commands include `/create req`, `/create bug`, `/create step`, `/create test`, `/create feature`, `/create adr`,
`/roadmap` (named roadmaps, their rows' dates and links) and `/show-backlog` (open work, blockers and missing links,
refreshed into `development/BACKLOG.md`).

## The knowledge-base module

The module for building and maintaining knowledge bases, `catalyst-knowledge-base`: a project's knowledge base from
any information outside catalyst, or a reflective one of catalyst's own ecosystem. Its types are the knowledge base
(`KB-`), its sources (`KSOURCE-`) and its builds (`KBUILD-`). It runs alone in a criterion, as it does for this
documentation, or beside any other module. See [Knowledge bases](10-knowledge-bases.md).

## Writing a module

The [module specification](https://github.com/cantica-tech/catalyst/blob/main/framework/kernel/MODULE-SPECIFICATION.md)
is the contract: the repository's layout, the manifest's fields (format 3 keeps a module in one file), the entity
type schema and its workflow, how a module's contributions are composed, and the kernel's purity rule. catalyst's
repository ships a sample module (`framework/modules/sample-process/`) to start from, and lists the production
modules in `framework/modules/catalog.md`.

## Where to go next

- [Knowledge bases](10-knowledge-bases.md).
- Reference: the
  [module specification](https://github.com/cantica-tech/catalyst/blob/main/framework/kernel/MODULE-SPECIFICATION.md),
  [20 — modules, version 3](https://github.com/cantica-tech/catalyst/blob/main/docs/design/20-module-v3-design.md),
  the software-engineering module's
  [README](https://github.com/cantica-tech/catalyst-software-engineering/blob/main/README.md),
  [commands](https://github.com/cantica-tech/catalyst-software-engineering/blob/main/code-of-conduct.module.md) and
  [roadmap](https://github.com/cantica-tech/catalyst-software-engineering/blob/main/ROADMAP.md).
