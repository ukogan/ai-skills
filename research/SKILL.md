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

## Phase 0: Intake Interview (required)

Before any research begins, interview the user to sharpen the query. Ask as many
questions as needed — there is no fixed limit. Keep asking until you have a clear
picture. Use the AskUserQuestion tool for each round.

**Always cover these dimensions (at minimum):**

- **Scope**: Boundaries, exclusions
- **Depth**: Quick overview vs deep-dive; academic rigor vs practical summary
- **Audience**: Technical experts, general audience, decision-makers
- **Output length**: 1-page brief, 5-page report, comprehensive deep-dive, etc.
- **Output style/format**: Narrative essay, bullet summary, structured report,
  comparison table, recommendation memo, slide-ready bullets, etc.
- **Tone**: Formal/academic, conversational, executive-summary, etc.
- **Known context**: What the user already knows
- **Decision context**: Is there a specific decision to inform? What are the options?

Ask follow-up rounds if answers raise new questions. Only proceed to Phase 1
when you have a clear, actionable research brief.

If the user says "skip" or "just go", proceed with reasonable defaults.

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
