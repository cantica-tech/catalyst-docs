# catalyst

| | |
| --- | --- |
| **Knowledge base** | `KB-000002-erFhHt6m` (recorded in the catalyst-docs criterion) |
| **Classification** | Public |
| **Default classification** | Internal |
| **Hosted embedding up to** | Local only (`nomic-embed-text`) |
| **Basis** | Documents |
| **Document base** | Dynamic |
| **Target** | Vector database (the realm build); Folders (the public build) |
| **Curator** | Olivier Steck |

catalyst's central documentation, in depth: the kernel, its modules, the engine, catalyst-users, catalyst-ui and the
shared standards. Public as a site built from catalyst's public repositories alone; searchable in depth by the realm
over every source, private criteria included, which are never published. Serves `kn-DOCS-000001-erFhHt6m`.

## Policy

Nothing here is above `Public`. A document is `Public` only when its source is a public repository; anything new,
imported or unmarked is `Internal`, and stays out of the public site. Every document read from a private source (a
criterion, a private repository) is at least `Internal`, and is read only by the realm build, never written here.

Two builds, never one build filtered afterwards: the public build reads the public sources only, within `Public`;
the realm build reads every source and is kept in private storage. Embedding runs locally, never on a hosted service.

## Sources

| Source | Kind | Visibility | Public build | Realm build |
| --- | --- | --- | --- | --- |
| `catalyst` | Repository | Public | yes | yes |
| `catalyst-software-engineering` | Repository | Public | yes | yes |
| `catalyst-knowledge-base` | Repository | Public | yes | yes |
| `catalyst-kb-engine` | Repository | Public | yes | yes |
| `catalyst-users` | Repository | Public | yes | yes |
| `catalyst-ui` | Repository | Public | yes | yes |
| `standards` | Repository | Public | yes | yes |
| `catalyst-users-kb` | Repository | Private | **never** | yes |
| six criteria of the fleet | Criterion | Private | **never** | yes |

## Contents

```text
KB.md       this file
docs/       the guide's written pages, public only, reviewed before they land (none yet)
pointers/   pointers to outside documents, mirroring docs/ (none yet)
builds/     one directory per public build version, its manifest only (none yet)
```
