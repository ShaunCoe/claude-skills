# Handback and outliers templates

Every build ends with these two files from the executor, written to the repo root, named after the prompt file, and kept local (fenced from git — see the private-doc fence in SKILL.md).

## HANDBACK-<prompt-name>.md

```
# HANDBACK — <prompt title>

## Shipped
What was built, in plain language. One paragraph.

## Commits
Each commit hash + one-line summary. (Stage-by-name means this list IS the change inventory.)

## Verified
Exactly what was checked and how: tests run, screenshots taken (desktop + mobile for UI),
grep sweeps for leftovers, local click-throughs. Claims here must be specific enough
for the PM to re-check them independently.

## Operator actions required
Anything only a human can finish: env vars to set (name them and where the values live —
never print the values), auth grants to click, DNS to point. Empty section = nothing needed.

## Deviations from plan
Anything done differently than the reviewed plan said, and why. An empty section is
a claim, and the PM will verify it.
```

## OUTLIERS-<prompt-name>.md

```
# OUTLIERS — <prompt title>

Everything noticed during the build that was OUTSIDE scope and deliberately not touched.
One line each: what, where (file/line if known), and why it matters.
Severity-tag anything that smells like security (P1/P2) so the PM can triage same-day.
```

Why outliers matter: scope discipline says don't touch it; the ledger says don't lose it. A security smell found mid-build and filed as a P1 outlier can be a reviewed, scoped fix within a day — found-to-fixed in 24 hours is a real, repeatable outcome of this pattern. And for an operator learning to read code, small outliers (a one-line class fix, a stale label) are ideal first hand-written reps: known shape, contained blast radius, real production value.

## The PM's verification pass (step 10)

For each handback claim, check the primary source, not the prose:

- "Endpoint gated" → read the actual router file at its current commit.
- "Migration applied" → query the schema, count the rows.
- "Uploaded to storage" → list the bucket and confirm the object keys.
- "Tests green / build green" → look at the run output, not the summary sentence.
- Diff scope → `git show <hash> --stat` and confirm only allowed files appear.

If every claim survives contact with reality, the build is done: archive the prompt, fold outliers into the running ledger, update working memory. If a claim fails, it goes back to the executor as a named, specific gap — the same way a plan review rejection works.
