# /research

A multi-agent research pipeline for [Claude Code](https://claude.com/claude-code): it produces a thoroughly researched, skeptically reviewed, and citation-verified report instead of a single-pass answer.

A one-shot answer is only as good as the first draft. `/research` runs the draft through an adversarial pipeline — a skeptic tears it apart, a reviser rebuilds it, a fact-checker verifies every citation — so what you get back has already survived the critique you would otherwise have to give it yourself.

## When to use

- "Research X" / "investigate" / "deep dive into X."
- "What does the evidence say about X?"
- "Compare X vs Y, with sources."
- Any question where you need cited, defensible findings — not a vibe.

Skip it for quick factual lookups; the pipeline is deliberately heavyweight.

## How it works

The skill orchestrates specialized sub-agents in sequence:

1. **Intake interview** — pins down scope, depth, audience, length, format, and the decision the research informs. No research starts until the brief is clear.
2. **SME consultation** (optional) — a subject-matter-expert agent briefs the researcher on specialized or interdisciplinary topics.
3. **Research agent** — produces the first cited report against the refined brief.
4. **Skeptical reviewer** — critically attacks the report for weak claims, gaps, and bias.
5. **Revision pass** — a fresh agent rewrites the report against the critique.
6. **Fact-checker** — verifies every citation; flags anything that can't be confirmed.
7. **Final revision** (if needed) — corrects flagged citations.
8. **Understanding check** (optional, for deep-dives) — generates questions that test whether you actually understand the topic, not just memorized the report.

Each agent's prompt template lives in `references/`.

## What you get out

- A fully cited, verified report formatted to the length, style, and tone agreed in the intake interview.
- A confidence summary: citations verified vs. flagged, the reviewer's quality rating, any remaining `[UNVERIFIED]` claims.
- The option to deep-dive a section, research more angles, or export to a file.

## Install

```bash
git clone https://github.com/ukogan/ai-skills.git
cp -r ai-skills/research ~/.claude/skills/research/
```

Or symlink so you get updates:

```bash
ln -s "$(pwd)/ai-skills/research" ~/.claude/skills/research
```

## Use

```
/research <your question or topic>
```

Answer the intake questions (or say "just go" for sensible defaults), then let the pipeline run. Expect it to take a while — it launches multiple agents and does real web research.

## Other skills

See the [repo root](../) for the full list.

## License

MIT.
