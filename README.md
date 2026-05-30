# Doc Curator Skill

[![Version](https://img.shields.io/badge/version-0.0.1-blue?style=flat-square)](https://github.com/manfredk/doc-curator/releases)
[![Status](https://img.shields.io/badge/status-First%20Draft-yellow?style=flat-square)](https://github.com/manfredk/doc-curator/issues)

> This is an early prototype. It hasn't been tested thoroughly in real projects yet. Please try it on your repositories and [post your experience as an issue](https://github.com/manfredk/doc-curator/issues) — feedback on what works, what doesn't, and edge cases you hit is invaluable.

A documentation maintenance skill that watches your docs for staleness, misalignment, and consistency issues — then generates cleanup plans. Designed to keep documentation high-quality, intent-driven, and synchronized with code.

## What This Skill Does

**Watches for:**
- Stale documentation (docs older than related code changes)
- Broken or misaligned links (orphaned references)
- Redundancy (code comments that replace formal docs)
- Speculative content (TODO, FIXME, experimental content in shared docs)
- Age-based candidates for condensing or archiving

**Generates:**
- **Planning documents** (`.docs/.plan.md`) — review before applying
- **Guidelines** (`.docs/.guidelines.md`) — for other agents on doc quality standards
- No docs are written by this skill; it only maintains and cleans up

## Installation

1. Copy this directory to your skills folder
2. Create `.docs/curator-config.yaml` in your project (see `curator-config.yaml.example`)
3. Run: `doc-curator` to generate your first cleanup plan

## Philosophy

In the age of AI assistance, documentation serves two purposes:

1. **Context for AI agents** — guide them on intent, architecture, decisions
2. **Teaching for humans** — reveal why we built it this way, the big picture

Documentation should **not** describe what code does (code is readable; use comments for intricate problems).

## Files

- **SKILL.md** — Main instructions and quick start
- **REFERENCE.md** — Advanced configuration, CI/CD integration, troubleshooting
- **curator-config.yaml.example** — Configuration template
- **README.md** — This file

## Quick Start

```bash
# Generate cleanup plan (default mode)
doc-curator

# Review .docs/.plan.md, then apply
doc-curator --apply

# Interactive mode (approve each change)
doc-curator --interactive-diff

# Or step-by-step walkthrough
doc-curator --interactive-walk
```

## Key Features

### Planning-Only Mode (Default)

1. Audit all docs
2. Generate `.docs/.plan.md` with proposed changes
3. You review and approve
4. Run `--apply` to execute

### Aggression Levels

- **High** (default for `/docs`): Aggressively remove speculative, redundant, old content
- **Medium**: Flag issues for review
- **Low**: Note issues without removing

Customize per folder in `curator-config.yaml`.

### Multi-Signal Staleness Detection

- **Git history** — docs older than related code
- **Broken links** — orphaned references
- **Doc comments** — code comments vs formal docs
- **Age-based** — features > threshold
- **Speculative** — TODO, FIXME, WIP markers

### Frontmatter Management

Auto-maintains:
```yaml
created: 2024-01-15        # Locked, never changes
updated: 2026-05-30        # Auto-updated when doc changes
status: current            # Suggested by skill
relevance: high            # Optional; skill suggests
linked-to: [PRD#5, ADR#2]  # Bidirectional link tracking
```

### Guidelines for Other Agents

Generates `.docs/.guidelines.md` with standards for:
- What to document (intent, not implementation)
- How to maintain frontmatter
- Staleness prevention
- Link conventions
- Quality standards

## Repository Structure

Works with your convention:

```
repo/
├── README.md
├── docs/                  # Shared, permanent
│   ├── prd/
│   ├── adr/
│   └── stories/
├── .docs/                 # Temporary (gitignored)
│   ├── .plan.md
│   ├── .guidelines.md
│   └── curator-config.yaml
└── .gitignore             # Add: /.docs
```

Auto-detects and adapts.

## Example Workflow

1. **Run the skill** (generates plan):
   ```bash
   doc-curator
   ```

2. **Review `.docs/.plan.md`**:
   - Check detected staleness issues
   - Review suggested status changes
   - Adjust if needed

3. **Apply changes**:
   ```bash
   doc-curator --apply
   ```

4. **Commit**:
   ```bash
   git add docs/ .docs/ && git commit -m "docs: curator maintenance"
   ```

## Configuration

See `curator-config.yaml.example` and [REFERENCE.md](REFERENCE.md) for:
- Aggression levels
- Staleness signal tuning
- Custom rules
- Frontmatter schema
- CI/CD integration

## When to Use This Skill

- Auditing docs for staleness
- Ensuring docs stay synchronized with code
- Removing redundant or speculative content
- Checking for broken links
- Guiding other agents on documentation standards
- Integrating doc maintenance into CI/CD

## When NOT to Use This Skill

- Writing new documentation (use other skills like `grill-with-docs`)
- One-off documentation questions
- Style/grammar review (focus is on staleness/alignment)

## See Also

- **grill-with-docs** — Stress-test docs against your domain model
- **grill-me** — Interview about design decisions (for recording in docs)

---

For advanced usage, see [REFERENCE.md](REFERENCE.md).
