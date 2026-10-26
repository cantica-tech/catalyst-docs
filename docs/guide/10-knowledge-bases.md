---
title: Knowledge bases
classification: Public
---

The knowledge-base module (KBM) and its engine build documentation and search from any project: a classified
repository of documents, and versioned builds of it that any model can read, without catalyst. This documentation
is built that way. This page explains how a knowledge base is set up, fed, classified, built and searched.

## The records

A knowledge base lives in a criterion as three types:

- **`KB-`, the knowledge base**: the rules it serves, its kind, its classification policy, what onboarding decided, its
  repository and its curator.
- **`KSOURCE-`, a source** it draws on: where it is, what kind of source it is, whether it is public or private, how
  it is read, the mark its documents get.
- **`KBUILD-`, a build**: its version, its target, its ceiling, its embedding model, where it is kept, and how it was
  verified.

KBM runs alone in a criterion (catalyst-docs does) or beside any other module.

## Onboarding: four decisions

Every knowledge base is onboarded before its first build (`/kb-setup`). An agent or the engine proposes each choice;
the curator, a person, decides:

1. **Its basis and sources.** *Criterion-based*: the project's own criterion is the first source, enriched by others.
   *Documents-based*: the curator gathers the sources.
2. **Its location and storage**: where its repository lives, and how documents are organized in folders.
3. **Its target**: organized markdown folders, or a vector database.
4. **Its document base**: *static* (defined once) or *dynamic* (grows and shrinks; the default for a criterion-based
   one).

The `KB-` record keeps those answers, with the policy: its classification, the default mark for anything new or
unmarked, and how far a hosted embedding service may go.

## Sources, and what stays private

Each source records its **kind** and its **visibility**:

| Kind | What it is |
| --- | --- |
| `Repository` | a git repository of documents |
| `Folder` | a directory or a drive folder |
| `Site` | a website or a wiki space |
| `Criterion` | a catalyst project's criterion, read only through catalyst's CLI |

A `Criterion` source is read with `catalyst list` and `catalyst view`, never by reading its files: each entity and
rule becomes a document citing its ID, its criterion and its status, with its own history from the journal.

**Visibility is a floor.** Every document read from a private source is at least `Internal`, whatever its own mark.
Before anything is read, each source's recorded visibility is checked against its host (GitHub's for a repository or
a criterion's repository), and any disagreement stops the read. A read for a public build never opens a private source
at all. What a `.catalystignore` opts out of catalyst is never part of a knowledge base.

## Classification

Four levels: `Public` < `Internal` < `Confidential` < `Restricted`. A document's effective classification is the
highest of its own mark, its source's floor and the knowledge base's own classification. Nothing stays unmarked: a
build refuses while anything is. A mark is only ever lowered by a person's decision.

## Builds any model can read

A build is written once, into its own version, in an open format (`kbm-build/1`): UTF-8 markdown, JSON, JSON Lines
and Parquet, with a `manifest.json` (every file's SHA-256) and a `README.md` saying how to read it. It contains
nothing above its **ceiling**.

- **Folders**: the documents organized by topic, with an index giving each one's title, classification and summary.
- **Vector database**: the documents, their chunks (`chunks.jsonl`) and one vector per chunk (`vectors.parquet`). One
  embedding model per build, named in the manifest, local unless the policy allows a hosted one for everything in the
  build. Each chunk is embedded with its document's title and heading, and with the prefixes the model expects; the
  manifest records both, so a consumer embeds questions the same way.

With the engine:

```sh
catalyst-kb-engine read-sources KB-… staging/ --projects-dir ~/sources      # every source, staged
catalyst-kb-engine build-vectors staging/ builds/1.0.0 --kb KB-… --version 1.0.0 --floor Public --ceiling Restricted \
    --local-model ollama:nomic-embed-text --document-prefix "search_document: " --query-prefix "search_query: "
```

**Rebuilds are incremental.** `read-sources --previous` re-views only the entities the journal says changed since the
last read; `build-vectors --previous` reuses every vector whose exact input is unchanged. After one change to a
fleet of five criteria and seven repositories, a rebuild re-viewed five entities and embedded 8 of 6,925 chunks, in
seconds rather than most of an hour.

## Searching a build

```sh
catalyst-kb-engine search "who decides during onboarding?" --build builds/1.0.0
catalyst-kb-engine mcp --build builds/1.0.0                                  # the same, as an MCP tool
```

The search merges keywords (SQLite FTS5) and vectors (the build's own model) by reciprocal rank. Every passage cites
its document, heading and classification, its source, and, from a criterion, its entity's ID.

**A search is measured.** A knowledge base can keep an evaluation set: questions, each with the entity IDs or
documents a good answer cites. `catalyst-kb-engine evaluate` scores a build in each mode (recall within the top
results, mean reciprocal rank), naming only the missed questions' IDs, and `--record` keeps the score in the build's
`KBUILD-`. An evaluation set drawn from private sources is itself private, kept in the criterion.

## The worked example: this documentation

catalyst's documentation is one knowledge base, `KB-000002` "catalyst", in the catalyst-docs project, with two builds
of it, never one build filtered afterwards:

| Build | Reads | Ceiling | For |
| --- | --- | --- | --- |
| **public** | catalyst's public repositories only | `Public` | this site |
| **realm** | every source, private criteria included | `Restricted` | the realm's search, kept privately |

The public build never reads a private source, so no private text can reach this site, even through a summary.
Each guide page cites only public documents, and a person reviews it before it lands.

## Where to go next

- [Analysing existing code](11-analysing-existing-code.md).
- Reference: KBM's [guide](https://github.com/cantica-tech/catalyst-knowledge-base/blob/main/GUIDE.md),
  [engine and onboarding](https://github.com/cantica-tech/catalyst-knowledge-base/blob/main/docs/engine-and-onboarding.md)
  and [best practices](https://github.com/cantica-tech/catalyst-knowledge-base/blob/main/docs/best-practices.md);
  the [engine](https://github.com/cantica-tech/catalyst-kb-engine/blob/main/README.md) and its design,
  [01 — catalyst-docs](https://github.com/cantica-tech/catalyst-kb-engine/blob/main/docs/design/01-catalyst-docs.md).
