---
title: People, roles and signing
classification: Public
---

Every rule, artifact and journal entry in catalyst is signed by someone. This page explains who that someone is, how
catalyst knows, and what each person may do.

## Who signs: a verified identity

catalyst never takes a signer's word for who they are, and never guesses from `git config`. It asks
**catalyst-users**, a small command-line facade to an identity provider (Keycloak is built in), and signs as the
identity catalyst-users verifies at that moment:

- **the person logged in** on this machine (`catalyst identity login`, the device flow: open a link, confirm a code);
- or, with no one logged in, **the machine's system account**, when its settings point at the same provider.

With neither, nothing is signed: the command fails and points to `catalyst identity login`. `catalyst identity whoami`
shows who you are, your roles and your grants.

The first time an identity signs in a criterion, it gets an eight-character **userid**, recorded in the criterion's
signers ledger (`IAM/signers.jsonl`). The ledger is append-only and never rewritten. Every ID that identity creates
ends with its userid, which is how two people creating rules at the same time never collide.

## What each person may do: grants

People, groups and roles live in the identity provider, never in the criterion. What a person may do is a set of
**grants** carried in their token:

- `cmd:<command>` allows one command (`cmd:/create`, `cmd:/status`); `cmd:*` allows every command;
- `create:<TYPE>` allows creating one type;
- `reconcile:<level>` sets how far they may go in deciding a reconciliation case: `full`, `propose` or `none`.

The provider composes grants into roles and roles into groups. A role's name grants nothing by itself: only its
grants count. Every writing command checks the signer's grants and refuses one that lacks the grant it needs, naming
that grant.

**Reading is never restricted.** Everyone who has the criterion can read all of it. Writes are what is controlled.
(Who may reach a project is its membership, built on the server; who may see which entities inside it is designed,
not yet built: see [Working together](07-working-together.md).)

## The default roles

catalyst-users provisions these once into the provider, from the kernel's roles template:

| Role | May, in short | Reconciliation |
| --- | --- | --- |
| Product Owner | create planning and new work, adopt changes made outside catalyst | `propose` |
| Scrum Master / Delivery Lead | move open work's status | `propose` |
| Tech Lead / Architect | approve rule changes and new domains, run analyses, migrate definitions | `full` |
| Developer | create and implement development artifacts, run analyses | `propose` |
| QA / Tester | report defects, verify a rule's coverage | `propose` |
| Stakeholder | propose ideas and planning items | `none` |
| Release Manager | synchronize catalyst, share the criterion, cut releases | `full` |
| Admin | everything | `full` |

A **criterion admin** administers one criterion: its members and their groups, and nothing else. The **catalyst
admin** administers the provider. After provisioning, roles change in the provider (`catalyst-users roles provision`),
never in the criterion.

## Who creates and changes what

- **Admins** (the Admin, and a criterion admin in their own criterion) create, edit and retire entities directly. A
  change is **forewarned** first: `--preview` on any writing command, or `catalyst try`, shows what it would change
  and every finding `catalyst check` would add, such as a dangling reference or a broken chain. Nothing is permanent
  before the confirmation, and a confirmed change that adds findings says so in its journal entry.
- **Everyone else** creates a type only when a module running in the criterion lets contributors create it and their
  grants allow it (`create:<TYPE>`, or `cmd:/create`).
- **Editing**: `catalyst edit <ID>` changes an entity's fields and sections, never its ID, name, signer or status. A
  contributor edits an unlinked entity directly. A change to a **linked** entity (one that cites or is cited by
  another) becomes a **proposal**, decided through the type's governing workflow, and applied only at its final move.
- **Deleting is retiring.** Nothing is ever deleted: an entity is retired in place and keeps its ID.
- **Tags and tag workflows** are designed, not yet built: a workflow that governs every entity carrying given tags,
  run by the people granted it.

## Approvals that need two people

A criterion may declare **approval gates** (`IAM/gates.json`): an entity cannot move into a gated state, and a gated
rule's text cannot change, until its current version has the approvals the gate asks for (`catalyst approve`), from
people holding its role, never its author. `--force` does not pass a gate.

## What actually controls a shared criterion

catalyst checks grants where the change is made, against the verified identity. In a shared criterion, the controls
that do not rely on anyone's good faith are the hosting service's: who may merge into the shared branch, pull-request
review, and the required `catalyst` check that `catalyst criterion protect` turns on.

## When the agent signs

The agent signs as the identity catalyst-users verifies on the machine, the logged-in person or the system account,
never as anyone else. It never approves its own work, and never decides a reconciliation case.

## Where to go next

- [Working together](07-working-together.md): sharing, proposals, reconciliation, membership and visibility.
- Reference:
  [access control](https://github.com/cantica-tech/catalyst/blob/main/framework/kernel/ACCESS-CONTROL.md),
  [28 — identity providers](https://github.com/cantica-tech/catalyst/blob/main/docs/design/28-identity-providers.md),
  [29 — people outside catalyst](https://github.com/cantica-tech/catalyst/blob/main/docs/design/29-users-outside-catalyst.md),
  [31 — who changes what](https://github.com/cantica-tech/catalyst/blob/main/docs/design/31-who-changes-what.md),
  [catalyst-users](https://github.com/cantica-tech/catalyst-users/blob/main/README.md) and its
  [onboarding](https://github.com/cantica-tech/catalyst-users/blob/main/docs/onboarding.md).
