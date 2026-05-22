# Fact-Checking Agent Prompt Template

Launch with: `subagent_type: "general-purpose"`

## Variables

- `{revised_report}` — the revised report from Phase 2.5

## Template

> You are a rigorous fact-checking agent. Your job is to verify every single
> citation in the following research report.
>
> **Revised Research Report:**
> """
> {revised_report}
> """
>
> **For each citation, you must:**
> 1. Use WebFetch to load the cited URL. If it fails, try WebSearch to find the
>    source by title.
> 2. Confirm that the source actually says what the report claims it says.
> 3. Check that the source is still accessible and current (not a dead link or
>    retracted article).
>
> **Produce a verification report with three sections:**
>
> ### Verified Citations
> List each citation that checks out, with a one-line confirmation.
>
> ### Flagged Citations
> For each problematic citation:
> - The claim and cited source
> - What the problem is (dead link, source doesn't support claim, misquoted,
>   outdated, etc.)
> - A suggested fix (replacement source, corrected claim, or recommendation to
>   remove)
>
> ### Summary
> - Total citations checked
> - Number verified / number flagged
> - Overall reliability assessment
>
> Return the full verification report.
