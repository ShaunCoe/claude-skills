---
name: operator-build-loop
description: >-
  A disciplined two-agent workflow for shipping real production software without
  writing code yourself, by pairing a planning/PM agent with an isolated coding
  agent like Claude Code. Use this whenever someone wants to build, ship, or change
  a feature, turn an idea into shipped code, write or review a prompt for a coding
  agent, plan a build, or direct Claude Code, especially a non-engineer or a small
  team who wants to ship safely while staying out of the codebase. Trigger on
  things like "build X", "ship this change", "write a prompt for Claude Code",
  "plan this feature", "review this plan before I run it", "help me direct the
  coding agent", or any request to move from idea to shipped code through an AI
  coding agent, even if the workflow is not named explicitly.
---

# Operator Build Loop

A repeatable way for a non-engineer (or a small team) to ship production software with AI, without ever touching code directly and without breaking what is live. It works by splitting the job across two agents with clear lanes, and never letting either do the other's work.

This is a real, battle-tested operating model: it has shipped multiple production apps. The point is not speed for its own sake. The point is shipping fast **and** safe, so a sharp operator can run engineering-grade output without being an engineer.

## The two roles (never blur them)

**The PM / strategist** (a planning agent, e.g. Claude in a chat or Cowork). Owns context and prompts. It absorbs what is actually going on, writes the build instructions, and reviews the coding agent's plan. It does **not** edit code. Keeping it out of the code is deliberate: its job is judgment, scope, and safety, and the moment it starts hand-editing files, that judgment gets compromised and mistakes leak straight into the repo.

**The executor** (the coding agent, e.g. Claude Code). Owns the code. It runs in its own isolated lane, takes **one** prompt at a time, plans before it builds, and verifies before it ships. The operator stays out of its blast radius. That isolation is the whole safety model: if the executor only ever acts on a reviewed, scoped prompt, a non-engineer can ship confidently because the dangerous surface is contained.

The operator (the human) directs. They give the goal and the product judgment. They never have to read the diff line by line, because the workflow catches problems before they land.

## The loop

Run this every time you ship a change. Each step exists to remove a specific way things go wrong.

1. **Absorb context first.** Before writing anything, the PM reads the real state: prior session notes, the actual git log and live code, recent email or messages, and any saved memory. Trust the code and the git history over project docs. Docs drift; the running app is the truth. Skipping this is how you write a prompt against a codebase that no longer exists.

2. **Frame one scoped objective.** Decide the single thing this build does. Write down explicitly what it must **not** touch, the acceptance criteria, and a mandatory verification step. Scope is the cheapest place to prevent damage: a tightly bounded prompt cannot wander into the live payment path.

3. **Write the prompt to the repo root.** Save it as a dated file (YYYY-MM-DD-NN-topic-slug.md). The prompt names the files the executor may touch, the files it must not, the steps, and how to verify. See references/prompt-template.md for the full structure. Writing it to the repo root (not pasting it into a chat) gives the executor a durable, reviewable artifact and leaves a paper trail of every change.

4. **Executor plans first — do not let it build yet.** Run the coding agent in plan mode on the prompt. This costs one extra round trip and is the single highest-leverage safety step, especially for a big or one-shot build you cannot easily redo. A plan is cheap to throw away; an hour of wrong building is not.

5. **PM reviews the plan against the checklist** (below) before any code is written. Green-light or send it back. This is where a non-engineer gets engineering-grade safety: the reviewer catches scope creep, missing modules, and missing verification while it is still just text.

6. **Execute.** Once the plan passes, the executor runs it (autonomous/bypass is fine here, because the scope and plan are already vetted). Higher reasoning effort is worth it for large or one-shot builds.

7. **Verify before you trust it — locally where the stack allows, live where it does not.** Decide which mode applies *while writing the prompt*, not after the build.

   **Local mode (the default).** Run it locally, take screenshots or run tests, confirm it works before pushing. Catch regressions on your machine, not in front of customers.

   **Live mode (when local is impossible).** Some stacks make local verification impossible for entire classes of screen, permanently. The common one: hosted auth providers refuse production keys on `localhost` and refuse to mint sessions against a production instance, so every signed-in screen is unreachable locally. That is a property of the stack, not a per-build obstacle you can work around. When that is true, say so in the prompt and run the live loop instead:

   - **Baseline first.** Capture the current, broken behavior on the deployed environment *before* you push — verbatim output, response bodies, screenshots.
   - **Push.**
   - **Re-run the identical checks.** Same URLs, same steps, same order. Changing the check between runs destroys the comparison.
   - **Paste before and after verbatim** in the handback. Not a summary. The actual output.
   - **On failure, revert.** Never patch forward on a live system at speed. A revert is one command; a half-fixed production path at midnight is a night you do not get back.
   - **Name who verifies.** Either the PM agent independently confirms the live surfaces, or the human does. "It deployed" is not verification, and the executor confirming its own work is not independent.

   Choosing the wrong mode is worse than choosing neither. A prompt that demands a local check on a screen the executor cannot reach locally will get skipped, worked around, or quietly reported as passing.

8. **Ship, then archive.** Push, run whatever verification the mode you chose calls for, and stop there. Do not sit and poll production past your own checks. Move the prompt into a local archive and update the working-memory doc so the next session starts from truth, not guesswork.

## The plan-review checklist (step 5)

When the executor hands back its plan, the PM checks four things, fast:

- **Scope is contained.** It only touches the files the prompt allowed. Nothing reaches into live routers, auth, schema, or production pages it had no business in.
- **It covers the goal.** Every part of the objective is in the plan. Nothing important is quietly dropped.
- **It is real work, not a reskin.** It actually rebuilds or implements the thing, rather than surface-patching to look done.
- **It ends with verification the executor can actually perform.** There is a concrete check — tests, screenshots, a click-through — and it is in a mode the stack permits. If the plan promises a local check on a screen that cannot be reached locally, that is a failed review, not a detail.

If all four pass, green-light it. If not, name the gap and send it back. Be specific about what to fix.

## Guardrails (the principles underneath the loop)

- **One cook at a time.** Do not run two builds in parallel, especially when tired or unwell. Split attention is how live systems get broken. Finish or park one before starting the next.
- **"Just X" means just X.** If the ask is "change the favicon," the build changes the favicon and nothing else. No bonus pages, no drive-by refactors. Unrequested scope is unreviewed risk.
- **Reframe the one-off as the general case.** Before building a per-customer or per-instance hack, ask what the proper, product-grade version is and build that instead. One-off code paths rot.
- **Isolation keeps the operator safe.** The executor works in its own lane on a vetted prompt. The human never has to be the last line of defense reading raw diffs.
- **Capture context to memory.** Save decisions, names, and state so sessions stay continuous and you never re-litigate solved problems.
- **Verify completion against the code, not the symptom.** Before calling something done, confirm the actual change shipped. "It looks fixed" is not "it is fixed."

- **A verification method that cannot run is not a verification method.** If the prompt demands a check the executor physically cannot perform, it will be skipped or a passing result will be assumed, and you will ship unverified while believing you did not. Match the check to what the stack actually permits, and write that into the prompt.

- **Revert, don't patch forward.** When a live change fails after deploy, the first move is always to put production back. Diagnose from a stable base. Patching forward under time pressure is how one bad deploy becomes three.

## When to reach for plan-first vs. just run

Plan-first (step 4) is always safe and is mandatory for anything large, irreversible-feeling, or single-shot. For a tiny, obviously contained change you have run a hundred times, you can let the executor run directly. When in doubt, plan first. The cost is minutes; the downside it prevents is a broken live app.

## Files in this skill

- references/prompt-template.md — the structure for the build prompt you write to the repo root in step 3. Read it when you are about to write a prompt for the coding agent.
