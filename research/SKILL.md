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

**Brief the user between phases — findings, not activity.** As each phase
completes, give a one- or two-line update on what was *concluded*, the way a
research team reports back — not a description of the work performed. "Initial
findings: vanilla is clearly the most popular flavor; the by-state data is
contested." — not "Phase 3 complete, ran a research agent." Phases 6 and 7
(fact-check, final revision) are mechanical; a short status line is enough there.

Agent prompt templates are in the `references/` directory of this skill. Read
the relevant reference file before launching each agent, and inject the
appropriate variables (topic, prior outputs, etc.) into the template.

---

## Phase 1: Intake (always runs — one efficient round)

Phase 1 always runs. Never skip it, even on a terse request or a "just go". It
is **one AskUserQuestion call** — no preamble, and **do not write out a brief
first**. The questions themselves carry your recommendations, so a separate
written brief is redundant.

### Standing defaults

These are the values you recommend unless the request overrides one:

| Dimension | Default |
|-----------|---------|
| **Depth & length** | Practical summary, ~one page — findings and what they imply, not background lectures. Depth and length move together: a quick overview is short, a deep-dive runs long. |
| **Audience & tone** | Audience inferred from the conversation and the user's memory / CLAUDE.md; tone direct, unemotional, just the facts — no hedging, no praise, no throat-clearing. Audience sets the tone. |
| **Output style** | Structured report, visual wherever structure beats prose — tables, concept-relationship diagrams, comparison matrices. When visuals do real work, deliver an HTML artifact (see below), not a long markdown file. |

### Decide your recommendations (your own prep — not shown to the user)

For each dimension, work out the value you will recommend:

- **Depth & length, Audience & tone, Output style** — start from the default,
  then sanity-check it against the request. A default is a starting point, not
  a straitjacket: when the request makes one a poor fit, recommend an
  alternative instead and hold a one-line reason for it.
  - "Summarize every element of the periodic table" / "the first stanza of
    every Beatles song" — can't fit one page; recommend a length and format
    that can.
  - "Teach me X well enough to hold my own with experts" — practical-summary
    depth is too shallow; recommend a deep-dive.
  - "Draft a leave-behind for a customer" — terse factual tone is too blunt for
    the audience; recommend matching it.
- **Scope, Known context, Decision context** — can't be defaulted. Form your
  best read from the conversation. Pick the one point that most needs the
  user's confirmation; infer the rest silently.

### Ask — go straight to the questions

Run a **single AskUserQuestion call** — no written brief before it:

- Four questions: depth & length, audience & tone, output style, and the one
  scope/context point you most need confirmed.
- In every question, the **first option is your recommendation**, labelled
  "(Recommended)", with your reason in its description. Realistic alternatives
  after it. "Other" is always available for a free-text override.
- The user accepts all your recommendations by taking the first option on each,
  or changes any dimension in place — one fast interaction.

After the user answers, state the final picks in one line and proceed. Re-ask
only if an answer opens a genuine new fork.

### HTML output

When findings carry visual structure — multi-source data, concept
relationships, comparisons, a decision matrix — deliver the report as a
**single self-contained HTML file**, not markdown. HTML carries what markdown
can't (CSS layout, real tables, SVG diagrams, light interactivity), and a
self-contained `.html` shares by link, so it actually gets read.

- One file: inline all CSS, JS, and SVG — no external assets, no build step.
- Lead with the one-page answer; push depth into collapsible sections or tabs.
- SVG diagrams for relationships and flows; real `<table>` markup for data.
- Restrained, professional styling — no emojis, one accent color, legible type.

Use plain markdown only when the output is linear prose with no structure worth
showing visually.

---

## Phase 2: SME Consultation (optional)

If the topic is highly specialized, launch an SME Agent before or during the
research phase.

Read `references/sme-agent.md` for the prompt template. Pass the SME brief to
the Research Agent alongside the refined query.

For interdisciplinary topics, launch multiple SME agents in parallel (one per
domain) and merge their briefs.

---

## Phase 3: Research Agent

Launch a general-purpose agent to research the topic and produce a cited report.

Read `references/research-agent.md` for the prompt template. Include the
refined query, output specs from intake, and any SME brief. Store the research
report for Phases 4 and 6.

**Brief the user** with the headline findings — e.g. "Initial findings: vanilla
wins both the survey and the purchase data; the by-state picture is contested."
Not "ran a research agent."

---

## Phase 4: Skeptical Reviewer Agent

Launch an agent to critically review the research report.

Read `references/reviewer-agent.md` for the prompt template.

**Brief the user** with what the review exposed — e.g. "Review flags 3 soft
spots: the survey may be multi-select, no retail scanner data was used, and the
state map measures over-indexing, not raw volume."

---

## Phase 5: Revision Pass

Send the reviewer's critique back to a new research agent instance.

Read `references/revision-agent.md` for the prompt template.

**Brief the user** with how the conclusions changed — e.g. "Updated: the
national vanilla finding is downgraded from 'well-established' to 'consistent
across two sources'; the two weakest sources were dropped."

---

## Phase 6: Fact-Checking Agent

Launch an agent to verify every citation in the revised report — and, in the
same pass, check the report for internal inconsistencies: counts that don't
match their own lists or tables, figures that disagree between sections, totals
that don't add up. This is a document-against-itself check, no web needed.

Read `references/factcheck-agent.md` for the prompt template.

**Brief the user** with a short status line — e.g. "7 of 10 citations verified,
3 flagged; one internal count mismatch found."

---

## Phase 7: Final Revision (if needed)

If the fact-checker flagged citations or internal inconsistencies, send them
back for correction. Consistency fixes are numbers/text edits only — no new
research.

Read `references/final-revision-agent.md` for the prompt template.

**Brief the user** with a short status line — e.g. "Flagged citations and the
count mismatch corrected."

---

## Phase 8: Compile and Deliver

Present the user with:

1. **The final research report** (fully cited and verified), formatted per the
   depth/length, style, and tone agreed in Phase 1.
2. **A confidence summary**:
   - Citations verified vs. flagged
   - Internal inconsistencies found and fixed
   - Overall quality rating from the reviewer
   - Any remaining `[UNVERIFIED]` claims
3. **Ask** whether the user wants to:
   - Deep-dive into any section
   - Research additional angles
   - Export the report (write to a file)

---

## Phase 9: Understanding Check (optional)

If intake specified "deep-dive" depth, learning-oriented goals, or the topic is
one where the user needs to hold their own in expert conversations (not just
reference a report), offer an understanding check after delivering the report.

Using the final verified report, generate 5-7 questions that would distinguish
someone who deeply understands the topic from someone who just memorized the
report's facts. Good questions:

- Require applying mental models from the report to a novel scenario
- Ask the user to argue one side of an expert disagreement, then the other
- Probe the "why" behind facts rather than the facts themselves
- Expose whether someone understands the relationships between concepts

Present the questions to the user as a self-assessment. If they answer any,
evaluate the response and explain what they're missing.

Skip this phase for quick overviews, decision memos, or comparison tables where
the goal is a reference artifact, not personal understanding.

---

## Extensibility

- **Iterative refinement**: The user can ask follow-ups at any point. Re-run
  any phase with updated context.
- **Domain-specific verification**: For technical topics, the fact-checker can
  verify claims against code, docs, or APIs using Bash, Read, and Grep.
