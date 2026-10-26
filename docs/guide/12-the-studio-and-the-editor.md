---
title: The Studio and the editor
classification: Public
---

catalyst's command line is the engine; **Catalyst Studio** is its interface. It makes the chain from a change to its
rule navigable in both directions, shows drift as it happens, and lets a person act on the criterion without
bypassing the agent or the journal. This page explains where it runs, what it shows, and how it acts.

## Where it runs

- **In VS Code**: the extension is published on the Visual Studio Marketplace as **Catalyst Studio**. Besides the
  Studio's views, it adds what belongs in an editor: diagnostics on the project's files, click-to-jump from an ID to
  its rule or artifact, CodeLens, and code actions.
- **On the desktop**: a standalone app for macOS, Windows and Linux, built for each release. Each platform's build is
  signed once its signing credentials are set up; until then, it is built unsigned and says so.

Both mount the same Studio. It reads a project through catalyst itself, `catalyst serve --local` or a served
criterion: the chain, its validation and the journal come from catalyst's `graph`, `check` and `journal`, refreshed as
they change. The desktop app uses the machine's `catalyst` when there is one, otherwise its own copy (which needs
Python 3.11 or later).

## What it shows

**Understanding the project**

- **The traceability explorer**: where an entity comes from (the rule and domain it serves) and what depends on it, as
  a tree and as a graph.
- **Entity pages**, generated from the module's types: fields, sections, the moves its workflow allows, and what
  blocks each one.
- **The rule browser**: every rule with its lineage, what cites it, and its strength (what a breach is reported as),
  filterable by it.
- **The map** of the module's types and how they reference each other, coloured by state.
- **Search** over every artifact, roadmap row and rule, from any view (`Ctrl`/`Cmd` + `/`).

**Seeing what needs attention**

- **The dashboard**: the check's health, open work, drift, versions, coverage, recent activity.
- **Drift**, marked wherever an entity is shown: failing checks, changes made outside catalyst, stale references, each
  with the command that fixes it.
- **The inbox**: what waits for a person, each item with its evidence, settled in a few clicks.
- **Approvals**: reconciliation cases with their trail, and who signed off what.

**Following the work**

- **The board**: a type's states as columns. A move is catalyst's own `status set`, refused by a guard just as on the
  command line, and a batch move makes one journaled change per card.
- **The journal**, as a timeline: entries grouped by date, each with who and why, and side-by-side differences
  between versions.
- **The team view**: each person's roles, open work and recent changes.
- **A roadmap's Gantt chart**, from the dates its rows declare.

## How it acts

The Studio never writes catalyst's files itself. Every action is one of catalyst's own commands, run through its host:

- it shows the exact command first, and runs it only on your yes;
- each run is journaled like any other change, signed by your verified identity;
- the command palette lists every command of the deployment (`catalyst spec`), and the agent console dispatches one
  to your agent, streaming its output and recording the run and its cost.

A write the Studio offers is forewarned: like `--preview` on the command line, it shows what the change would do, and
any finding `catalyst check` would add, before it is made.

## Where to go next

- Back to [What catalyst is](01-what-catalyst-is.md), or on to the reference.
- Reference: the Studio's
  [roadmap](https://github.com/cantica-tech/catalyst-ui/blob/main/docs/roadmap.md),
  [decisions](https://github.com/cantica-tech/catalyst-ui/blob/main/docs/decisions.md) and
  [desktop builds](https://github.com/cantica-tech/catalyst-ui/blob/main/docs/desktop.md), and its
  [README](https://github.com/cantica-tech/catalyst-ui/blob/main/README.md).
