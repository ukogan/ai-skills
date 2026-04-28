# ai-skills

Skills I've built for [Claude Code](https://claude.com/claude-code). Each skill is a self-contained folder you can install on its own.

## Skills

| Skill | What it does |
|-------|--------------|
| [`/sharpen`](sharpen/) | A read-only dialogue mode for sharpening requirements before plan mode. Captures the ask, probes edge cases, optionally generates wireframes, and runs a supportability check against your codebase before plan mode commits engineering work. |

## Install one skill

```bash
git clone https://github.com/ukogan/ai-skills.git
cp -r ai-skills/<skill-name> ~/.claude/skills/<skill-name>/
```

Or symlink so the skill stays in sync as the repo updates:

```bash
ln -s "$(pwd)/ai-skills/<skill-name>" ~/.claude/skills/<skill-name>
```

See each skill's own README for details.

## License

MIT.
