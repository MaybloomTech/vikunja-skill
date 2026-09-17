# vikunja-skill

A Claude Code plugin for the shared Vikunja task board, and the marketplace
that serves it:

```
.claude-plugin/marketplace.json   this repo as a one-plugin marketplace
.claude-plugin/plugin.json        the plugin, `vikunja`
skills/vikunja/SKILL.md           when and how the agents use the board
bin/vk                            the client; on the Bash tool's PATH while the plugin is enabled
```

`vk` is a small Vikunja client that prints one line per task. Python
standard library only; Vikunja API v2 (2.4+). The skill is what makes it
more than a CLI: the session routine, the project layout and the label
conventions live there.

## Install

```
claude plugin marketplace add git@github.com:MaybloomTech/vikunja-skill.git
claude plugin install vikunja@vikunja-skill
```

(or `/plugin marketplace add …` and `/plugin install …` inside a session).
The repo is private, so the SSH URL; the machine's GitHub key needs read
access. The skill shows up as `vikunja:vikunja`.

Then give the machine its own token:

1. Sign in to Vikunja as `claude`, Settings → API Tokens, new token named
   after the host. Scopes: projects, tasks (incl. comments), labels,
   assignees, buckets/views.
2. `vk init <token>` from a Claude Code session (`! vk init <token>`), or
   `bin/vk init <token>` from a checkout. It stores the token in
   `~/.config/vikunja/token` (0600) and prints how many projects it can see.

The token never goes in a repo. One token per machine, so a lost laptop is
one token to revoke.

To type `vk` in an ordinary shell, clone the repo anywhere and
`ln -s <checkout>/bin/vk ~/.local/bin/vk`.

## Reaching the board

The default URL is only served to the LAN and the tailnet, so the machine
has to be on one of them and resolve the name through the home resolver.
A 404 from `vk init` on a machine that is on the tailnet means the name
resolved through a public resolver and the request came back in through
the WAN side. `vk init <token> --url URL` (or `VIKUNJA_URL`, or
`~/.config/vikunja/url`) points it somewhere else.

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

Moved out of MaybloomTech/infra (`skills/vikunja`, MaybloomTech/infra#52).
