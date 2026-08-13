# Build prompt template

Use this structure when writing a prompt to the repo root for the coding agent (step 3 of the loop). Fill every section. The discipline of the template is what keeps the executor inside the lines.

```
# YYYY-MM-DD-NN — <short title>

## Goal
One paragraph. The single thing this build accomplishes and why. If you cannot say it in a few sentences, the scope is too big — split it.

## Scope — files you MAY touch
List the exact files/directories the executor is allowed to modify or create.

## Scope — DO NOT touch
List what is off limits: live routers, auth, schema, production pages, payment paths, anything outside the goal. Be explicit. This is the safety boundary.

## Step 0 — audit first
Tell the executor to read the relevant live code before changing anything, so it builds against reality, not stale docs.

## Build steps
Numbered, each a working/testable slice. Commit per step where it makes sense.

## Verify (mandatory)
Exactly how to confirm it works locally: run the dev server, take screenshots (desktop + mobile if UI), run tests, grep for leftovers. Do NOT verify against the production URL.

## Acceptance criteria
The concrete checklist that means "done." Include "live app untouched, only allowed files changed."

## Out of scope (explicit)
Restate what this build is NOT doing, to stop scope creep mid-build.

## Handback
On completion write HANDBACK-<this-file's-name>.md and OUTLIERS-<this-file's-name>.md
per the handback template (references/handback-template.md). Local-only, never committed.
```

## Why each section earns its place

- **Goal** forces one objective. Vague goals produce sprawling builds.
- **MAY / DO NOT touch** is the literal blast-radius fence. The plan review checks the plan against it.
- **Step 0 audit** prevents prompts written against a codebase that has since changed.
- **Verify** is non-negotiable because "looks done" is not "is done," and a non-engineer cannot catch a silent regression by reading code.
- **Out of scope** is repeated at the end because that is where a smart, eager executor is most tempted to "improve" things you did not ask for.

## Naming

Name the file `YYYY-MM-DD-NN-topic-slug.md` at the repo root, where NN is the build number for that day. Dated, ordered, greppable. After the build ships, move it to a local archive so the repo root stays clean and git only tracks what runs the app.
