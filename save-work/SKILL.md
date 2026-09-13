---
name: save-work
description: Commit and push the current repository's work to GitHub so it is safe and available on every other device. Use this whenever the user says "save", "save my work", "commit and push", "sync this", "push this up", "I'm switching devices", "I'm heading out", or otherwise signals they are done with a chunk of work or about to move between their laptop, phone, or a cloud session. Also use it proactively at the end of a completed task when the user works across multiple machines, since unpushed work on one device is the main way their workflow breaks.
---

# Save work

Commit and push the current repository so the work exists on GitHub and any other
device can pick it up.

The user works across a MacBook Pro and cloud sessions (phone, web, desktop app).
GitHub is the single source of truth between them. The failure this skill exists to
prevent is work stranded on one device: if it was never pushed, the next device
starts from stale code and the divergence has to be untangled by hand later. That is
why pushing matters more than tidy history here.

## Steps

Work through these in order. Report at the end, not at each step.

### 1. Confirm the situation

Run `git status` and `git rev-parse --abbrev-ref HEAD`.

Stop and tell the user if any of the following is true, rather than guessing:

- Not inside a git repository
- HEAD is detached (not on a branch)
- There is no remote configured (`git remote -v` is empty)

If there is nothing to commit and the local branch is not ahead of the remote, say
so plainly and stop. Do not create an empty commit.

If there is nothing staged or modified but the branch is ahead of its remote, skip
to step 4 and just push. This is the common case where a previous session committed
but failed to push.

### 2. Pull first

Run `git pull --rebase`.

Pulling before committing catches work pushed from the user's other device. Rebase
keeps the history linear rather than littering it with merge commits from routine
device switching.

If the branch has no upstream set yet, skip the pull. Step 4 handles setting it.

**If the rebase hits a conflict, stop immediately.** Run `git rebase --abort` to put
the repository back exactly where it was, then tell the user which files conflicted
and that their work is untouched and uncommitted. Do not attempt to resolve it.
Conflicts mean two devices edited the same lines and only the user knows which
version is right. Wait for their direction.

### 3. Stage and commit

Run `git add -A`, then review `git diff --cached --stat` and enough of
`git diff --cached` to actually understand what changed.

Write a commit message that describes the change, not the moment. A future reader
scanning `git log` should be able to tell what happened without opening the diff.
Use the conventional-commit shape:

```
<type>(<scope>): <what changed, imperative, under ~70 chars>
```

Types: `feat`, `fix`, `refactor`, `docs`, `chore`, `test`, `style`.

**Example 1**
Changes: added JWT validation middleware and wired it into the auth routes
Message: `feat(auth): add JWT validation middleware to auth routes`

**Example 2**
Changes: edits across three unrelated files during an exploratory session
Message: `chore: update parser config, README, and test fixtures`

**Example 3**
Changes: a single typo in a heading
Message: `docs: fix typo in setup heading`

Never write a message that is only a timestamp, "update", "wip", or "save". Those
are exactly what makes history useless six weeks later. If the changes genuinely
span several unrelated things, say so in the message rather than picking one and
ignoring the rest.

Add a body only when the *why* is not obvious from the subject line.

### 4. Push

Run `git push`.

If the branch has no upstream, run `git push -u origin <branch>` instead so future
pushes are a bare `git push`.

**Cloud sessions can only push to the session's current working branch.** The
GitHub proxy that authenticates cloud sessions refuses pushes to any other branch.
If a push is rejected this way, do not try to work around it: tell the user which
branch the session is on and let them decide.

If the push is rejected because the remote moved during the commit, run
`git pull --rebase` once and push again. If it is rejected a second time, stop and
report rather than looping.

Never use `--force` or `--force-with-lease`. This skill is invoked casually and
often; a force push is never the right default in that context.

### 5. Report

One line, in this shape:

```
Saved: <commit subject> — pushed to <branch> on <remote>
```

If anything was unusual (nothing to commit, upstream newly set, a pull that brought
in changes from another device), add one more line saying so. The user is often
walking out the door when they invoke this, so the report needs to be scannable at
a glance and honest about anything that did not go cleanly.

## What this skill does not do

- Does not create branches, open pull requests, or merge anything
- Does not resolve conflicts
- Does not force push
- Does not commit secrets: if `git status` shows a `.env`, a `*.pem`, an
  `id_rsa`, a `*.tfvars`, or anything else that looks like credentials and it is
  not already tracked and not already ignored, stop and tell the user before
  staging. Once a secret is pushed it is in history and effectively public.
