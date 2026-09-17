# vikunja-skill

A Claude Code plugin for a shared [Vikunja](https://vikunja.io) task board,
and the marketplace that serves it:

```
.claude-plugin/marketplace.json   this repo as a one-plugin marketplace
.claude-plugin/plugin.json        the plugin, `vikunja`
skills/vikunja/SKILL.md           when and how an agent uses the board
bin/vk                            the client; on the Bash tool's PATH while the plugin is enabled
```

`vk` is a small Vikunja client that prints one line per task, so a session
spends a few dozen tokens on the board instead of pages of JSON. Python
standard library only; Vikunja API v2 (2.4+).

Nothing in the repo is specific to one team. The team's own rules (project
layout, what the buckets and labels mean, who the agents' account is) live
on the board itself, as the description of a project named `Agents`.
`vk conventions` prints them, every machine with a token sees the same
text, and humans edit them in the Vikunja UI.

## Install

```
claude plugin marketplace add git@github.com:MaybloomTech/vikunja-skill.git
claude plugin install vikunja@vikunja-skill
```

(or `/plugin marketplace add …` and `/plugin install …` inside a session).
The skill shows up as `vikunja:vikunja`.

Then point the machine at the instance:

1. Sign in to Vikunja as the account the agents will use, Settings → API
   Tokens, new token named after the machine. Scopes: projects, tasks
   (incl. comments), labels, assignees, buckets/views.
2. `vk init <token> --url https://tasks.example.org`, from a Claude Code
   session (`! vk init …`) or as `bin/vk` from a checkout. It stores both
   in `~/.config/vikunja/` (`token` is 0600) and prints how many projects
   it can see.

The token never goes in a repo. One token per machine, so a lost laptop is
one token to revoke. `VIKUNJA_URL` and `VIKUNJA_TOKEN` override the files.

To type `vk` in an ordinary shell, clone the repo anywhere and
`ln -s <checkout>/bin/vk ~/.local/bin/vk`.

## Conventions

```
vk conventions                 print them
vk conventions --set FILE      write FILE (or - for stdin) as the description of the
                               Agents project, creating the project if needed
```

Descriptions are plain text with a little structure that survives the
Vikunja editor: blank lines make paragraphs, `# ` headings, `- ` and `1. `
lists, `code` spans. `VIKUNJA_CONVENTIONS` names another project than
`Agents`. A `~/.config/vikunja/conventions.md`, if present, is appended for
machine-local notes.

The first time, write them from a file and share the project with the
humans who should edit it. The skill tells the agent to read them once
per session and to follow them over its own defaults.

## Update

`plugin.json` carries no `version` on purpose: every commit on `main` is a
release.

```
claude plugin marketplace update vikunja-skill
claude plugin update vikunja@vikunja-skill
```

Changes go through a PR here; `SKILL.md` documents every command, so keep
it in step with `vk`. `claude plugin validate .` checks the manifests, and
`claude --plugin-dir .` tries a working copy without installing it.
