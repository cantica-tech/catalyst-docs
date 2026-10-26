# The guide: outline (approved by the owner, 2026-10-26)

The guide is the written layer of catalyst's documentation (catalyst-kb-engine's design 01 §5.3). Each page is drafted
with the realm's search, reviewed by a person before it lands, and published only if it cites public reference pages
alone. Below, each page has its purpose, its outline, and the public documents it rests on.

## Audience and voice

- **For**: a developer or team lead meeting catalyst for the first time, and a contributor who needs the concepts
  before the reference.
- **Voice**: plain, short sentences, catalyst's own terms defined once (`GLOSSARY.md`). It says what is. What is
  designed but not yet built is described too, always marked "designed, not yet built" and linked to its design.
- **Shape**: about twelve pages of 800 to 2,000 words. Each page ends with "Where to go next", and the reference pages
  are linked inline.

## 1. What catalyst is

Why rules come first, the chain from a change to its rule, and who catalyst is for.

- The problem: agents and people changing code without a record of why.
- The chain: domain, rule, artifact, change; every change journaled and signed.
- The kernel and its modules; the criterion beside the project.
- **Rests on**: `catalyst/README.md`, `framework/kernel/GLOSSARY.md`, `framework/kernel/INVARIANTS.md` (the laws).

## 2. Getting started

From nothing to a governed project in one sitting.

- Install the CLI; `catalyst init` with a module; what lands in the project (`catalyst.toml`, nothing else).
- The agent wired once per machine (`catalyst agent install`).
- A first rule, a first requirement, a first check.
- **Rests on**: `framework/kernel/INSTANTIATION-GUIDE.md`, `framework/kernel/INSTANTIATION-CHECKLIST.md`,
  `framework/kernel/CLI.md`.

## 3. The ten laws

What an agent judges for itself, and what the CLI enforces.

- The ten laws, each with an example of keeping and of breaking it.
- Invariants: how a law becomes a check.
- **Rests on**: `framework/kernel/INVARIANTS.md`, `docs/design/17-ten-laws-draft.md`.

## 4. Rules, domains and the criterion

How a project's rules are written, numbered, retired and checked.

- Rule documents and domains; IDs that never change; strength (`must`, `should`, `may`).
- The criterion's layout and its indexes; what is generated and never edited.
- **Rests on**: `framework/kernel/FORMAT.md`, `framework/kernel/ARTIFACT-LAYOUT.md`,
  `docs/design/01b-kernel-rules-sync-commands.md`, `docs/design/13-criterion-home-requirement.md`.

## 5. Working day to day

The commands a person and an agent use, and the journal behind them.

- Tiers (chore, fix, feature) and the artifacts each needs.
- The journal: who, why, which rules, before and after; adopting hand-made changes.
- Checking, the backlog, the inbox.
- **Rests on**: `framework/kernel/CLI.md`, the software-engineering module's `code-of-conduct.module.md`.

## 6. People, roles and signing

Who signs, how they are verified, what they may do.

- Identity from a provider (Keycloak built in) through catalyst-users; the system account.
- Roles, grants, reconciliation levels; who may create and edit which type.
- **Rests on**: `framework/kernel/ACCESS-CONTROL.md`, `docs/design/28-identity-providers.md`,
  `docs/design/29-users-outside-catalyst.md`, `docs/design/31-who-changes-what.md`, `catalyst-users/README.md`,
  `catalyst-users/docs/onboarding.md`.

## 7. Working together

Sharing a criterion, proposals, approvals and reconciliation.

- Sharing through git (`catalyst share`); pull requests and the criterion's CI.
- Proposals; approval gates; reconciliation cases decided by people.
- Project membership and visibility (as designed, marked as such).
- **Rests on**: `docs/design/14-sharing-backends.md`, `docs/design/26-proposals.md`, `docs/design/25-approval-gates.md`,
  `docs/design/32-project-membership.md`, `docs/design/30-visibility.md`, `catalyst-users/docs/membership.md`.

## 8. Keeping catalyst up to date

Kernel and module versions, migrations, and what a sync changes.

- `catalyst sync`, module upgrades, frozen definitions and migrations.
- **Rests on**: `framework/kernel/SYNCHRONIZE.md`, `docs/design/21-module-install-design.md`, the modules'
  `migrations/migrations.md`.

## 9. Modules

What a process module is, and the two that exist.

- The module specification: entity types, commands, tiers, its own laws.
- Software engineering: requirements, steps, bugs, features, roadmaps.
- Knowledge bases (KBM): next page.
- **Rests on**: `framework/kernel/MODULE-SPECIFICATION.md`, `docs/design/20-module-v3-design.md`,
  `catalyst-software-engineering/README.md`, `catalyst-software-engineering/ROADMAP.md`.

## 10. Knowledge bases

Building documentation and search from any project with KBM and its engine.

- Onboarding's four decisions; sources, their kind and visibility; classification and its floors.
- Builds any model can read; searching and scoring a build; incremental rebuilds.
- This documentation as the worked example.
- **Rests on**: `catalyst-knowledge-base/GUIDE.md`, `catalyst-knowledge-base/docs/engine-and-onboarding.md`,
  `catalyst-knowledge-base/docs/best-practices.md`, `catalyst-kb-engine/README.md`.

## 11. Analysing existing code

Inferring domains and rules from code already written, four-eyes.

- **Rests on**: `framework/kernel/ANALYSIS-PLAYBOOK.md`.

## 12. The Studio and the editor

catalyst's user interfaces: the Studio, the editor extension, the desktop app.

- **Rests on**: the UI repository's `docs/roadmap.md`, `docs/decisions.md`, `docs/desktop.md`.

## Decided (owner, 2026-10-26)

1. **All twelve pages, in this order.**
2. **Designs not yet built are described**, each marked "designed, not yet built" and linked to its design.
3. **The owner reviews every page** before it lands. Pages are drafted in batches, starting with pages 1 to 3.
