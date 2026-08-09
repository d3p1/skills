# [GUIDE]

## Introduction

This repository is a collection of [Agent Skills](https://agentskills.io/home), meant to be cloned into an agent's skills folder (e.g. `.claude/skills`).

## Architecture

- Each skill lives in its own directory under `src/`, named after the skill (e.g. `src/test/`).
- Every skill directory contains a `SKILL.md` with YAML frontmatter (`name`, `description`) followed by the skill's instruction body.
- When adding a new skill, create a new `src/<skill-name>/SKILL.md` following this same frontmatter + body structure.

## Commands

```shell
npm run format:code        
npm run format:code:fix 
```

## Conventions

- Commit messages must follow the `@d3p1/commitlint-config` (Angular-style) convention, enforced by a husky `commit-msg` hook running `commitlint`. Non-conforming commits will be rejected.
- Releases are fully automated via `semantic-release` (`d3p1/semantic-releasify` GitHub Action) on push to `main` — version bumps and `CHANGELOG.md` are derived from commit messages, so do not hand-edit the version in `package.json` or `CHANGELOG.md`.
- Formatting is enforced by Prettier with project-specific settings in `.prettierrc.json` (4-space tabs, no semicolons, single quotes).
