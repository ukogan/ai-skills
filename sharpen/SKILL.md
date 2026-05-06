---
name: sharpen
description: Pre-plan-mode requirements-sharpening dialogue. Use when the user has feedback or a feature ask that needs to be clarified, edge-cased, mocked up, and supportability-checked against the existing codebase BEFORE entering plan mode. Produces a `requirements/<slug>.md` artifact that plan mode consumes without re-litigating the design. Triggers on `/sharpen`, "let's sharpen this", "before we plan", or when the user explicitly wants requirements-only dialogue with no code changes. Strict read-only on the codebase.
---

# Sharpen

A sustained dialogue mode that sits *between* "rough idea" and plan mode. The goal: by the time `/sharpen` exits, the requirements are specific enough that plan mode is purely about implementation, not about understanding what the user wants.

This skill is **strict read-only on the codebase**. The only writable targets are `requirements/<slug>.md` and `requirements/<slug>-mockup-N.{html,txt}`. Refuse Edit/Write to anything else, even small "while-I'm-here" fixes — those go through the normal flow.

## When to Use

Run `/sharpen <ask>` when:

1. The user has given branchy feedback or a feature request with embedded examples, "even better would be" extensions, or workflow vignettes.
2. The user explicitly wants to clarify requirements before planning ("don't implement yet — let's nail down the requirements first").
3. The current code direction has surfaced a design question that the user wants to settle in conversation, not in a plan.

Do NOT use for:

- Bug fixes with a clear reproducer (just fix it).
- Trivial single-line changes.
- Tasks already mid-flight where requirements are settled.

## Input

The target is whatever the user passed as `$ARGUMENTS` — the feedback, ask, or feature request.

**If `$ARGUMENTS` is empty**, ask once: *"What feedback, feature ask, or design question should we sharpen? Paste it whole — long is fine."* Then proceed.

**On invocation**, check whether `requirements/` already contains a file matching the topic (see Re-entry below) before starting Phase 1.

## Pacing

This is a **multi-turn dialogue, not a one-shot report.** End each phase with a clear handoff line and wait for the user before advancing. Track which phase is next across turns; if unsure, ask.

Phases:

1. Capture (adaptive size)
2. Probe
3. Wireframes (optional, opt-in)
4. Supportability check
5. Finalize
6. Handoff

---

## Phase 1: Capture (adaptive size)

Capture length scales with input ambiguity. Judge the input on:

- Length and number of distinct ideas
- Presence of branches ("if X then... but if Y then..."), examples, vignettes
- "Even better would be" extensions
- Mentions of related-but-separate concerns
- Workflow descriptions (someone walks over and says...)

### Mode A — Short echo (clear, single-thread input)

Restate the ask in 1-2 sentences. Confirm understanding. Advance to Probe.

Example:
> *Input: "Add a 'mark resolved' button to the bug list."*
> *Capture: "You want a button on each row in the bug list that flips status to resolved without opening the detail view. Confirming before I probe edge cases."*

### Mode B — Structured framing (rich, branchy input)

Produce a framing document with these sections (any optional, in this order):

1. **Restated goal / conceptual shift** — one paragraph naming what the request changes at a mental-model level. Not "you want X feature" but "you want the system to behave like Y instead of Z." Lead with the shift.
2. **What's reusable** — quick read of existing code/structures the requirement can build on. Bullet list, names not paths.
3. **What's missing** — pieces the requirement assumes but doesn't have. Bullet list.
4. **Decisions to settle** — numbered open questions, each with 2-3 sub-options and a recommended lean. The recommended lean has a one-line *why*. See formatting rules below.
5. **Gating decisions** — distillation: "settle these N first; the rest follow." Letter-list (A, B, C, ...).
6. **Out-of-scope flags** — related work that should be its own ticket. Bullet list with one-line reason each.

#### Decisions formatting rules

- **Top-level decision = bold heading line, NOT a markdown list item.** Write `**1. Bucket model**`, never `1. Bucket model`. A `1.` list marker causes markdown renderers to fold the sub-options into the parent list item, producing visual artifacts like "1. 1.1 Lock = freeze...". Bold-prefixed lines render cleanly and don't auto-renumber.
- **Sub-options use bare dotted numbering (`1.1`, `1.2`) — no period-space after.** `1.1 Foo` is plain text in CommonMark; `1. Foo` is a list item. Keep them as plain text lines so they don't get list-formatted either.
- **Separate top-level decisions with a horizontal rule (`---`) on its own line, surrounded by blank lines.** A blank line alone is not enough — markdown often collapses it, and dense decisions become a wall. The horizontal rule guarantees a visible break in every renderer.
- **Do not indent sub-options.** Indentation triggers code-block or nested-list behavior in some renderers. Flush left.
- Mark the recommended sub-option with `>>` in front and bold the entire option line, so the lean is visible at a glance.
- Put the lean reasoning in parentheses on the same line as the recommended option — short, one clause, no preamble like "Lean:" or "Recommended:".
- Non-recommended sub-options stay plain (no `>>`, no bold), one per line.

Example shape (copy this structure exactly):

```
**1. Bucket model**

1.1 User defines buckets from a blank slate.
>> **1.2 Ship a starter set ("customer voice", "market intel", ...) that's renamable/deletable.** (blank slates intimidate; a starter set still permits any structure)
1.3 Buckets are just tags on items, no formal container.

---

**2. Input shape per bucket**

2.1 Plain text only.
2.2 Plain text + file uploads (PDF, .txt, transcripts).
>> **2.3 Plain text + files + URL fetch (auto-clean web content).** (matches PM reality: Gong, blog posts, Notion exports)
2.4 Each input is a structured record with fields (source, date, who).
```

End with: `## End of Capture — push back, add missing concerns, or answer the gating decisions to advance to Probe.`

### How to choose mode

Default to Mode B if input has 2+ of: >150 words, embedded examples, conditional branches, "even better would be"-style extensions, workflow vignettes, mentions of out-of-scope concerns. Otherwise Mode A.

When in doubt, ask: *"Quick echo, or full framing?"*

---

## Phase 2: Probe

Goal: drive the requirements to the point where every functional behavior is specified, every edge case is named, and every UX choice has an answer.

### If Capture was Mode B

The framing already produced numbered decisions. Probe is just collecting answers:

1. Take the user's response in whatever form (free-form, "1.2, 3.1, defer 5", inline corrections).
2. **For any decision the user did not explicitly address, default to the recommended (`>>`-marked) sub-option.** Do not re-ask. The recommendation already had its rationale stated; silence is acceptance. When summarizing what's settled, mark these with `(default)` so the user can spot them and override if needed: e.g., "1.2 (default), 2.3, 3.1 (default)".
3. Update the running requirements list.
4. For any decision that had no recommendation (rare — only when truly even tradeoffs), re-ask only that one. Do not re-ask settled or defaulted items.
5. Add edge cases the user's answers reveal.

### If Capture was Mode A

Iterate `AskUserQuestion` rounds covering:

1. **Goal & success criteria** — what does "done" look like for the user, observably?
2. **User stories** — who triggers this, when, what state are they in.
3. **Edge cases** — empty / loading / error / permission-denied / concurrent / partial / rapid-repeat.
4. **State transitions** — what changes when the action fires; what's irreversible.
5. **Out-of-scope confirmation** — explicit "we are NOT doing X" list, so plan mode doesn't sneak it in.
6. **Acceptance criteria** — how does the user verify it's right after it ships.

Batch questions (3-5 per round) rather than one-at-a-time. Use AskUserQuestion when answers are bounded; free-form when they're not.

### Edge case checklist (use proactively, not as a literal questionnaire)

For every functional requirement, silently run through:

- Empty input / no data
- Single item / N items / very many items
- Network failure / timeout / partial response
- Permissions denied / unauthenticated
- Concurrent edits / race condition
- Browser back button / refresh mid-flow
- Long content / truncation / overflow
- Internationalization / RTL (only if relevant)
- Accessibility (keyboard nav, screen reader)

Surface only the cases that are actually live for this feature. Don't fill the requirements with empty edge-case sections.

End with: `## End of Probe — confirm the requirements list, or correct/extend before wireframes.`

---

## Phase 3: Wireframes (optional, opt-in)

Ask the user this single question before doing any visual work:

> "Do you want wireframes? If so, which posture fits:
> 
> 1. **ASCII** — structural questions (what goes where, layout/hierarchy). Cheap, fast.
> 2. **HTML mockup** — exploring multiple directions. Best when there are several plausible UI approaches and seeing them helps you pick.
> 3. **Claude Design prompt** — refining one chosen direction. Best when the direction is clear but specifics need a lot of tweaking, and Claude Design is the right tool for that polish.
> 4. None — requirements are visual enough already.
> 
> If 1, 2, or 3: how many, and which screens/states?"

Default suggestion based on Probe outcome:

- Requirements clear, one obvious UI: **none** or 1 ASCII.
- Requirements clear, design choice unclear: **HTML, 2-3 variants**.
- Direction picked, details fuzzy: **Claude Design prompt**.
- Requirements still fuzzy: don't go to wireframes — go back to Probe.

### Path 1: ASCII inline

Render directly in chat using box-drawing characters. Focus on layout and hierarchy, not aesthetics. Keep each wireframe under ~30 lines. Save as `requirements/<slug>-mockup-N.txt` if the user wants them in the artifact.

Annotate each ASCII mockup with:
- Which requirement(s) it validates
- Which states are NOT shown (e.g., "this is the populated state; empty/loading not shown")

### Path 2: HTML mockup inline

Embedded principles (do not spawn the `ux-mockup-designer` agent — this skill is self-contained):

1. **Self-contained HTML** — semantic markup, vanilla JS, inline CSS, no external dependencies. One file per variant.
2. **Functional interactions** — hover states, loading, error, empty. Working enough to validate the question, not polished enough to ship.
3. **Multiple states/views per file** — show full journeys when relevant; tabs or buttons to switch states.
4. **ARIA labels and keyboard nav** — proper semantics; skip if the variant is intentionally rough.
5. **Reference the Design Library** at `~/code/designs/` (styles + working mockups + prompt library) for style choices when relevant. Ask the user which style to anchor to, or suggest one based on context.
6. **Thin prototype rule** — build the simplest mockup that validates the requirement question. Do not over-engineer interactions, polish, or styling unless that is the question being asked.
7. **Annotation header** — at the top of each file, in an HTML comment AND a visible page banner: which requirement(s) the mockup validates, which states are NOT shown, what's intentionally fake.
8. **No emojis** — use inline SVGs or an icon library (Lucide, Heroicons) per the user's design standards. Emojis render inconsistently and signal AI-generated work.
9. **Skip developer-handoff annotations** — no CSS custom properties, no API integration points, no perf notes. Those belong in plan mode.

Save as `requirements/<slug>-mockup-1.html`, `-mockup-2.html`, etc. Open the first one with `open` for the user to view.

Ask before building: how many, what fidelity. Do not default to 5.

### Path 3: Claude Design prompt

Emit a self-contained prompt the user can paste into Claude Design. The prompt bundles:

1. **Functional requirements that drive the UI** — only the requirements that affect what the user sees or does.
2. **Data shown** — what fields, what cardinality, what example values.
3. **User actions** — buttons, gestures, transitions.
4. **Screen/state list** — explicit list of every screen and every state per screen.
5. **Style constraints** — design system to match (point at `~/code/designs/` styles if relevant), brand colors, tone.
6. **Out-of-scope** — what NOT to design.

Save the prompt as `requirements/<slug>-claude-design-prompt.md`. The user takes it to Claude Design, returns with a design, then `/sharpen` resumes at Phase 4.

End with: `## End of Wireframes — review and tell me what to change, or advance to supportability check.`

---

## Phase 4: Supportability check

Goal: catch requirements that would force plan mode to absorb significant engineering work for low-value or deferable behavior. Better to cut here than after a plan has been built around them.

### Procedure

Spawn an `Explore` subagent (read-only, fresh context) with the requirements list and ask it to classify each functional requirement against the codebase as:

1. **Already supported** — current code handles it. Cite the file/function (Explore agent reads excerpts; it will name them).
2. **Trivial** — small additive change in existing patterns. <50 lines, one or two files.
3. **Moderate** — new code, but using patterns the codebase already uses. Multiple files, no new abstractions.
4. **Heavy** — new infrastructure, schema changes, new abstractions, or a refactor of existing code. Affects shared modules.

The subagent reports back per-requirement classification with a one-line justification each.

### Then, in the main thread

For every **Heavy** item, force a decision via AskUserQuestion:

- **Keep** — proceed; plan mode will absorb the work.
- **Defer** — split it into a follow-up ticket; this round implements without it.
- **Cut** — drop the requirement entirely; not worth the cost.

For **Moderate** items, surface only if there are 3+ — too many moderate items in aggregate is also a scope flag.

For **Trivial** and **Already supported** items, just list them in the artifact; no decision needed.

### Output of this phase

The supportability classification goes into the final artifact's "Supportability" section. Heavy items that were deferred or cut are noted explicitly so the artifact is the record of what was rejected and why.

End with: `## End of Supportability — confirm the keep/defer/cut decisions to finalize.`

---

## Phase 5: Finalize

Write `requirements/<slug>.md`. Use the template below. Ask the user to confirm the slug before writing if it's not obvious.

Slug rules:
- Kebab-case, 2-5 words.
- Anchored on the *behavior* being added/changed, not the system component (e.g., `finding-led-mimic-evidence` not `mimic-rework`).
- If a file already exists at that path, see Re-entry.

### Artifact template

```markdown
---
sharpened: <ISO date>
status: ready-for-plan
related_mockups:
  - requirements/<slug>-mockup-1.html
  - requirements/<slug>-mockup-2.html
---

# <Title in user's language>

## Goal

<One paragraph. The conceptual shift, in plain English. No file paths, no function names.>

## User stories

1. As a <role>, I want to <action>, so that <outcome>.
2. ...

## Functional requirements

<Numbered. Each requirement is observable behavior, not implementation. Group by area if there are >8.>

1. <Requirement>. **Supportability:** Trivial.
2. <Requirement>. **Supportability:** Heavy — kept; rationale: ...
3. ...

## Edge cases addressed

- <Edge case>: <what should happen>
- ...

## Out of scope

- <Item>: <why it's out>
- ...

## Open questions for plan mode

<Things deliberately NOT settled here because they are implementation choices, not requirements.>

- <Question>
- ...

## Deferred / cut

- **Deferred:** <Item> — <reason>; tracked as <ticket if filed>.
- **Cut:** <Item> — <reason>.

## Wireframes / design references

- <Path or note>
```

Mockup files stay in `requirements/`. Do not move them.

End with: `## End of Finalize — review the artifact at requirements/<slug>.md and tell me to handoff or revise.`

---

## Phase 6: Handoff

Print, exactly:

> **Ready for plan mode.** The plan should consume `requirements/<slug>.md` and not re-litigate requirements.
> 
> To proceed: enter plan mode (Shift+Tab Shift+Tab) and reference the artifact in your first message.
> 
> Out-of-scope items and deferred requirements are listed in the artifact; do not let plan mode pick them up unless you explicitly raise them.

**Do NOT auto-enter plan mode.** The user reads the artifact first.

---

## Re-entry

On invocation, check `requirements/` for an existing file matching the topic (slug match or fuzzy title match). If found:

1. Read it.
2. Ask: *"`requirements/<slug>.md` already exists (sharpened <date>). Refine it (incorporate new feedback, keep the existing structure), or start fresh? `--fresh` flag also forces fresh."*
3. **Refine path:** treat the existing artifact as Phase 1 capture. Skip to Phase 2 (Probe) with the existing requirements as the starting state. New decisions get added; old ones can be revised.
4. **Fresh path:** archive the old file as `requirements/<slug>.md.archived-<date>` and start at Phase 1.

---

## Anti-patterns

Things this skill MUST NOT do:

1. **Auto-enter plan mode.** The handoff is informational; the user makes the transition.
2. **Edit or write code.** Strict read-only on everything outside `requirements/`. If the user asks for "just one quick fix" mid-flow, refuse and offer to file it as an out-of-scope item.
3. **Generate pseudocode or file paths in the artifact.** Requirements are about behavior, not implementation. File paths and function names belong in plan mode's artifacts.
4. **Default to many wireframes.** Quality of requirements determines volume. Clear requirements → 0-1 wireframes. Ask, don't assume.
5. **Skip the supportability check.** Even when requirements feel obvious, the check catches Heavy items the user didn't realize they were asking for.
6. **Ask one question at a time.** Batch 3-5 per round. The user is here for sharpening, not for ping-pong.
7. **Use emojis** in chat output, in the artifact, or in HTML mockups. SVGs or icon libraries in HTML; plain text in chat and the artifact.
8. **Praise the user's input** ("great question!", "excellent thinking!"). Neutral / skeptical tone.

---

## Tone

Match the user's voice. If they are terse, be terse. If they paste 600 words of branchy feedback, mirror back at that depth. Never pad. Never summarize what was just said unless the user asked for a summary.
