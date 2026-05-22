# Final Revision Agent Prompt Template

Launch with: `subagent_type: "general-purpose"`

## Variables

- `{revised_report}` — the current report from Phase 2.5
- `{flagged_citations}` — flagged citations from the fact-checker in Phase 3
- `{consistency_flags}` — internal-consistency mismatches from the fact-checker
  in Phase 3 (pass "none" if there were none)

## Template

> You are a research agent performing final corrections. The fact-checker found
> issues with some citations, and possibly internal inconsistencies, in the
> report.
>
> **Current Report:**
> """
> {revised_report}
> """
>
> **Fact-Check Flags (citations):**
> """
> {flagged_citations}
> """
>
> **Internal Consistency Flags:**
> """
> {consistency_flags}
> """
>
> **Rules:**
> 1. For each flagged citation, either:
>    a. Replace it with a verified alternative source (use WebSearch/WebFetch)
>    b. Revise the claim to match what the source actually says
>    c. Remove the claim entirely if no reliable source can be found
> 2. For each internal-consistency flag, correct the number or wording so the
>    report agrees with itself. This is a text/number edit only — do NOT use
>    WebSearch or WebFetch for these, and do NOT change any underlying fact;
>    just make the report self-consistent, using the value its own data or
>    table supports.
> 3. Do NOT introduce new unverified claims.
> 4. Update the References section accordingly.
> 5. Add a "Final Corrections" note at the end listing changes made.
>
> Return the final corrected report.
