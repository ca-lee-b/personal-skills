---
name: create-prd
description: >
  Synthesizes conversation into a concise, implementation-ready PRD. Trigger on "write a PRD",
  "create a PRD", "document requirements", "write a spec", or when the user has fully
  specified a feature and is ready to hand off to a build agent. Output is a self-contained
  .md file.
---

This skill reads the current conversation and produces a concise, implementation-ready Product Requirements Document. Optimized for **build agent consumption**: every section must be unambiguous, actionable, and free of filler. A downstream agent reading this PRD should be able to implement immediately.

---

## Guiding Principles

- **No dates, authors, revision history, em-dashes, or stakeholder lists**: these waste tokens and are irrelevant to an agent.
- **No passive voice or vague language.** "The system should…" → "The system must…" or "The system does…"
- **Decisions, not discussions.** If the conversation showed debate, capture the final decision only. If something is still unresolved, flag it as an open question (not a design discussion).

---

## PRD Structure

Write in this exact order. Omit a section only if truly not applicable (note the omission with one line). Use `##` headers, no deeper than `###`.

### 1. `## Overview`

2–4 sentences max. What is being built, why, and who uses it. Include minimal context.

### 2. `## Goals`

Bullet list of 3–7 outcomes this product/feature must achieve. Measurable where possible.

- Bad: "Improve user experience"
- Good: "Users can complete checkout in under 3 clicks"

### 3. `## Non-Goals`

Explicit list of what is **out of scope**. Prevents scope creep. At least 2–3 items even if you have to infer them.

### 4. `## Users & Personas`

Only if multiple distinct user types exist. One line per persona: name, role, core need. Skip if there's only one user type and it's obvious.

### 5. `## Functional Requirements`

The core of the PRD. Use numbered requirements (`FR-01`, `FR-02`, …) for traceability.

Format each as:

```
**FR-01** [Module/Area] — Description of what the system must do.
  - Sub-requirement or constraint if needed
  - Edge case behavior
```

Group by feature area with `###` subheadings if there are more than ~8 requirements.

Rules:

- Use "must" for hard requirements, "should" for strong preferences, "may" for optional.
- Include error states and edge cases inline.
- Be explicit about default values, limits, and thresholds.
- If a user flow is complex, write it as a numbered step sequence under the requirement.

### 6. `## Data Model` _(omit if no persistent data)_

Fields, types, relationships. Use a simple table or code block. No ORM specifics unless the user named them.

```
Entity: User
- id: uuid, PK
- email: string, unique, required
- role: enum(admin, member), default=member
```

### 7. `## API / Interface Contract` _(omit if not applicable)_

Endpoints, methods, request/response shape. Use concise pseudocode or JSON examples.

```
POST /api/items
  Body: { name: string, quantity: int }
  Returns: { id: uuid, created: true }
  Errors: 400 if name missing, 409 if duplicate
```

### 8. `## Non-Functional Requirements`

Performance, security, accessibility, scalability, only what was mentioned or is clearly implied. Use `NFR-01`, `NFR-02` numbering.

```
NFR-01 Performance — Page load must complete in < 2s on a 4G connection.
NFR-02 Auth — All endpoints except /health require a valid JWT.
```

### 9. `## Tech Stack & Constraints` _(omit if user specified nothing)_

Only what the user explicitly stated. No recommendations. No opinions.

---

## Execution Instructions

1. **Read the full conversation.** Extract all stated requirements, decisions, examples, and constraints. The user may have given requirements scattered across many messages.
2. **Resolve conflicts.** If the user changed their mind, use the most recent stated preference.
3. **Fill gaps by inference, not invention.** If something is a standard industry default (e.g., passwords must be hashed), include it. If it's a genuine decision the user hasn't made, put it in Open Questions.
4. **Write the PRD.** Follow the structure above. Be ruthlessly concise.
5. **Save to file.** Create a `docs/` directory if it doesn't exist. Derive the filename from what the PRD is about — lowercase, hyphenated, descriptive (e.g., `docs/implement-user-auth-flow.md`, `docs/checkout-redesign.md`, `docs/create-admin-dashboard.md`).
6. **Report back.** After saving, give the user a 2-sentence summary of what was written.

---

## Quality Checklist (run mentally before saving)

- [ ] Every FR is testable and uses must/should/may correctly
- [ ] No requirement is ambiguous
- [ ] All error states are handled
- [ ] No dates, authors, or revision history
- [ ] The PRD could be handed to a lower model with no other context and could build it
