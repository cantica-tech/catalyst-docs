---
title: Rules, domains and the criterion
classification: Public
---

A project's rules are its documented behaviour: what the system must do, in statements short enough to check. This
page explains how they are written, numbered, grouped, retired and checked, and where they live.

## A rule

A rule is a heading in a rule document, its ID first, then its title:

```markdown
### `br-AUTH-000003-Ab3xR9pQ` A password-reset link expires after 30 minutes

- **Domain**: `AUTH`
- **Status**: ✅ working — `reset_link()`, tested by `test_reset.py`
- **Strength**: must

A reset link stops working 30 minutes after it is sent, whether or not it was used.
```

- **Its ID** reads `<document prefix>-<DOMAIN>-<number>-<userid>`: the rule document's lowercase prefix (`br`), the
  domain's uppercase code (`AUTH`), a six-digit number, and the eight-character userid of the person who created it.
  The CLI hands it out (`catalyst id next-rule br AUTH`), never a person.
- **Its status** says how far it is: ❌ not started, 🟡 specified, ✅ working, 🗑 retired.
- **Its strength** says how a breach is reported, never whether it is: `must` (a violation), `should` (a warning),
  `may` (a note). A rule that states none is `must`.
- **Its body** is the statement itself, in plain words.

A rule may also be a file of its own, `<ID>-<slug>.md`, listed in a rule document's table. Both shapes are read and
checked alike.

## Rule documents and domains

- **A rule document** holds the rules of one natural seam of the project: the business rules, the UI, one service.
  Each has a short prefix, chosen at install (`--rule-doc business-rules:br`), a `## Contents` section and a
  `## Linked Artifacts — Quick Index`.
- **A domain** groups rules inside a document: `AUTH`, `BILLING`. It has a file of its own in `rules/domains/`, saying
  its scope and its relationship to other domains, and a row in `rules/domains/domains.md`. A domain is registered
  before its first rule: `id next-rule` refuses a domain it does not know. A sub-domain is written `PARENT.SUB`.
- **`rules/rules.md`** is the index: every rule document with its domains, and every rule ID with its title. A rule
  missing from it fails the check.

## The life of a rule

1. **Check for conflicts.** Before a rule is added or changed, the rule documents are read for a rule it would
   contradict, duplicate or weaken. On a conflict, the user decides which wins; neither is edited silently.
2. **Gathered, implemented, tested, documented.** A rule is done only when all four hold: written with its domain and
   ID, the code doing what it says through a tracked artifact, a test or explicit verification showing it holds, and
   the user-facing documentation saying so. Its status marker shows which part is missing.
3. **Retired, never deleted.** A rule that no longer applies is marked 🗑 in place, with why and what replaces it. Its
   ID stays valid for ever, so every reference to it, in code, tests or old commits, still resolves. Retiring a rule
   never closes the work that targeted it, and closing work never retires a rule.

Numbers are never reused, not even a number that was only ever mentioned. The CLI counts every ID the journal names as
taken.

## Meta-rules

The rules about rules are themselves rules, `rr-META-NNN`, in the criterion's `rules/Rules-of-Rules.md`: how IDs are
formed, how a rule is retired, what a tier is, how the journal is kept, how a shared criterion works. They come from
the kernel, and each module adds its own (`rr-SE-`, `rr-KB-`). `catalyst why rr-META-3` explains any of them.

## The criterion

The criterion is the project's whole record. It lives in catalyst's home, never in the project:

```text
~/.catalyst/projects/<name>/criterion/
├── CODE-OF-CONDUCT.md     how work is proposed, tracked and closed; every command
├── INVARIANTS.md          the ten laws and the invariants behind them
├── DEPLOYMENT.md          the kernel and module versions installed
├── rules/                 rule documents, domains/, rules.md, Rules-of-Rules.md
├── <one folder per type>  requirements/, steps/, development/bugs/, …: each with its index and templates
├── definitions/           what each type is for, one versioned file per type
├── workflows/             WORKFLOW-: repeatable procedures
├── reconciliations/       RECON-: diverging versions, decided by people
├── IAM/                   who signs: the pointer to catalyst-users, the signers ledger
├── development/journal/   the journal, one file per person, machine and month
└── bin/catalyst.pyz       the CLI this criterion runs
```

Every artifact type has the same shape: its folder, an index of its entities, and its templates. An entity is one
markdown file whose name starts with its ID, and whose fields sit in a table at its top.

Some files are **generated** and never edited by hand: the indexes, the backlog, the composed `CODE-OF-CONDUCT.md`,
back-references such as a requirement's `Steps`. Others are **frozen**: a type's definition, once deployed, changes
only by migration. `catalyst check` reports a hand edit to either.

## Finding things

- `catalyst list rules`, `catalyst list REQ`: what exists, filtered by any field.
- `catalyst view <ID>`: one entity or rule, its fields, its links both ways, and its own history.
- `catalyst graph`: every rule, domain and artifact with its links.
- `catalyst backlog`: open work, missing links, and idle rules no work targets.

## Where to go next

- [Working day to day](05-working-day-to-day.md): tiers, the journal, commits.
- [The ten laws](03-the-ten-laws.md): L6 (generated and frozen) and L7 (permanent IDs) in full.
- Reference: the [format](https://github.com/cantica-tech/catalyst/blob/main/framework/kernel/FORMAT.md) (§5,
  rules and domains), the
  [criterion's layout](https://github.com/cantica-tech/catalyst/blob/main/framework/kernel/ARTIFACT-LAYOUT.md), the
  [meta-rules](https://github.com/cantica-tech/catalyst/blob/main/framework/kernel/rules-of-rules.template.md), and
  [13 — the criterion's home](https://github.com/cantica-tech/catalyst/blob/main/docs/design/13-criterion-home-requirement.md).
