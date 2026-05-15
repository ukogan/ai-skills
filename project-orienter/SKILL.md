---
name: project-orienter
description: Generate an executive-level project orientation when returning to a project. Combines git history, Claude session history, and project planning files to produce a feature-oriented status briefing. Use when starting a new session, returning after time away, or when the user asks "where did we leave off?" or "what's the status of this project?"
---

# Project Orienter

Generate a feature-oriented, executive-level project orientation by synthesizing multiple sources of project context. Designed for the "cold start" problem: returning to a project after hours, days, or weeks away.

## When to Use

- User runs `/orient`
- User asks "where did we leave off?", "what's the project status?", "catch me up"
- Start of a new session on a project with prior history

## Arguments

| Argument | Description |
|----------|-------------|
| `--days N` | Look back N days (default: 14) |
| `--brief` | Short version: just the 5 sections, no detail bullets |
| `--save` | Save orientation to `.claude-team/orientations/YYYY-MM-DD.md` |

## Output Format

Present a structured briefing with these 5 sections. Use plain bullets, no numbering. Feature-oriented language, not implementation details.

```
## Project Orientation — {Project Name}
As of {date}, looking back {N} days

### Current State
* [What the project does today, key capabilities that are live/working]
* [Overall health: tests passing, deploys working, known broken things]

### Recently Shipped
* [Feature or capability completed, with approximate date]
* [Feature or capability completed, with approximate date]
* [Refactors or infrastructure changes worth noting]

### Discovered and Addressed
* [Issue found and resolved during recent work]
* [Idea that came up and was implemented]
* [Decision made that closed off an alternative]

### Discovered and Deferred
* [Issue found but intentionally left for later, with reason if known]
* [Idea noted but not yet pursued]
* [Bug filed but not yet fixed]
* [Incomplete Claude plan that was started but not finished]

### Near-Term Roadmap
* [Next planned feature or milestone]
* [Backlog item slated for upcoming work]
* [Open question that needs resolution before proceeding]
```

## Workflow

### Phase 1: Gather Context (use subagents in parallel)

Launch these data collection tasks concurrently:

#### 1a. Git History

```bash
# Recent commits (feature-oriented, not individual lines)
git log --oneline --since="{N} days ago" --no-merges

# Branches with recent activity
git branch --sort=-committerdate | head -10

# Current branch and uncommitted state
git status --short
git stash list
```

Group commits by feature/theme, not chronologically. Look for patterns:
- `feat:` commits = Recently Shipped
- `fix:` commits = Discovered and Addressed
- Reverted or abandoned branches = Discovered and Deferred

#### 1b. Claude Session History

Use the `claude-history` agent to search recent conversations for:
- What tasks were worked on
- What decisions were made
- What was explicitly deferred ("we'll do this later", "parking this", "backlog")
- What problems were encountered
- What plans were created but not fully executed

Query patterns:
- "deferred", "later", "backlog", "TODO", "parking", "skip for now"
- "decided", "chose", "going with", "instead of"
- "bug", "issue", "broken", "failing"
- "plan", "next steps", "roadmap"

#### 1c. Project Planning Files

Search for and read planning/roadmap files. Check these locations:
- `VISION.md`, `ROADMAP.md`, `BACKLOG.md`, `TODO.md`
- `.claude-team/status.md`, `.claude-team/handoff-*.md`
- `.claude-team/bugs/*.md`
- `.claude-team/reviews/*.md`
- `docs/**/next-sprint*`, `docs/**/roadmap*`
- Any `*.plan.md` or `*-plan.md` files
- GitHub issues (if `gh` CLI is available): `gh issue list --limit 10`

#### 1d. Claude Auto-Memory

Check the project's memory directory for relevant context:
- Recent project memories (type: project)
- Feedback memories that indicate ongoing concerns
- Reference memories pointing to external tracking

### Phase 2: Synthesize

Merge all sources into the 5-section format. Rules:

1. **Feature-oriented**: "Added real-time collaboration" not "implemented WebSocket handler in api/ws.py"
2. **Executive-level**: A product manager should understand every bullet
3. **Honest about gaps**: If you can't determine something, say so
4. **Attribute sources**: When a bullet comes from a single source, note it parenthetically: (git), (session history), (status.md)
5. **Deduplicate**: If git and session history both mention the same thing, merge into one bullet
6. **Recency bias**: Weight recent activity higher. Things from 2 days ago matter more than 12 days ago
7. **No speculation**: Only include what evidence supports

### Phase 3: Present and Wait

Present the briefing. Do NOT start working on anything. End with:

```
Ready when you are. What would you like to focus on?
```

### Phase 4: Save (if --save)

If `--save` flag is set, write the orientation to `.claude-team/orientations/YYYY-MM-DD.md` so it can be referenced later.

## What This Is NOT

- **Not a session resume**: It doesn't replay what happened in the last Claude session. It orients you to the project.
- **Not a handoff**: It doesn't capture failed approaches or debugging context. Use `/save-session` patterns for that.
- **Not a standup**: It covers weeks of context, not yesterday's work.

## Graceful Degradation

Not all sources will be available. Degrade gracefully:
- No Claude history? Git + planning files are still valuable.
- No planning files? Git + Claude history still work.
- Git only? Still useful: group commits into themes and present.
- Note which sources were available: `Sources: git (47 commits), claude history (3 sessions), status.md, 2 bug reports`
