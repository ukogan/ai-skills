---
name: research
description: >
  Multi-agent research system that produces thoroughly researched, reviewed, and
  citation-verified output. Use when the user asks to research a topic, investigate
  a question, compare options with evidence, or produce a cited report. Triggers on:
  "research X", "investigate", "what does the evidence say about", "deep dive into",
  "compare X vs Y with sources".
---

# Multi-Agent Research System

You are orchestrating a multi-agent research system. Your job is to coordinate
several specialized sub-agents to produce thoroughly researched, rigorously
reviewed, and citation-verified output.

## User's Research Query

$ARGUMENTS

## Process Overview

Run the following phases in order. Each phase uses the Agent tool with specific
instructions. Track progress with TodoWrite.

**Ask questions at any stage.** The intake interview is the primary place, but
also ask whenever ambiguity arises during any phase — if the research reveals
the topic is broader than expected, if there are conflicting findings that
require the user's judgment, or if the output format needs adjustment. Never
guess when you can ask.

Agent prompt templates are in the `references/` directory of this skill. Read
the relevant reference file before launching each agent, and inject the
appropriate variables (topic, prior outputs, etc.) into the template.

---

## Phase 0: Intake (defaults + a short scoping round)

Most output dimensions have **standing defaults** — apply them silently, do not
interview the user about them. Only ask about the dimensions that are genuinely
specific to this query.

### Standing defaults

Apply these unless the user's request explicitly overrides one:

| Dimension | Default |
|-----------|---------|
| **Depth** | Practical summary, no fluff — findings and what they imply, not background lectures. |
| **Audience** | Infer from the conversation so far and the user's memory / CLAUDE.md. Do not ask. |
| **Output length** | One page. Expand only if the user explicitly asks for more. |
| **Output style** | Structured report, visual wherever structure beats prose — tables, concept-relationship diagrams, comparison matrices. When visuals do real work, deliver an HTML artifact (see below), not a long markdown file. |
| **Tone** | Direct, unemotional, just the facts. No hedging, no praise, no throat-clearing. |

### Sanity-check the defaults against the request

A default is a starting point, not a straitjacket. Before applying one
silently, judge whether the request makes it a poor fit. When it does, do not
apply it silently — raise it in the scoping round below with a **recommended
alternative** and a one-line reason.

Examples of a request that overrides a default:

- "Summarize every element of the periodic table" / "the first stanza of every
  Beatles song" — the content can't fit in one page. Recommend a length and
  format that can (e.g. a structured table, or a longer artifact).
- "Teach me X well enough to hold my own with experts" — practical-summary
  depth is too shallow. Recommend a deep-dive.
- "Draft a leave-behind for a customer" — the terse factual tone may be too
  blunt for the audience. Recommend matching it.

### What to still ask

Run **one short round** (AskUserQuestion) covering:

- **Scope** — boundaries and explicit exclusions.
- **Known context** — what the user already knows, so the report doesn't repeat it.
- **Decision context** — what decision this informs, and the options on the table.
- **Any default flagged above** as a poor fit — presented with your recommended
  alternative as the suggested option.

If the request already answers the query-specific dimensions and no default was
flagged, skip the round. If the user says "skip" or "just go", apply the
defaults — except for any default you flagged as unworkable, where you apply
your recommended alternative instead and note the swap. Ask further questions
later only when the research surfaces a real fork needing the user's judgment.

### HTML output

When findings carry visual structure — multi-source data, concept
relationships, comparisons, a decision matrix — deliver the report as a
**single self-contained HTML file**, not markdown. HTML carries what markdown
can't (CSS layout, real tables, SVG diagrams, light interactivity), and a
self-contained `.html` shares by link, so it actually gets read.

- One file: inline all CSS, JS, and SVG — no external assets, no build step.
- Lead with the one-page answer; push depth into collapsible sections or tabs.
- SVG diagrams for relationships and flows; real `<table>` markup for data.
- Add a copy button that exports the core findings as markdown.
- Restrained, professional styling — no emojis, one accent color, legible type.

Use plain markdown only when the output is linear prose with no structure worth
showing visually.

---

## Phase 0.5: SME Consultation (optional)

If the topic is highly specialized, launch an SME Agent before or during Phase 1.

Read `references/sme-agent.md` for the prompt template. Pass the SME brief to
the Research Agent alongside the refined query.

For interdisciplinary topics, launch multiple SME agents in parallel (one per
domain) and merge their briefs.

---

## Phase 1: Research Agent

Launch a general-purpose agent to research the topic and produce a cited report.

Read `references/research-agent.md` for the prompt template. Include the
refined query, output specs from the intake interview, and any SME brief.

Store the research report for Phases 2 and 3.

---

## Phase 2: Skeptical Reviewer Agent

Launch an agent to critically review the research report.

Read `references/reviewer-agent.md` for the prompt template.

---

## Phase 2.5: Revision Pass

Send the reviewer's critique back to a new research agent instance.

Read `references/revision-agent.md` for the prompt template.

---

## Phase 3: Fact-Checking Agent

Launch an agent to verify every citation in the revised report.

Read `references/factcheck-agent.md` for the prompt template.

---

## Phase 3.5: Final Revision (if needed)

If the fact-checker flagged citations, send them back for correction.

Read `references/final-revision-agent.md` for the prompt template.

---

## Phase 4.5: Understanding Check (optional)

If the user's intake specified "deep-dive" depth, learning-oriented goals, or
the topic is one where the user needs to hold their own in expert conversations
(not just reference a report), generate understanding-check questions.

Using the final verified report, generate 5-7 questions that would distinguish
someone who deeply understands the topic from someone who just memorized the
report's facts. Good questions:

- Require applying mental models from the report to a novel scenario
- Ask the user to argue one side of an expert disagreement, then the other
- Probe the "why" behind facts rather than the facts themselves
- Expose whether someone understands the relationships between concepts

Present the questions to the user as a self-assessment. If they want to answer
any, evaluate their response and explain what they're missing — the same
follow-up loop from the thread: "Explain why this is wrong and what I'm
missing."

Skip this phase for quick overviews, decision memos, or comparison tables where
the goal is a reference artifact, not personal understanding.

---

## Phase 4: Compile and Deliver

Present the user with:

1. **The final research report** (fully cited and verified), formatted per the
   length/style/tone agreed in Phase 0
2. **A confidence summary**:
   - Citations verified vs. flagged
   - Overall quality rating from the reviewer
   - Any remaining `[UNVERIFIED]` claims
3. **Ask** whether the user wants to:
   - Deep-dive into any section
   - Research additional angles
   - Export the report (write to a file)

---

## Extensibility

- **Iterative refinement**: The user can ask follow-ups at any point. Re-run
  any phase with updated context.
- **Domain-specific verification**: For technical topics, the fact-checker can
  verify claims against code, docs, or APIs using Bash, Read, and Grep.
