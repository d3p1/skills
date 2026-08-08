# [GUIDE]

<!--
Fill each section by reading the repository first, then asking the user
only for what the code cannot reveal. Keep every line high-signal: this
file is reloaded into context on every session. Delete any section that
does not apply, and remove HTML comments from the final file.
-->

## System

Functional overview: what the system does, its goal, main features, and
third-party integrations, from a business/product perspective.

Ask the user (they may refuse any of these):
- The domain(s) where the system is accessible.
- Whether admin panels exist and how to reach them.
Record only the location of any stored credentials here, never the credentials
themselves — see "Access & secrets" below.

## Environment

Technical overview: stack, services, and how they run. If the project uses
containers, list each one, its role, and the command used to interact with
it.

Prefer discovering this from docker-compose.*, Dockerfile, package.json,
Makefile, or CI config before asking the user.

## Commands

The commands an agent will actually run, one per line and copy-pasteable:
install, build, run, test, lint, format.

Pull these from package.json scripts, Makefile, composer.json, etc.

## Development

Development practices and rules to be taken into consideration.

### Conventions

Coding style, naming, formatting, commit-message convention, and any
repo-specific patterns. Reference the config that enforces each rule
(`.editorconfig`, `.prettierrc`, linter config, commitlint) instead of
restating it, so this file cannot drift out of sync.

### Workflow

How the agent is expected to implement features.

Ask the user for the following. Mark the last two as IMPORTANT in the
final file, since they change how the agent behaves:
- The `@author` value for file DocBlock headers (check nearby files first
  to match the existing value).
- IMPORTANT: May the agent commit its changes, or will a human review them
  first?
- IMPORTANT: Should the agent write tests? Which kind (unit, integration,
  e2e)? Should it run them after finishing a change?

## Access & secrets

Mention the use of the `.env.agent` file to 
access credentials and secrets.