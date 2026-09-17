---
name: vikunja
description: The shared task board (Vikunja at tasks.maybloom.space) for Miguel and the coding agents. Use at the start of a homelab, Maybloom or OCF session to see open work, and whenever work is filed, picked up, blocked, shipped or closed. Also when Miguel mentions tasks, the board, or asks what is pending.
---

# Vikunja task board

Talk to it only through `vk`. The plugin puts it on the Bash tool's PATH;
outside Claude Code it is `${CLAUDE_PLUGIN_ROOT}/bin/vk`.
Never curl the API by hand and never dump raw JSON into the conversation:
`vk` prints one line per task and that is all the context a session needs.

## Session routine

1. `vk recent -d 7` (what Miguel closed or commented since last time), then
   `vk ls -a claude` (mine) and `vk ls -p <project>` for the area at hand.
2. Pick up a task: `vk mv <id> Doing` when starting it.
3. Ship: `vk done <id> -c "infra #51 merged, deployed 2026-09-16 21:10"`.
   The closing comment is the log: PR numbers, deploy time, what was verified.
4. Something for Miguel to decide or do: `vk add`/`vk edit` with
   `-a miguel` and the right label, and say so in chat once. Do not ask him
   about it again; he marks it done on the board.
5. New work discovered mid-session: file it right away (`vk add`), do not
   keep it in memory files. Memory files hold knowledge; the board holds
   state.

## Commands

```
vk ls [-p PROJECT] [-a USER] [-l LABEL] [--all]   open tasks, priority first
vk show ID...                                     description + comments
vk add -p PROJECT "Title" [-d DESC] [-a USER] [-l LABEL] [-P 0-5] [--due YYYY-MM-DD] [-b BUCKET]
vk edit ID... [-t TITLE] [-d DESC] [-a USER] [--unassign USER] [-l LABEL] [--unlabel LABEL] [-P N] [--due D|none] [-b BUCKET] [-p PROJECT]
vk done ID... [-c "closing note"] [--undo]
vk comment ID "text"
vk mv ID BUCKET
vk recent [-d DAYS] [-p PROJECT]
vk projects | vk labels | vk buckets PROJECT
vk mkproject "Title" [--parent PROJECT] | vk mkbucket PROJECT Name... [--done Name]
```

PROJECT is a name (`Homelab`), a path (`Maybloom/Homelab`) or an id.
Descriptions and comments are plain text; blank lines make paragraphs.

## Layout and conventions

Projects: `Maybloom/Homelab` (network, NUC, intranet services, the infra
repo), `Maybloom/System` (the Maybloom app and ecosystem, the maybloom
repo), `Maybloom/Homestead` (Miguel's in-person land work; read, never
file into it), `OCF/IMS` (the ocf-ims work and its staging).

Buckets in every project: Backlog, Next, Doing, Waiting, Done (Done is the
done bucket: `vk done` moves the card there and `vk mv ID Done` marks it
done; new tasks land in Backlog).

Labels say what kind of step is needed, assignee says who:
`decision` (needs Miguel's call, nothing to build), `hands-on` (physical or
credential work only Miguel can do), `waiting` (blocked on something
external, say what in a comment). Priority 0-5 is urgency; due dates only
when a date is real.

Title = one imperative sentence. Description = the facts needed to resume
(paths, ids, the verified gotchas). Comments = the log.

Security trade-offs still get an explicit yes from Miguel in chat before
they are applied, card or no card.

Projects are owned by `claude` and shared with Miguel as admin (children
inherit). User search is closed to API tokens, so sharing is by username:
`POST /projects/ID/users {"username": ..., "permission": 2}` (v2), a
one-off not in `vk`.

## Setup on a new machine

Install the plugin (the repo is its own marketplace; SSH because it is
private):

```
claude plugin marketplace add git@github.com:MaybloomTech/vikunja-skill.git
claude plugin install vikunja@vikunja-skill
```

Token per machine, named after the host, created in the `claude` account
(Settings → API Tokens; scopes: projects, tasks incl. comments, labels,
assignees, buckets/views). Then `vk init <token>`; it lands in
`~/.config/vikunja/token` (0600), never in a repo. The URL defaults to
https://tasks.maybloom.space (LAN + tailnet); `--url` or
`~/.config/vikunja/url` overrides it.
