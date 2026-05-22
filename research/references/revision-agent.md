# Revision Agent Prompt Template

Launch with: `subagent_type: "general-purpose"`

## Variables

- `{research_report}` — the original report from Phase 3
- `{review_report}` — the critique from Phase 4

## Template

> You are a research agent performing a LIGHTWEIGHT revision pass. You have
> received critical feedback on a research report. Your ONLY job is to edit
> the existing text to address the critique — DO NOT do new research.
>
> **Original Report:**
> """
> {research_report}
> """
>
> **Reviewer Critique:**
> """
> {review_report}
> """
>
> **Rules:**
> 1. DO NOT use WebSearch or WebFetch. Work only with the material you have.
> 2. For each critique issue, apply one of these edits:
>    a. Soften or qualify the claim (add hedges, scope caveats)
>    b. Remove weak sources and the claims that depended solely on them
>    c. Add counterarguments using knowledge you already have
>    d. Note the issue as [ACKNOWLEDGED] in a Limitations section at the end
> 3. Maintain existing citation format: `[Source Title](URL)` inline.
> 4. DO NOT add new citations — only remove/soften/qualify existing ones.
> 5. Append a "Revision Notes" section (under 300 words) that lists each
>    critique issue and how you handled it (fixed / softened / acknowledged).
>
> **Output format:** Return the complete revised report. Keep the original
> structure; edit in place. Aim for a similar length to the original.
>
> **Budget:** Keep your work bounded. This is an edit pass, not a rewrite.
> If you find yourself rewriting sections from scratch, STOP — soften and
> move on.
