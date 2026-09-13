---
name: start-project
description: Scaffold a brand new project and create its GitHub repository in one step, without going to github.com. Use this whenever the user says "start a new project", "spin up a repo", "new repo", "create a project for X", "set up a repo for this", "make this a repo", or describes wanting to begin something new that should live in version control. Also use it when the user has loose files or an idea that is not yet under git and wants it turned into a proper repository they can reach from their other devices.
---

# Start project

Create a new project locally, create its GitHub repository, connect the two, and
push the first commit — so the project exists on every device from minute one.

The user works across a MacBook Pro and cloud sessions. A project that exists only
on one machine cannot be picked up from the phone later. Creating the remote at
birth, rather than "when I get around to it", is what makes the rest of their
workflow work.

## Before starting

Confirm these with the user in a single question if they are not already clear from
the conversation. Do not invent answers.

- **Name** — becomes the folder name and the repository name. Lowercase, hyphenated.
- **Owner** — the user belongs to several GitHub organizations as well as their
  personal account, so this is never assumed. See "Choosing the owner" below.
- **What it is** — one sentence, used for the repo description and the README.

Ask for all of these in one message rather than one at a time.

**Visibility is not one of the questions.** Every new repository is private, for
every owner, personal and organizational alike. Create it public only when the user
says public in that specific request. Do not ask which they want, do not offer
public as an option, and do not treat a previous public repo as a precedent.

If the user is already inside a folder of existing files and wants *that* turned
into a repo, skip the scaffolding in step 1 and work with what is there.

## Choosing the owner

The user has a personal GitHub account and belongs to multiple organizations. The
right owner is a judgment call about who the work belongs to, and getting it wrong
means a repo sitting in the wrong place with the wrong people able to see it. So
never guess and never default to personal silently.

If the user already named an owner in their request ("spin up a repo under
<org> for..."), use it and don't ask.

Otherwise, run `gh org list` to get the organizations the authenticated account can
actually create in, and present those plus the personal account as a short list for
the user to pick from. Listing them live rather than from memory means the choice
stays correct as memberships change.

If `gh org list` returns nothing or errors, ask the user to name the owner directly
rather than assuming personal.

One caveat worth surfacing if the user seems unsure: creating a repo in an
organization requires the member permission to do so, and the repo belongs to that
organization, not to them. Moving it afterwards needs admin rights on both sides,
so it is much cheaper to pick correctly now than to correct later.

## Steps

### 1. Scaffold locally

Create the directory and the starting files:

- `README.md` — the project name as an H1, the one-sentence description, and a
  short "Getting started" placeholder
- `.gitignore` — appropriate to the stack. If the stack is not obvious, ask rather
  than guessing. Always include OS and editor noise (`.DS_Store`, `.vscode/`,
  `.idea/`) and always include `.env` and `.env.*` regardless of stack.
- `CLAUDE.md` — a short file capturing what this project is and any conventions
  established so far. This one matters more than it looks: a repo's `CLAUDE.md`
  travels into cloud sessions with the clone, so it is how context reaches the
  phone.
- `docs/decisions/` — decision records, seeded with the template and first record
  described below.

Then add stack-specific scaffolding only where the user named a stack. Do not
invent `src/`, `tests/`, `lib/`, or similar ahead of need: git does not track empty
directories, so they either vanish or have to be propped up with placeholder files,
and either way they are clutter that implies structure the project has not earned
yet. Add each folder when there is a real file to put in it.

#### Decision records

Create `docs/decisions/TEMPLATE.md`:

```markdown
# <number>. <short title in the form of the decision made>

Date: <YYYY-MM-DD>
Status: Accepted

## Context

What situation forced a choice. The constraints, what was already true, what was
unknown at the time.

## Decision

What was chosen, stated plainly.

## Consequences

What this makes easy, what it makes hard, and what would have to change to undo it.
```

Then create the first record, `docs/decisions/0001-use-decision-records.md`, filled
in from the template: the context is that this project will be picked up across
devices and across long gaps, the decision is to record significant choices here as
they are made, and the consequence is a small habit cost now against not having to
reconstruct reasoning later.

Number records sequentially with a four-digit prefix and a hyphenated title. Never
edit an accepted record to reflect a change of mind: write a new record that
supersedes it and mark the old one `Status: Superseded by <number>`. The value is
in seeing what was believed at the time, which an edited record destroys.

#### The `.claude/` directory

A repo's `.claude/` directory travels into cloud sessions with the clone, unlike
the user's machine-level `~/.claude/`, so it is the only way to give project
context to a phone session. Anything project-specific belongs there:
`.claude/skills/`, `.claude/commands/`, `.claude/rules/`.

Do not create it empty at scaffold time, for the same reason as any other empty
directory. Create it the moment there is a first rule or command to put in it, and
mention in `CLAUDE.md` that it is where project-scoped Claude configuration goes so
the convention is discoverable later.

If the user states any project conventions during setup, those are the first
content: put them in `.claude/rules/` and create the directory then.

### 2. Initialize git

```bash
git init
git add -A
git commit -m "chore: initial commit"
```

Set the branch to `main` if it is not already: `git branch -M main`.

### 3. Create the GitHub repository

Use the `gh` CLI, which authenticates as the user without a browser trip:

```bash
gh repo create <owner>/<name> --private --source=. --remote=origin --description "<description>"
```

Always qualify the name with the owner, even for the personal account. Passing a
bare `<name>` silently creates it under whichever account `gh` happens to be
authenticated as, which is exactly the mistake the owner question exists to
prevent.

Use `--public` only when the user explicitly asked for a public repo in this
request. Otherwise `--private` stands, whatever the owner.

`--source=.` tells `gh` this is an existing local repo, and `--remote=origin` wires
the remote up in the same command, so there is no separate `git remote add` step.

**If `gh` is not authenticated**, it will say so. Tell the user to run
`gh auth login` and stop; do not try to authenticate on their behalf.

**If creation is refused for the chosen organization**, report the refusal as-is
and ask whether to use a different owner. Do not quietly fall back to the personal
account. The usual causes are the `gh` token lacking organization scope (fixed with
`gh auth refresh -s read:org,repo`) or the organization restricting who can create
repositories.

**If this is running in a cloud session, repository creation will likely fail.**
Cloud sessions reach GitHub through a proxy scoped to the repositories attached to
that session, so creating a brand new repo is usually refused. If the command is
rejected that way, say so plainly and tell the user this step needs to run on their
MacBook. Do not retry with a different method or look for a workaround.

### 4. Push

```bash
git push -u origin main
```

The `-u` matters: it sets the upstream so every later `git push` — including the
ones the `save-work` skill runs — works without arguments.

### 5. Report

Report in this shape:

```
Created <owner>/<name> (<private|public>) — pushed to <repo URL>
Local path: <path>
```

Then state in one line what the project contains, so the user can confirm the
scaffolding matched what they had in mind.

## What this skill does not do

- Does not choose the owner on the user's behalf, or fall back to personal when an
  organization is refused.
- Does not add collaborators, branch protection, or CI.
- Does not create a public repository unless the user says public in that request.
  Private is the recoverable choice: a private repo can be opened up later, but
  code that was briefly public has to be treated as exposed.
