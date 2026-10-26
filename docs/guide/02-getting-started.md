---
title: Getting started
classification: Public
---

From nothing to a governed project in one sitting: people who can sign, catalyst installed into a project, a first
rule, a first piece of work traced to it, and a passing check. Most of it is done for you by the agent once you ask;
this page shows each piece so you know what it did.

catalyst installs only when you ask. Reading this repository, or opening a project without a `catalyst.toml`,
installs nothing.

## What you need

- **Python 3.11** or later, and **git**, with `git config user.name` set.
- **[uv](https://docs.astral.sh/uv/)** and **Docker**, for catalyst-users and its local identity provider.
- **An agent**, if you want one: Claude Code is the supported and tested agent. Everything below also works by hand.
- Checkouts, side by side in one directory:

```sh
git clone git@github.com:cantica-tech/catalyst.git
git clone git@github.com:cantica-tech/catalyst-software-engineering.git
git clone git@github.com:cantica-tech/catalyst-users.git
```

## 1. People who can sign

Every rule, artifact and journal entry is signed by a verified person, never a name typed in. catalyst learns who you
are from an identity provider, through catalyst-users. For a first project, run its local Keycloak:

```sh
cd catalyst-users
uv sync --all-extras
docker compose up -d                  # Keycloak on 127.0.0.1:8080, realm "catalyst"
uv run catalyst-users setup           # the provider, its settings, its admin
uv run catalyst-users whoami --system
```

`setup` writes catalyst-users' settings to `~/.catalyst/users/`. With no one logged in, catalyst signs as this
machine's system account, verified by the same provider; `catalyst identity login` signs you in as a person instead.

## 2. The command line

Build the CLI from the catalyst checkout:

```sh
cd catalyst && task build:cli         # writes dist/catalyst.pyz
```

The CLI is a single file. Each project's criterion keeps its own copy, so a project always runs the version it was
installed with.

## 3. Install catalyst into a project

From the project's root, with the module and a first rule document:

```sh
python3 ../catalyst/dist/catalyst.pyz init --kernel ../catalyst/framework/kernel \
    --name shop --module software-engineering --git-username ada \
    --rule-doc business-rules:br --agent claude-code
```

`init` does everything mechanical:

- it creates the criterion in catalyst's home, `~/.catalyst/projects/shop/criterion`: rule documents, entity folders,
  definitions, the journal, and a copy of the CLI with its own Python runtime;
- it records you in the signers ledger, as the first person who signs;
- it writes **one file into the project**, `catalyst.toml`, which names the criterion and holds no path;
- it commits nothing in your project.

Once per machine, put `catalyst` on your path and wire your agent:

```sh
python3 ~/.catalyst/projects/shop/criterion/bin/catalyst.pyz runtime install   # the launcher, ~/.catalyst/bin/catalyst
export PATH="$HOME/.catalyst/bin:$PATH"
catalyst agent install claude-code
```

`agent install` works at user level only. It registers catalyst's MCP server, so its commands reach the agent as
prompts (`/catalyst:check-rules`), plus an end-of-turn hook that runs `catalyst check`. Nothing is written into the
project.

Commit the project file, and, if you want every commit traced, install the commit-message hook:

```sh
git add catalyst.toml && git commit -m "chore: install catalyst"
catalyst hook install                 # each commit must cite a rule or artifact ID, or start with chore:
```

`catalyst where` says which criterion a project uses; `catalyst open` makes it ready on a new machine or a fresh
clone.

## 4. A first rule

A rule lives in a domain, and its ID comes from the CLI, never from you. With an agent, ask: "add a rule that a
password-reset link expires after 30 minutes". By hand, the agent's steps are:

1. Register the domain: its file in `rules/domains/` and its row in `rules/domains/domains.md`.
2. Allocate the ID: `catalyst id next-rule br AUTH` prints `br-AUTH-000001-<your userid>`.
3. Write the rule under that heading in `rules/business-rules.md`, with its domain, its status and its strength, and
   list it in `rules/rules.md`.
4. Journal the change: `catalyst journal append --command chore --action create --artifact "the AUTH domain" --intent
   "why" --file <each file> --tier chore`.

The eight-character suffix of every ID is your userid, so IDs from different people never collide.

## 5. A first piece of work

Work is an artifact that names the rule it serves. With the software-engineering module, a feature is a requirement:

```sh
catalyst new REQ --title "Reset links expire" --field Targets=br-AUTH-000001-<userid> --field Domain=AUTH \
    --command /create-req --tier feature
```

`new` allocates the ID, fills the template, signs it, indexes it and journals it. As the work goes, the agent opens
steps under the requirement, writes the code, and journals each file it changes, saying which rule the change served
and why. A commit then cites the requirement's ID.

## 6. Check

```sh
catalyst check
```

`check` verifies the chain: every artifact grounded in a rule, every rule in a domain, every ID well formed and unique,
every changed file journaled. An agent's turn ends on it: the end-of-turn hook blocks the agent until what it broke
passes again.

## Next steps

- **A team**: `catalyst share create <url>` publishes the criterion to its own private repository; teammates run
  `catalyst open` in their clone. See [Working together](07-working-together.md).
- **An existing codebase**: rules can be gathered as you touch each area, or bootstrapped by a four-eyes analysis of
  the code. See [Analysing existing code](11-analysing-existing-code.md).

## Where to go next

- [The ten laws](03-the-ten-laws.md): what the agent judges for itself.
- [Working day to day](05-working-day-to-day.md): tiers, the journal, adopting hand-made changes.
- Reference: the
  [installation guide](https://github.com/cantica-tech/catalyst/blob/main/framework/kernel/INSTANTIATION-GUIDE.md) and
  its [checklist](https://github.com/cantica-tech/catalyst/blob/main/framework/kernel/INSTANTIATION-CHECKLIST.md), the
  [CLI](https://github.com/cantica-tech/catalyst/blob/main/framework/kernel/CLI.md), and
  [catalyst-users](https://github.com/cantica-tech/catalyst-users/blob/main/README.md).
