---
name: agent-context-file-creator
description: >
    Create or update an agent context file (`CLAUDE.md`, `AGENTS.md`,
    `.cursorrules`, and similar) that tells AI agents how to work in a
    project. Use whenever the user wants to bootstrap, write, or improve a
    context/instructions file for an agent, onboard an agent to a repository,
    or capture a project's purpose, environment, conventions, and workflow
    for AI use.
---

# Goal

Produce a context file that lets an agent work correctly in the project
from the first session: what the system does, how to run it, the
conventions to follow, and the guardrails to respect.

The file is reloaded into context on every session, so every line costs
tokens on every run. Favor short, high-signal statements over prose, and
record only what an agent cannot infer on its own.

## Discover before you ask

Inspect the repository first, and only ask the user about what the code
cannot reveal. This keeps the interview short and the file accurate.

Good sources to read before asking:

-   `README*`, existing `CLAUDE.md`/`AGENTS.md`, docs
-   `package.json`, `composer.json`, `Makefile`, or equivalent (commands)
-   `docker-compose.*`, `Dockerfile` (environment and services)
-   CI config, `.editorconfig`, formatter/linter/commitlint config
    (conventions)
-   recent git history (workflow, commit-message style)

Only after this, ask the user for the things that are not in the code —
business purpose, access domains, permissions, and testing expectations
(see the template's inline notes).

## Interview the user

Group the open questions into one short exchange instead of asking them
one at a time. Some questions are sensitive (domains, admin panels,
credentials) — tell the user they may refuse any of them.

Highlight the two questions that most change how an agent behaves, and
make sure they are answered:

-   **May the agent commit its own changes, or will a human review them
    first?**
-   **Should the agent write tests, which kind, and run them when done?**

## Never persist secrets

A context file is usually committed to version control. Never write
plaintext credentials, API keys, or tokens into it. 
Create a custom `.env.agent` file to persist any access information.

## Keep it maintainable

-   Reference existing config files instead of restating their rules
    (e.g. point to `.prettierrc` rather than re-listing formatting rules) —
    restated rules drift out of sync with the source.
-   Delete sections that do not apply rather than filling them with filler.
-   When updating an existing file, preserve its structure and only change
    what is stale or missing.

## Template

Use [this template](./references/template.md) as the starting structure
for the context file, and follow its notes for what to gather in
each section.
