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

1. `vk projects -v` once per session. Each project's description says what
   belongs in it and any rules or labels of its own; they override the
   defaults below.
2. `vk inbox`: my account's unread Vikunja notifications (comments by
   others, assignments, mentions), by task. Act on each (answer, do the
   work, or file it), then `vk ack <id>` marks them read. Never ack what
   you have not handled: the read state is shared by every machine, so an
   ack hides it everywhere. Then `vk recent -d 7` (everything that
   changed, including cards humans closed, which do not notify), `vk ls -a
   <my account>` and `vk ls -p <project>` for the area at hand.
3. Pick up a task: `vk mv <id> Doing` when starting it.
4. Ship: `vk done <id> -c "PR #51 merged, deployed 2026-09-16 21:10"`.
   The closing comment is the log: PR numbers, deploy time, what was verified.
5. Something for a human to decide or do: `vk add`/`vk edit` assigned to
   them with the right label, and say so in chat once. Do not ask again;
   they mark it done on the board.
6. New work discovered mid-session: file it right away (`vk add`) in the
   project whose description fits, do not keep it in memory files. Memory
   files hold knowledge; the board holds state.

## Commands

```
vk projects [-v]                                  projects as Parent/Child (-v: with descriptions)
vk project PROJECT [-d DESC] [--identifier LAB]                   one project: description and buckets (-d sets it)
vk ls [-p PROJECT] [-a USER] [-l LABEL] [--all]   open tasks, priority first
vk show ID...                                     description + comments
vk add -p PROJECT "Title" [-d DESC] [-a USER] [-l LABEL] [-P 0-5] [--due YYYY-MM-DD] [-b BUCKET]
vk edit ID... [-t TITLE] [-d DESC] [-a USER] [--unassign USER] [-l LABEL] [--unlabel LABEL] [-P N] [--due D|none] [-b BUCKET] [-p PROJECT]
vk done ID... [-c "closing note"] [--undo]
vk comment ID "text"
vk mv ID BUCKET
vk recent [-d DAYS] [-p PROJECT]
vk inbox                                          unread notifications, by task
vk ack ID...                                      mark a task's notifications read, once handled
vk labels | vk buckets PROJECT
vk mkproject "Title" [--parent PROJECT] [-d DESC] [--no-buckets]
vk mkbucket PROJECT Name... [--done Name] [--default Name]
```

ID is what `vk` prints: `LAB-3` (the project's identifier plus the
per-project number, the same thing the web UI shows) or a global `#29` for
projects without an identifier. Both forms are accepted everywhere; use the
printed one in chat, comments and PRs so humans can find the card. Set a
project's prefix with `vk project NAME --identifier LAB`.

PROJECT is a name (`Homelab`), a path (`Team/Homelab`) or an id.
Descriptions and comments are plain text: blank lines make paragraphs,
`# ` headings, `- ` and `1. ` lists and `code` spans survive the trip
through Vikunja's editor.

## Defaults

A project's description can override any of these for that project.

**Projects.** Parents group, children hold tasks. A project's description
is its conventions: what belongs in it, what does not, and any rule,
label or bucket set of its own. There is no other place for them. So:

- Creating a project means writing that description: `vk mkproject -d`,
  never a bare `vk mkproject`. A parent's says whether tasks go in it at
  all.
- When the user states a rule that applies to a project, put it in the
  description (`vk project PROJECT -d`) so every machine and session sees
  it, and say so in chat. Memory files are for what you learned, not for
  the team's rules.
- Reading it comes before filing into it.

**Buckets.** `vk mkproject` gives every project Backlog, Next, Doing,
Waiting, Done: new tasks land in Backlog, Done is the done bucket, so
`vk done` moves the card there and `vk mv ID Done` marks it done. A
project that needs different columns says so in its description and uses
`vk mkbucket`.

**Labels** say what kind of step is needed; the assignee says who:

- `decision`: needs a human's call, nothing to build.
- `hands-on`: physical or credential work only a human can do.
- `waiting`: blocked on something external; say what in a comment.

Labels are global in Vikunja, so keep the set small: use these three
first, use a label a project's description names, and create a new one
only when the user asks for it.

**Tasks.** Title = one imperative sentence. Description = the facts needed
to resume (paths, ids, the verified gotchas). Comments = the log.
Priority 0-5 is urgency; due dates only when a date is real.

**Approval.** A task being on the board is not approval. Anything the
team treats as needing a human's explicit yes (security trade-offs,
destructive changes) still gets it in chat before it is applied.

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
`~/.config/vikunja/` (`token` is 0600), never in a repo. `VIKUNJA_URL` and
`VIKUNJA_TOKEN` override them.
