# Final Revision Agent Prompt Template

Launch with: `subagent_type: "general-purpose"`

## Variables

- `{revised_report}` — the current report from Phase 2.5
- `{flagged_citations}` — flagged items from the fact-checker in Phase 3

## Template

> You are a research agent performing final citation corrections. The
> fact-checker found issues with some citations in the report.
>
> **Current Report:**
> """
> {revised_report}
> """
>
> **Fact-Check Flags:**
> """
> {flagged_citations}
> """
>
> **Rules:**
> 1. For each flagged citation, either:
>    a. Replace it with a verified alternative source (use WebSearch/WebFetch)
>    b. Revise the claim to match what the source actually says
>    c. Remove the claim entirely if no reliable source can be found
> 2. Do NOT introduce new unverified claims.
> 3. Update the References section accordingly.
> 4. Add a "Final Corrections" note at the end listing changes made.
>
> Return the final corrected report.
