# First-commit runbook (the guided ceremony)

The operator's first hand-written commit is a milestone session. Run it with less-more-often pacing: ONE instruction per message, wait for "done" before the next. Everything below assumes the edit itself is already made and verified (source it from the outliers ledger — a one-line fix with known shape).

## Pre-flight (tutor, silently)

- Verify the edit is on disk and correct by reading the file yourself, before any git ceremony starts.
- Confirm what SHOULD appear in status: which files modified, which untracked files are expected noise (handbacks, scratch dirs). Untracked expected-noise is worth naming to the operator up front: `??` means "git isn't watching this" — untracked ≠ problem.
- Know the repo's fence (ignore patterns) so you can explain any file that is conspicuously absent.

## The ceremony, one command at a time

1. `git status --short` — teach the columns: left = staged (green), right = unstaged (red), `??` = untracked. Ask what they see before telling them what they should see.
2. `git add <file1> <file2>` — by name, never `-A`. Explain: silence is success in git; only what you name goes in.
3. `git status --short` again — the M moves to the left column and turns green. "In the loading bay."
4. `git commit -m "<message>"` — write the message together, format `type: what and why`. Their hash comes back: that is their name in the permanent history.
5. `git push` — the commit exists only on their machine until this. The output narrates objects being counted, compressed, written; the last line shows old..new on the branch.

Then the tutor verifies independently — log shows the commit, the commit's stat matches exactly the intended files, remote and local are the same hash — and reports the verification to the operator. Then celebrate. Specifically and honestly: name what they did that they could not do a week ago.

## The standard errors (teach through them, never around them)

**Paste-the-prose.** The operator pastes your whole message — prose, file summaries, and all — into the shell, which erupts in red parser errors. Nothing ran; shells reject a paste they cannot parse (in PowerShell the whole block fails as one parse; in bash partial lines may run — know your shell). Teach: only what is inside the code box is a command. This error is a rite of passage; treat it lightly.

**Stale status.** `git status` output missing files you know are modified (or showing ghosts). On Windows, git's filesystem-watcher cache (fsmonitor) can go stale when other processes touch the repo. Prove it and bypass in one move: `git -c core.fsmonitor=false status --short`. Teach: tools cache for speed, and caches lie; when output contradicts known reality, suspect the cache before the reality.

**Lock files.** `index.lock` exists and git refuses to run: a previous git process died holding the lock. If the lock is genuinely stale (no git running), it is safe to remove — but on restricted mounts where deletion is blocked, move it aside instead. Teach: locks protect the repo from two writers; a dead process's lock is a leftover, not a corruption.

**Wrong-column confusion.** The operator stages, edits again, and sees the file in BOTH columns. Teach: git tracks the staged snapshot separately from the working file; re-add to update the snapshot.

## Why the ceremony is manual

Every step above could be automated away. That is exactly why it is not: the point of the first commit is not the commit, it is the operator feeling the machine respond to their own hands — status, stage, commit, push, each with visible cause and effect. Automation resumes the day after.
