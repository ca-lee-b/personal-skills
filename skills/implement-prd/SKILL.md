---
name: implement-prd
description: >
  Reads a PRD file and implements it — first producing an execution plan, then writing code.
  Trigger on "implement the PRD", "build this PRD", "code this up", or any instruction to
  act on a PRD file. If no file is specified, ask the user before proceeding.
---

Reads a PRD, builds an execution plan, then implements it fully. No guessing: if something is ambiguous, stop and ask.

---

## Guiding Principles

- **Plan before coding.** Always produce and confirm the execution plan before writing a single line of code, unless user said not to.
- **One question at a time.** If the PRD is ambiguous, surface all blockers upfront in a single message — not one by one mid-implementation.
- **Full implementation.** Do not stub, scaffold, or leave TODOs unless the user explicitly asks. Every FR must be implemented.
- **Stay faithful to the PRD.** Do not add features, opinions, or improvements not specified. If you see a problem, flag it, don't silently deviate.

---

## Execution Instructions

### Step 1 — Locate the PRD

If the user specified a file path, read it. If not, ask:
> "Which PRD file should I implement? (e.g. `docs/my-feature.md`)"
Do not proceed until you have the file.

### Step 2 — Quickly Parse & Validate

Read the full PRD. Check for:

- **Ambiguities** — requirements that could be interpreted two or more ways
- **Contradictions** — two requirements that conflict
- **Missing decisions** — anything a build agent cannot reasonably default

If any blockers are found, list them all in a single message and wait for the user to resolve them before continuing. Format as:

```
Before I start, I need to clarify a few things:
1. FR-03 says X — should that mean A or B?
2. FR-07 and FR-11 conflict: X vs Y. Which takes precedence?
```

If the PRD is fully clear, proceed immediately.

### Step 3 — Execution Plan

Produce a task list derived from the PRD's FRQs. Each task must:

- Map to one or more `FR-XX` identifiers
- Name the file(s) to be created or modified
- Be sequenced so dependencies come first

Format:

```
Execution Plan
1. [FR-01] Create `src/models/user.ts` — User data model and types
2. [FR-02, FR-03] Create `src/api/auth.ts` — Login and signup endpoints
3. [FR-04] Create `src/middleware/auth.ts` — JWT validation middleware
...
```

Present the plan and wait for the user to confirm ("looks good", "go ahead", etc.) before writing any code.

### Step 4 — Implement

Execute the plan in order. For each task:

- Write complete, working code — no stubs, no TODOs
- Follow any tech stack, constraints, or conventions specified in the PRD
- If the PRD specifies NFRs (performance, security, etc.), apply them inline

After all tasks are done, give a brief completion summary listing what was created/modified.

---

## Ambiguity Rules

- **Stop and ask** for anything that changes behavior or architecture.
- **Default silently** only for trivially standard decisions (e.g. using 4-space indentation, hashing passwords with bcrypt, returning 404 for missing resources).
- Never skip a requirement. If something is unimplementable as written, flag it immediately.
