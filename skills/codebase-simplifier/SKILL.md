---
name: codebase-simplifier
description: Audits a codebase and produces a prioritized report on what to cut to reach a lean MVP and what existing code can be simplified or made more concise. Use this whenever the user describes their codebase as messy, bloated, over-scoped, or over-engineered; asks what they can remove, cut, descope, or trim; mentions "feature creep," "MVP," "minimum viable product," "too many features," or "too complex"; or wants a second opinion on what's essential vs. optional in their project — even if they never use the word "simplify." Also trigger when a user has a working but sprawling project and wants help getting back to a shippable core.
---

# Codebase Simplifier

A messy, over-scoped codebase usually isn't messy everywhere at once — it has a small core that matters and a long tail of features, abstractions, and config that crept in around it. This skill's job is to find that line and hand the user a concrete, prioritized map of what to cut, what to shrink, and what to leave alone.

The output is a judgment call, not a linter report. Be specific and opinionated — vague advice like "consider simplifying this module" is not useful. Cite file paths, function names, and line counts wherever possible.

## Step 0: Get oriented yourself — don't make the user describe their own repo

Use `view`/`bash` to explore the codebase directly rather than asking the user to summarize it. Build a map before judging anything:

```bash
# Shape of the repo, excluding noise
find . -type f \
  -not -path '*/node_modules/*' -not -path '*/.git/*' \
  -not -path '*/dist/*' -not -path '*/build/*' -not -path '*/venv/*' \
  -not -path '*/.next/*' -not -path '*/vendor/*' \
  | sed 's|/[^/]*$||' | sort | uniq -c | sort -rn | head -30

# Where the bulk of the code actually lives
find . -name '*.py' -o -name '*.js' -o -name '*.ts' -o -name '*.tsx' -o -name '*.go' -o -name '*.rb' \
  2>/dev/null | xargs wc -l 2>/dev/null | sort -rn | head -20

# Dependency surface — a strong proxy for scope
cat package.json requirements.txt Gemfile go.mod Cargo.toml 2>/dev/null
```

If `.git` exists, churn vs. staleness is one of the highest-signal things you can pull cheaply:

```bash
# Files nobody has touched in 6+ months (candidates for dead/speculative code)
git log --since="6 months ago" --name-only --pretty=format: | sort -u > /tmp/recent_files.txt
find . -type f -name '*.py' -o -name '*.js' -o -name '*.ts' 2>/dev/null | grep -vFf /tmp/recent_files.txt

# Hottest files (the real core, almost always)
git log --name-only --pretty=format: | sort | uniq -c | sort -rn | head -20
```

Identify entry points (`main`, `index`, `app.py`, route definitions, `cmd/`) — these tell you the actual paths a request or user action takes, which matters more than directory names.

For a large codebase, don't read every file. Use this map to prioritize: read entry points first, then the largest and hottest directories, then sample the long tail.

## Step 1: Pin down what the MVP actually needs to do

Everything in this audit is judged against one yardstick: **the smallest end-to-end path that delivers real value to one user.** Not "all the planned features with less polish" — an MVP is a different, smaller shape than the full vision, not a watered-down version of it.

- If the user has already described the core use case in this conversation, use that — don't ask again.
- If they haven't, ask directly: *"What's the one thing this absolutely has to do, end-to-end, for a single user, for it to be worth shipping?"* This is worth one clarifying question even mid-skill, because guessing wrong here invalidates the whole report.
- Write this core path down in one or two sentences before moving on. It anchors everything else.

## Step 2: Classify every feature/module against that core path

Walk through the major directories, routes, and modules and bucket each one:

- **Core** — directly on the critical path the user just described.
- **Supporting** — makes the core path more robust or usable, but the core path works without it (e.g. retries, nicer error messages, an admin view of data the core flow already produces).
- **Adjacent feature** — solves a real, separate problem the MVP doesn't need yet (e.g. teams/permissions, billing tiers, exports, integrations nobody's asked to use yet).
- **Speculative/dead** — built for a future need that hasn't materialized, or genuinely unused (check for it being imported/called anywhere; check recency from Step 0).

For everything in Adjacent and Speculative, note the price tag: roughly how many files/LOC it touches, what dependencies it alone justifies, and how much it adds to tests/config/build. This is what makes "cut this" persuasive instead of arbitrary.

## Step 3: Separately, flag complexity in whatever stays

This is independent of scope — it's about code that does a necessary job in a needlessly complicated way. Look for:

- Duplicated logic that should be one shared function, not three near-copies.
- Abstraction built for flexibility that's never used — interfaces/factories/plugin systems with exactly one real implementation.
- Long files, long functions, or deep nesting that make the core hard to follow.
- Config flags or feature switches whose alternate branch nobody exercises.
- Dead code: commented-out blocks, unused exports, unreachable branches.
- The same problem solved 2–3 different ways in different parts of the codebase (e.g. three different HTTP client wrappers).

These go in the "simplify" bucket even when the feature itself is staying — the fix is to shrink the code, not remove the capability.

## Step 4: Write the report

Write this to a markdown file (don't just paste it in chat) so the user can keep it next to their codebase. ALWAYS use this structure:

```markdown
# Simplification & MVP Audit — [project name]

## TL;DR
[2-4 sentences: current state, the single biggest lever, the recommended core]

## Proposed MVP core
[One paragraph restating the critical path everything below is judged against]

## Cut for MVP (defer or delete)
| Feature / module | Why it's not core | Cost to keep (LOC, deps, surface) | Effort to remove |
|---|---|---|---|

## Simplify / refactor (keep the feature, shrink the code)
| Location | Issue | Suggested fix | Impact |
|---|---|---|---|

## Quick wins (do these first)
[Ranked list, highest impact / lowest effort first — this is what the user should tackle today]

## Keep as-is
[Short list with one-line rationale each, so the user knows what NOT to touch]

## Risks / double-check before cutting
[Anything that looks unused but might have a non-obvious external dependency — a webhook consumer, a cron job, a route something else calls]
```

## Calibration notes

- Don't recommend a rewrite-from-scratch unless the evidence genuinely points there. The goal is the smallest set of changes that gets to a lean, shippable MVP — not a redesign.
- If something's ambiguous (core vs. adjacent), say so explicitly in the report rather than silently picking a side — wrong guesses in either direction either tell the user to cut something load-bearing or clutter the report with hedged "maybe" items.
- Prefer "here's what I found and why" over a one-line summary. The user is trying to make real cut decisions on a real codebase; specificity is the entire value of this skill.