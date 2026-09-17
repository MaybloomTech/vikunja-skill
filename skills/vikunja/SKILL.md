---
name: vikunja
description: The team's shared Vikunja task board, where humans and coding agents track open work. Use at the start of a working session to see what is open, and whenever work is filed, picked up, blocked, shipped or closed. Also when the user mentions tasks, the board, or asks what is pending.
---

# Vikunja task board

Talk to it only through `vk`. The plugin puts it on the Bash tool's PATH;
outside Claude Code it is `${CLAUDE_PLUGIN_ROOT}/bin/vk`.
Never curl the API by hand and never dump raw JSON into the conversation:
`vk` prints one line per task and that is all the context a session needs.

## Session routine

1. `vk conventions` once per session: the team's own rules (project layout,
   bucket and label meanings, the agents' account name, anything else they
   decided). They live on the board itself, so they are the same on every
   machine. Follow them over the defaults below.
2. `vk recent -d 7` (what changed since last time), then `vk ls -a <agent
   account>` (mine) and `vk ls -p <project>` for the area at hand.
3. Pick up a task: `vk mv <id> <doing bucket>` when starting it.
4. Ship: `vk done <id> -c "PR #51 merged, deployed 2026-09-16 21:10"`.
   The closing comment is the log: PR numbers, deploy time, what was verified.
5. Something for a human to decide or do: `vk add`/`vk edit` assigned to
   them with the right label, and say so in chat once. Do not ask again;
   they mark it done on the board.
6. New work discovered mid-session: file it right away (`vk add`), do not
   keep it in memory files. Memory files hold knowledge; the board holds
   state.

## Commands

```
vk conventions [--set FILE]                       the team's rules (FILE or - to write them)
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

PROJECT is a name (`Homelab`), a path (`Team/Homelab`) or an id.
Descriptions and comments are plain text: blank lines make paragraphs,
`# ` headings, `- ` and `1. ` lists and `code` spans survive the trip
through Vikunja's editor.

## Defaults (the conventions may override any of them)

- Buckets are per project (`vk buckets PROJECT`). `vk done` marks the task
  done, which Vikunja moves to the view's done bucket when one is set;
  `vk mv ID <done bucket>` does the reverse.
- Labels say what kind of step is needed, assignee says who. Priority 0-5
  is urgency; due dates only when a date is real.
- Title = one imperative sentence. Description = the facts needed to resume
  (paths, ids, the verified gotchas). Comments = the log.
- A task being on the board is not approval. Anything the team treats as
  needing a human's explicit yes (security trade-offs, destructive changes)
  still gets it in chat before it is applied.

## Setup on a new machine

Install the plugin (the repo is its own marketplace):

```
claude plugin marketplace add git@github.com:MaybloomTech/vikunja-skill.git
claude plugin install vikunja@vikunja-skill
```

One API token per machine, named after the host, created in the agents'
Vikunja account (Settings → API Tokens; scopes: projects, tasks incl.
comments, labels, assignees, buckets/views). Then
`vk init <token> --url https://<instance>`; both land in
`~/.config/vikunja/` (`token` is 0600), never in a repo. `VIKUNJA_URL` /
`VIKUNJA_TOKEN` override them, `VIKUNJA_CONVENTIONS` names another
project than `Agents` for the conventions, and a
`~/.config/vikunja/conventions.md` is appended to `vk conventions` for
machine-local notes.
