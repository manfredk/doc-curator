---
name: doc-curator
description: Audit and maintain documentation quality, staleness, and alignment without writing docs itself. Generates cleanup plans and provides guidelines for other agents. Use when reviewing docs for staleness, detecting inconsistencies, or ensuring docs stay aligned with code intent and purpose.
---

# Doc Curator

A documentation maintenance skill that watches your docs for staleness, misalignment, and consistency issues — then generates cleanup plans. Guides other agents on documentation quality standards.

## Quick Start

```bash
# Audit all docs and generate a cleanup plan (default)
doc-curator

# Review .docs/.plan.md, then apply changes
doc-curator --apply

# Interactive diff preview (review each change)
doc-curator --interactive-diff

# One-by-one walkthrough
doc-curator --interactive-walk
```

## Core Workflow

### Default: Planning Mode

Generates `.docs/.plan.md` with:
- **Staleness detected** (git history, broken links, doc comments, age)
- **Redundancy flags** (where code comments replace formal docs)
- **Status recommendations** (current → deprecated, archived, draft)
- **Condensing suggestions** (for features > threshold age)
- **Guidelines** for other agents to prevent these issues

You review, then:
```bash
doc-curator --apply  # Execute all changes
```

### Optional: Interactive Diff

```bash
doc-curator --interactive-diff
# Shows git diff-style changes, approve/skip each
```

### Optional: Interactive Walkthrough

```bash
doc-curator --interactive-walk
# Step through each doc, decide per-issue
```

## Aggression Levels

Configure in `.docs/curator-config.yaml`:

```yaml
aggression: high  # high | medium | low

# For /docs (shared): high by default
# For /.docs (temp): medium by default
# Tunable per folder

age-threshold-months: 6  # When is content "old"?

staleness-signals:
  - git-history      # Doc outdated vs code changes
  - broken-links     # Orphaned docs
  - doc-comments     # Code comments vs formal docs
  - speculative      # TODO/FIXME in docs
  - redundancy       # When code already documents it
```

## What Gets Detected

1. **Staleness** — multi-signal:
   - Git history: `updated` older than related code changes
   - Broken links: References to deleted PRDs/ADRs/stories
   - Doc comments: Code comments vs formal docs (redundancy or conflict)
   - Age: Features > threshold should be condensed or archived
   - Relevance: Features refactored but docs unchanged

2. **Redundancy**:
   - Formal doc duplicates code comment explanation
   - Low-level implementation details (should be comments)
   - Speculative/TODO content in shared docs

3. **Misalignment**:
   - Docs reference removed features
   - Outdated architecture descriptions
   - Closed PRD/story references

## Frontmatter Convention

Auto-maintained by the skill:

```yaml
---
title: Document Title
created: 2024-01-15        # Locked, never changed
updated: 2026-05-30        # Auto-updated when doc changes
status: current            # Suggested by skill
relevance: high            # Optional; skill suggests
linked-to: [PRD#5, ADR#2]  # Bidirectional links
---
```

## Guidelines Generated for Other Agents

The skill outputs `.docs/.guidelines.md`:

1. **Documentation Purpose**
   - Explain *intent* and *why*, not *what the code does*
   - Intricate problems: document the *why*, not the *what*
   - Low-level details belong in code comments

2. **Frontmatter**
   - Always maintain: title, created, updated, status
   - Keep relevance current when code changes
   - Link to related PRDs/ADRs/stories

3. **Staleness Prevention**
   - Link docs to code paths
   - Update `updated` when doc changes
   - Review docs when closing related PRs

4. **Quality Standards**
   - Remove docs that just describe code
   - Preserve intent-driven architecture docs
   - Single source of truth: CONTEXT.md for domain language

5. **Cleanup Signals**
   - Docs > X months: condense or archive
   - Broken links: orphaned content
   - Doc comments supersede formal docs

## Repository Structure

Adapts to your convention:

```
repo/
├── README.md              # Entry point
├── docs/                  # Shared, permanent
│   ├── prd/
│   ├── adr/
│   └── stories/
├── .docs/                 # Temp (gitignored)
│   ├── .plan.md           # Generated plan
│   ├── .guidelines.md     # For other agents
│   └── curator-config.yaml
└── .gitignore             # /.docs
```

Auto-detects and adapts to your structure.

## Commands

```bash
doc-curator                     # Generate plan
doc-curator --apply             # Execute plan
doc-curator --interactive-diff  # Review diffs
doc-curator --interactive-walk  # Step-by-step
doc-curator --aggression high   # Override default
doc-curator --dry-run           # Preview only
doc-curator --folder docs/prd   # Specific folder
```

## Advanced

See [REFERENCE.md](REFERENCE.md) for:
- Custom staleness rules
- CI/CD integration
- Bulk operations
