# Claude Skills for Operators

I have run companies for 25 years. Restaurants, a recruiting firm, a consulting practice. I am not a software engineer. I run a production SaaS today, built by directing Claude, and these skills are the actual operating system I use to do it.

Not theory. Not a course funnel. These are the working methods behind a live multi-tenant platform with paying customers, extracted into skills you can install and run yourself.

## The skills

### operator-build-loop

The core method: how a non-engineer ships production software with an AI coding agent without breaking what is live. Two agents with hard lanes (a PM that never touches code, an executor that never sets its own scope), plan-before-build, a four-check plan review, scoped prompts with explicit blast-radius fences, independent verification of every handback, and an outliers ledger so nothing found mid-build gets lost.

This loop has shipped, among other things: a full email automation engine (Google Workspace domain-wide delegation, sequence workers, provider switching), security hardening found-to-fixed in under 24 hours, and off-vendor database backups with retention. All reviewed, all verified, none of it typed by me.

### sunday-brunch-tutor

The second act: learning to actually read the code you have been shipping. A tutoring format built on real production files instead of toy examples. Annotated walkthroughs, tiny hand-written reps sourced from the outliers ledger, commit-history-as-curriculum, and the checklists that let an operator review an AI's work with real judgment. Named for the weekly ritual it came from.

My first hand-written commit went through this format, guided one command at a time, including a live debugging detour when git's cache lied to us. That commit is in my production repo's history. The method works.

## Installing

Each skill folder follows the standard Claude skill layout (SKILL.md plus references). Use the packaged .skill files in this repo, or point your Claude setup at the skill folders directly.

## Why give this away

Most AI content is either hype or fear. I think both are wrong. The real play is using these tools to multiply what a sharp operator can do, with discipline, so nothing live ever breaks. That method deserves to be public. The company I run with it stays private; the method does not.

Built with Claude. Directed by a human.

MIT licensed. Use it, fork it, ship something.
