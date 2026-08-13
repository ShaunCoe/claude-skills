---
name: operator-build-loop
description: >-
  A disciplined two-agent workflow for shipping real production software without
  writing code yourself, by pairing a planning/PM agent with an isolated coding
  agent like Claude Code. Use this whenever someone wants to build, ship, or change
  a feature, turn an idea into shipped code, write or review a prompt for a coding
  agent, plan a build, review a coding agent's plan or handback, run an overnight
  build queue, or direct Claude Code, especially a non-engineer or a small
  team who wants to ship safely while staying out of the codebase. Trigger on
  things like "build X", "ship this change", "write a prompt for Claude Code",
  "plan this feature", "review this plan before I run it", "review this handback",
  "help me direct the coding agent", or any request to move from idea to shipped
  code through an AI coding agent, even if the workflow is not named explicitly.
---

# Operator Build Loop

A repeatable way for a non-engineer (or a small team) to ship production software with AI, without ever touching code directly and without breaking what is live. It works by splitting the job across two agents with clear lanes, and never letting either do the other's work.

This is a real, battle-tested operating model: it has shipped multiple production apps and runs a live multi-tenant SaaS today. The point is not speed for its own sake. The point is shipping fast **and** safe, so a sharp operator can run engineering-grade output without being an engineer.

## The two roles (never blur them)

**The PM / strategist** (a planning agent, e.g. Claude in a chat or Cowork). Owns context and prompts. It absorbs what is actually going on, writes the build instructions, reviews the coding agent's plan, and independently verifies the finished work. It does **not** edit code. Keeping it out of the code is deliberate: its job is judgment, scope, and safety, and the moment it starts hand-editing files, that judgment gets compromised and mistakes leak straight into the repo.

**The executor** (the coding agent, e.g. Claude Code). Owns the code. It runs in its own isolated lane, takes **one** prompt at a time, plans before it builds, verifies before it ships, and writes a handback when it is done. The operator stays out of its blast radius. That isolation is the whole safety model: if the executor only ever acts on a reviewed, scoped prompt, a non-engineer can ship confidently because the dangerous surface is contained.

The operator (the human) directs. They give the goal and the product judgment, and they hold the keys: credentials, auth grants, and irreversible clicks stay human. They never have to read the diff line by line, because the workflow catches problems before they land.

## The loop

Run this every time you ship a change. Each step exists to remove a specific way things go wrong.

1. **Absorb context first.** Before writing anything, the PM reads the real state: prior session notes, the actual git log and live code, recent email or messages, and any saved memory. Trust the code and the git history over project docs. Docs drift; the running app is the truth. Skipping this is how you write a prompt against a codebase that no longer exists.

2. **Frame one scoped objective.** Decide the single thing this build does. Write down explicitly what it must **not** touch, the acceptance criteria, and a mandatory verification step. Scope is the cheapest place to prevent damage: a tightly bounded prompt cannot wander into the live payment path.

3. **Write the prompt to the repo root.** Save it as a dated file (YYYY-MM-DD-NN-topic-slug.md). The prompt names the files the executor may touch, the files it must not, the steps, how to verify, and what the handback must contain. See references/prompt-template.md for the full structure. Writing it to the repo root (not pasting it into a chat) gives the executor a durable, reviewable artifact and leaves a paper trail of every change.

4. **Executor plans first — do not let it build yet.** Run the coding agent in plan mode on the prompt. This costs one extra round trip and is the single highest-leverage safety step, especially for a big or one-shot build you cannot easily redo. A plan is cheap to throw away; an hour of wrong building is not. If the executor comes back with questions instead of a plan, that is a feature: it found something the prompt got wrong about reality (a schema detail, a file that moved). Answer the questions — usually by taking its recommended option — and let it re-plan. The executor knows the code better than the prompt does; the prompt knows the intent better than the executor does.

5. **PM reviews the plan against the four-check list** (below) before any code is written. Green-light or send it back. This is where a non-engineer gets engineering-grade safety: the reviewer catches scope creep, missing modules, and missing verification while it is still just text.

6. **Execute.** Once the plan passes, the executor runs it (autonomous/bypass is fine here, because the scope and plan are already vetted). Higher reasoning effort is worth it for large or one-shot builds.

7. **Verify locally before pushing.** The executor runs it locally, takes screenshots or runs tests, confirms it actually works. Do not verify by poking the production URL. Catch regressions on your machine, not in front of customers.

8. **Stage by name, never blanket-add.** The executor commits with an explicit file list (`git add <file> <file>`), never `git add -A` or `git add .`. A blanket add is how a stray env file, a private strategy doc, or an editor's scratch file ends up in history forever. Naming each staged file is a five-second act that makes every commit an intentional statement of exactly what changed. Write commit messages that narrate the change — the git log doubles as the project's story.

9. **Ship fire-and-forget, then hand back.** Push. Do not sit and poll production. The executor writes a HANDBACK file (what shipped, commits, how it was verified) and an OUTLIERS file (everything noticed but deliberately not touched). See references/handback-template.md.

10. **PM verifies independently — trust the handback, then check it.** The PM does not mark the build done on the executor's word. It reads the actual diff or the live files, checks the database or infrastructure state where relevant, and confirms each claim in the handback against reality. This is not distrust of the executor; it is the same discipline as step 7 applied one level up. A handback that says "endpoint gated" gets a read of the actual router. When verification passes, archive the prompt and update working memory so the next session starts from truth, not guesswork.

## The plan-review four-check (step 5)

When the executor hands back its plan, the PM checks four things, fast:

- **Scope is contained.** It only touches the files the prompt allowed. Nothing reaches into live routers, auth, schema, or production pages it had no business in.
- **It covers the goal.** Every part of the objective is in the plan. Nothing important is quietly dropped.
- **It is real work, not a reskin.** It actually rebuilds or implements the thing, rather than surface-patching to look done.
- **It ends with verification.** There is a concrete local check (tests, screenshots, a click-through) before any push.

If all four pass, green-light it. If not, name the gap and send it back. Be specific about what to fix.

## The handback and the outliers ledger (steps 9-10)

Every build ends with two short files from the executor, named after the prompt (HANDBACK-<prompt-name>.md, OUTLIERS-<prompt-name>.md):

**The HANDBACK** is the executor's account: what shipped, the commit hashes, what was verified and how, and anything the operator must do by hand (set an env var, click an auth grant). It is the input to the PM's independent verification, not a substitute for it.

**The OUTLIERS ledger** is where scope discipline pays compound interest. Mid-build, the executor will notice things outside its lane — a stale doc, a security smell, a dead code path. Touching them would violate scope; forgetting them would waste the discovery. So they go in OUTLIERS. The PM folds them into a running ledger, and the ledger becomes the raw material for future prompts (and, for an operator learning to code, perfect first hand-written reps — one-line fixes with a known shape). Nothing gets fixed silently; nothing gets lost.

These files stay local (see the fence below). They never ship in a commit — they are operating notes, not product.

## The private-doc fence

The repo root accumulates operator files that must never reach the public history: build prompts, handbacks, outliers, strategy and marketing docs, ops runbooks. Fence them with two layers:

1. **Tracked .gitignore patterns** for the standing names and the dated-prompt pattern (e.g. `/2[0-9][0-9][0-9]-[0-9][0-9]-[0-9][0-9]-*.md`) — tracked means every clone, including the executor's, inherits the fence automatically.
2. **`.git/info/exclude`** for local-only, machine-specific ignores that do not belong in the shared file.

Verify the fence with `git check-ignore -v <file>` rather than assuming a pattern works. Combined with stage-by-name (step 8), a private file now needs two independent failures to leak — a fence hole AND an explicit add. That is the redundancy a non-engineer needs to move fast without fear.

## Credentials and consoles: "you stage, I click"

Some work runs through dashboards and consoles (hosting env vars, OAuth grants, DNS). The division of labor there mirrors the code loop: **the agent stages, the human clicks.** The agent drives the browser to the exact screen, fills what it safely can, names the value that goes in each field and where to find it — and the human pastes the secrets and clicks the auth grants, the payments, the irreversible buttons. The agent never types credentials, and the human never has to guess which of nine dashboard tabs the setting lives in. Each side does the part the other is worst at.

## Overnight cook queues

Once the loop is trusted, it scales to unattended runs: the PM writes several prompts in one sitting (numbered NN in order), the operator feeds them to the executor **serially — one finishes and is verified before the next starts.** Never run builds in parallel; two agents in one repo means racing writes and unreviewable interleaved changes. The morning after: PM reviews each handback in order, verifies independently, folds outliers into the ledger. A night of queued cooks with morning verification routinely ships what a contractor would bill a week for — but only because every prompt was scoped and every plan was reviewed before the queue started.

## Guardrails (the principles underneath the loop)

- **One cook at a time.** Do not run two builds in parallel, especially when tired or unwell. Split attention is how live systems get broken. Finish or park one before starting the next.
- **"Just X" means just X.** If the ask is "change the favicon," the build changes the favicon and nothing else. No bonus pages, no drive-by refactors. Unrequested scope is unreviewed risk — anything else noticed goes in OUTLIERS.
- **Reframe the one-off as the general case.** Before building a per-customer or per-instance hack, ask what the proper, product-grade version is and build that instead. One-off code paths rot.
- **Isolation keeps the operator safe.** The executor works in its own lane on a vetted prompt. The human never has to be the last line of defense reading raw diffs.
- **Capture context to memory.** Save decisions, names, and state so sessions stay continuous and you never re-litigate solved problems.
- **Verify completion against the code, not the symptom.** Before calling something done, confirm the actual change shipped. "It looks fixed" is not "it is fixed." This applies to the executor (step 7) and the PM (step 10) alike.

## When to reach for plan-first vs. just run

Plan-first (step 4) is always safe and is mandatory for anything large, irreversible-feeling, or single-shot. For a tiny, obviously contained change you have run a hundred times, you can let the executor run directly. When in doubt, plan first. The cost is minutes; the downside it prevents is a broken live app.

## Files in this skill

- references/prompt-template.md — the structure for the build prompt you write to the repo root in step 3. Read it when you are about to write a prompt for the coding agent.
- references/handback-template.md — the structure for the HANDBACK and OUTLIERS files the executor writes in step 9, and how the PM verifies them in step 10. Read it when a build is finishing or you are reviewing one.
