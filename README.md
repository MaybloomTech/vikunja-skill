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
standard library only; Vikunja API v2 (2.4+). One HTTPS connection per
run, so a command is one name lookup and one handshake however many
requests it makes. `vk start` is the whole session routine in one call;
`vk recent` reports what changed since it last ran on the machine (the
stamp lives in `~/.config/vikunja/last-recent`) and shows only the
comments other people wrote, clipped to two lines.

The skill carries the team-independent part: a session routine, and
defaults for buckets (`Backlog, Next, Doing, Waiting, Done`, created by
`vk mkproject`), labels (`decision`, `hands-on`, `waiting`) and how tasks
are written. What is specific to a team lives on its board: each project's
description says what goes in it and any rules or labels of its own, and
the agent reads them at the start of a session with `vk projects -v`.

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

3. Once per instance, in the web UI as the agents' account: open each
   top-level project, ⋯ → Subscribe (children inherit). `vk inbox` reads
   that account's notifications, and Vikunja only notifies subscribers;
   without it, a card someone else creates and does not assign to the
   agent stays silent. API tokens cannot subscribe, and the token needs
   the notifications scope.

The token never goes in a repo. One token per machine, so a lost laptop is
one token to revoke. `VIKUNJA_URL` and `VIKUNJA_TOKEN` override the files.

To type `vk` in an ordinary shell, clone the repo anywhere and
`ln -s <checkout>/bin/vk ~/.local/bin/vk`.

## Setting up a board

```
vk mkproject "Team"                                   a parent
vk mkproject "Backend" --parent Team -d "The API and its deploys."
vk project Backend -d "..."                           change the description later
vk project Backend                                    description and buckets
```

Descriptions are plain text with a little structure that survives the
Vikunja editor: blank lines make paragraphs, `# ` headings, `- ` and `1. `
lists, `code` spans. Humans edit them in the Vikunja UI like any project
description.

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
