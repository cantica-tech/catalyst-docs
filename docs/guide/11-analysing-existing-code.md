---
title: Analysing existing code
classification: Public
---

A project that already has code has rules nobody wrote down: what the code actually does, and where it fails to. An
**analysis** (`/run-analysis`) infers them, the domains, the rules and the defects where the code breaks a rule. It
uses two independent readings of the code, and a person decides every finding. This page explains when to use one and
how it runs.

## When to use one

- **Bootstrap**: the project has no rules yet. The analysis finds its domains, a small set of rules, and the defects
  where the code breaks them. This is the usual start of a retrofit (see [Getting started](02-getting-started.md)).
- **Incremental** (the default): rules exist. The analysis finds what they miss, and where the code breaks them.

The alternative is gathering rules by hand, one area at a time, as you touch it. Both are fine. An analysis is faster
on a large codebase; gathering by hand keeps every rule's wording yours from the start.

An analysis extracts what already exists in code. It is not how a project's process is changed: the ID scheme, how
rules are retired, what becomes a meta-rule are decided by whoever owns the process, written once.

## Four eyes

Every analysis runs **two independent passes** over the same code, by two agents that cannot see each other's work,
then a **reconciliation** that compares them and checks against the code whatever only one found or the two disagree
on:

```text
pass A (independent) ──┐
                        ├──▶ diff ──▶ reconciliation ──▶ you decide each finding
pass B (independent) ──┘
```

Two blind passes catch what one misses, such as an agent that stops early, misreads a code path, or reports a rule
the code does not have. They do it far better than one pass and a self-review, because the second agent carries none
of the first one's framing.

A project may make one pass its default (`analysis_passes = 1` in `catalyst.toml`), or choose it for one analysis. A
single-pass analysis is recorded as such, says how each finding was verified, and still leaves every decision to you.
It is cheaper and catches less: use it for a narrow scope, never because two passes disagreed.

## The process

Each phase is a `catalyst analysis` command, and none can be skipped. The record is an `ANALYSIS-` entity in the
criterion, with its reports beside it.

1. **Start.** `catalyst analysis start <paths…> --mode bootstrap|incremental` records the scope, the commit analysed,
   and an inventory of every file in scope (and, when incremental, the rules and artifacts that already exist).
2. **Two passes.** Agents A and B each answer the same prompt with a findings file, recorded with
   `catalyst analysis record <ID> --pass A <file>`, then `--pass B`. A file that fails validation goes back to its
   agent; it is never edited to pass.
3. **Diff.** `catalyst analysis diff <ID>` pairs the findings: agreed, conflicting, found by A only, found by B only.
4. **Reconcile.** A third agent (or you) accounts for every finding of both passes, kept or dropped with a reason,
   exactly once, and says how each one that is not a plain agreement was verified against the code.
5. **Decide.** You see the reconciled findings, domains first, then rules, then defects, each with its statement,
   evidence and verification. For each, you accept, edit then accept, or reject. On acceptance, the agent writes it:
   the domain, the rule with its ID, or the bug a defect needs. `catalyst analysis decide` records each decision. A
   finding you and the reconciler still disagree on can become a reconciliation case.
6. **Close.** `catalyst analysis close <ID>` once every finding is decided; it refuses while an accepted finding's
   artifact does not exist.

## What a finding looks like

```json
{"id": "A2", "kind": "rule", "title": "Sessions expire", "area": "SESSION",
 "statement": "An idle session expires after the configured timeout.",
 "status": "partial", "confidence": "medium",
 "evidence": [{"path": "src/session.py", "line": 12}], "notes": "no test covers expiry"}
```

A finding is a `domain`, a `rule` (with a status: `holds`, `partial` or `missing`) or a `defect` (naming the rule it
breaks). Every rule and defect cites evidence: a file from the inventory and, where it applies, a line.

## After an analysis

The accepted rules are ordinary rules: they go through the same conflict check, and count as done only once they are
gathered, implemented, tested and documented. The accepted defects are ordinary bugs, fixed at the fix tier. Run
another incremental analysis whenever a large area changes.

## Where to go next

- [The Studio and the editor](12-the-studio-and-the-editor.md).
- Reference: the
  [analysis playbook](https://github.com/cantica-tech/catalyst/blob/main/framework/kernel/ANALYSIS-PLAYBOOK.md) (the
  pass and reconciliation prompts, the findings format) and the
  [CLI](https://github.com/cantica-tech/catalyst/blob/main/framework/kernel/CLI.md) (`catalyst analysis`).
