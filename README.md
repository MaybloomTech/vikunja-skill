# vikunja-skill

A Claude Code skill for the shared Vikunja task board: `SKILL.md` (when and
how the agents use the board) and `vk`, a small Vikunja client that prints
one line per task. Python standard library only; Vikunja API v2 (2.4+).

The repo root is the skill directory, so a checkout is an install.

## Install

```
git clone git@github.com:MaybloomTech/vikunja-skill.git ~/.claude/skills/vikunja
```

Then give the machine its own token:

1. Sign in to Vikunja as `claude`, Settings → API Tokens, new token named
   after the host. Scopes: projects, tasks (incl. comments), labels,
   assignees, buckets/views.
2. `~/.claude/skills/vikunja/vk init <token>` stores it in
   `~/.config/vikunja/token` (0600) and prints how many projects it can see.

The token never goes in a repo. One token per machine, so a lost laptop is
one token to revoke.

Optional, to type `vk` in a shell: `ln -s ~/.claude/skills/vikunja/vk ~/.local/bin/vk`.

## Reaching the board

The default URL is only served to the LAN and the tailnet, so the machine
has to be on one of them and resolve the name through the home resolver.
A 404 from `vk init` on a machine that is on the tailnet means the name
resolved through a public resolver and the request came back in through
the WAN side. `vk init <token> --url URL` (or `VIKUNJA_URL`, or
`~/.config/vikunja/url`) points it somewhere else.

## Update

```
git -C ~/.claude/skills/vikunja pull
```

Changes go through a PR here; `SKILL.md` documents every command, so keep
it in step with `vk`.

Moved out of MaybloomTech/infra (`skills/vikunja`, MaybloomTech/infra#52).
