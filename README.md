# catalyst-docs

catalyst's central documentation, built by catalyst's knowledge-base module (KBM) and
[catalyst-kb-engine](https://github.com/cantica-tech/catalyst-kb-engine): a public site, in depth, built from
catalyst's public repositories only, and an in-depth search for the realm.

The site is live at <https://cantica-tech.github.io/catalyst-docs/>. Beside its pages it serves the same documents as
a vector database, [`vectors/catalyst-vectors.tar.gz`](https://cantica-tech.github.io/catalyst-docs/vectors/catalyst-vectors.tar.gz),
embedded with Ollama's `nomic-embed-text`. [`KB.md`](KB.md) describes the knowledge base: its classification, its
policy and its sources. The design is catalyst-kb-engine's
[`docs/design/01-catalyst-docs.md`](https://github.com/cantica-tech/catalyst-kb-engine/blob/main/docs/design/01-catalyst-docs.md).

GitHub Actions builds and deploys the site (`.github/workflows/pages.yml`) on each push to `main` and daily. Actions
cannot reach the private criterion, so after a deployment the curator records what the site serves, from this
repository's root:

```sh
catalyst-kb-engine record-site https://cantica-tech.github.io/catalyst-docs/
```

Each build the site serves becomes a `KBUILD-`, `Published`, superseding the one before it. A build already recorded
is skipped.

This repository is governed by catalyst; its criterion is private.
